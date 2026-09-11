# Website and Storefront

This domain specifies the public-facing side of the system: the **website** (a content management
capability with sites, pages, menus, themes, editable content, redirects, visitor tracking and
multi-site resolution), the **storefront** (the online sales channel built on top of it: published
products, categories, the shopping cart, the checkout flow, delivery selection, online payment,
abandoned-cart recovery, wish lists, product comparison, reviews and product feeds), and the
**community capabilities** that publish content on the same site: the discussion forum with its
reputation economy, the blog, the public contact directory, the tracked-link service and the public
profile pages.

It also specifies the plumbing that every public page depends on: the request-to-site resolution
algorithm, the language segment in the address, the slug grammar, the record-visibility filters that
decide which records a given site may show, the page-visibility gate, the **copy-on-write algorithm
that gives one site its own version of a shared template**, the editable-content saving contract
with its sanitisation rules, the generic form builder with its field allow lists, and the anti-robot
challenge providers.

The domain is the publishing surface of the whole system. Other domains (events, recruitment,
courses, live chat, projects, marketing) do not define their own web server, their own page store or
their own visitor identity: they attach their published records to the mixins defined here, register
their request endpoints in the routing table described here, and appear inside the navigation menus,
the site index and the site search defined here.

Five algorithms in this folder are load-bearing for any re-implementation and are written out step
by step:

1. The **site resolution algorithm** — how one incoming request is mapped onto exactly one site, and
   what that decision then governs ([multi-site-and-languages.md](multi-site-and-languages.md) §1,
   [calculations.md](calculations.md) §2).
2. The **copy-on-write algorithm** — what happens the first time a shared template or page is edited
   in the context of one site, including the fate of every template that inherits from the copied one
   ([multi-site-and-languages.md](multi-site-and-languages.md) §3, [workflows.md](workflows.md) §15).
3. The **cart algorithm** — how a line is found, added, updated, verified and removed, how the
   quantity actually granted may differ from the quantity requested, and how the cart is revalidated
   after every change ([storefront-checkout.md](storefront-checkout.md) §2,
   [calculations.md](calculations.md) §14).
4. The **checkout algorithm** — the ordered step list, the address rules, the delivery-method
   selection, the payment hand-off and the confirmation
   ([storefront-checkout.md](storefront-checkout.md) §§4–10).
