# Configuration

Every setting, system parameter, fixed constant, shipped record, access group, model access rule, record
rule, scheduled job, message template and message subtype of this folder, with its data type, its default
value and its effect.

Settings marked **per site** are read from and written to the Website record selected in the settings
screen; settings marked **per company** are stored as company defaults; settings marked **feature group**
grant or revoke an access group for every internal user; settings marked **package** install or uninstall
a capability package; settings marked **parameter** are stored as a configuration parameter.

---

## 1. Settings exposed in the configuration screen

### 1.1 The site itself

| Setting | Scope | Type | Default | Effect |
|---|---|---|---|---|
| Website Name | per site | text | none | The site name, used in the browser title suffix and in the sharing metadata. |
| Website Domain | per site | text | empty | The public domain. Drives site resolution, absolute addresses, indexing decisions and cross-site navigation. Normalised by WS-004 and validated by WS-002 and WS-003. |
| Company | per site | many_to_one to Company | the current company | Drives the currency, the public user and the allowed company list during a front-end request. |
| Languages | per site | many_to_many to Language | every active language | The languages the site is published in. |
| Default Language | per site | many_to_one to Language | the default contact language, otherwise the first active language | The language served without a prefix. |
| Redirect to the browser language | per site | boolean | true | Redirect a visitor whose browser prefers another available language. |
| Homepage Url | per site | text | empty | Requests to the site root are internally rerouted to this path. |
| Logo | per site | binary | the shipped default logo | Also the default sharing picture. |
| Favicon | per site | binary | the shipped default icon | Centre-cropped and resized to 256 by 256 on write. |
| Default Social Share Image | per site | binary | empty | Replaces the logo as the default sharing picture. |
| Social links | per site | eight text fields | the company's values | Links to the site's accounts on the short-message network, the social network, the code-hosting service, the professional network, the video service, the picture network, the short-video network and the chat service. |
| Cookies Bar | per site | boolean | false | Shows the consent bar and refuses optional cookies until consent is given. Switching it on creates the cookie policy page (WS-017); switching it off deletes it (WS-018). |
| Block 3rd-party domains | per site | boolean | true | Neutralises embedded frames and scripts of blocked domains while optional consent is missing. |
| Custom Blocked Third-Party Domains | per site, Editor and Designer only | long text | empty | One host per line; a comment line starts with a number sign; a first line of `#ignore_default` replaces the shipped list instead of extending it. |
| Google Analytics Key | per site | text | empty | The measurement identifier of the Google Analytics audience measurement service; when set, the measurement script is injected in the document head of every page that is not being edited. |
| Google Search Console | per site | text | empty | The verification token of the Google Search Console service. |
| Google Maps Api Key | per site | text | empty | The access key used to render static and interactive maps. |
| Plausible Shared Key, Plausible Site | per site | text | empty | The shared authentication key and the registered site name of the second audience measurement service, used to embed its dashboard. |
| Custom head code, Custom end-of-body code | per site | rich text, never sanitised | empty | Markup injected at the end of the document head and of the document body. |
| Robots.txt | per site, Editor and Designer only | rich text, never sanitised, never translated | empty | Custom text appended to the crawler exclusion response. |
| Content Delivery Network | per site | boolean, text, long text | false, empty, the six shipped patterns of §3 | Rewrites matching asset addresses to the network base address. |
| Theme | per site | many_to_one to a capability package | none | The theme applied to the site. |
| Shared Customer Accounts | per site | boolean, stored inverted as "site-specific accounts" | shared | When accounts are site-specific, an account created on one site cannot sign in on another. |
| Customer Account | per site | selection: on invitation, free sign-up | on invitation | Whether visitors may create an account themselves. Written together with the checkout account policy of §1.3. |
| Karma to view a profile | per site | integer | 150 | Minimum reputation score required to view another account's public profile. |

### 1.2 Shop and catalogue

| Setting | Scope | Type | Default | Effect |
|---|---|---|---|---|
| Base Unit Price | feature group | boolean | false | Enables the per-unit price on storefront pages and adds the reference unit label and the price per reference unit to the combination information payload. |
| Comparison Price | feature group | boolean | false | Enables the manually entered strikethrough price on the product page and on the shop tiles. |
| Product Feed | feature group, mirrored on every site | boolean | false | Enables the product syndication feed, its menu and its public endpoint. Writing it also writes the feed flag on **every** site, so that the behaviour is consistent, and creates one starter feed per site that has none and is within the soft product limit. |
| Ecommerce Access | per site | selection: `everyone` = All users, `logged_in` = Logged in users | `everyone` | Whether anonymous visitors may reach shop pages. |
| Prevent Sale of Zero Priced Product | per site | boolean | false | Hides the price and the add-to-cart action of a zero-priced product, refuses zero-priced lines and blocks checkout when one exists. |
| Contact Us Button Url | per site | text, translatable | `/contactus` | Destination of the button that replaces the add-to-cart action when the price is hidden. |
| Display Product Prices | per site | selection: `tax_excluded`, `tax_included` | computed, reset to `tax_excluded` when the company's fiscal country changes | The storefront tax display mode. |
| Add to Cart action | per site | selection: `stay`, `go_to_cart` | `stay` | What the page does after a successful add to cart. |
| Number of products in the grid | per site | integer | 21 | Page size of the shop listing. |
| Number of grid columns | per site | integer | 3 | Columns of the shop listing; a falsy value is read as 4 at render time. |
| Grid gap | per site | text | `16px` | Gap between tiles. |
| Shop page width | per site | selection: `regular`, `fluid` | `regular` | Page container of the shop listing. |
| Default sort | per site | selection, five values | `website_sequence asc` | The order applied when the shopper has not chosen one. |
| Shop product design | per site | text | a fixed token list | Opaque presentation tokens of the product tile. |
| Product page settings | per site | see [entities.md](entities.md) §5.1 | see the same section | Media layout, width, spacing, roundness, ratio, ratio on narrow screens, grid columns, column order and container. |
| Wish list page settings | per site | see [entities.md](entities.md) §5.1 | see the same section | Wish list tile design, columns, columns on narrow screens and gap. |
| Product Page Extra Fields | per site | list of Storefront Extra Field | empty | Which Product Template fields are printed in the product details block. |

