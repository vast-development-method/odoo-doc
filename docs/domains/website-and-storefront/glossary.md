# Glossary

Every term this folder uses, in alphabetical order. A term owned by another folder carries a pointer
to its owner; the definition given here is the meaning it has on the public site.

**Abandoned cart.** A draft Sales Order attached to a site, holding at least one line, whose customer
is not the site's public contact and whose order date is at or before the present moment minus the
site's abandoned-cart delay. It is a derived flag, not a state.

**Abandoned-cart delay.** The number of hours, configurable per site, after which an untouched cart
counts as abandoned. Default 10.0; a value of zero is read as one hour.

**Access token (order).** The opaque secret stored on a Sales Order that lets a person who holds the
link open, pay or revive that order without signing in. Owned by
[customer portal](../customer-portal/).

**Access token (product feed).** The opaque secret that must accompany a feed request. It is compared
in constant time.

**Access token (visitor).** The value that identifies one browsing identity: the decimal contact
identifier of the signed-in user, or a 32-character digest of the network address, the browser user
agent and the session identifier.

**Accessory product.** A product suggested in the cart alongside a product already in it. The link is
template to variant.

**Activation moment.** The moment at which the abandoned-cart recovery feature was switched on for a
site. Carts created before it are never mailed.

**Active menu entry.** The menu entry whose target matches the address being served; the matching
rules are in [entities.md](entities.md) §1.4.

**Add-to-cart permission check.** The set of conditions a variant must satisfy before it may enter a
cart: active, published, matching the site sellable-product condition, priced when the site forbids
zero prices, and the site granting shop access.

**Alternate-language link.** A declaration emitted in the document head that points at the same page
in another language of the site.

**Alternative product.** A product suggested on another product's page as a higher-tier or different
choice. The link is template to template.

**Anonymous cart.** A cart whose customer is still the site's public contact. It stops being
anonymous as soon as an address is submitted.

**Answer.** A forum post that carries a parent question.

**Approximate matching.** The search behaviour that, when the exact term matches nothing, proposes
the closest word found in the searched fields and searches that word instead.

**Architecture.** The stored markup tree of a template. A page's content is the architecture of the
template it delegates to.

**Asset customisation.** A replacement file stored per site under the customisation address
prefix, together with the asset record that makes the compiled bundle use it instead of the
shipped file.

**Attribute filter.** The shop listing filter carried by repeated query parameters of the form
attribute identifier, a hyphen, then one or more attribute value identifiers separated by commas.
Values of one attribute widen the result; different attributes narrow it.

**Autocompletion answer.** The compact search answer returned while a visitor types, carrying the
rendered results, the total count, the mapped slot names and the substituted term when one was
used.

**Badge.** A reward shown on a public profile when a participant reaches a stated achievement.
Owned by [learning, surveys and gamification](../learning-surveys-and-gamification/); this folder
ships the forum badges and adds the publication flag that shows or hides one.

**Base unit.** See reference unit.

**Blog.** A publication channel holding blog posts, with a cover, a subtitle, free landing-page
content and an optional site restriction.

**Blog post.** One article of a blog, with an author, a content, a teaser, tags, a cover, a
publishing date and a view counter.

**Business-to-business fields.** The optional address form block that collects a company name and a
tax identification number.

**Canonical address.** The single address a page declares as its own for search engines: the current
address rebuilt for the current language against the site's base address, without a query string and,
for a product page, without a category segment.

**Cart.** A draft Sales Order attached to a site and carried by a browsing session.

**Cart quantity.** The whole-number sum of the displayed line quantities of a cart, used by the header
counter. Promotion reward quantities are subtracted.

**Checkout step.** One publishable, ordered stage of the checkout flow, with a path, a forward label
and a backward label. Each site owns a copy of the generic steps.

**Click and collect.** See collection in store.

**Closing reason.** The reason recorded when a forum question is closed or a post is marked
offensive. Two shipped reasons carry a reputation penalty.

**Collection in store.** The delivery arrangement in which the shopper reserves online and collects
in one of the seller's stores. Implemented as a delivery method of the collect-in-store kind whose
stores are warehouses.