5. The **reputation economy of the forum** — every award, every deduction and every gate
   ([forum-and-blog.md](forum-and-blog.md) §§3–5, [calculations.md](calculations.md) §21).

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Multi-site hosting | Several independent sites in one database, each with its own domain name, company, languages, theme, public user, menus, pages and settings; a deterministic resolution of an incoming request to exactly one site. |
| Record visibility per site | A uniform rule: a record either belongs to one site or to all sites; one filter expression, applied everywhere, decides what a visitor of a given site may see. Combined with the publication flag and the company filter. |
| Pages | Static pages backed by editable templates, with an address, a publication flag, a scheduled publication date, indexing control, header and footer options, per-page visibility (public, signed in, restricted group, password) and per-page tracking. |
| Model pages | Generated list-and-detail page pairs that expose a whole business entity publicly under a chosen path prefix, with a restriction condition and a default layout. |
| Menus | A per-site menu tree of at most two levels, with ordering, nesting, mega-menu content, group restriction and automatic visibility derived from the target page. |
| Copy-on-write | Editing a shared template while acting for one site silently produces a site-specific copy, relocates the inheriting templates, duplicates the attached pages, and repoints the menus — leaving the other sites untouched. |
| Address rewriting | Permanent and temporary redirects, rewrites that rename a route while keeping the old address alive, and address suppression returning "not found", each optionally scoped to one site. |
| Language in the address | A language prefix in the path, automatic redirection to the visitor's preferred language, canonical addresses and alternate-language links. |
| Editable content | Inline editing of template fragments and of record fields, with a saving contract, a sanitisation policy, an image and attachment library, a content-block library, revision history for rich text fields and per-site style customisation files. |
| Themes | Installable themes shipping templates, assets, attachments, pages and menus as *theme records* that are copied into live records per site, with a documented removal and re-installation behaviour. |
| Content blocks with live data | Named filters that select records of an allowed entity, cap the number of results and feed a rendering template, with sample data when the selection is empty. |
| Form builder | A generic public form that writes a record of an opted-in entity, with per-entity field allow lists, required-field enforcement, file attachment handling, an anti-robot challenge and an electronic mail notification. |
| Visitors and tracking | An anonymous visitor record keyed by a browsing token, its link to a contact once known, and a page-visit log with visit counts, merging at sign-in and scheduled cleanup. |
| Crawler instructions and site index | A generated crawler exclusion response and a cached, paginated site index enumerating pages and enumerable endpoints. |
| Site search | One search contract implemented by every published entity, with exact matching, approximate matching, highlighting, truncation and an autocompletion answer. |
| Site configurator | A guided first-run sequence that collects the industry, purpose, palette and desired features, installs the matching capability packages, creates the matching pages and menus, and fills the generated pages with content blocks. |
| Storefront catalogue | Published product templates and variants, storefront categories as a tree, ribbons, tags, attribute filters, full-text search with approximate matching, sorting, pagination and a grid layout model. |
| Product page | The combination contract (which variant a set of attribute choices resolves to), price and comparison price, availability text, images and documents, alternative and accessory products, unit-price display, reviews. |
| Cart | Session-bound draft order, line matching, quantity verification, stock limitation, combo and optional-product lines, zero-price blocking, accessory suggestions, reorder history, warnings. |
| Checkout | An ordered, per-site configurable step list: cart, address, extra information, payment; guest, optional-account and mandatory-account modes; billing and delivery addresses; delivery method and rate; pickup points. |
| Payment and confirmation | Hand-off to the payment domain, the landing page, the confirmation page, the order state change and the confirmation message. |
| Abandoned-cart recovery | Detection of abandoned carts after a configurable delay, the filter that decides which of them deserve a message, the scheduled job and the recovery link that revives the cart. |
| Wish list and comparison | Per-visitor or per-contact wish lists with back-in-stock notifications, and a side-by-side comparison of products by attribute category. |
| Stock display and collection | Sell-when-out-of-stock policy, availability threshold and message, back-in-stock subscriptions, warehouse per site, collection in a store and collection at an external parcel shop. |
| Product syndication | A token-protected product feed per site, language and price list, with a daily cache and product-count limits. |
| Forum | Questions, answers, comments, votes, accepted answers, tags, moderation, closing reasons and the complete reputation economy with every award and every gate. |
| Blog | Blogs, posts, tags, tag categories, cover properties, per-post visibility, comments, the subscription feed and the reading statistics. |
| Public contacts and profiles | Public detail pages for contacts, a filterable customer reference directory with country and industry facets and a map, and a public profile per user with ranks and badges. |
| Tracked links | Short addresses with campaign parameters and click statistics, served from the site of the company that owns them. |

---

## 2. Actors

| Actor | Description | Typical rights |
|---|---|---|
| Public visitor | An unauthenticated person browsing the site. Runs as the site's public user. | Read published pages and published records, build a cart, submit forms, accept or refuse optional cookies. |
| Portal user | An authenticated external person (customer, applicant, student, forum participant). | Everything the public visitor may do, plus pages whose visibility is `connected`, their own orders, wish lists and forum posts. |
| Restricted Editor | An internal user allowed to edit page content but not to create or delete structural records. | Read template definitions, edit content of records they may write, use the content editor and the media dialogue, reorder storefront products. |
| Editor and Designer | An internal user responsible for the site. Implies Restricted Editor and the sanitisation override. | Full create, read, update and delete on Website, Website Menu, Website Page, Website Model Page, Website Rewrite, Website Route, template definitions, assets, Blog, Blog Post, Blog Tag, Blog Tag Category, Website Checkout Step and Product Feed; read, update and delete on Website Visitor; full rights on Website Visit Track. |
| Multi-site user | A flag group granted automatically as soon as a second site exists. | Sees the site selector and the site column on multi-site records. |
| Salesperson | Internal user with the sales user group. | Sees online orders, unpaid orders and abandoned carts, sends recovery messages, confirms orders paid by wire transfer. |
| Sales Administrator | Internal user with the sales manager group. | Configures storefront categories, ribbons, base units, attribute categories, price lists and product feeds. Implicitly receives the Restricted Editor group. |
| Forum moderator | Any participant whose reputation reaches the moderation threshold of a forum. | Validates pending posts, closes and reopens questions, marks posts as offensive, edits and deletes any post. |
| Administrator | System administrator. | Everything above, plus theme template records, global settings, verification service keys and the audience measurement keys. |
| Scheduled job runner | The background process that runs recurring jobs. | Deletes inactive visitors, deactivates unused content-block assets, sends abandoned-cart and back-in-stock messages, releases abandoned coupons. |
| Search engine crawler | An automated agent identified by its user agent text. | Reads published pages without being tracked and without being redirected by browser language preference. |
| External services | Anti-robot verification, address autocompletion, product syndication, parcel-shop network, print-on-demand fulfilment, image library, mapping and audience measurement. | Contact the system, or are contacted by it, through the contracts of [interfaces.md](interfaces.md) §11. |