### 1.3 Orders, accounts and checkout

| Setting | Scope | Type | Default | Effect |
|---|---|---|---|---|
| Customer Accounts | per site | selection: `optional`, `disabled` = buy as guest, `mandatory` = no guest checkout | `optional` | Guest checkout policy. Writing it also writes the sign-up policy of §1.1: free sign-up for `optional` and `mandatory`, invitation-only for `disabled`. The settings screen reports `disabled` when the site has no value. |
| Salesperson | per site | many_to_one to User, internal users only | none | Assigned to orders of this site at confirmation; also the sender of back-in-stock messages and the recipient of feed warnings. |
| Sales Team | per site | many_to_one to Sales Team | the shipped site sales team | Assigned to orders of this site. |
| Order Confirmation email template | per site | many_to_one to Message Template restricted to Sales Order | the platform default confirmation template, or the one named by `sale.default_confirmation_template` | The template used to confirm an order of this site. |
| Abandoned Email | per site | boolean | false | Enables the recovery job for this site; switching it on stamps the activation moment. |
| Send after | per site | decimal, hours | 10.0 | The abandoned-cart delay. Zero is read as one hour. |
| Cart Recovery Email | per site | many_to_one to Message Template restricted to Sales Order | the shipped cart recovery template | The template used by the manual recovery action. |
| Extra Step | per site | boolean, stored as the active state of the extra-information page option and the publication flag of the extra-information checkout step | false | Adds the extra-information step to the checkout flow. |
| Newsletter | per site | boolean, stored as the active state of the newsletter page option | false | Shows the newsletter box on the address form. |
| Newsletter List | per site | many_to_one to Mailing List | none | The list a shopper is subscribed to from the checkout. |

### 1.4 Delivery and inventory

| Setting | Scope | Type | Default | Effect |
|---|---|---|---|---|
| Click & Collect | package | boolean | false | Installs the collect-in-store capability, which adds the collect-in-store delivery kind, the store selector and the pay-on-site provider. |
| Warehouse | per site | many_to_one to Warehouse, restricted to the site's company | none | The warehouse used for storefront availability and assigned to carts of this site. |
| Continue selling when out-of-stock | per company | boolean | true | Default out-of-stock ordering flag of new products. |
| Show Available Quantity | per company | boolean | false | Default availability display flag of new products. |
| Show Threshold | per company | decimal | 5.0 | Default availability threshold of new products. |

Changing a product default does not change existing products.

### 1.5 Integrations and verification

| Setting | Scope | Type | Default | Effect |
|---|---|---|---|---|
| Address Autocomplete | package | boolean | false | Installs the address autocompletion capability. |
| Google Places access key | per site, administrator only | text | none | The access key used by the address autocompletion during checkout. The storefront uses the site key rather than the back-office key. |
| Enable reCAPTCHA | parameter `enable_recaptcha`, administrator only | boolean | true | Enables the score-based human verification service. |
| Site key, Secret key of the score-based service | parameters `recaptcha_public_key` and `recaptcha_private_key` | text | none | The public and private keys of the Google reCAPTCHA service. When no private key is stored, the check passes. |
| Minimum score | parameter `recaptcha_min_score` | decimal | 0.5 | A verification answer below this score is treated as suspicious. |
| Site key, Secret key of the challenge-widget service | parameters `cf.turnstile_site_key` and `cf.turnstile_secret_key` | text | none | The keys of the Cloudflare Turnstile service. When no secret key is stored, the check passes. |
| Picture library access | parameters `unsplash.access_key` and `unsplash.app_id` | text | none | The access key and the application identifier of the external picture library offered inside the media dialogue. |

### 1.6 Actions offered from the settings screen

| Action | Effect |
|---|---|
| Configure Methods | Opens the shipping method list. |
| Find a Delivery Provider | Opens the list of installable delivery integration packages. |
| Attributes | Opens the product attribute list. |
| Pricelists | Opens the price list list. |
| Customize Abandoned Email Template | Opens the shipped cart recovery template in a form. |
| Customize Email Templates | Opens the list of every message template of the Sales Order entity. |
| Manage feeds | Opens the product feeds of the selected site in a dialogue, with that site preset and its column hidden. |
| Extra Info | Opens the extra-information page of the selected site in the editor, with the editing flag in the address. |
| Configure Pickup Locations | Opens the collect-in-store shipping methods: the form when exactly one exists, the list filtered on the collect-in-store kind otherwise. |
| Activate a payment provider | Starts the payment onboarding from the site settings menu. |
| Edit robots.txt | Opens the crawler exclusion dialogue for the selected site. |
| Edit blocked third-party domains | Opens the blocked-domain dialogue for the selected site. |