**Combination.** A set of attribute values chosen for one product template. It may or may not
correspond to an existing variant.

**Combination information.** The payload returned for a combination: the resolved variant, the
display name, the price, the reference price, the discount flag, the extra-price flag, the
availability, the media and every other value the product page needs.

**Combo.** A product sold as a bundle of choices. A combo line in a cart is accompanied by one combo
item line per selected choice, all carrying the same quantity.

**Comment (forum).** A message on the discussion thread of a post. It is not a post and carries no
vote.

**Comparison price.** A manually entered reference price shown struck through on the shop tile and
the product page, only when no price list discount applies and the comparison-price feature is
enabled. It is never taxed.

**Consent bar.** The banner that asks the visitor to accept or refuse optional cookies.

**Content block.** A reusable, editable markup fragment dropped into a page by the editor. A *dynamic*
content block renders records selected by a Website Content Block Filter.

**Content delivery network.** An external service that serves static asset files. When it is
enabled, an asset address matching one of the configured patterns is rewritten to the network base
address.

**Copy-on-write.** The rule that writing to a generic record while acting for one site creates and
writes a site-specific copy instead, leaving the other sites untouched.

**Cover properties.** The serialised background picture, colour class, opacity and height class of a
blog, a post, an event or a course.

**Crawler.** An automated agent identified by its user agent text. Crawlers are never tracked and
never redirected by browser language preference.

**Crawler exclusion response.** The plain-text document served at `/robots.txt` that tells crawlers
what they may fetch and where the site index is.

**Custom value.** A free-text value a shopper types for an attribute value marked as accepting one.
It is stored on the cart line and prevents that line from merging with another.

**Decoy field.** A form field that no human fills, used to recognise an automated submission. Also
called a honeypot field. The observed behaviour records its value as free text; refusing the
submission is the industry-standard default WS-900 of [business-rules.md](business-rules.md).

**Delivery line.** The ordinary Sales Order Line carrying the delivery product of the selected
shipping method, at the computed rate, quantity one.

**Display currency.** The currency in which the storefront shows prices: the currency of the price
list in force during a storefront request, and the company currency otherwise.

**Editable region.** An element of a rendered page that the editor may change, either because it
carries the editable marker or because it is bound to a record field.

**Editor and Designer.** The internal group responsible for a site. It implies the Restricted
Editor group and the sanitisation override, and it may create and delete structural records.

**Embargo.** The state of a published page or blog post whose publishing date is still in the
future: visitors receive "not found" and designers still see it.

**Express checkout.** A one-click flow in which a payment wallet supplies the addresses and the
shipping choice, and the shopper pays the order amount excluding delivery.

**Extra price.** The additional amount an attribute value adds to a product's price, shown next to
the value on the product page, converted to the display currency and taxed like the price. Hidden
when a fixed-price price list rule applies.

**Favourite (forum).** A question a participant marked to follow; marking it also subscribes the
participant to the thread.

**Feed.** See product feed.

**Fiscal position.** The mapping that turns a product's own taxes into the taxes actually charged and
the accounts into the accounts actually used. Owned by [taxes](../taxes/README.md); the storefront
resolves one per session and per cart.

**Flagged post.** A forum post a participant reported for moderation; it leaves the public listings
until a moderator validates or refuses it.

**Forum.** One discussion space, with its own reputation table, answering mode, privacy,
guidelines and welcome banner.

**Forum tag.** A label attached to the questions of one forum. Its name is unique within the
forum.

**Free quantity.** The quantity of a product available to promise, that is on hand minus reserved.
Owned by [inventory operations](../inventory-operations/README.md); the storefront reads it with the
site warehouse or the selected store in context and rounds it down to a whole number.

**Free-shipping threshold.** The order amount excluding delivery, expressed in the company currency,
at or above which a shipping method's rate becomes zero. The delivery line is kept at price zero.

**Front end.** Request handling for a public page, as opposed to the back office.

**Generic checkout step.** A checkout step with no site. It is the template from which each site's
copy is made.

**Generic record.** A record whose site reference is empty: it is shared by every site.

**Guest checkout.** Completing a purchase without creating an account. Allowed when the site's
account policy is optional or disabled.