---

## 3. Entities this folder owns

Every entity below is specified field by field in [entities.md](entities.md), which links each one to
its generated reference page.

### 3.1 Site, structure and content

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Website | `website` | `website` | One public site: domain name, company, languages, theme, public user and every site-level setting. |
| Website Menu | `website.menu` | `website_menu` | One entry of one site's navigation tree. |
| Website Page | `website.page` | `website_page` | One static page: an address plus the template that renders it plus publication data. |
| Website Model Page | `website.controller.page` | `website_controller_page` | A generated list-and-detail page pair for an arbitrary entity, published under a chosen path prefix. |
| Website Technical Page | `website.technical.page` | database view `website_technical_page` | A read-only listing of the endpoint-backed addresses of the site, for the page manager. |
| Website Route | `website.route` | `website_route` | The catalogue of endpoint paths, refreshed from the routing table, used when defining a rewrite. |
| Website Rewrite | `website.rewrite` | `website_rewrite` | One redirect, rewrite or suppression rule, optionally scoped to one site. |
| Website Visitor | `website.visitor` | `website_visitor` | One anonymous or identified browsing identity, keyed by an access token. |
| Website Visit Track | `website.track` | `website_track` | One visit of one visitor to one address at one instant. |
| Website Content Block Filter | `website.snippet.filter` | `website_snippet_filter` | A named, parameterised record selection that a dynamic content block renders. |
| Website Configurator Feature | `website.configurator.feature` | `website_configurator_feature` | One selectable feature (a page template or a capability package) offered by the first-run configurator. |
| Page Properties Wizard | `website.page.properties` | transient | The dialogue that edits a page's address, name, publication, visibility and options. |
| Page Properties Base Wizard | `website.page.properties.base` | transient | The shared part of the page properties dialogue, usable for any published record. |
| Robots Editor | `website.robots` | transient | The dialogue that edits the crawler exclusion text of a site. |
| Blocked Domain List Editor | `website.custom_blocked_third_party_domains` | transient | The dialogue that edits the list of third-party domains blocked before consent. |
| Assets Utility | `website.assets` | abstract | The service that writes, reads and resets per-site style customisation files. |
| Text Processor | `website.html.text.processor` | abstract | The service that turns rendered content blocks into placeholders, requests generated text and re-applies the original formatting. |

### 3.2 Theme records

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Theme Template | `theme.ir.ui.view` | `theme_ir_ui_view` | A template shipped by a theme, copied into a live template when the theme is applied to a site. |
| Theme Asset | `theme.ir.asset` | `theme_ir_asset` | An asset declaration shipped by a theme. |
| Theme Attachment | `theme.ir.attachment` | `theme_ir_attachment` | A file shipped by a theme. |
| Theme Menu | `theme.website.menu` | `theme_website_menu` | A menu entry shipped by a theme. |
| Theme Page | `theme.website.page` | `theme_website_page` | A page shipped by a theme. |
| Theme Utilities | `theme.utils` | abstract | The service that enables and disables named templates when a theme is applied and resets the default style configuration. |

### 3.3 Mixins this folder defines and other domains inherit

| Full name | Transport name | One-line purpose |
|---|---|---|
| Search Engine Metadata | `website.seo.metadata` | Title, description, keywords, social image and slug name for any published record. |
| Multi-site Restriction | `website.multi.mixin` | The single optional site reference and the "may this record be shown on the current site" test. |
| Publication Flag | `website.published.mixin` | The publication flag, the publication right, the public address of a record. |
| Multi-site Publication Flag | `website.published.multi.mixin` | The combination of the two above, with a site-aware publication computation and search. |
| Searchable | `website.searchable.mixin` | The contract an entity implements to appear in the site-wide search. |
| Cover Properties | `website.cover_properties.mixin` | The serialised cover-image settings of a blog, post, event or course. |
| Page Options | `website.page_options.mixin` | Header overlay and header colours of a page-like record. |
| Page Visibility Options | `website.page_visibility_options.mixin` | Header and footer visibility of a page-like record. |
| Rich Text History | `html.field.history.mixin` | A bounded revision history of designated rich text fields, with restore, compare and difference. |