---

## 2. System parameters

| Parameter | Type | Default | Effect |
|---|---|---|---|
| `website.visitor.live.days` | integer, days | 60 | Retention of anonymous visitors before the daily cleanup deletes them. |
| `website.disable_delay_translations` | text read as a boolean | unset | When set to any value other than `False`, `0` or empty, delayed translation is disabled when content is saved, so that a save discards the translations of unchanged terms. |
| `website_form_enable_metadata` | text read as a boolean | unset | When set, the network address, the user agent, the accepted languages and the referring address of a form submission are appended to the created record under the heading `Metadata`. |
| `website.apply_new_theme` | text | unset | Restricts a theme load started from the interface to the current site. |
| `enable_recaptcha` | boolean | true | See §1.5. |
| `recaptcha_public_key`, `recaptcha_private_key`, `recaptcha_min_score` | text, text, decimal | none, none, 0.5 | See §1.5. |
| `cf.turnstile_site_key`, `cf.turnstile_secret_key` | text | none | See §1.5. |
| `unsplash.access_key`, `unsplash.app_id` | text | none | See §1.5. |
| `auth_signup.invitation_scope` | selection | invitation-only, set to free sign-up when the forum capability is installed | The platform sign-up policy, overridden per site by the site's own policy. |
| `website_profile.uuid` | text | generated | The secret used to derive the address-validation token of a public profile. |
| `database.secret` | text | generated | The key used for the form signature of WS-214. |
| `database.uuid` | text | generated | Sent to the text generation service so that it can attribute its answers. |
| `web.base.url` | text | the installation address | The platform base address, used when no site domain applies. |
| `website_sale.require_billing_details_for_services` | text read as a boolean | true | When false, an order containing only services requires only a name and an address at checkout, which shortens the flow but prevents a country-based fiscal position and price list from being resolved. |
| `website_sale.markup_data_limit_variants` | integer | unset | Caps the number of variants described in the structured product description of a template. |
| `sale.automatic_invoice` | boolean | false | When true, a completed payment transaction creates, posts and sends the invoice of its orders, including for a partial payment. |
| `sale.default_confirmation_template` | reference to a Message Template | unset | The default value of the site confirmation template. |
| `sale.async_emails` | boolean | false | When true, the automatic invoice is sent by the deferred sending job instead of immediately. |
| `website_sale_coupon.abandonned_coupon_validity` | integer, days | 4 | How long a coupon stays attached to an untouched storefront cart before it is released. |
| `portal.allow_api_keys` | boolean | false | Whether a portal user may create a programmatic access key; read by the portal pages this folder serves. |

---

## 3. Fixed constants and shipped default values

| Constant | Value | Use |
|---|---|---|
| Shipped content delivery network patterns | Six matching patterns anchored at the start: any package static folder, the compiled style and script folder, the picture endpoint, the content endpoint, the compiled asset folder, and the legacy picture folder | The default value of the network filters. |
| Page cache lifetime | 3600 seconds | §29 of [calculations.md](calculations.md). |
| Site index page size | 45000 locations | One index document. |
| Site index cache lifetime | 12 hours | Before a full regeneration. |
| Visitor visit interval | 8 hours | Below it, a page view does not count as a new visit. |
| Visitor connected window | 5 minutes | Above it, a visitor is no longer reported as connected. |
| Track deduplication window | 30 minutes | Outside the page pipeline. |
| Visitor cleanup batch | 1000 visitors | One run of the daily job. |
| Consent cookie retention | 999 days | The stored consent document. |
| Approximate matching band | 4 characters | The maximum edit distance considered. |
| Teaser length | 200 characters plus an ellipsis | Blog post teaser. |
| Blog listing page size | 12 posts | A common multiple of 2, 3 and 4. |
| Blog feed size | 15 entries by default, 50 at most | The subscription feed. |
| Reference directory page size | 20 contacts | The public customer directory. |
| Map limit | 80 contacts by default | The public map frame. |
| Shop path | `/shop` | The root of every catalogue address. |
| Product feed soft limit | 5000 products | Validation limit on a feed's product count. |
| Product feed hard limit | 6000 products | Rendering limit; products beyond it are omitted. |
| Product feed warning threshold | 5500 products, the midpoint of the two limits | Above it the site salesperson is notified at most once a week. |
| Price list cache lifetime | 3600 seconds | After it, the cached price list is resolved again on the next shop listing request. |
| Anonymous wish list lifetime | 5 weeks | After it, rows without a contact are deleted. |
| Attribute value preview count | 20 values | The maximum previewed on a product card; the rest are reported as a hidden count. |
| Supported feed measurement units | ounce, pound, milligram, gram, kilogram, fluid ounce, pint, carat, quart, gallon, millilitre, centilitre, litre, cubic metre, inch, foot, yard, centimetre, metre, square foot, square metre | A reference unit outside this set suppresses the feed's reference-measure fields. |
| Variant reference separator | A rarely used control character | Joins the internal references of the variants of one template into one searchable text. |
| Tracked link code length | 3 characters, increased on collision | §24 of [calculations.md](calculations.md). |
| Font archive entry limit | 10485760 bytes | Above it an archive entry is refused. |
| Verification service timeout | 2 seconds | Both human verification services. |
| Industry picture download timeout | 3 seconds | The configurator. |