**Guidelines.** The page of a forum that explains its rules and its reputation table.

**Home page bootstrap.** The sequence that gives a newly created site a home page, a root menu tree
and a menu entry pointing at that page.

**Honeypot field.** Another name for the decoy field; see decoy field.

**Indexing flag.** The page flag that decides whether the page is enumerated in the site index and
whether a no-index instruction is emitted in the document head.

**Language prefix.** The first path segment of an address when it names a language, for example
`/fr/`. The default language of a site carries no prefix.

**Linked line.** A cart line attached to a parent line: an optional product line or a combo item
line. It is deleted with its parent.

**Listing condition.** The complete search condition of the shop listing: the site sellable-product
condition, narrowed by the search term, the category, the attribute filter, the tag filter and the
price range.

**Media list.** The ordered list of things shown in the product carousel or grid: the main picture
(the variant or the template record itself) followed by the extra media rows.

**Mega menu.** A menu entry whose panel is a markup fragment rather than a list of child entries. A
mega menu has no parent and no child.

**Menu entry.** One node of a site's navigation tree. The tree is at most two levels deep; an
entry targets a page, a model page or a free address.

**Model page.** A page that exposes a whole business entity publicly under `/model/<segment>`, with a
listing template, a record template and a restriction condition.

**Moderator.** A forum participant whose reputation reaches the moderation threshold of that
forum. Moderation is a score, not a group.

**Most-specific selection.** The rule that, for one key or one address, the record bound to the
current site wins over the shared record.

**Multi-site user.** The flag group granted automatically as soon as a second site exists. It
makes the site selector and the site column visible.

**No-variant attribute.** An attribute whose values do not create variants. Its chosen values are
stored on the cart line rather than on a variant, and they participate in line matching.

**Page.** A static publishable address whose content is a template architecture, with a
publication flag, a publishing date, an indexing flag, a visibility mode and optional menu
entries.

**Page visibility.** The gate applied to the main content template of a request: public, signed in,
restricted group or password.

**Palette.** The named set of colours a theme applies. Setting it resets the user palette, the
grey palette, the semantic colours and the preset gradients.

**Pending post.** A forum question created by a participant whose reputation is below the validation
threshold. It waits for a moderator.

**Pickup location.** The payload describing the place at which an order is collected: a store of a
collect-in-store method, or a parcel shop of an external relay network.

**Preview of variants.** The small selection of an attribute's values shown directly on a product
card of the shop grid, each linking to the product page pre-filtered on that value.

**Price before discount.** The price a product would have without the applied price list rule, used
as the strikethrough reference when the rule is of a kind that shows a discount on the catalogue.

**Price list in force.** The price list used for the current request, resolved from the session, the
cart, the visitor's country group, the contact's assignment and the site's price lists, in that order
of precedence.

**Product feed.** A published, token-protected syndication document exposing the published catalogue
of one site to an external product syndication service, in one language and one price list, rendered
at most once a day.

**Product ribbon.** A coloured banner or badge overlaid on a product picture, assigned manually or
automatically when the product is on sale, newly published or out of stock.

**Public contact.** The contact of the site's public user. It owns anonymous carts and is excluded
from abandoned-cart detection.

**Public profile.** The page that shows a participant's reputation, rank, badges and activity, gated
by the site's minimum reputation.

**Public user.** The anonymous identity a request runs as when nobody is signed in; each site names
its own.

**Question.** A forum post with no parent.

**Quick add.** The add-to-cart action offered directly on a product card, without opening the product
page.

**Rank.** A named level derived from a reputation score, shown on a public profile. Owned by
[learning, surveys and gamification](../learning-surveys-and-gamification/).

**Recovery message.** The message sent for an abandoned cart, carrying a link that revives or merges
that cart.

**Reference unit.** A named unit used only for the storefront per-unit price, for example `100 g` or
`750ml`. It is unrelated to the unit of measure used for stock and invoicing. Its multiplier on one
sales unit is the base unit count; a count of zero hides the per-unit price.

**Relay point.** A parcel shop of an external pickup network, stored as a dedicated contact under the
customer and used as the delivery address.