### 3.4 Storefront

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Website Product Category | `product.public.category` | `product_public_category` | One node of the storefront category tree (distinct from the internal product category). |
| Product Ribbon | `product.ribbon` | `product_ribbon` | A coloured corner label or badge displayed over a product picture. |
| Product Image | `product.image` | `product_image` | One extra picture or video of a product template or variant. |
| Base Unit Display | `website.base.unit` | `website_base_unit` | A named reference unit used to display a price per unit of measure. |
| Storefront Extra Field | `website.sale.extra.field` | `website_sale_extra_field` | A product field added to the details block of the product page. |
| Website Checkout Step | `website.checkout.step` | `website_checkout_step` | One step of the checkout progress bar of one site, with its address, order, labels and publication flag. |
| Product Feed | `product.feed` | `product_feed` | A syndication document definition with its filter, its address, its access token and its daily cache. |
| Product Wishlist | `product.wishlist` | `product_wishlist` | One product a visitor or contact saved for later, with the price seen at the time. |
| Product Attribute Category | `product.attribute.category` | `product_attribute_category` | A grouping of product attributes used to organise the specification and comparison tables. |
| Coupon Share Wizard | `coupon.share` | transient | Produces an address that applies a promotional code and lands on a chosen page. |

### 3.5 Forum, blog and public directories

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Forum | `forum.forum` | `forum_forum` | One discussion space with its reputation table, its default mode, its privacy and its welcome content. |
| Forum Post | `forum.post` | `forum_post` | One question or answer, with votes, acceptance, state and moderation data. |
| Forum Post Vote | `forum.post.vote` | `forum_post_vote` | One up-vote or down-vote of one user on one post. |
| Forum Tag | `forum.tag` | `forum_tag` | One label attached to questions of one forum. |
| Forum Post Closing Reason | `forum.post.reason` | `forum_post_reason` | A reason offered when closing a question or marking it offensive, with its kind. |
| Blog | `blog.blog` | `blog_blog` | One blog with its cover, subtitle and site restriction. |
| Blog Post | `blog.post` | `blog_post` | One article with its cover, tags, publication data and reading statistics. |
| Blog Tag | `blog.tag` | `blog_tag` | One label attached to articles. |
| Blog Tag Category | `blog.tag.category` | `blog_tag_category` | A grouping of blog labels for the filter side bar. |
| Partner Website Tag | `res.partner.tag` | `res_partner_tag` | A publishable label used to filter the public customer reference directory. |

---

## 4. Entities owned elsewhere that this folder extends

The extensions listed here are specified in [entities.md](entities.md) part 3.