---

## 4. Access groups

| Group | Kind | Granted by | May do |
|---|---|---|---|
| Restricted Editor | role | configuration, or implied by the sales manager group | Read template definitions; edit the content of records they may write; use the content editor and the media dialogue; add, clear and reorder product media; declare storefront extra fields; reorder products on the grid; change an attribute's display type; write the page style settings. |
| Editor and Designer | role | configuration | Everything above, plus create, read, update and delete on Website, Website Menu, Website Page, Website Model Page, Website Rewrite, Website Route, page templates, assets, Blog, Blog Post, Blog Tag, Blog Tag Category, Website Checkout Step and Product Feed; read, update and delete on Website Visitor; full rights on Website Visit Track; create and write Website Product Category; the sanitisation override. |
| Multi-website | flag | automatically implied into the portal, internal user and public groups as soon as a second site exists | Sees the site selector and the site column on multi-site records. |
| Public access to arbitrary exposed model | flag | implied into the public and portal groups | Reads published Website Model Pages and the records they expose. |
| Unit of Measure Price Display | feature | the Base Unit Price setting | Sees the per-unit price on storefront pages. |
| Comparison Price | feature | the Comparison Price setting | Sees and sets the manually entered strikethrough price. |
| Product Feed | feature | the Product Feed setting | Sees the feed menu; the feed endpoint additionally requires the site flag. |
| Sales user | role | configuration | Reads online orders, unpaid orders and abandoned carts; sends recovery messages; confirms orders. Sees the site dashboard menu and the storefront configuration menu. |
| Sales manager | role | configuration | Everything above, plus create, write and delete on Website Product Category, Product Ribbon, Base Unit Display, Product Attribute Category and Product Image; reads every wish list row; sees the online sales report menu and the spreadsheet dashboard. Implies the Restricted Editor group. |
| Administrator | role | configuration | Everything above, plus theme records, global settings, verification service keys, audience measurement keys and every template. |
| Public | role | automatic | Reads published pages and published records; uses the cart, the wish list, the comparison and the checkout; submits public forms. No access at all to the wish list entity itself, anonymous rows being handled with elevated privileges. |
| Portal | role | automatic on sign-up | Everything the public role may do, plus pages whose visibility is signed-in, their own wish list rows, their own orders and invoices, and their own forum activity. |
| Internal user | role | automatic | Reads the catalogue including unpublished products. Implicitly receives the delivery and invoice address group. |

### 4.1 Model access

"full" means create, read, write and delete.

| Entity | Public | Portal | Internal | Restricted Editor | Editor and Designer | Sales manager | Administrator |
|---|---|---|---|---|---|---|---|
| Website | read | read | read | read | full | read | full |
| Website Page, Website Model Page | read | read | read | read | full | read | full |
| Website Menu | read | read | read | read | full | read | full |
| Website Rewrite, Website Route | none | none | read | read | full | read | full |
| Website Visitor | none | none | read | read | read, write, delete | read | read, write, delete |
| Website Visit Track | none | none | read | read | full | read | full |
| Website Content Block Filter | read | read | read | read | full | read | full |
| Website Configurator Feature | read | read | read | read | read | read | full |
| Website Technical Page | none | none | none | none | none | none | read |
| Theme Template, Theme Asset, Theme Attachment, Theme Menu, Theme Page | none | none | none | none | none | none | full |
| Blog, Blog Post, Blog Tag, Blog Tag Category | read | read | read | read | full | read | full |
| Forum, Forum Post, Forum Tag, Forum Post Vote | read | read, write, create | read, write, create | read | full | read | full |
| Forum Post Closing Reason | read | read | read | read | read | read | full |
| Partner Website Tag | read | read | read | read | full | read | full |
| Product Variant, Product Template, internal product category, Product Tag | read | read | read | read | read | full through the catalogue folder | full |
| Website Product Category | read | read | read | read | create, read, write | full | full |
| Price List, Price List Rule | read | read | read | read | read | full through the pricing folder | full |
| Product Ribbon | read | read | read | read | read | full | full |
| Product Attribute, Product Attribute Value, Product Template Attribute Value, attribute exclusion, Product Template Attribute Line, custom attribute value | read | read | read | read | read | full through the catalogue folder | full |
| Fiscal position, payment term | none | read | read | read | read | read | full |
| Tax | read | read | read | read | read | read | full |
| Product Image | read | read | read | full | full | full | full |
| Unit of measure | read | read | read | read | read | read | full |
| Storefront Extra Field | read | read | read | full | full | full | full |
| Base Unit Display | read | read | read | read | read | full | full |
| Website Checkout Step | none | none | none | none | full | none | full |
| Product Feed | none | none | none | none | full | none | full |
| Product Attribute Category | read | read | read | read | read | full | full |
| Product Wishlist | none | full, own rows | full, own rows | full, own rows | full, own rows | full, every row | full |

### 4.2 Record rules