**Relevance (forum).** The ordering score that combines the vote count and the age of a question,
using the two parameters of the forum.

**Reputation.** The numeric score a forum participant accumulates. Every forum operation is gated by
a reputation threshold, and every vote, acceptance and moderation decision moves the score.

**Restricted Editor.** The internal group allowed to edit page content and product presentation
but not to create or delete structural records.

**Revival method.** How an abandoned cart is brought back: "squash" replaces the current cart with the
old one, "merge" moves the old lines into the current cart and cancels the old order.

**Rewrite rule.** A record that redirects, aliases or suppresses one address, optionally on one site
only.

**Search descriptor.** The structure an entity returns to declare how it participates in the site
search: entity, base condition, searched fields, read fields, slot mapping, icon and ordering.

**Sellable-product condition.** The condition that decides whether a product may appear on a given
site: sellable, matching the site, matching the company and, for non-internal users, published with a
sellable service tracking value.

**Session cart key.** The browsing-session entry holding the identifier of the current cart, or the
value false to record that the signed-in shopper has none.

**Shop ordering value.** The integer that orders products on the shop grid under the featured sort.
New products receive the current maximum plus five.

**Shop-access check.** The test that refuses shop pages to anonymous visitors when the site restricts
the shop to signed-in users.

**Short address.** A tracked link of the form `/r/<code>` that records a click and redirects to a
target address.

**Similarity index.** A storage-engine index on a text column that makes approximate matching
affordable. The storefront search fields and the internal reference carry one.

**Site.** One record of the Website entity. "Current site" is the one the present request resolved
to.

**Site configurator.** The guided first-run sequence that collects the industry, the purpose, the
palette and the wanted features, installs the matching packages, creates the matching pages and
menus and fills them with content blocks.

**Site index.** The generated, cached document that enumerates the addresses of a site for crawlers.

**Site resolution.** The algorithm that maps one incoming request onto exactly one site, described
in [calculations.md](calculations.md) §2.

**Slug.** A path segment made of a readable, language-dependent label, a hyphen and the record
identifier, for example `my-first-post-7`.

**Sold out.** A tracked product whose out-of-stock ordering is disabled and whose storefront quantity
free to use is at or below zero.

**Specific record.** A record whose site reference names one site: only that site sees it.

**Storefront extra field.** A declaration that one text or binary field of the Product Template must
be printed in the details block of the product page of one site.

**Strikethrough price.** The reference price shown crossed out next to the price. It comes either
from the price list rule or from the comparison price, never from both.

**Style variable.** A named design value stored in a variable map inside a style source file.
Editing a design option in the theme panel writes a pair into that map.

**Tax display mode.** The site setting that decides whether storefront prices and cart subtotals are
shown tax excluded or tax included. It never changes what is stored.

**Teaser.** The short excerpt of a blog post shown in listings: the manual teaser when one is
written, otherwise the first 200 characters of the plain text of the content followed by an ellipsis.

**Template.** A stored, renderable markup tree. Pages, layouts and content blocks are all
templates; a template may be shared by every site or specific to one.

**Theme.** An installable capability package that ships templates, assets, attachments, pages and
menus, copied into live records per site when it is applied.

**Tracked link.** A short address that records each click before redirecting to its target. Owned
by [marketing and mass mailing](../marketing-and-mass-mailing/); this folder serves it from the
site.

**Tracking payload.** The analytics description of the cart or of the order, listing each line with
an item identifier, a name, a category, a price, a discount and a quantity.

**Visit track.** One recorded page view of one visitor: the visitor, the full address, the page
when the served content was a page, optionally the product viewed, and the moment.

**Visitor.** One browsing identity, keyed by a browsing token, carrying the site, the language,
the country, the time zone, the visit count and the page-view history. It becomes identified when
the person signs in.

**Vote.** A forum participant's up-vote, down-vote or withdrawn vote on one post. A participant may
not vote on their own post.

**Website Product Category.** A storefront-facing, hierarchical product category used for navigation,
filtering, category pages, content blocks and feed filtering. Distinct from the internal product
category used for accounting.

**Wish list row.** One product saved for later by a contact or by a browsing session, with the price
and the price list captured at the moment it was saved.