| Entity | Owning folder | What this folder adds |
|---|---|---|
| View (template definition) | [platform foundation](../platform-foundation/README.md) | The site reference, the page and model-page back links, the tracking flag, the four visibility modes with their password, the theme template pointer and the search engine metadata; plus copy-on-write, copy-on-delete, most-specific selection, visibility enforcement and content saving. |
| Asset | [platform foundation](../platform-foundation/README.md) | The key, the site reference and the theme template pointer; plus copy-on-write and most-specific selection. |
| Attachment | [platform foundation](../platform-foundation/README.md) | The key, the site reference, the theme template pointer, the local address, the image source, width and height, and the original attachment; plus per-site serving order. |
| Model Definition | [platform foundation](../platform-foundation/README.md) | The form opt-in flag, the default free-text field, the form label and the form key; plus the writable-field computation of the form builder. |
| Field Definition | [platform foundation](../platform-foundation/README.md) | The form exclusion flag; plus the deletion guard for fields used in a published form. |
| Server Action | [platform foundation](../platform-foundation/README.md) | The public path, the public address, the publication flag and the shipped-document identifier; plus the public execution endpoint. |
| Request routing | [platform foundation](../platform-foundation/README.md) | Site resolution, language matching and redirection, slug generation and parsing, rewrite application, page and redirect fallback serving, error page selection, the visitor tracking hook and the cookie permission check. |
| Capability package | [platform foundation](../platform-foundation/README.md) | Theme images and the installed-on-this-site flag; plus the theme load, unload, cleanup, upgrade, choose, remove and refresh operations, and the propagation of checkout-step translations. |
| Record Rule | [identity and access](../identity-and-access/README.md) | The current site in the rule evaluation context and in the rule cache key. |
| User | [identity and access](../identity-and-access/README.md) | The site reference; plus per-site login uniqueness, per-site sign-up scope, visitor linking at authentication, wish list migration and the public profile fields. |
| Contact | [contacts and organizations](../contacts-and-organizations/README.md) | The visitors, the public descriptions, the publication flag, the public tags and the wish lists; plus the public detail page address, the map helpers and the publication tracking subtypes. |
| Company | [contacts and organizations](../contacts-and-organizations/README.md) | The site back link; plus the archive guard and the theme selector action. |
| Language | [contacts and organizations](../contacts-and-organizations/README.md) | The frontend language list per site, the alternate-language code computation and the deactivation guard. |
| Sales Order and Sales Order Line | [sales](../sales/README.md) | The site link, the cart behaviour, abandoned-cart detection and recovery, the storefront warning, the cart operations, delivery selection, pickup location, express checkout and payment readiness. |
| Product Template, Product Variant, Product Attribute, attribute lines and values, Product Document, Product Tag | [products and catalog](../products-and-catalog/README.md) | Publication and address, storefront descriptions, categories, ribbons, extra media, cross-selling links, comparison price, base unit, availability settings and the combination information service. |
| Price List and Price List Rule | [pricing and price lists](../pricing-and-pricelists/README.md) | The site link, the promotional code, the selectable flag, the availability rules and the strikethrough decision. |
| Shipping Method and Warehouse | [delivery and shipping](../delivery-and-shipping/README.md) and [inventory operations](../inventory-operations/README.md) | Publication on the site, the storefront description, the collect-in-store type with its stores, opening hours and the pickup-location payload. |
| Payment Provider, Payment Token, Payment Transaction, Payment | [payment providers](../payment-providers/README.md) | The site restriction, the base address override, the pay-on-site mode and its filters, the donation flag and the confirmation of orders paid on site. |
| Loyalty Program and Loyalty Rule | [loyalty and promotions](../loyalty-and-promotions/README.md) | Availability on the site, the site restriction, the code uniqueness across reachable sites and the unpublished trigger-product warning. |
| Journal Entry and Transfer | [general ledger](../general-ledger/README.md) and [inventory operations](../inventory-operations/README.md) | The site stamp of the originating order. |
| Sales Team and Sales Analysis Report | [customer relationship management](../customer-relationship-management/README.md) and [sales](../sales/README.md) | The sites of the team, the abandoned-cart counters, and the site, abandoned-cart flag and storefront categories of the report. |
| Badge, Rank and Reputation Tracking | [learning, surveys and gamification](../learning-surveys-and-gamification/README.md) | The publication flag on a badge, so ranks and badges can be shown on public profiles. |
| Digest Email | [human resources core](../human-resources-core/README.md) | The online sales measure. |
| Configuration Settings | [platform foundation](../platform-foundation/README.md) | Every site and storefront setting exposed in the settings screen. |

---

## 5. Entities in this folder's candidate scope that another folder owns

The scope of this folder lists entities that belong to neighbouring folders because the capability
packages that create them are published on the site. They are named here so that no reader looks for
them in the wrong place.

| Entity | Owned by | Why it appears in this scope |
|---|---|---|
| Portal Record mixin (`portal.mixin`), Portal Share Wizard (`portal.share`), Portal Access Wizard (`portal.wizard`), Portal Access Wizard Line (`portal.wizard.user`) | [customer portal](../customer-portal/) | The authenticated document area shares the site layout and the pager of this folder; this folder only adds the site-aware duplicate-account detection described in [entities.md](entities.md) §3.16. |
| Tracked Link (`link.tracker`), Tracked Link Click (`link.tracker.click`), Tracked Link Code (`link.tracker.code`) | [marketing and mass mailing](../marketing-and-mass-mailing/) | The short-address service is served from the site; the site-specific behaviour is specified in [entities.md](entities.md) §3.17 and [interfaces.md](interfaces.md) §7. |
| Event Track, Event Track Stage, Event Track Tag, Event Track Location, Event Track Visitor, Event Sponsor, Event Sponsor Type, Event Quiz and its questions and answers, Website Event Menu | [events](../events/) | The event capability publishes agendas, talks, exhibitors and booths on the site through the mixins of this folder. |
| Course, Course Slide, Slide Tag, Slide Channel Tag, Slide Channel Partner, Slide Answer, Slide Question, Slide Resource, Slide Embed, Course Invitation | [learning, surveys and gamification](../learning-surveys-and-gamification/) | Courses are published, searched and sold through this folder; the paid-enrolment link is specified in [storefront-engagement.md](storefront-engagement.md) §8. |
| Lead assignment, lead forwarding, partner assignment report, reveal rule and reveal view | [customer relationship management](../customer-relationship-management/) | The public contact form and the partner assignment pages are served by this folder; the lead itself belongs to that folder. |
| Content editor conversion test entities | [platform foundation](../platform-foundation/) | Test fixtures of the rich text editor; they carry no business behaviour. |