| Rule | Entity | Groups | Condition | Permissions |
|---|---|---|---|---|
| Published pages | Website Page | public, portal | the per-site publication is true | read only |
| Published model pages | Website Model Page | public, portal, through the exposure group | the per-site publication is true | read only |
| Menu visibility | Website Menu | all | the authorised groups are empty, or the reader belongs to one of them | read only |
| Template visibility, public | template | public | the visibility is empty or public | read only |
| Template visibility, signed in | template | portal | the visibility is empty, public or signed-in | read only |
| Designer templates | template | Editor and Designer | page templates only | full |
| All templates | template | administrator | always true | full |
| Active blogs | Blog | public, portal | the blog is active | read only |
| Published blog posts | Blog Post | public, portal | the per-site publication is true | read only |
| Published site tags | Partner Website Tag | public, portal | the tag is published | read only |
| Visible forum posts | Forum Post | public, portal | the post is viewable under WS-605 | read only |
| Public product template | Product Template | public, portal | the product is published and may be sold | read only |
| Hide empty storefront categories | Website Product Category | public, portal | the published-products flag is true | read only |
| Own custom attribute values | custom attribute value | public, portal, internal | the creating user is the current user | full |
| Sales roles see all custom attribute values | custom attribute value | sales user, product manager | always true | full |
| Price list company rule | Price List | all | the company is empty, or the site's company, or one of the allowed companies | full |
| Price list rule company rule | Price List Rule | all | the same condition | full |
| Own wish list | Product Wishlist | portal, internal | the owner is the current user's contact | full |
| All wish lists | Product Wishlist | sales manager | always true | full |

The two generic price list company rules shipped by the pricing folder are deactivated and replaced by the
two site-aware rules above, so that a shopper may read the price lists of the site's company even when
that company is not among their own allowed companies. Deactivating rather than rewriting them is what
allows them to be restored if the storefront capability is removed.

Every computed record rule condition is cached with the site identifier in its key (WS-287).

---

## 5. Shipped records

### 5.1 Site structure

| Record | Values |
|---|---|
| Default site | One Website record named after the company, with the shipped logo, the shipped icon and no domain. |
| Default main menu | A root menu entry whose address is `/default-main-menu`; it is the template from which each new site's root menu is copied, and it may not be deleted (WS-093). |
| Home menu entry | Address `/`, ordering value 10, child of the root. |
| Contact us menu entry | Address `/contactus`, child of the root. |
| Home page | A page at `/`, published, pointing at the shipped home page template. |
| Contact page, thank-you page, privacy policy page, cookie policy page, page-not-found page, password prompt page | Shipped page templates, created per site on demand. |
| Site layout, header templates, footer templates | Shipped templates that a theme enables or disables; the default header and the default footer are the last entries of their lists. |
| Configurator features | One record per offered feature, each naming either a page template or a package, with its ordering value, its icon, its preselection list and its menu ordering value. |

### 5.2 Storefront content

| Record | Values |
|---|---|
| Shop menu | Name `Shop`, address `/shop`, child of the main menu, ordering value 20. |
| Forum menu | Name `Forum`, address `/forum`, child of the main menu. |
| Blog menu | Name `Blog`, created by the blog capability, child of the main menu. |
| Checkout steps | The four generic steps of [storefront-checkout.md](storefront-checkout.md) §5.1: the cart step at ordering value 0, the address step at 250, the extra-information step at 500 and the payment step at 999. |
| Product ribbons | `Sold out` at ordering value 1, white on red; `Out of stock` at 2, white on amber; `Sale` at 3, white on green; `New!` at 4, white on blue. All are positioned on the left and none carries an automatic assignment mode until an administrator sets one. |
| Site sales team | Activated, and used as the default sales team of a site. |
| Free delivery method | Published on the site. |
| Collect-in-store method | Name `Pick up in store`, the collect-in-store kind, the delivery product `Pick up in store`, attached to the default site, in production mode. Created by the collect-in-store capability. |
| Collect-in-store product | Name `Pick up in store`, a service, sales price 0, neither purchasable nor sellable. |
| Print-on-demand standard and express delivery methods | Published on the site. |
| Pay-on-site payment method | Name `Pay on site`, code `pay_on_site`, ordering value 1000, no tokenisation, no express checkout, no manual capture, no refund. |
| Pay-on-site payment provider | Name `Pay on Site`, a custom provider whose mode is the pay-on-site value, enabled, published, carrying the pay-on-site payment method and using the custom redirect form. |
| Default site assignment | The default site receives the site sales team and the administrator as salesperson. |
| Customer creation form | The Contact entity is declared as a target of public forms, with the form key `create_customer` and the label `Create a Customer`. |

### 5.3 Forum records

| Record | Values |
|---|---|
| Default forum | One Forum named `Help`, with the shipped guidelines template rendered into its guidelines field and the shipped welcome block in its welcome message. |
| Closing reasons, basic | `Duplicate post`, `Off-topic or not relevant`, `Too subjective and argumentative`, `Not a real post`, `Not relevant or out dated`, `Contains offensive or malicious remarks`, `Spam or advertising`, `Too localized`. |
| Closing reasons, offensive | `Insulting and offensive language`, `Violent language`, `Inappropriate and unacceptable statements`, `Threatening language`, `Racist and hate speech`. |
| Message subtypes | `New Question` and `New Answer` on the post, `Question Edited` and `Answer Edited` on the post, and the two forum-level subtypes `New Question` and `New Answer` that carry the notification to the forum's followers. |
| Sign-up policy | Installing the forum capability sets the platform sign-up policy to free sign-up, because a forum without self-registration cannot grow. |