---

## 6. Reading order

1. `README.md` — this file: scope, actors, entity list, vocabulary.
2. [glossary.md](glossary.md) — every term used below, in full. Read it before anything else.
3. [entities.md](entities.md) — every field of every entity with type, default, computation and rules.
4. [state-machines.md](state-machines.md) — the publication machine, the page-visibility machine, the rewrite machine, the theme machine, the visitor machine, the blog post machine, the forum post machine, the cart-to-order machine, the checkout step machine, the product feed machine and the wish list machine.
5. [calculations.md](calculations.md) — every formula: site resolution, slug grammar, approximate matching, pagination, teaser extraction, price display, availability, cart totals, delivery cost, reputation points, distances, feed measures.
6. [workflows.md](workflows.md) — the operational procedures end to end.
7. [business-rules.md](business-rules.md) — every validation, constraint, permission check and edge case with the exact message text.
8. [accounting-effects.md](accounting-effects.md) — what this domain does and does not post, and the exact hand-off to the selling and payment domains.
9. [configuration.md](configuration.md) — settings, parameters, default records, groups, access rights, record rules, scheduled jobs and message templates.
10. [interfaces.md](interfaces.md) — navigation, views, named operations, the complete address catalogue, printable documents, message templates and external services.
11. [acceptance-criteria.md](acceptance-criteria.md) — numbered Given, When and Then scenarios with concrete numbers.

The deep topic files are read after the document that introduces them:
[content-management.md](content-management.md),
[multi-site-and-languages.md](multi-site-and-languages.md),
[visitors-and-tracking.md](visitors-and-tracking.md),
[forum-and-blog.md](forum-and-blog.md),
[storefront-catalogue.md](storefront-catalogue.md),
[storefront-checkout.md](storefront-checkout.md),
[storefront-stock-and-pickup.md](storefront-stock-and-pickup.md),
[storefront-engagement.md](storefront-engagement.md).

---

## 7. Every file in this folder

| File | Content |
|---|---|
| [README.md](README.md) | Scope, capabilities, actors, the entities this folder owns and extends, reading order, dependencies and vocabulary. |
| [entities.md](entities.md) | Every field of every owned entity and of every field added to entities owned elsewhere, with types, defaults, derivations, constraints, validation messages and lifecycles. |
| [state-machines.md](state-machines.md) | Every state-bearing field, with stored values, labels, meanings, transitions, guards, side effects, refusal messages and a diagram per machine. |
| [workflows.md](workflows.md) | End-to-end operational procedures with actors, preconditions, numbered steps, records written and postconditions. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with exact messages, permissions, guards, locking and consistency rules, and the mapping of the former rule identifiers. |
| [calculations.md](calculations.md) | Every formula and algorithm with rounding, precision, currency and unit handling, and worked numeric examples. |
| [accounting-effects.md](accounting-effects.md) | The statement that this domain posts no ledger entry, and the exact boundary with the domains that do. |
| [configuration.md](configuration.md) | Settings, system parameters, shipped records, access groups, model access, record rules, scheduled jobs and message templates. |
| [interfaces.md](interfaces.md) | Menus, screens, named operations, the complete address catalogue, generated documents, notifications and external integrations. |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given, When and Then scenarios with concrete records, inputs and results. |
| [glossary.md](glossary.md) | Every term of the domain, defined. |
| [content-management.md](content-management.md) | Deep specification of editable regions, architecture versioning, images and fonts, style variables, themes, dynamic content blocks, the form builder, the anti-robot services and the first-run configurator. |
| [multi-site-and-languages.md](multi-site-and-languages.md) | Deep specification of site resolution, most-specific selection, copy-on-write, language prefixes, canonical and alternate links, the crawler exclusion response, the site index and the site search. |
| [visitors-and-tracking.md](visitors-and-tracking.md) | Deep specification of visitor identity, tracking, merging, statistics, cleanup and cookie consent. |
| [forum-and-blog.md](forum-and-blog.md) | Deep specification of forums, questions, answers, comments, votes, moderation, the reputation economy, blogs, posts, tags, feeds and public profiles. |
| [storefront-catalogue.md](storefront-catalogue.md) | Deep specification of the shop listing, category navigation, search and filtering, sorting, the product page, the combination information contract, ribbons, media, documents, extra fields, base unit prices, price list selection, tax display and the product feed. |
| [storefront-checkout.md](storefront-checkout.md) | Deep specification of the cart, the checkout steps, addresses, delivery selection and rating, express checkout, payment, confirmation, reordering, promotions and the abandoned cart flow. |
| [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) | Deep specification of storefront availability, out-of-stock behaviour, back-in-stock notifications, warehouse selection, collection in store and external parcel-shop pickup. |
| [storefront-engagement.md](storefront-engagement.md) | Deep specification of wish lists, product comparison, cross-selling blocks, visitor product tracking, promotions on the storefront, newsletter subscription, donations, course products and print-on-demand rules. |