The forum also ships the reputation badges below. The badge entity itself belongs to
[learning, surveys and gamification](../learning-surveys-and-gamification/README.md); this folder only
declares them and adds the publication flag that shows them on a public profile.

| Group | Badge and the achievement it rewards |
|---|---|
| Answers | `Teacher`, at least 3 up-votes on an answer for the first time; `Nice Answer`, an answer voted up 4 times; `Good Answer`, 6 times; `Great Answer`, 15 times; `Enlightened`, an answer accepted with 3 or more votes; `Guru`, an answer accepted with 15 or more votes; `Self-Learner`, answering one's own question with at least 4 up-votes. |
| Questions | `Popular Question`, a question with at least 150 views; `Notable Question`, 250 views; `Famous Question`, 500 views; `Credible Question`, set as a favourite by 1 participant; `Favorite Question`, by 5; `Stellar Question`, by 25; `Student`, a first question with at least one up-vote; `Nice Question`, a question voted up 4 times; `Good Question`, 6 times; `Great Question`, 15 times; `Scholar`, asking a question and accepting an answer. |
| Participation | `Autobiographer`, completing one's own biography; `Commentator`, 10 comments; `Chief Commentator`, 100 comments; `Pundit`, 10 answers with a score of 10 or more; `Taxonomist`, creating a tag used by 15 questions. |
| Moderation | `Cleanup`, a first rollback; `Critic`, a first down-vote; `Supporter`, a first up-vote; `Editor`, a first edit; `Disciplined`, deleting one's own post with 3 or more up-votes; `Peer Pressure`, deleting one's own post with 3 or more down-votes. |

### 5.4 Blog records

| Record | Values |
|---|---|
| Default blog | One Blog named `Travel`, with the shipped cover configuration. |
| Message subtype | `Published Post`, posted on the blog when one of its posts is published. |
| New-post template | The message template rendered when a post is published; its subject is the post title. |
| Blog content blocks | The shipped content blocks that list posts and tags. |

### 5.5 Content block filters

| Filter | Data provider | Fields requested | Limit | Needs a product |
|---|---|---|---|---|
| Newest Products | a stored condition on published products sorted by creation date descending | display name, sales description, 512-pixel picture | 16 | no |
| Recently Sold Products | the recently-sold provider | the same | 16 | no |
| Recently Viewed Products | the recently-viewed provider, per visitor | the same | 16 | no |
| Accessories for Product | the accessories provider | the same | 16 | yes |
| Products Recently Sold With Product | the sold-with provider | the same | 16 | yes |
| Alternative Products | the alternatives provider | the same | 16 | yes |
| Category List | the category provider | identifier, name, cover picture | 10 | no |

Two content block defaults are shipped: the product block starts with the Newest Products filter, as a
four-element carousel with a five-second interval, two elements on narrow screens, variants shown, the card
design and the comparison action; the category block starts with the Category List filter, showing the
parent, four columns, medium size, centred.

The recently-viewed filter carries the editor note
`The building block will remain empty until the user visits a product page.`

### 5.6 Reporting records

| Record | Values |
|---|---|
| Online Sales Analysis | A pivot and graph action over the sales analysis report restricted to rows with a site, with the confirmed filter preselected. |
| Sales, carts | The same report restricted to rows with a site, without the confirmed filter, used by the dashboard. |
| Storefront spreadsheet dashboard | Name `eCommerce`, main data entity Sales Order, visible to the sales manager group, ordering value 200, published, with a sample dashboard for empty databases. |
| Digest measure | The online sales measure is enabled on the shipped default periodic digest. |

### 5.7 Back-office navigation

| Menu | Parent | Action | Group |
|---|---|---|---|
| Website | root | The site dashboard | Restricted Editor |
| Site | Website | Pages, Model Pages, Menus, Redirects, Files | Editor and Designer |
| Content | Website | Pages, Blogs, Blog Posts, Forums, Products, Product Pages | Restricted Editor |
| Reporting | Website | Visitors, Online Sales, Page Views | Restricted Editor |
| Configuration | Website | Settings, Domains, Languages, Checkout Steps, Product Feeds | Editor and Designer |
| eCommerce | Website configuration | none | sales user |
| Orders | eCommerce | Confirmed orders of any site, with the from-site and confirmed filters preselected | sales user |
| Unpaid Orders | Orders | Orders in the sent state with a site, creation disabled | sales user |
| Abandoned Carts | Orders | Orders whose abandoned-cart flag is true, with the recovery filter preselected, creation disabled, the public contact supplied in the context | sales user |
| Customers | Orders | The customer list | sales user |
| Products | eCommerce | Product Templates with the published filter preselected, in card, list, form and activity views | sales user |
| Pricelists | Products | The price list list | price list feature group |
| eCommerce Categories | Products | Website Product Category | sales user |
| Attributes | Products | Product attributes | variant feature group |
| Combo Choices | Products | Product combos | sales user |
| Product Tags | Products | Product tags | sales user |
| Product Ribbons | Products | Product ribbons | sales user |
| Attribute Categories | Products | Product attribute categories | developer group |
| eCommerce settings | Website global configuration | none | sales user |
| Payment Providers, Payment Methods | eCommerce settings | The payment lists | sales user |
| Payment Tokens, Payment Transactions | eCommerce settings | The payment lists | developer group |
| Delivery Methods | eCommerce settings | Shipping methods | sales user |
| Postal Code Prefixes | eCommerce settings | Delivery postal-code prefixes | developer group |
| Product Feeds | eCommerce settings | Product feeds | product feed group |
| Online Sales | Website reporting | Online Sales Analysis | sales manager |
| Forum | Website content | Forums, posts and tags | Editor and Designer |
| Blogs | Website content | Blogs, posts, tags and tag categories | Editor and Designer |

Additional actions not attached to a menu: New Product, a small dialogue form for creating a product from
the storefront; Orders To Invoice, confirmed site orders with lines whose invoice status is "to invoice",
creation disabled; Base Units; Product Views History, the visit entries of one visitor that carry a
product, in list and graph views; and Reset Cache on a product feed, visible only to the developer group.

The site dashboard menu becomes visible to the sales user group. Opening the back-office dashboard as a
user holding that group redirects to the site dashboard. The suggested-page list offered by the editor
gains the entries `("eCommerce", "/shop")` and `("Forum", "/forum")`. On installation of the storefront
capability, the start menu action is set to open the shop.

---

## 6. Scheduled jobs

| Job | Interval | Effect | Idempotence |
|---|---|---|---|
| Website Visitor: clean inactive visitors | every 1 day | Deletes at most one batch of 1000 anonymous visitors older than the retention period, together with their tracks (§31 of [workflows.md](workflows.md)). | Deletion is idempotent; a full batch makes the job report the remaining count. |
| Disable unused snippets assets | every 1 week | Toggles the active flag of every content block asset according to its actual usage (§26 of [workflows.md](workflows.md)). | Recomputed from scratch each run. |
| Abandoned cart emails | every 1 hour | Runs the recovery job over every site (§57 of [workflows.md](workflows.md)). | Every examined cart is marked as mailed, so a second run in the same hour does nothing. |
| Product availability emails | every 1 hour | Runs the back-in-stock notification job (§54 of [workflows.md](workflows.md)). | Each subscriber is removed as soon as their message is created. |
| Anonymous wish list cleanup | the platform cleanup cycle | Deletes wish list rows with no contact older than five weeks. | Deletion is idempotent. |
| Abandoned coupon cleanup | the platform cleanup cycle | Releases the coupons of untouched storefront carts and re-evaluates them. | Re-running is harmless. |
| Deferred invoice sending | on demand, triggered by automatic invoicing when asynchronous messages are enabled | Sends the invoices produced by completed transactions. | Owned by the sales folder. |

---

## 7. Message templates, subtypes and generated documents

| Template | Entity | Subject | Recipients | Trigger |
|---|---|---|---|---|
| Cart recovery | Sales Order | `You left items in your cart!` | the record's default recipients | The hourly recovery job, or the manual action. The sender is the order salesperson, else the company, else the current user. The portal button is relabelled `Resume Order`. It is not deleted after sending, and its description is `If the setting is set, sent to authenticated visitors who abandoned their cart`. |
| Order confirmation | Sales Order | the platform confirmation subject | the customer | Order confirmation; the site's own template wins when one is set. |
| Back in stock | Product Variant | `The product '%(product_name)s' is now available` | every subscriber, in their own language | The hourly availability job. The sender is the site company's contact, else the site salesperson. |
| Feed product limit | Product Feed | `GMC: Product Limit Exceeded` | the site salesperson | A feed rendering above the warning threshold, at most once a week. The three capital letters abbreviate the name of the Google Merchant Center product listing service. |
| Donation confirmation | Payment Transaction | `Donation confirmation` | the donor | A donation transaction reaching the completed state. |
| Donation notice | Payment Transaction | `A donation has been made on your website` | the address supplied as the donation recipient | A donation transaction is created. |
| New blog post | Blog | the post title | the blog's followers | Publishing a post; the subtype is `Published Post`. |
| New forum question | Forum Post | the question title | the followers of the post and of its tags | A question becomes active. |
| New forum answer | Forum Post | `Re: ` followed by the question title | the followers of the question | An answer becomes active. |
| Forum validation request | Forum Post | the question title | every moderator and tag follower, as an internal note | A question is created in the pending state. |
| Profile address validation | User | the platform validation subject | the account | The account asks for its address to be validated. |

| Generated document | Media type | Produced by |
|---|---|---|
| Crawler exclusion response | plain text | §14 of [workflows.md](workflows.md) |
| Site index | the extensible markup language | §15 of [workflows.md](workflows.md) |
| Blog subscription feed | the syndication format | §38 of [workflows.md](workflows.md) |
| Product syndication feed | the extensible markup language, compressed | §62 of [workflows.md](workflows.md) |
| Order document | the printable document format | The order report of the [sales](../sales/README.md) folder |

No activity type is defined by this folder.

---

## 8. Site onboarding styles

The site configurator offers six shop page styles and six product page styles. Applying one writes a set of
site fields, enables and disables a set of page options, writes the three category page options on every
category, and may write presentation parameters of the theme.

### 8.1 Shop page styles