---

## 8. Dependencies on other domains

| Domain | What this domain takes from it | Where it is specified |
|---|---|---|
| Platform foundation | Template definitions and template inheritance, assets and bundles, attachments, the rendering engine, the routing table, external identifiers, package installation, configuration parameters, scheduled jobs and the translation machinery. | [`../platform-foundation/README.md`](../platform-foundation/README.md) |
| Identity and access | Users, groups, the public user, sign-up, access rights and record rules. | [`../identity-and-access/configuration.md`](../identity-and-access/configuration.md) |
| Contacts and organizations | Contacts, addresses, countries, country states, companies and languages. | [`../contacts-and-organizations/entities.md`](../contacts-and-organizations/entities.md) |
| Messaging and activities | The message thread on a record, the message templates, the notification recipients and the composer. | [`../messaging-and-activities/workflows.md`](../messaging-and-activities/workflows.md) |
| Products and catalogue | Product templates, variants, attributes, attribute values, combinations, combos, documents and tags. | [`../products-and-catalog/entities.md`](../products-and-catalog/entities.md) |
| Pricing and price lists | The price computation algorithm, rule selection and the price list of a visitor. | [`../pricing-and-pricelists/calculations.md`](../pricing-and-pricelists/calculations.md) |
| Sales | The sales order, its lines, the confirmation algorithm, the invoicing policy and the sales analysis report. | [`../sales/workflows.md`](../sales/workflows.md) |
| Taxes | Tax computation for displayed prices and for the order, and fiscal position mapping. | [`../taxes/calculations.md`](../taxes/calculations.md) |
| Payment providers | Transactions, tokens, the express-checkout contract and the post-processing that confirms the order. | [`../payment-providers/state-machines.md`](../payment-providers/state-machines.md) |
| Delivery and shipping | Delivery methods, rate requests, pickup points and the shipping line. | [`../delivery-and-shipping/calculations.md`](../delivery-and-shipping/calculations.md) |
| Inventory operations | Quantities free to use, warehouses and transfers. | [`../inventory-operations/calculations.md`](../inventory-operations/calculations.md) |
| Manufacturing | The availability of a product whose composition is a kit. | [`../manufacturing/calculations.md`](../manufacturing/calculations.md) |
| Loyalty and promotions | Programs, codes, rewards, gift cards and electronic wallets applied to the cart. | [`../loyalty-and-promotions/workflows.md`](../loyalty-and-promotions/workflows.md) |
| Accounts receivable | Customer invoices produced from confirmed online orders. | [`../accounts-receivable/workflows.md`](../accounts-receivable/workflows.md) |
| Customer portal | The pager helper reused by public listings, the portal layout and the access-granting wizard extended here. | [`../customer-portal/README.md`](../customer-portal/README.md) |
| Customer relationship management | The lead created by a public contact form and the sales teams of a site. | [`../customer-relationship-management/workflows.md`](../customer-relationship-management/workflows.md) |
| Learning, surveys and gamification | Reputation points, ranks and badges shown on public profiles, and the courses sold on the storefront. | [`../learning-surveys-and-gamification/entities.md`](../learning-surveys-and-gamification/entities.md) |
| Marketing and mass mailing | Tracked links, campaign parameters and the mailing lists used by the newsletter checkbox. | [`../marketing-and-mass-mailing/entities.md`](../marketing-and-mass-mailing/entities.md) |
| Events | The events, talks, exhibitors and booths published on the site. | [`../events/README.md`](../events/README.md) |

Platform-wide material is in [`../../overview/architecture.md`](../../overview/architecture.md),
[`../../overview/security-model.md`](../../overview/security-model.md),
[`../../overview/views-and-actions.md`](../../overview/views-and-actions.md),
[`../../runtime/README.md`](../../runtime/README.md),
[`../../data/README.md`](../../data/README.md) and
[`../../interfaces/README.md`](../../interfaces/README.md).

---

## 9. What this domain does not cover

- The back-office user interface framework, its client-side rendering and its asset pipeline, except
  where public page rendering depends on them; see
  [`../../overview/views-and-actions.md`](../../overview/views-and-actions.md).
- Hosting, deployment, process management and server sizing — excluded by
  [`../../references/documentation-rules.md`](../../references/documentation-rules.md) rule four.
- The price computation itself: only its *invocation* from the storefront and its *display* rules are
  specified here; the algorithm lives in
  [`../pricing-and-pricelists/calculations.md`](../pricing-and-pricelists/calculations.md).
- The sales order confirmation side effects (procurement, invoicing) — see
  [`../sales/workflows.md`](../sales/workflows.md).
- The payment provider protocols — see
  [`../payment-providers/interfaces.md`](../payment-providers/interfaces.md).
- The reputation engine itself (ranks, badges, challenges) — see
  [`../learning-surveys-and-gamification/entities.md`](../learning-surveys-and-gamification/entities.md);
  this folder specifies every award and gate the forum contributes to it.

---

## 10. Vocabulary shortcuts used throughout

| Term | Meaning in this folder |
|---|---|
| **Site** | One record of the Website entity. "Current site" is the one the present request resolved to. |
| **Generic record** | A record whose site reference is empty: it is shared by every site. |
| **Specific record** | A record whose site reference names one site: only that site sees it. |
| **Public user** | The anonymous identity a request runs as when nobody is signed in; each site names its own. |
| **Front end** | Request handling for a public page, as opposed to the back office. |
| **Cart** | A draft Sales Order bound to a site and to the visitor's session. |
| **Reputation** | The numeric score a forum participant accumulates; it gates every forum operation. |
| **Copy-on-write** | The rule that writing to a generic record while acting for one site creates and writes a specific copy instead. |
| **Content block** | A reusable, editable markup fragment dropped into a page by the editor; a *dynamic* content block renders records selected by a Website Content Block Filter. |
| **Template** | A stored, renderable markup tree; pages, layouts and content blocks are all templates. |

---

## 11. Reconciliation notes

Two independently written versions of this folder were merged. Where they disagreed, the source tree
decided; the resolutions that affect names or placement are recorded here, and the resolutions that
affect behaviour are recorded at the end of the file that carries them.

1. **Names of entities.** One version named entities by their transport name ("Website Controller
   Page", "Public Category", "Page Visit", "Storefront Unit", "Extra Field", "Wish List Entry",
   "Website Snippet Filter"), the other by a business reading ("Website Model Page", "Website Product
   Category", "Website Visit Track", "Base Unit Display", "Storefront Extra Field", "Product
   Wishlist", "Website Content Block Filter"). The business readings are kept in prose because they
   are self-explanatory to a reader who has never seen the system; every table nevertheless carries
   the transport name and the storage name, which are the contractual strings.
2. **Delivery methods.** One version placed Shipping Method in the inventory operations folder, the
   other in the delivery and shipping folder. The charter's folder list contains
   [delivery and shipping](../delivery-and-shipping/README.md), which owns delivery methods and rate
   computation; warehouses and quantities stay with
   [inventory operations](../inventory-operations/README.md). Both links are used accordingly.
3. **The customer portal.** One version treated the authenticated document area as part of this
   folder. It is owned by [customer portal](../customer-portal/); this folder keeps only the
   site-aware extensions listed in §5.
4. **Tracked links.** Both versions agree that the entity belongs to
   [marketing and mass mailing](../marketing-and-mass-mailing/); its public serving on a site is
   specified here.
5. **Folder renames.** Links written for the working branch were renamed: the messaging folder is
   [messaging and activities](../messaging-and-activities/), the electronic-learning folder is
   [learning, surveys and gamification](../learning-surveys-and-gamification/), the digest and
   gamification material is in [human resources core](../human-resources-core/), and the two source
   folders of this consolidation (site content management and commerce storefront) are this one.