| Style | Site fields written | Page options enabled | Page options disabled | Category page options |
|---|---|---|---|---|
| Classic Grid | design tokens: catalogue layout, thumbnail design, regular name colour, cover thumbnails, secondary picture shown, light zoom-out on hover, call to action, actions on hover, wish list fixed, first colour combination, rounding level 2, comparison action, promoted actions | the storefront footer | none | title hidden, description shown, content not centred |
| Modern Grid | products per row 5, gap `0px`, full-width container, design tokens: cover thumbnails, light zoom-out, call to action, wish list, comparison, actions on hover, wish list fixed, catalogue layout, grid design, themed actions, secondary picture shown, four-by-five thumbnails, centred text | header with a search bar, full-width header, single column on narrow screens, filters on top, grid category strip, storefront footer, full-width footer | side filters | title shown, description shown, content not centred |
| Showcase | gap `0px`, full-width container, design tokens: regular name colour, cover thumbnails, call to action, wish list, description, inline actions, fifth colour combination, themed actions, four-by-three thumbnails, list layout, showcase design, no rounding | sales header, full-width header, centred shop title, pill category strip, filters on top, floating toolbar, storefront footer, full-width footer | side filters | title shown, description shown, content centred |
| Chips Contained | products per row 4, gap `16px`, design tokens: regular name colour, cover thumbnails, secondary picture shown, light zoom-out, call to action, wish list, comparison, inline actions, inline wish list, promoted actions, first colour combination, rounding level 4, catalogue layout, chip design | sales header, centred shop title, single column on narrow screens, bordered category strip, filters on top, storefront footer | side filters | title shown, description shown, content centred |
| Condensed List | gap `16px`, design tokens: regular name colour, cover thumbnails, call to action, wish list, inline actions, first colour combination, rounding level 2, secondary picture shown, promoted actions, list layout, thumbnail design | collapsed header, header without automatic hiding, picture category strip, storefront footer | none | title shown, description shown, content not centred |
| Cards | products per row 4, gap `8px`, design tokens: regular name colour, cover thumbnails, secondary picture shown, light zoom-out, call to action, wish list, actions on hover, wish list fixed, subtle actions, rounding level 2, catalogue layout, card design, four-by-five thumbnails, comparison | single column on narrow screens, large-picture category strip, filters on top, storefront footer | side filters | title hidden, description shown, content not centred |

### 8.2 Product page styles

| Style | Site fields written | Page options enabled | Page options disabled |
|---|---|---|---|
| Classic | media rounding medium | none | none |
| Image Grid | media width 66 percent, inverse column order, grid layout, medium spacing, medium rounding, two-by-three ratio | none | none |
| Focused | media width 66 percent, grid layout, 1 grid column, small spacing, small rounding | large call-to-action wrapper, large buy-now button, large quantity input | boxed call-to-action wrapper |
| Large Image | media width 100 percent, twenty-one-by-nine ratio | bottom carousel indicators, boxed call-to-action wrapper | left carousel indicators, call-to-action separator, large call-to-action wrapper, large buy-now button, large quantity input |
| Functional | media width 33 percent, small rounding | bottom carousel indicators, boxed call-to-action wrapper | left carousel indicators, large call-to-action wrapper, large buy-now button, large quantity input |
| Large Grid | media width 100 percent, grid layout, big spacing, big rounding, sixteen-by-nine ratio | large call-to-action wrapper, large buy-now button, large quantity input | call-to-action separator, boxed call-to-action wrapper |

The six category strip options are mutually exclusive: enabling one disables the other five (WS-466). The
configurator may additionally generate a set of storefront categories appropriate to the industry chosen
during onboarding.

---

## 9. Master data prerequisites

Before a site can operate:

1. One Website with a company, a public user, at least one installed language and a root menu.
2. A home page, either the bootstrapped one or one created by the configurator.
3. For a multi-language site, every offered language installed and attached to the site.

Before a storefront can operate, additionally:

4. At least one Price List usable on that site: attached to the site, or generic and selectable, or
   generic and carrying a promotional code.
5. Sales taxes on the products, and the fiscal positions that map them for the countries served.
6. At least one published Shipping Method when physical goods are sold, and its delivery product.
7. At least one enabled and published Payment Provider, unless every order can be free.
8. A payment term, so that a storefront order always has one; the immediate-payment term is used when it
   exists.
9. For availability: a Warehouse, and the products marked as tracked.
10. For collection in store: warehouses with complete addresses and, optionally, working schedules for the
    opening hours.
11. Message templates for the order confirmation and the cart recovery.
12. A public contact on the site, used as the customer of anonymous carts and excluded from abandoned-cart
    detection.

---

## Reconciliation notes

1. **Two configuration documents.** Only the storefront half had one. The site half's settings, parameters,
   groups, model access, record rules, scheduled jobs and shipped records are written here for the first
   time, from the entity behaviour of [entities.md](entities.md) and the rules of
   [business-rules.md](business-rules.md).
2. **Group names.** One version called the two editing groups "website designer" and "restricted website
   editor". The labels shown to a user are `Editor and Designer` and `Restricted Editor`; those are the
   names used throughout this folder, and the model access table above states what each may do.
3. **The sign-up policy.** The site-level policy and the checkout account policy are two views of the same
   decision; §1.1 and §1.3 state the writing rule once, in §1.3, and cross-refer.
