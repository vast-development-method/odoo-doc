# Entities

Complete field-by-field specification of every entity this folder owns, and of every field this
folder adds to an entity owned by another folder. Each entity names its generated reference page,
which carries the machine-readable field list.

## Conventions

Data types follow the catalogue defined by the platform: `boolean`, `integer`, `decimal`,
`monetary`, `text`, `long_text`, `rich_text`, `date`, `datetime`, `binary`, `image`, `selection`,
`reference`, `many_to_one`, `one_to_many`, `many_to_many`, `structured_data`, `properties`.

Every persistent entity carries the common shared fields of the platform: `identifier` (surrogate
integer primary key), `created_on` (datetime), `created_by_user` (many_to_one to User),
`last_updated_on` (datetime) and `last_updated_by_user` (many_to_one to User). Entities that support
archiving carry `active` (boolean, default true; archived records are excluded from default
queries). These fields are not repeated in the tables below unless the entity gives them a special
label, a special access restriction or a special meaning.

Three storage notions recur and are defined once here.

* **Stored** means the value is written in the entity's own table. **Derived** means the value is
  computed on read from the stated inputs; a derived field that is also stored is recomputed and
  written whenever one of its inputs changes.
* **Translated** means the field keeps one value per active language. Two translation modes exist:
  *term translation* (each sentence-level term inside the markup is translated separately, used for
  rich text content and for template architecture) and *value translation* (the whole value is
  translated as one unit, used for single-line and plain text fields).
* **Architecture** is the stored markup tree of a View record. Pages do not store their content
  directly; they delegate to a View through a delegation link, which means every field of View is
  readable and writable directly on the page record.

Currency amounts are stored with the precision of the currency named by the record's currency field
("currency precision" below, two decimal places for most currencies). Quantities use the product unit
precision, a configurable decimal precision whose default is two decimal places; the storefront
rounds shopper-entered quantities to whole numbers.

Names chosen for this specification that differ from a mechanical expansion of the stored field name
are listed once, in §6, so that a reader comparing a field table with a reference page can match them.

---

# Part 1: Site, structure and content

## 1.1 Website

Reference page: [`website`](../../references/entities/website.md).

The Website entity is the root configuration record of one public site.

* Default ordering: `sequence` ascending, then `identifier` ascending.
* Display name rule: `name`.
* Company scoping: every site belongs to exactly one company through `company_id`; the reverse link
  on Company is derived as the first site of that company in ordering order.
* Archiving: not supported. A site is deleted, not archived.
* Database constraint: `unique(domain)`, message `Website Domain should be unique.`

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Access restriction | Meaning |
|---|---|---|---|---|---|---|---|---|
| `name` | Site name | text | yes | none | stored | yes | all | The site name, shown in the browser title suffix and in social sharing metadata. |
| `sequence` | Ordering | integer | no | 10 | stored | yes | all | Ordering among sites; the first site in this order is the fallback site and the company's site. |
| `domain` | Site domain name | text | no | empty | stored | yes | all | The public domain of the site, for example `https://www.example.com`. Used for site resolution, absolute addresses, indexing decisions and cross-site navigation. |
| `domain_punycode` | Domain in ascii-compatible encoding | text | no | none | derived, not stored | no | all | The domain with its host part encoded in the ascii-compatible internationalized domain name encoding. |
| `company_id` | Owning company | many_to_one to Company | yes | the current company | stored | yes | all | Drives the currency, the public user and the allowed company list during frontend requests. Deletion behaviour: restrict. |
| `language_ids` | Available languages | many_to_many to Language | yes | all active languages | stored | yes | all | The languages the site is published in. Drives the language selector, the alternate-language links and the accepted language prefixes. |
| `language_count` | Number of languages | integer | no | none | derived, not stored | no | all | Number of entries in `language_ids`. |
| `default_lang_id` | Default language | many_to_one to Language | yes | the default contact language if set, otherwise the first active language | stored | yes | all | The language served when no language prefix is present in the address. |
| `auto_redirect_lang` | Redirect to the browser language | boolean | no | true | stored | yes | all | When true, a visitor whose browser prefers another available language is redirected to that language's prefix. |
| `cookies_bar` | Cookie consent bar | boolean | no | false | stored | yes | all | When true, the consent bar is displayed and optional cookies are refused until consent is given. |
| `configurator_done` | Configurator completed | boolean | no | false | stored | yes | all | True once the first-run configurator has been completed or explicitly skipped. |
| `block_third_party_domains` | Block third-party content | boolean | no | true | stored | yes | all | When true, and the consent bar is enabled and optional consent has not been given, embedded frames and scripts pointing at blocked domains are neutralised. |
| `custom_blocked_third_party_domains` | Custom blocked domains | long_text | no | empty | stored | yes | Editor and Designer | One host per line, added to the built-in block list. A line starting with `#` is a comment. When the first line starts with `#ignore_default`, the built-in list is replaced instead of extended. |
| `blocked_third_party_domains` | Effective blocked domains | long_text | no | none | derived, not stored | no | all | The effective block list, computed as described in [business-rules.md](business-rules.md) WS-020. |
| `logo` | Site logo | binary | no | the shipped default logo | stored | yes | all | Also used as the default social sharing image when no dedicated sharing image is set. |
| `social_twitter` | Social link, short-message network | text | no | the company's value | stored | yes | all | Link to the site's account on that network. |
| `social_facebook` | Social link, social network page | text | no | the company's value | stored | yes | all | Link to the site's page on that network. |
| `social_github` | Social link, code-hosting service | text | no | the company's value | stored | yes | all | Link to the site's organisation on that service. |
| `social_linkedin` | Social link, professional network | text | no | the company's value | stored | yes | all | Link to the site's page on that network. |
| `social_youtube` | Social link, video service | text | no | the company's value | stored | yes | all | Link to the site's channel on that service. |
| `social_instagram` | Social link, picture network | text | no | the company's value | stored | yes | all | Link to the site's account on that network. |
| `social_tiktok` | Social link, short-video network | text | no | the company's value | stored | yes | all | Link to the site's account on that network. |
| `social_discord` | Social link, chat service | text | no | the company's value | stored | yes | all | Link to the site's server on that service. |
| `social_default_image` | Default sharing image | binary | no | empty | stored | yes | all | When set, replaces the logo as the default social sharing image. |
| `has_social_default_image` | Has a sharing image | boolean | no | false | derived and stored | yes | all | True when a dedicated social sharing image is set. |
| `google_analytics_key` | Audience measurement identifier | text | no | empty | stored | yes | all | Measurement identifier of the Google Analytics audience measurement service. When set, the measurement script is injected in the document head of every non-editing page. |
| `google_search_console` | Search console verification token | text | no | empty | stored | yes | all | The verification token of the Google Search Console service, stored in the form `google<token>.html`. |
| `google_maps_api_key` | Mapping service access key | text | no | empty | stored | yes | all | Key used to render static and interactive maps from the Google Maps mapping service. |
| `plausible_shared_key` | Second measurement service key | text | no | empty | stored | yes | all | Shared authentication key of the Plausible audience measurement service, used to embed its dashboard. |
| `plausible_site` | Second measurement service site | text | no | empty | stored | yes | all | Site name registered with that service. |
| `user_id` | Public user | many_to_one to User | yes | the public user of the company, otherwise the global public user | stored | yes | all | The account under which unauthenticated requests execute on this site. |
| `partner_id` | Public contact | many_to_one to Contact | no | none | derived from the public user's contact, writable | yes | all | The contact of the public user. It owns anonymous carts and is excluded from abandoned-cart detection. |
| `cdn_activated` | Content delivery network enabled | boolean | no | false | stored | yes | all | When true, static asset addresses matching the filters are rewritten to the delivery network base address. |
| `cdn_url` | Content delivery network base address | text | no | empty | stored | yes | all | Base address of the content delivery network. |
| `cdn_filters` | Content delivery network filters | long_text | no | the six shipped patterns listed in [configuration.md](configuration.md) §3 | stored | yes | all | One matching pattern per line; an address matching any of them is rewritten. |
| `menu_id` | Root menu | many_to_one to Website Menu | no | none | derived, not stored | no | all | The first menu of this site that has no parent. |
| `homepage_url` | Home page address | text | no | empty | stored | yes | all | When set, requests to `/` are internally rerouted to this path. Must start with `/`; trailing slashes are stripped on write. |
| `custom_code_head` | Custom head markup | rich_text | no | empty | stored, never sanitised | yes | all | Markup injected at the end of the document head of every page. |
| `custom_code_footer` | Custom footer markup | rich_text | no | empty | stored, never sanitised | yes | all | Markup injected at the end of the document body of every page. |
| `robots_txt` | Crawler exclusion text | rich_text | no | empty | stored, never sanitised, not translated | yes | Editor and Designer | Custom text appended to the generated crawler exclusion response. |
| `favicon` | Site icon | binary | no | the shipped default icon | stored, transformed on write | yes | all | On write the supplied picture is centre-cropped, resized to 256 by 256 and converted to icon format. |
| `theme_id` | Applied theme | many_to_one to capability package | no | none | stored | yes | all | The installed theme package currently applied to this site. |
| `specific_user_account` | Site-specific accounts | boolean | no | false | stored | yes | all | When true, accounts created through sign-up on this site are bound to this site and cannot sign in on another. |
| `auth_signup_uninvited` | Sign-up policy | selection: `b2b` = On invitation, `b2c` = Free sign up | no | `b2b` | stored | yes | all | Whether visitors may create an account themselves. The two stored values are reproduced; they mean invitation-only and free sign-up. |
| `app_icon` | Installable application icon | image | no | none | derived and stored from `favicon` | yes | all | A square icon of at least 512 by 512 in portable network graphics format; skipped when the icon is a scalable vector image. Added by the events capability. |
| `events_app_name` | Installable application name | text | no | derived from `name` | derived and stored, writable | yes | all | The name used by the events progressive site. Added by the events capability. |
| `karma_profile_min` | Reputation to view a profile | integer | no | 150 | stored | yes | all | Minimum reputation score required to view another user's public profile. Added by the public profile capability. |
| `forum_count` | Number of forums | integer | no | 0 | stored, read only | no | all | Number of forums reachable from this site; recomputed whenever a forum is created, deleted, archived or re-scoped. |

Additional fields are contributed to Website by other capabilities and are specified in the sections
of this file that own them: the storefront settings (§5.1), the live chat channel, the sales team of
the contact form, the newsletter list and the warehouse. The complete settings screen is listed in
[configuration.md](configuration.md).

### Validation rules

| Rule | Condition | Message |
|---|---|---|
| Domain must parse | `domain` is set and cannot be parsed as a web address | `The provided website domain is not a valid URL.` |
| Domain path must not be relative | the path part of `domain` contains `/./` or `/../` | `The domain path cannot contain relative path segments like '/./' or '/../'.` |
| Home page address must be relative | `homepage_url` is set and does not start with `/` | `The homepage URL should be relative and start with '/'.` |
| Events application name required | the events capability is installed and `events_app_name` is empty | `"Events App Name" field is required.` |
| Default site cannot be deleted | the site referenced by the shipped external identifier of the default site is part of the deletion set | `You cannot delete default website %s. Try to change its settings instead` where the placeholder is the site name. |

### On-change behaviour

When the user edits `language_ids` in the form and the current `default_lang_id` is no longer among
them, `default_lang_id` is set to the first language of the new list.

### Normalisation applied on create and on update

1. A supplied `favicon` is replaced by its centre-cropped 256 by 256 icon-format rendering.
2. A non-empty `domain` that does not start with `http` receives the prefix `https://`; trailing
   slash characters are removed.
3. Trailing slash characters are removed from a non-empty `homepage_url`.

### Record lifecycle

1. **Create.** The normalisation above runs. When `user_id` is not supplied, the public user of the
   supplied company is used, or the global public user when no company is supplied. After insertion
   the derived site link of every affected company is recomputed, the home page bootstrap runs
   ([workflows.md](workflows.md) §22), the per-site checkout steps are created (§5.7), the forum
   count is refreshed, and, when the creating user is not in the multi-site group and more than one
   site now exists, that group is implied into the portal, internal user and public groups so that
   everyone sees the site dimension.
2. **Update.** The normalisation runs, then the whole registry cache is cleared. When `company_id`
   changes and `user_id` is not written at the same time, sites whose public user belongs to another
   company receive the new company's public user. When any content delivery network field changes,
   the registry cache is cleared again so compiled static nodes are rebuilt. When `sequence` or
   `company_id` changes, the derived company link is recomputed for the previous and the new company.
   Turning `cookies_bar` off deletes the page at `/cookie-policy` of this site; turning it on creates
   that page from the shipped cookie policy template, as a site-specific copy, published and not
   indexed.
3. **Delete.** Deletion of the default site is refused. Otherwise attachments belonging to this site
   that are theme attachments (they carry a `key`), customised asset attachments (their address
   starts with `/_custom/`) or compiled bundle attachments (their address contains `.assets_`) are
   deleted first; then the site row is deleted; then the derived company link and the forum count are
   recomputed.

## 1.2 Website Page

Reference page: [`website.page`](../../references/entities/website.page.md).

A page is a static publishable address whose content is a template architecture. It delegates to View
through `view_id`, so every field of View — notably `name`, `key`, `arch`, `active`, `priority`,
`visibility`, `group_ids` and `track` — is readable and writable on the page.

* Default ordering: by site.
* Display name rule: the delegated template name.
* Company scoping: indirect, through the site's company.
* Archiving: through the delegated `active` flag of the template.
* Inherited mixins: Multi-site Publication Flag, Searchable, Page Options.

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `url` | Page address | text | yes | none | stored | copied with a uniqueness suffix | The path the page answers on, always starting with `/`. Slugified on every write. |
| `view_id` | Content template | many_to_one to View | yes | none | stored | a fresh copy of the template is made | The template holding the page content. Deletion behaviour: cascade. |
| `write_uid` (of the template) | Last content editor | many_to_one to User | no | none | derived from the template | no | Who last changed the page content. |
| `write_date` (of the template) | Last content change | datetime | no | none | derived from the template | no | When the page content last changed. |
| `website_indexed` | Indexed | boolean | no | true | stored | yes | When false, the page is excluded from the site index and a no-index instruction is emitted in the document head. |
| `date_publish` | Publishing date | datetime | no | empty | stored | yes | Embargo timestamp. The page is visible only once this moment has passed. |
| `menu_ids` | Menu entries | one_to_many to Website Menu | no | none | inverse of the menu's page link | no | Navigation entries that target this page. |
| `is_in_menu` | Is in the menu | boolean | no | none | derived from `menu_ids` | no | True when at least one menu entry targets the page. |
| `is_homepage` | Is the home page | boolean | no | none | derived, not stored | no | True when `url` equals the current site's `homepage_url`, or, when that setting is empty and the page belongs to the current site, when `url` is `/`. |
| `is_visible` | Is visible | boolean | no | none | derived, not stored | no | True when the page is published on the current site and either `date_publish` is empty or already past. |
| `is_new_page_template` | Offered as a template | boolean | no | false | stored | yes | When true, the page appears in the custom group of the new-page template picker. |
| `website_id` | Site | many_to_one to Website | no | none | derived and stored from the template's site, writable | no | The site the page belongs to; empty means the page is shared by every site. Deletion behaviour: cascade. |
| `arch` | Content markup | long_text | no | none | derived from the template, writable, depends on the site in context | no | The page content markup. |
| `theme_template_id` | Theme template | many_to_one to Theme Page | no | none | stored, not copied | no | The theme template this page was generated from, used to delete it again when the theme is removed. |
| `header_overlay` | Header overlays the content | boolean | no | false | stored | yes | From the page options mixin. |
| `header_color` | Header colour | text | no | empty | stored | yes | A colour class name or a colour value. |
| `header_text_color` | Header text colour | text | no | empty | stored | yes | A colour class name or a colour value. |
| `header_visible` | Show the header | boolean | no | true | stored | yes | From the page visibility options mixin. |
| `footer_visible` | Show the footer | boolean | no | true | stored | yes | From the page visibility options mixin. |

Fields reachable through the delegation to View and relevant to pages:

| Delegated identifier | Full name | Type | Meaning on a page |
|---|---|---|---|
| `name` | Page title | text | Used in the browser title and as the menu entry name when the page is added to the menu. |
| `key` | Template key | text | The stable key, of the form `<package>.<slug>`, unique per site. Regenerated when `name` changes. |
| `active` | Active | boolean | Archiving. |
| `priority` | Rendering priority | integer | Also used to derive the site index priority. |
| `track` | Track visits | boolean | When true, serving this page creates or updates a Website Visitor and a Website Visit Track. |
| `visibility` | Page visibility | selection: empty = Public, `connected` = Signed In, `restricted_group` = Restricted Group, `password` = With Password | Who may open the page. |
| `visibility_password` | Visibility password | text | The hashed password, readable only by the administrator group. |
| `visibility_password_display` | Visibility password shown | text | The masked password shown in the dialogue (`********` when set, empty otherwise); writing it hashes the value. Readable by the Editor and Designer group. |
| `group_ids` | Authorised groups | many_to_many to Access Group | The groups allowed when `visibility` is `restricted_group`. |
| `website_meta_title`, `website_meta_description`, `website_meta_keywords`, `website_meta_og_image`, `seo_name`, `is_seo_optimized` | Search engine metadata | see §3.1 | Search engine and social sharing metadata of the page. |

### Validation, normalisation and on-change behaviour

On every write:

1. When `url` is written, the value is slugified as a path and prefixed with `/`. When the resulting
   address differs from the current one, a uniqueness suffix is applied against the pages of the same
   site, every menu entry targeting the page is repointed to the new address, and, when the site's
   `homepage_url` equalled the old address, it is updated to the new one.
2. When `name` is written and differs, `key` is regenerated from the slug of the new name with a
   uniqueness suffix.
3. When `visibility` is written with a value other than `restricted_group`, the authorised group list
   is emptied.
4. When `url`, `visibility` or the authorised groups were written, the template cache is cleared,
   because both routing and rendering depend on them.

### Record lifecycle

1. **Create.** Normally created through the New Page operation, which first copies a template, then
   creates the page pointing at that copy with tracking enabled.
2. **Duplicate.** Duplicating a page copies its template into a new template (inheriting the target
   site), sets the page key to the new template key, and gives the copy a unique address derived from
   the original one.
3. **Clone.** The Clone Page operation duplicates the page under the current site, optionally renames
   it (which also re-slugifies the address), and, when the clone stays on the same site, duplicates
   the menu entry that targeted the original.
4. **Delete.** Templates used only by the deleted pages and without inheriting children are deleted
   together with the pages; the pages already cascaded from those templates are removed from the
   deletion set to avoid deleting them twice. The template cache is cleared so that the menu cache
   flag is recomputed.

## 1.3 Website Model Page

Reference page: [`website.controller.page`](../../references/entities/website.controller.page.md).

A model page publishes a whole business entity under `/model/<slug of the name>`. Like a page it
delegates to View.

* Default ordering: site ascending, then `identifier` descending.
* Display name rule: `name`.
* Database constraint: `UNIQUE(name_slugified)`, message `url should be unique`.
* Inherited mixins: Multi-site Publication Flag, Searchable.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `view_id` | Listing template | many_to_one to View | yes | none | stored | The listing template. Deletion behaviour: cascade. |
| `record_view_id` | Record template | many_to_one to View | no | none | stored | The single-record template. Deletion behaviour: cascade. |
| `menu_ids` | Menu entries | one_to_many to Website Menu | no | none | inverse of the menu's model-page link | Navigation entries targeting this model page. |
| `website_id` | Site | many_to_one to Website | no | none | derived and stored from the template's site, writable | The site the model page belongs to. Deletion behaviour: cascade. |
| `name` | Page name | text | yes | the name of the listing template | derived and stored, precomputed, writable through an inverse that writes the template name | Used to build the address and shown in the browser title. |
| `name_slugified` | Address segment | text | no | the slug of `name` | derived and stored, precomputed, writable through an inverse that re-slugifies the written value | The path segment used in `/model/<segment>`. Recomputed when the exposed entity or `name` changes; empty when no entity is set. |
| `page_url` | Example address | text | no | none | derived, not stored | `/model/<segment>`, empty when no segment exists. |
| `record_domain` | Record restriction | text | no | empty | stored | A condition restricting which records of the exposed entity may be viewed publicly. |
| `default_layout` | Default layout | selection: `grid` = Grid, `list` = List | no | `grid` | stored | The layout used when the visitor has not chosen one in the session. |
| `is_published` | Published | boolean | no | false | stored | Publication flag; the default publication state of a model page is unpublished. |

Delegated fields of interest: the exposed entity, `name`, `key`, `arch` and `active`.

### Validation rules

* The exposed entity must be a concrete stored entity: a transient, abstract or table-less entity is
  refused with `A page must be set to display a concrete model.`
* The user creating or changing the exposed entity must have read access to that entity; the platform
  access check runs and its message is raised when it fails.

### Record lifecycle

1. **Create.** After insertion the exposed-entity access check runs.
2. **Update.** After the write, every menu entry targeting the model page is rewritten with address
   `/model/<segment>` and name `name`. When the exposed entity changed, the access check runs again.
3. **Delete.** Templates used only by the deleted model pages and without inheriting children are
   deleted together; the template cache is cleared.

## 1.4 Website Menu

Reference page: [`website.menu`](../../references/entities/website.menu.md).

* Default ordering: `sequence` ascending, then `identifier` ascending.
* Hierarchy: a materialised path is maintained in `parent_path`; the tree is limited to two levels.
* Display name rule: `name`, suffixed with ` [<site name>]` when the caller asked for the site
  dimension or when the user is in the multi-site group.

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Access restriction | Meaning |
|---|---|---|---|---|---|---|---|---|
| `name` | Entry label | text | yes | none | stored, translated (value translation) | yes | all | The label of the entry. |
| `url` | Target address | text | yes | `#` | derived and stored from the page link, the mega-menu flag and the children, writable | yes | all | Derived as `#` when the entry is a mega menu or has children, otherwise the address of the linked page when there is one, otherwise the stored value, otherwise `#`. |
| `page_id` | Target page | many_to_one to Website Page | no | none | stored | yes | all | The page this entry targets. Deletion behaviour: cascade. |
| `controller_page_id` | Target model page | many_to_one to Website Model Page | no | none | stored | yes | all | The model page this entry targets. Deletion behaviour: cascade. |
| `new_window` | Open in a new tab | boolean | no | false | stored | yes | all | Open the target in a new browser tab. |
| `sequence` | Ordering | integer | no | the highest existing sequence, or 0 | stored | yes | all | Ordering among siblings. |
| `website_id` | Site | many_to_one to Website | no | none | stored | yes | all | The site this entry belongs to. Deletion behaviour: cascade. |
| `parent_id` | Parent entry | many_to_one to Website Menu | no | none | stored | yes | all | Parent entry. Deletion behaviour: cascade. |
| `child_id` | Child entries | one_to_many to Website Menu | no | none | inverse of the parent link | no | all | Child entries. |
| `parent_path` | Materialised path | text | no | none | maintained by the hierarchy machinery | no | all | Used for subtree queries. |
| `is_visible` | Visible | boolean | no | none | derived, not stored | no | all | False when the entry targets a page or model page that the current non-internal user cannot see. |
| `group_ids` | Authorised groups | many_to_many to Access Group | no | empty | stored | yes | visible only to internal users | The visitor must belong to at least one of these groups to see the entry. Empty means everyone. |
| `is_mega_menu` | Is a mega menu | boolean | no | none | derived from the mega-menu content, writable through an inverse | no | all | True when mega-menu content exists. Setting it to true fills the content with the shipped default mega-menu template; setting it to false clears the content and the classes. |
| `mega_menu_content` | Mega-menu markup | rich_text | no | empty | stored, never sanitised, translated (term translation) | yes | all | The markup of the mega-menu panel. |
| `mega_menu_classes` | Mega-menu layout classes | text | no | empty | stored | yes | all | Layout classes applied to the mega-menu panel. |
| `theme_template_id` | Theme template | many_to_one to Theme Menu | no | none | stored, not copied | no | all | The theme template this entry was generated from. |

### Validation rules

All three checks run whenever the parent, the children, the mega-menu flag or the mega-menu content
changes.

| Rule | Condition | Message |
|---|---|---|
| Two levels maximum | the chain of ancestors of the entry is longer than two | `Menus cannot have more than two levels of hierarchy.` |
| Mega menus are flat | the parent is a mega menu, or the entry is a mega menu and has a grandparent or children | `A mega menu cannot have a parent or child menu.` |
| Containers stay at the top | the entry has children and either its parent has a parent or its children have children | `Menus with child menus cannot be added as a submenu.` |
| Root menu protected | the shipped default main menu is part of the deletion set | `You cannot delete this website menu as this serves as the default parent menu for new websites (e.g., /shop, /event, ...).` |

### Record lifecycle

1. **Create.** The template cache is cleared. Then, per value set: an entry whose address is exactly
   `/default-main-menu` is created as is (this is the shipped template root); otherwise an entry that
   names a site is created as is; otherwise, when a site is present in the execution context, that
   site is written into the values and the entry is created; otherwise the entry is duplicated once
   per existing site, each copy taking the supplied parent when it is a site-specific parent and
   otherwise the root menu of that site, and, when the supplied parent was the shipped template root,
   one entry is additionally created under that template root. The operation returns the last created
   record so that external identifiers bind to a real record.
2. **Update.** The template cache is cleared. When the authorised groups were written, the Editor and
   Designer group is added to the groups of every entry that now has groups, so that designers never
   lose sight of a restricted entry. The recursion is stopped by a context flag.
3. **Delete.** The template cache is cleared. Deleting a direct child of the shipped template root
   also deletes every site-specific entry with the same address, so that removing a shared entry
   removes its per-site copies.

### Derived visibility

An entry is not visible when the current user is not internal and one of the following holds.

* The entry targets a page that is unpublished or embargoed, or whose template visibility check fails
  while the visibility mode is not `password`.
* The entry targets a model page that is unpublished, or whose template visibility check fails while
  the visibility mode is not `password`.

A mode of `password` keeps the entry visible on purpose, so that a visitor can reach the page and be
asked for the password.

### Active entry detection

An entry is considered active for the current request when all of the following hold. Mega-menu
entries are never active, and an entry is never active outside a request.

* With no children: the request path and the entry path are equal after replacing the last path
  segment of each by the numeric identifier it contains, if any; and, when the entry targets a page,
  the two raw paths must also be equal; and every query parameter of the entry address must be
  present with the same value in the request address; and, when the entry address carries a host,
  that host must equal the request host.
* With children: at least one child is active.

## 1.5 Website Rewrite

Reference page: [`website.rewrite`](../../references/entities/website.rewrite.md).

* Default ordering: by `identifier`.
* Display name rule: the action type, a space, a hyphen, a space, then `name`.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `name` | Rule label | text | yes | none | stored | Human label of the rule. |
| `website_id` | Site | many_to_one to Website | no | none | stored | Restricts the rule to one site; empty means every site. Deletion behaviour: cascade. |
| `active` | Active | boolean | no | true | stored | Archiving. |
| `url_from` | Source address | text | no | empty | stored, indexed | The source path. |
| `route_id` | Endpoint helper | many_to_one to Website Route | no | none | stored | Selecting a registered endpoint path fills both addresses. |
| `url_to` | Target address | text | no | empty | stored | The target path. |
| `redirect_type` | Action type | selection: `404` = 404 Not Found, `301` = 301 Moved permanently, `302` = 302 Moved temporarily, `308` = 308 Redirect / Rewrite | no | `302` | stored | The action to take. |
| `sequence` | Ordering | integer | no | 0 | stored | Ordering. |

### Meaning of each action type

| Stored value | Effect on the routing table | Effect at request time |
|---|---|---|
| `301` | none | The path is not in the routing table; the fallback handler answers with a permanent redirect to the target. Browsers cache the new address. |
| `302` | none | The same with a temporary redirect; browsers do not cache the new address. |
| `308` | The endpoint is registered at the target path, and the source path is registered as a duplicate endpoint that redirects permanently to the target while preserving the slug values and the query string. | Both addresses work; the source answers with a permanent redirect. |
| `404` | The endpoint is removed from the routing table for this site. | The address answers not found. |

### Validation rules

For `301`, `302` and `308`:

| Condition | Message |
|---|---|
| target empty | `"URL to" can not be empty.` |
| source empty | `"URL from" can not be empty.` |
| either address starts with `#` | `URL must not start with '#'.` |
| the two addresses are equal once their fragment is removed | `base URL of 'URL to' should not be same as 'URL from'.` |

Additionally for `308`:

| Condition | Message |
|---|---|
| target does not start with `/` | `"URL to" must start with a leading slash.` |
| a parameter placeholder of the form `/<...>` present in the source is absent from the target | `"URL to" must contain parameter %s used in "URL from".` |
| a parameter placeholder present in the target is absent from the source | `"URL to" cannot contain parameter %s which is not used in "URL from".` |
| the target is exactly `/` | `"URL to" cannot be set to "/". To change the homepage content, use the "Homepage URL" field in the website settings or the page properties on any custom page.` |
| a registered endpoint already answers on the target path, ignoring a trailing slash | `"URL to" cannot be set to an existing page.` |
| the target cannot be compiled as a routing pattern | `"URL to" is invalid: %s` where the placeholder is the compilation error. |

### Record lifecycle

Creating, updating or deleting a rule whose type is or was `308` or `404` clears the routing cache on
every worker, because those two types change the routing table. Types `301` and `302` do not change
the routing table and are applied by the fallback handler.

## 1.6 Website Route

Reference page: [`website.route`](../../references/entities/website.route.md).

* Default ordering: `path`.
* Display name rule: `path`.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `path` | Endpoint path | text | no | none | stored | A registered endpoint path pattern. |

Refresh operation: the catalogue is rebuilt from the live routing table. Every generated path whose
endpoint answers the retrieval verb is kept; paths present in the catalogue but no longer generated
are deleted; paths generated but missing are created. The catalogue is refreshed automatically when a
name search returns nothing, and the search is then retried once.

## 1.7 Website Visitor

Reference page: [`website.visitor`](../../references/entities/website.visitor.md).

* Default ordering: `identifier` descending.
* Display name rule: the contact name when a contact is linked, otherwise `Website Visitor #` followed
  by the identifier.
* Database constraint: `unique(access_token)`, message `Access token should be unique.`

| Identifier | Full name | Type | Required | Default | Stored or derived | Access restriction | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Display label | text | no | none | derived from the contact name | all | Display label. |
| `access_token` | Browsing token | text | yes | computed at creation | stored, not copied | all | Either the decimal contact identifier of the signed-in user, or a 32-character hexadecimal digest of the remote network address, the browser user agent and the session identifier. |
| `website_id` | Site | many_to_one to Website | no | none | stored, read only | all | The site on which the visitor was first seen. |
| `partner_id` | Contact | many_to_one to Contact | no | none | derived and stored from the token | all | The contact of the last signed-in user. Empty while the token is a 32-character digest. |
| `partner_image` | Contact picture | binary | no | none | derived from the contact picture | all | Avatar. |
| `country_id` | Country | many_to_one to Country | no | none | stored, read only | all | Resolved at creation from the geolocation of the network address; empty when the resolved code is unknown. |
| `country_flag` | Country flag address | text | no | none | derived from the country | all | Flag picture address. |
| `lang_id` | Language | many_to_one to Language | no | none | stored | all | The frontend language in effect when the visitor was created. |
| `timezone` | Time zone | selection of the recognised time zone names | no | empty | stored | all | Resolved from the browser time zone cookie when it is a recognised name, otherwise from the signed-in user's time zone, otherwise empty. |
| `email` | Electronic mail address | text | no | none | derived, computed with elevated rights | all | The normalised address of the linked contact. |
| `mobile` | Telephone | text | no | none | derived, computed with elevated rights | all | The telephone number of the linked contact. |
| `visit_count` | Number of visits | integer | no | 1 | stored, read only | all | A new visit is counted when the previous connection was more than eight hours ago. |
| `website_track_ids` | Visit history | one_to_many to Website Visit Track | no | none | inverse of the visitor link, read only | all | The page view history. |
| `visitor_page_count` | Tracked page views | integer | no | none | derived from the history | all | Total number of tracked page views. |
| `page_ids` | Pages visited | many_to_many to Website Page | no | none | derived from the history | Editor and Designer | Distinct pages visited. |
| `page_count` | Distinct pages | integer | no | none | derived from the history | all | Number of distinct tracked pages. |
| `last_visited_page_id` | Last page | many_to_one to Website Page | no | none | derived from the history | all | The page of the most recent track that carries one. |
| `create_date` | First connection | datetime | no | insertion time | stored, read only, relabelled First Connection | all | First time the visitor was seen. |
| `last_connection_datetime` | Last connection | datetime | no | insertion time | stored, read only | all | Timestamp of the most recent page view. |
| `time_since_last_action` | Time since the last action | text | no | none | derived from the last connection | all | Human phrasing of the elapsed time, for example `2 minutes ago`. |
| `is_connected` | Connected | boolean | no | none | derived from the last connection | all | True when the last page view happened less than five minutes ago. |

The entity does not keep the platform's standard access-logging columns in the usual way: the
creation moment is relabelled First Connection, and both the creation moment and the last connection
moment are written directly by the insert-or-update statement described in
[visitors-and-tracking.md](visitors-and-tracking.md) §2.

Other capabilities add fields to Website Visitor and are specified in their own folders: leads and
lead count ([customer relationship management](../customer-relationship-management/)), event
registrations and wish-listed talks ([events](../events/)), live chat channels, operator and session
count, and the product counters of §5.12 of this file.

### Validation rules

* A visitor may only be created from a frontend request; computing a token outside a request fails
  with `Visitors can only be created through the frontend.`
* Merging requires a target linked to a contact; otherwise the message is
  `The `target` visitor should be linked to a partner.`
* Opening the message composer on a visitor without a contact or without an address fails with
  `There are no contact and/or no email linked to this visitor.`

## 1.8 Website Visit Track

Reference page: [`website.track`](../../references/entities/website.track.md).

* Default ordering: visit moment descending.
* Access-logging columns are not kept on this entity.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `visitor_id` | Visitor | many_to_one to Website Visitor | yes | none | stored, read only, indexed | The browsing identity. Deletion behaviour: cascade. |
| `page_id` | Page | many_to_one to Website Page | no | none | stored, read only, indexed | The page that was served, when the served content was a page. Deletion behaviour: cascade. |
| `url` | Address | long_text | no | none | stored, indexed | The full address of the request. |
| `visit_datetime` | Visit moment | datetime | yes | the current moment | stored, read only | When the view happened. |
| `product_id` | Product viewed | many_to_one to Product Variant | no | none | stored, read only | The product whose page was viewed. Added by the storefront capability. Deletion behaviour: cascade. |

## 1.9 Website Content Block Filter

Reference page: [`website.snippet.filter`](../../references/entities/website.snippet.filter.md).

* Default ordering: `name` ascending.
* Inherited mixin: Multi-site Publication Flag.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `name` | Filter label | text | yes | none | stored, translated | Label shown in the block configuration panel. |
| `action_server_id` | Record-returning action | many_to_one to Server Action | no | none | stored | A code action that returns the records. Deletion behaviour: cascade. |
| `field_names` | Exposed fields | text | yes | empty | stored | Comma-separated list of field names exposed to the rendering template. A name may carry a forced presentation after a colon, as in `price:monetary`. |
| `filter_id` | Saved filter | many_to_one to Saved Filter | no | none | stored | A stored search condition and ordering that selects the records. Deletion behaviour: cascade. |
| `limit` | Result cap | integer | yes | none | stored | Maximum number of records retrieved. |
| `website_id` | Site | many_to_one to Website | no | none | stored | Restricts the block to one site. Deletion behaviour: cascade. |
| `model_name` | Exposed entity | text | no | none | derived from the filter or the action | The exposed entity name. |
| `help` | Editor help | long_text | no | empty | stored, translated | Optional explanation shown to the editor. |
| `product_cross_selling` | Needs a reference product | boolean | no | false | stored | True only for product filters that need a reference product. Added by the storefront capability. |

### Validation rules

| Rule | Condition | Message |
|---|---|---|
| Exactly one data source | both the action and the filter are set, or neither is set | `Either action_server_id or filter_id must be provided.` |
| Limit range | the cap is not strictly greater than 0, or is greater than 16 | `The limit must be between 1 and 16.` |
| Field names not empty | any comma-separated part of the field list is blank after trimming | `Empty field name in “%s”` where the placeholder is the whole field list. |

## 1.10 Website Configurator Feature

Reference page:
[`website.configurator.feature`](../../references/entities/website.configurator.feature.md).

* Default ordering: `sequence`.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `sequence` | Ordering | integer | no | 0 | stored | Order in the feature picker. |
| `name` | Feature label | text | no | none | stored, translated | Feature label. |
| `description` | Explanation | text | no | none | stored, translated | One-line explanation. |
| `icon` | Tile icon | text | no | none | stored | Icon class of the feature tile. |
| `iap_page_code` | Suggestion service page code | text | no | none | stored | Page code sent to the content suggestion service so that it can generate a block list for that page. |
| `website_config_preselection` | Preselection list | text | no | none | stored | Comma-separated list of site types or purposes for which the feature is preselected. |
| `page_view_id` | Page template | many_to_one to View | no | none | stored | The template of the page to create. Deletion behaviour: cascade. |
| `module_id` | Capability package | many_to_one to capability package | no | none | stored | The package to install. Deletion behaviour: cascade. |
| `feature_url` | Feature address | text | no | none | stored | The address the feature lives at, used for the page and for the menu entry. |
| `menu_sequence` | Menu ordering | integer | no | 0 | stored | When non-zero, a menu entry is created for the feature at this ordering. |
| `menu_company` | Group under Company | boolean | no | false | stored | When true, and a Company grouping menu was created, the entry is created under it. |

Validation rule: exactly one of the page template and the package must be set; otherwise the message
is `One and only one of the two fields 'page_view_id' and 'module_id' should be set`.

## 1.11 Website Technical Page

Reference page:
[`website.technical.page`](../../references/entities/website.technical.page.md).

A read-only projection with no table of its own. Each row is one endpoint that declared itself as
listable site content.

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `name` | Declared title | text | The declared title of the endpoint. |
| `website_url` | Address | text | The last static path of the endpoint, that is the last declared route that contains no parameter placeholder. |

The projection is built from the routing table and cached with the routing cache. Opening a row opens
the site preview at that address. Only the administrator group may read it.

## 1.12 Editing dialogues and abstract services

| Entity | Transport name | Kind | Fields and behaviour |
|---|---|---|---|
| Page Properties Wizard | `website.page.properties` | transient | Edits one page: title, address, in-menu flag, home-page flag, publication flag, publishing date, indexing flag, visibility, visibility password, authorised groups, new-page-template flag, and whether to create a redirect from the old address and with which type. The dialogue carries the old address so that the redirect can be built; the full procedure is [workflows.md](workflows.md) §6. |
| Page Properties Base Wizard | `website.page.properties.base` | transient | The shared part of the dialogue, usable for any published record: name, address, publication flag and search engine metadata. |
| Robots Editor | `website.robots` | transient | One rich text field carrying the custom crawler exclusion text of the current site; saving writes it onto the site. Readable, writable and creatable by the Editor and Designer group, never deletable. |
| Blocked Domain List Editor | `website.custom_blocked_third_party_domains` | transient | One long text field carrying the custom blocked-domain list of the current site. On save each line is trimmed and lower-cased, blank lines are dropped, a line starting with `#` is kept verbatim as a comment, and any other line is reduced to its host part; a line that cannot be parsed is refused with `The following domain is not valid:` followed by a newline and the offending line. |
| Assets Utility | `website.assets` | abstract | Reads, customises and resets style-sheet source and script files per site; the algorithm is [content-management.md](content-management.md) §4. |
| Theme Utilities | `theme.utils` | abstract | Enables and disables theme templates and assets and resets the default style configuration; the algorithm is [content-management.md](content-management.md) §6. |
| Text Processor | `website.html.text.processor` | abstract | Turns rendered content blocks into placeholders, requests generated text from the content suggestion service and re-applies the original formatting; the algorithm is [content-management.md](content-management.md) §7.13. |

---

# Part 2: Theme records

A theme is an installable capability package that ships template records; installing it on a site
copies each template into a real record bound to that site, and the real record keeps a pointer back
to its template. The processing order is fixed: templates, then assets, then pages, then menus, then
attachments, because pages need templates and menus need pages.

## 2.1 Theme Template

Reference page: [`theme.ir.ui.view`](../../references/entities/theme.ir.ui.view.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Template name | text | yes | none | Template name. |
| `key` | Template key | text | no | none | Key of the generated template. |
| `type` | Template type | text | no | none | When empty, the generated record is a plain template. |
| `priority` | Rendering priority | integer | yes | the platform default sequence | Rendering priority. |
| `mode` | Inheritance mode | selection: `primary` = Base view, `extension` = Extension View | no | none | When empty, the mode of the generated template is inferred from the presence of a parent. |
| `active` | Active | boolean | no | true | Whether the generated template is active. |
| `arch` | Architecture | long_text | no | none | The markup tree, translated with term translation. |
| `arch_fs` | Shipped file path | text | no | the package-relative path of the file the template was loaded from | Where the shipped architecture lives, used by the hard reset operation. |
| `inherit_id` | Parent | reference to View or Theme Template | no | none | The parent template; either a real template or another theme template. |
| `copy_ids` | Generated templates | one_to_many to View | no | none | The real templates generated from this one, read only, not copied. |
| `customize_show` | Shown in the theme panel | boolean | no | false | Whether the generated template appears in the theme options panel. |

Conversion to a real template: when the parent is a theme template, the already-generated copy for
this site is used, and when it does not exist yet the conversion is postponed to a later pass. When
the parent is a real template belonging to another site, a site-specific template with the same key
is looked up and used when found. The generated values are the type (defaulting to the template
type), the name, the architecture, the key, the parent, the shipped file path, the priority, the
active flag, the back pointer, the site and the panel flag; the inheritance mode is copied only when
the theme template sets it.

## 2.2 Theme Asset

Reference page: [`theme.ir.asset`](../../references/entities/theme.ir.asset.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `key` | Asset key | text | no | none | Key of the generated asset. |
| `name` | Asset name | text | yes | none | Asset name. |
| `bundle` | Bundle | text | yes | none | The bundle the asset belongs to. |
| `directive` | Insertion directive | selection: append, prepend, after, before, remove, replace, include | no | append | How the asset is inserted into the bundle. |
| `path` | Asset path | text | yes | none | The file path or pattern of the asset. |
| `target` | Directive target | text | no | none | The existing path the directive applies to, for the positional and replacing directives. |
| `active` | Active | boolean | no | true | Whether the generated asset is active. |
| `sequence` | Ordering | integer | yes | the platform default sequence | Order inside the bundle. |
| `copy_ids` | Generated assets | one_to_many to Asset | no | none | The real assets generated from this one. |

## 2.3 Theme Attachment

Reference page: [`theme.ir.attachment`](../../references/entities/theme.ir.attachment.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | File name | text | yes | none | File name. |
| `key` | Attachment key | text | yes | none | Key of the generated attachment. |
| `url` | Served address | text | no | none | The address the attachment is served from. |
| `copy_ids` | Generated attachments | one_to_many to Attachment | no | none | The real attachments generated from this one. |

The generated attachment is public, of the address kind, related to the View entity, and carries the
key, the name, the address, the site and the back pointer.

## 2.4 Theme Menu

Reference page: [`theme.website.menu`](../../references/entities/theme.website.menu.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Entry label | text | yes | none | Entry label, translated. |
| `url` | Target address | text | no | empty | Target address. |
| `page_id` | Target theme page | many_to_one to Theme Page | no | none | Target theme page. Deletion behaviour: cascade. |
| `new_window` | Open in a new tab | boolean | no | false | Open in a new tab. |
| `sequence` | Ordering | integer | no | 0 | Order. |
| `parent_id` | Parent theme entry | many_to_one to Theme Menu | no | none | Parent template entry. Deletion behaviour: cascade. |
| `mega_menu_content` | Mega-menu markup | rich_text | no | empty | Mega-menu markup. |
| `mega_menu_classes` | Mega-menu classes | text | no | empty | Mega-menu layout classes. |
| `use_main_menu_as_parent` | Attach to the root menu | boolean | no | true | When true and no parent template is set, the generated entry is attached to the site's root menu. |
| `copy_ids` | Generated entries | one_to_many to Website Menu | no | none | The real entries generated from this one. |

## 2.5 Theme Page

Reference page: [`theme.website.page`](../../references/entities/theme.website.page.md).

Inherits the Page Options mixin, so it also carries the header overlay, the header colour, the header
text colour and the header and footer visibility.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `url` | Page address | text | no | none | The page address. |
| `view_id` | Content theme template | many_to_one to Theme Template | yes | none | The template holding the page content. Deletion behaviour: cascade. |
| `website_indexed` | Indexed | boolean | no | true | Indexing flag of the generated page. |
| `is_published` | Published | boolean | no | false | Publication flag of the generated page. |
| `is_new_page_template` | Offered as a template | boolean | no | false | Whether the generated page is offered as a new-page template. |
| `copy_ids` | Generated pages | one_to_many to Website Page | no | none | The real pages generated from this one. |

Conversion to a real page is postponed when the theme template of the content has not yet produced a
copy for this site.

---

# Part 3: Mixins this folder defines

## 3.1 Search Engine Metadata

Reference page: [`website.seo.metadata`](../../references/entities/website.seo.metadata.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `is_seo_optimized` | Metadata complete | boolean | no | false | derived and stored from the three fields below | True only when the title, the description and the keywords are all filled. |
| `website_meta_title` | Meta title | text | no | empty | stored, translated | Overrides the page title and the sharing title. |
| `website_meta_description` | Meta description | long_text | no | empty | stored, translated | Overrides the page description and the sharing description. |
| `website_meta_keywords` | Meta keywords | text | no | empty | stored, translated | Keyword list emitted in the document head. |
| `website_meta_og_image` | Sharing image address | text | no | empty | stored | Overrides the sharing image address. |
| `seo_name` | Slug name | text | no | empty | stored, translated | Overrides the human part of the record slug in generated addresses. |

Default metadata produced for any record carrying the mixin:

* page title: the record name, a space, a vertical bar, a space, then the site name, when the record
  has a name; otherwise the site name;
* sharing type `website`; sharing title as above; sharing site name equal to the site name; sharing
  address equal to the site domain (or the request root when no domain is set) joined with the
  rewritten current path; sharing image equal to the site image address of the dedicated sharing
  image when one exists, otherwise of the logo;
* summary card type `summary_large_image`; summary title as above; summary image equal to the site
  image address of the same field at size 300 by 300; when the company declares an account on the
  short-message network, the summary site handle is that account's last path segment prefixed by `@`.

Final metadata is the default metadata with the stored overrides applied: a stored meta title
replaces both the sharing title and the summary title; a stored meta description replaces both
descriptions; a stored sharing image replaces both images after its host and scheme are stripped;
both image addresses are then made absolute against the site domain, or against the request root
stripped of its trailing slash when no domain is set.

## 3.2 Publication Flag

Reference page: [`website.published.mixin`](../../references/entities/website.published.mixin.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `website_published` | Published switch | boolean | no | false | derived from the stored flag, writable | The publication switch shown in the site editor. |
| `is_published` | Publication flag | boolean | no | the entity's own default, false unless overridden | stored, indexed, not copied | The stored publication flag. |
| `can_publish` | May publish | boolean | no | none | derived, depends on the current user | True when the current user may change the publication flag. |
| `website_url` | Public address | text | no | `#` | derived, depends on the language in context | The full relative address of the record on the site. |
| `website_absolute_url` | Absolute public address | text | no | `#` | derived from the relative address | The relative address joined with the record's base address; stays `#` when the relative address is `#`. |

Permission rule: the default computation of the publication right tries the platform write check on
the record and returns false when it is refused. Creating a record with the publication flag already
true, or writing the publication flag, is refused with `You do not have the rights to
publish/unpublish` when the right is false for any record in the set. The Toggle Publication
operation flips the switch and returns the new value.

## 3.3 Multi-site Restriction

Reference page: [`website.multi.mixin`](../../references/entities/website.multi.mixin.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `website_id` | Site | many_to_one to Website | no | none | stored, indexed | Restricts the record to one site; empty means every site. Deletion behaviour: restrict. |

Access helper: a record is accessible from the current site when its site is empty or equal to the
current site.

## 3.4 Multi-site Publication Flag

Reference page:
[`website.published.multi.mixin`](../../references/entities/website.published.multi.mixin.md).

Combines the two mixins above and redefines the publication switch as derived, writable and
searchable.

* Derivation: when a site is present in the execution context, the switch is true only when the
  stored flag is true and the record is either shared or bound to that site; otherwise the switch
  equals the stored flag.
* Writing the switch writes the stored flag.
* Searching for published records with a site in context yields "stored flag is true and the site is
  empty or the current site"; without a site in context it yields "stored flag is true". Only the
  equality-to-true search is supported.
* Opening the record on the site: when the record is bound to a site that has a domain, an absolute
  link into that domain's editor preview is produced; otherwise the editor preview of the current
  site is opened with the record's site passed along.

## 3.5 Searchable

Reference page: [`website.searchable.mixin`](../../references/entities/website.searchable.mixin.md).

This mixin declares no field. It defines the contract every searchable entity implements.

| Operation | Inputs | Output |
|---|---|---|
| Describe search | the site, the requested ordering, the search options | A search descriptor (below). Every entity that participates in site search must provide it; the base implementation raises a not-implemented error. |
| Fetch matches | the descriptor, the search text, a maximum number of results, the ordering | The matching records and the total count. The count is the exact number of matches when the result set is shorter than the cap, otherwise a separate count query is issued. |
| Render matches | the fields to read, the field mapping, the fallback icon, a maximum number of rows | One dictionary per result containing the read values, the fallback icon and the mapping; rich text fields named in the mapping are converted to whitespace-collapsed plain text, and for a template architecture the double-escaped entities are first unescaped. |

The search descriptor contains: the entity name; the base condition list; the fields the search text
is matched against; the fields to read for rendering; the mapping from template slot to field name,
field type and rendering flags (`match` to highlight the term, `truncate` to allow shortening, `html`
to convert markup to text); the fallback icon; optionally a flag requesting elevated reads;
optionally an extra condition builder for a search term; optionally a forced ordering.

Domain building: the base condition is combined with, for each whitespace-separated term of the
search text, the disjunction of "field contains term" over the search fields, plus the extra
condition when one is supplied. Special characters in the term are escaped so that they are matched
literally.

## 3.6 Cover Properties

Reference page:
[`website.cover_properties.mixin`](../../references/entities/website.cover_properties.mixin.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `cover_properties` | Cover configuration | long_text | no | a structured record with background colour class `o_cc3`, background image `none`, opacity `0.2` and height class `o_half_screen_height` | The cover configuration, stored as a structured data document in text form. |

Write guard: when the cover configuration is written and the new height class contains none of
`o_half_screen_height`, `o_full_screen_height`, `cover_auto`, the previous height class of each
record is preserved, so that a bulk edit from a list cannot destroy the cover height.

Background read helper: the stored background image expression is returned as is, except that when it
points at the image service (it starts with `url(/web/image/`) the requested height and width are
appended as query parameters, using `?` when the address carries no query string yet and `&`
otherwise.

## 3.7 Page Options and Page Visibility Options

Reference pages:
[`website.page_options.mixin`](../../references/entities/website.page_options.mixin.md) and
[`website.page_visibility_options.mixin`](../../references/entities/website.page_visibility_options.mixin.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `header_visible` | Show the header | boolean | no | true | Show the site header on this page or record page. |
| `footer_visible` | Show the footer | boolean | no | true | Show the site footer on this page or record page. |
| `header_overlay` | Header overlays the content | boolean | no | false | Header floats over the first content block. |
| `header_color` | Header colour | text | no | empty | Header background colour class or value. |
| `header_text_color` | Header text colour | text | no | empty | Header text colour class or value. |

The visibility mixin carries only the first two fields; the options mixin inherits it and adds the
other three.

## 3.8 Rich Text History

Reference page:
[`html.field.history.mixin`](../../references/entities/html.field.history.mixin.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `html_field_history` | Revision history | structured_data | no | empty | stored, read only | Per field, an ordered list of revisions, each holding a revision identifier and the reverse patch needed to restore the previous content. |
| `html_field_history_metadata` | Revision metadata | structured_data | no | none | derived from the history | Per field, the list of revision identifiers with their author and timestamp, without the patch payloads. |

Contract: the entity declares which of its fields are versioned. Every declared field must be a
sanitised rich text field; writing a record whose declared fields are not all sanitised fails with
`Ensure all versioned fields ( %s ) in model %s are declared as sanitize=True`. On create, an empty
history is initialised for each declared field; on write, the reverse patch between the old and the
new content is prepended to that field's history, the revision identifier is incremented, and the
history is truncated to the configured depth. Three read operations are offered: restore a field at a
revision, produce a marked-up comparison between the current content and a revision, and produce a
line-oriented difference between them.

## 3.9 Portal Record

Reference page: [`portal.mixin`](../../references/entities/portal.mixin.md).

Owned by [customer portal](../customer-portal/). It carries the access token and the public address
of a record shared with a customer. This folder uses it for two purposes: the abandoned-cart recovery
link of [storefront-checkout.md](storefront-checkout.md) §11.4, which is built from the order token,
and the payment transaction guard of [storefront-checkout.md](storefront-checkout.md) §10.4.

---

# Part 4: Forum, blog and public directories

## 4.1 Forum

Reference page: [`forum.forum`](../../references/entities/forum.forum.md).

One discussion space. A forum carries its own reputation table, so two forums of the same site can
demand different scores for the same operation.

* Default ordering: `sequence` ascending, then `identifier` ascending.
* Display name rule: `name`.
* Inherited mixins: discussion thread, picture, Search Engine Metadata, Multi-site Restriction,
  Searchable.
* Public address: `/forum/<slug of the forum>`.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `name` | Forum name | text | yes | none | stored, translated | The forum name. |
| `sequence` | Ordering | integer | no | 1 | stored | Order in the forum list. |
| `mode` | Answering mode | selection: `questions` = Questions (1 answer), `discussions` = Discussions (multiple answers) | yes | `questions` | stored | In questions mode a participant may post only one answer per question; in discussions mode several. |
| `privacy` | Privacy | selection: `public` = Public, `connected` = Signed In, `private` = Some users | no | `public` | stored | Who may reach the forum and its content. |
| `authorized_group_id` | Authorised group | many_to_one to Access Group | no | none | stored | The group allowed when the privacy is `private`. Cleared automatically when the privacy becomes `public` or `connected`. |
| `active` | Active | boolean | no | true | stored | Archiving. Writing it writes the same value on every post of the forum, archived ones included. |
| `faq` | Guidelines | rich_text | no | the rendered shipped guidelines template | stored, translated, sanitised, overridable by the sanitisation group | The guidelines page of the forum, filled at creation from the shipped template. |
| `description` | Description | text | no | none | stored, translated | Shown in listings and used as the search description. |
| `welcome_message` | Welcome message | rich_text | no | the shipped welcome block | stored, translated, attributes and forms not stripped | The banner shown to a visitor until it is dismissed. Its shipped text contains a heading `Welcome!`, the sentence `Share and discuss the best content and new marketing ideas, build your professional profile and become a better marketer together.`, a button labelled `Sign up` pointing at the sign-in page and a dismiss button labelled `Dismiss`. |
| `default_order` | Default ordering of questions | selection: `create_date desc` = Newest, `last_activity_date desc` = Last Updated, `vote_count desc` = Most Voted, `relevancy desc` = Relevance, `child_count desc` = Answered | yes | `last_activity_date desc` | stored | The ordering applied when the visitor has not chosen one. |
| `relevancy_post_vote` | First relevance parameter | decimal | no | 0.8 | stored | Exponent applied to the vote count in the relevance formula. |
| `relevancy_time_decay` | Second relevance parameter | decimal | no | 1.8 | stored | Exponent applied to the age in the relevance formula. |
| `allow_share` | Sharing options | boolean | no | true | stored | After posting, the participant is offered to share the question or the answer on social networks. |
| `post_ids` | Posts | one_to_many to Forum Post | no | none | inverse of the forum link | Every post of the forum. |
| `last_post_id` | Last question | many_to_one to Forum Post | no | none | derived | The highest-numbered active question of the forum. |
| `total_posts` | Number of questions | integer | no | none | derived | Count of active and closed questions. |
| `total_views` | Number of views | integer | no | none | derived | Summed view counters of those questions. |
| `total_answers` | Number of answers | integer | no | none | derived | Summed answer counts of those questions. |
| `total_favorites` | Number of favourites | integer | no | none | derived | Number of those questions that at least one participant marked as a favourite. |
| `count_posts_waiting_validation` | Posts waiting for validation | integer | no | none | derived | Count of posts in the pending state. |
| `count_flagged_posts` | Flagged posts | integer | no | none | derived | Count of posts in the flagged state. |
| `has_pending_post` | Has a pending question | boolean | no | none | derived, depends on the current user | True when the current user has a question of this forum waiting for validation. |
| `can_moderate` | Is a moderator | boolean | no | none | derived, depends on the current user | True when the current user's reputation reaches the moderation threshold. |
| `tag_ids` | Tags | one_to_many to Forum Tag | no | none | inverse of the forum link | The tags of this forum. |
| `tag_most_used_ids` | Most used tags | one_to_many to Forum Tag | no | none | derived | The five tags with the highest post count, ordered by count descending then name. |
| `tag_unused_ids` | Unused tags | one_to_many to Forum Tag | no | none | derived | The tags with no post. |

### Reputation generated by activity

| Identifier | Full name | Default | Meaning |
|---|---|---|---|
| `karma_gen_question_new` | Asking a question | 2 | Awarded to the author when a question becomes active. |
| `karma_gen_question_upvote` | Question upvoted | 5 | Awarded to the author of a question for each up-vote. |
| `karma_gen_question_downvote` | Question downvoted | −2 | Applied to the author of a question for each down-vote. |
| `karma_gen_answer_upvote` | Answer upvoted | 10 | Awarded to the author of an answer for each up-vote. |
| `karma_gen_answer_downvote` | Answer downvoted | −2 | Applied to the author of an answer for each down-vote. |
| `karma_gen_answer_accept` | Accepting an answer | 2 | Awarded to the participant who accepts an answer. |
| `karma_gen_answer_accepted` | Answer accepted | 15 | Awarded to the author of an accepted answer. |
| `karma_gen_answer_flagged` | Answer flagged | −100 | Applied to the author when a post is marked offensive or closed as spam or offensive. |

### Reputation required to act

| Identifier | Full name | Default | Gate |
|---|---|---|---|
| `karma_ask` | Ask questions | 3 | Creating a question. |
| `karma_answer` | Answer questions | 3 | Creating an answer. |
| `karma_edit_own` | Edit own posts | 1 | Editing a post the participant created. |
| `karma_edit_all` | Edit all posts | 300 | Editing any other post. |
| `karma_edit_retag` | Change question tags | 75 | Changing the tag list of a post. |
| `karma_close_own` | Close own posts | 100 | Closing or reopening an own question; also grants the right to see it while closed. |
| `karma_close_all` | Close all posts | 500 | Closing or reopening any question. |
| `karma_unlink_own` | Delete own posts | 500 | Deleting or reactivating an own post. |
| `karma_unlink_all` | Delete all posts | 1000 | Deleting or reactivating any post. |
| `karma_tag_create` | Create new tags | 30 | Creating a tag. |
| `karma_upvote` | Upvote | 5 | Casting an up-vote. |
| `karma_downvote` | Downvote | 50 | Casting a down-vote. |
| `karma_answer_accept_own` | Accept an answer on own questions | 20 | Accepting an answer on a question the participant asked. |
| `karma_answer_accept_all` | Accept an answer to all questions | 500 | Accepting an answer on any question. |
| `karma_comment_own` | Comment own posts | 1 | Commenting on an own post. |
| `karma_comment_all` | Comment all posts | 1 | Commenting on any post. |
| `karma_comment_convert_own` | Convert own comments to answers | 50 | Converting an own comment into an answer. |
| `karma_comment_convert_all` | Convert all comments to answers | 500 | Converting any comment into an answer. |
| `karma_comment_unlink_own` | Delete own comments | 50 | Deleting an own comment. |
| `karma_comment_unlink_all` | Delete all comments | 500 | Deleting any comment. |
| `karma_flag` | Flag a post as offensive | 500 | Flagging a post for moderation. |
| `karma_dofollow` | Links followed by crawlers | 500 | Below this score every link in the content receives the no-follow marker. |
| `karma_editor` | Editor features: pictures and links | 30 | Posting a picture, a link or a background picture. |
| `karma_user_bio` | Display detailed biography | 750 | Below this score the author's biography is hidden on the post. |
| `karma_post` | Ask questions without validation | 100 | Below this score a new question is created in the pending state. |
| `karma_moderate` | Moderate posts | 1000 | Validating, refusing and marking posts offensive; also makes the moderation queues visible. |

### Record lifecycle

1. **Create.** The creation is silent in the discussion thread and no follower is subscribed
   automatically. The forum count of every site is refreshed. The guidelines field is filled by
   rendering the shipped guidelines template with the new forum as value.
2. **Update.** Writing the privacy to `public` or `connected` clears the authorised group. Writing
   the active flag writes it on every post of the forum, archived posts included. Writing the active
   flag or the site refreshes the forum count of every site.
3. **Delete.** The forum count of every site is refreshed.

### Tag parsing helper

The question form submits its tags as one text: existing tags as their identifier, new tags as an
underscore followed by the name. Parsing keeps the identifiers, and, for each new name, reuses an
existing tag of the same forum when one already carries that name, and otherwise creates one only
when the participant's reputation reaches the tag-creation threshold and the name is not empty.

## 4.2 Forum Post

Reference page: [`forum.post`](../../references/entities/forum.post.md).

One question or one answer. A comment is not a post: it is a message on the discussion thread of a
post.

* Default ordering: accepted answers first, then vote count descending, then last activity
  descending.
* Inherited mixins: discussion thread, Search Engine Metadata, Searchable.
* Public address: `/forum/<slug of the forum>/<slug of the post>`, with the fragment
  `#answer_<identifier>` appended when the post is an answer.
* Notification behaviour: the recipient header limit is zero, so recipients are never disclosed to
  each other; comments are never pushed to the internal inbox, only sent as electronic mail.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `name` | Title | text | no | none | stored | The question title; an answer created from the reply form receives `Re: ` followed by the question title. |
| `forum_id` | Forum | many_to_one to Forum | yes | none | stored, indexed | The forum the post belongs to. |
| `content` | Content | rich_text | no | none | stored, inline styles stripped | The body of the post. |
| `plain_content` | Plain content | text | no | none | derived and stored from the content | The first 500 characters of the plain text of the content, used for search and sharing metadata. |
| `tag_ids` | Tags | many_to_many to Forum Tag | no | empty | stored | Labels of the question. |
| `state` | Status | selection: `active` = Active, `pending` = Waiting Validation, `close` = Closed, `offensive` = Offensive, `flagged` = Flagged | no | `active` | stored | The moderation state; see [state-machines.md](state-machines.md) §7. |
| `views` | Views | integer | no | 0 | stored, read only, not copied | Number of times the question page was served. |
| `active` | Active | boolean | no | true | stored | Archiving. Writing it writes the same value on every answer of the post. |
| `website_message_ids` | Public comments | one_to_many to Message | no | none | filtered view of the thread | Messages of this post whose type is electronic mail, comment or outgoing mail. |
| `website_id` | Site | many_to_one to Website | no | none | derived from the forum, read only | The site the post belongs to. |
| `create_date` | Asked on | datetime | no | insertion time | stored, read only, indexed | Creation moment. |
| `create_uid` | Author | many_to_one to User | no | the current user | stored, read only, indexed | The author. |
| `write_date` | Updated on | datetime | no | write time | stored, read only, indexed | Last change. |
| `write_uid` | Updated by | many_to_one to User | no | the current user | stored, read only, indexed | Last editor. |
| `last_activity_date` | Last activity on | datetime | yes | the current moment | stored, read only | Refreshed whenever the post is replied to or commented on, or one of its answers is. |
| `relevancy` | Relevance | decimal | no | none | derived and stored | The relevance score of [calculations.md](calculations.md) §22. |
| `vote_ids` | Votes | one_to_many to Forum Post Vote | no | none | inverse of the post link | The votes cast on the post. |
| `user_vote` | My vote | integer | no | none | derived, depends on the current user | The current user's vote: 1, 0 or −1. |
| `vote_count` | Total votes | integer | no | none | derived and stored | The sum of the votes cast. |
| `favourite_ids` | Favourites of | many_to_many to User | no | empty | stored | The participants who marked the question as a favourite. |
| `user_favourite` | Is my favourite | boolean | no | none | derived, depends on the current user | True when the current user marked it. |
| `favourite_count` | Favourite count | integer | no | none | derived and stored | Number of participants who marked it. |
| `is_correct` | Accepted answer | boolean | no | false | stored | True for the accepted answer. |
| `parent_id` | Question | many_to_one to Forum Post | no | none | stored, read only, indexed | The question this post answers; empty for a question. Deletion behaviour: cascade. |
| `self_reply` | Reply to own question | boolean | no | none | derived and stored | True when the author of the answer is the author of the question. |
| `child_ids` | Answers | one_to_many to Forum Post | no | none | inverse of the question link, restricted to the same forum | The answers. |
| `child_count` | Number of answers | integer | no | none | derived and stored | Number of answers. |
| `uid_has_answered` | I have answered | boolean | no | none | derived, depends on the current user | True when the current user already answered. |
| `has_validated_answer` | Is answered | boolean | no | none | derived and stored | True when one answer is accepted. |
| `flag_user_id` | Flagged by | many_to_one to User | no | none | stored | Who flagged the post. |
| `moderator_id` | Reviewed by | many_to_one to User | no | none | stored, read only | Who validated, refused or marked the post offensive. |
| `closed_reason_id` | Closing reason | many_to_one to Forum Post Closing Reason | no | none | stored, not copied | Why the post was closed or marked offensive. |
| `closed_uid` | Closed by | many_to_one to User | no | none | stored, read only, not copied | Who closed the post. |
| `closed_date` | Closed on | datetime | no | none | stored, read only, not copied | When the post was closed. |

### Derived rights

Every right below is derived per post and per current user; an administrator always passes. The
thresholds come from the forum of the post.

| Identifier | Full name | Value |
|---|---|---|
| `karma_accept` | Reputation to accept | The own-question threshold when the current user asked the question, otherwise the all-questions threshold. |
| `karma_edit` | Reputation to edit | The own threshold when the current user is the author, otherwise the all-posts threshold. |
| `karma_close` | Reputation to close | Same rule with the closing thresholds. |
| `karma_unlink` | Reputation to delete | Same rule with the deletion thresholds. |
| `karma_comment` | Reputation to comment | Same rule with the commenting thresholds. |
| `karma_comment_convert` | Reputation to convert | Same rule with the conversion thresholds. |
| `karma_flag` | Reputation to flag | The forum's flagging threshold. |
| `can_ask`, `can_answer`, `can_accept`, `can_edit`, `can_close`, `can_unlink`, `can_comment`, `can_comment_convert`, `can_post`, `can_flag`, `can_moderate`, `can_use_full_editor` | Permission flags | True when the current user's reputation reaches the matching threshold. |
| `can_upvote` | May up-vote | True when the reputation reaches the up-vote threshold, or the current user's present vote is a down-vote (withdrawing it is always allowed). |
| `can_downvote` | May down-vote | True when the reputation reaches the down-vote threshold, or the current user's present vote is an up-vote. |
| `can_view` | May view | True when the user may close the post, or the post is active and its author has a strictly positive reputation, or the current user is the author. Searchable, so that listings and the site index apply the same filter. |
| `can_display_biography` | Biography visible | True when the author's reputation reaches the biography threshold and the author's profile is published. |

### Validation rules

| Rule | Condition | Message |
|---|---|---|
| No cycles | the answer chain of a post contains the post itself | `You cannot create recursive forum posts.` |
| Answering a closed or deleted question | the parent question is closed or archived | `Posting answer on a [Deleted] or [Closed] question is not possible.` |
| Reputation to ask | creating a question below the asking threshold | `%d karma required to create a new question.` with the threshold substituted |
| Reputation to answer | creating an answer below the answering threshold | `%d karma required to answer a question.` |
| Reputation to edit | writing any field outside the trusted set below the editing threshold | `%d karma required to edit a post.` |
| Reputation to close or reopen | writing the state to `active` or `close` below the closing threshold | `%d karma required to close or reopen a post.` |
| Reputation to flag | writing the state to `flagged` below the flagging threshold | `%d karma required to flag a post.` |
| Reputation to delete | writing the active flag, or deleting, below the deletion threshold | `%d karma required to delete or reactivate a post.` for the write and `%d karma required to unlink a post.` for the deletion |
| Reputation to accept | writing the acceptance flag below the acceptance threshold | `%d karma required to accept or refuse an answer.` |
| Reputation to retag | changing the tag list below the retagging threshold | `%d karma required to retag.` |
| Reputation to comment | posting a comment below the commenting threshold | `%d karma required to comment.` |
| Reputation to convert a comment | converting a comment below the conversion threshold | `%d karma required to convert your comment to an answer.` when the participant is the comment author and the own threshold is lower, otherwise `%d karma required to convert a comment to an answer.` |
| Reputation to convert an answer | converting an answer into a comment below the conversion threshold | `%d karma required to convert an answer to a comment.` |
| Reputation to delete a comment | deleting a comment below the matching threshold | `%d karma required to delete a comment.` |
| Reputation to use pictures and links | the content contains a picture element, a link element or an element whose style declares a background picture, below the editor threshold | `%d karma required to post an image or link.` |
| Reputation to moderate | validating, refusing or marking offensive below the moderation threshold | `%d karma required to validate a post.`, `%d karma required to refuse a post.` and `%d karma required to mark a post as offensive.` |
| Empty content | the submitted content is visually empty | `Question should not be empty.` for a question and `Reply should not be empty.` for an answer, both rendered as a bad-request page |
| Empty title | the submitted title is blank on save | `Title should not be empty.` rendered as a bad-request page |

The trusted set of fields, whose security is checked individually rather than by the editing
threshold, is the active flag, the acceptance flag and the tag list, plus the state and its closing
fields when the state is being written to `active` or `close`, plus the state and the flagging user
when the state is being written to `flagged`.

### Content rewriting on create and on write

1. When the author's reputation is below the no-follow threshold, every link element in the content
   is rewritten with the no-follow marker, keeping its address.
2. When the author's reputation is below the editor threshold and the content contains a picture, a
   link or a background picture, the write is refused with the editor message above.

### Record lifecycle

1. **Create.** The creation is silent in the discussion thread. A question whose author is below the
   validation threshold is forced into the pending state. A question created active awards the
   asking points to its author. The state notification then runs.
2. **Update.** The rights above are checked field by field. Accepting or withdrawing acceptance moves
   reputation in both directions, except when the participant accepts their own answer. Writing the
   content or the title posts a tracked message on the question, with the subject `Question Edited`
   and its own subtype for a question and `Answer Edited` and its own subtype for an answer. Writing
   the active flag writes it on every answer.
3. **Delete.** Deleting an accepted answer withdraws the acceptance points from its author and the
   acceptance award from the participant who accepted it.
4. **State notification.** An active answer posts the shipped new-answer message on its question with
   the subject `Re: ` followed by the question title; an active question posts the shipped
   new-question message on itself with the question title as subject; a pending question posts the
   shipped validation message as an internal note addressed to every follower of the post and of its
   tags whose reputation reaches the moderation threshold. Followers of the tags of the post are
   added as recipients of the first two.
5. **Access from a notification.** Opening an active post from a notification redirects to its public
   address; a post in any other state opens the ordinary record form. For an active post every
   recipient group receives the access button.

### Structured description for search engines

A question that has at least one answer publishes a question-and-answer page description containing
the question, its accepted answer when there is one and at most five suggested answers. Each entry
carries the vote count, the publication moment, the public address and the author, and the author
carries a link to the public profile when that profile is published. The question additionally
carries its title, its plain content and its answer count.

### Related questions

The related questions of a question are the at most five questions with the highest tag similarity,
computed as described in [calculations.md](calculations.md) §23, ordered by similarity descending and
then by last activity descending. A question with no tag has no related questions.

## 4.3 Forum Post Vote

Reference page: [`forum.post.vote`](../../references/entities/forum.post.vote.md).

* Default ordering: creation moment descending, then `identifier` descending.
* Database constraint: `unique (post_id, user_id)`, message `Vote already exists!`

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `post_id` | Post | many_to_one to Forum Post | yes | none | stored, indexed | The post voted on. Deletion behaviour: cascade. |
| `user_id` | Voter | many_to_one to User | yes | the current user | stored | Who voted. Deletion behaviour: cascade. |
| `vote` | Vote | selection: `1` = 1, `-1` = −1, `0` = 0 | yes | `1` | stored | The direction of the vote; `0` records a withdrawn vote. |
| `create_date` | Cast on | datetime | no | insertion time | stored, read only, indexed | When the vote was cast. |
| `forum_id` | Forum | many_to_one to Forum | no | none | derived and stored from the post | Used for reporting and for the reputation update. |
| `recipient_id` | Beneficiary | many_to_one to User | no | none | derived and stored from the post author | The participant whose reputation the vote moves. |

### Rules

* A participant may not vote on their own post: `It is not allowed to vote for its own post.`
* A participant may not change somebody else's vote:
  `It is not allowed to modify someone else's vote.`
* Up-voting below the up-vote threshold is refused with `%d karma required to upvote.`; down-voting
  below the down-vote threshold with `%d karma required to downvote.`
* A non-administrator may never write the voter or the beneficiary; both are dropped from the
  submitted values.
* Every create and every change of the vote value moves the beneficiary's reputation by the
  difference between the new and the old award, as specified in [calculations.md](calculations.md)
  §21.

## 4.4 Forum Tag

Reference page: [`forum.tag`](../../references/entities/forum.tag.md).

* Inherited mixins: discussion thread, Searchable, Search Engine Metadata.
* Database constraint: `unique (name, forum_id)`, message `Tag name already exists!`
* Public address: `/forum/<slug of the forum>/tag/<slug of the tag>/questions`.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `name` | Tag name | text | yes | none | stored | The label. |
| `color` | Colour index | integer | no | 0 | stored | Colour of the tag chip. |
| `forum_id` | Forum | many_to_one to Forum | yes | none | stored, indexed | The forum the tag belongs to. |
| `post_ids` | Questions | many_to_many to Forum Post | no | empty | stored, restricted to active posts | The questions carrying the tag. |
| `posts_count` | Number of questions | integer | no | none | derived and stored | Number of active questions carrying the tag. |

Creating a tag below the tag-creation threshold is refused with
`%d karma required to create a new Tag.` Creation is silent in the discussion thread and subscribes
nobody.

## 4.5 Forum Post Closing Reason

Reference page: [`forum.post.reason`](../../references/entities/forum.post.reason.md).

* Default ordering: `name`.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Reason | text | yes | none | The reason shown in the closing dialogue, translated. |
| `reason_type` | Reason kind | selection: `basic` = Basic, `offensive` = Offensive | no | `basic` | Basic reasons are offered when closing a question; offensive reasons are offered when marking a post as offensive. |

The shipped reasons are listed in [configuration.md](configuration.md) §5.4. Two of them carry a
reputation consequence: the reason `Contains offensive or malicious remarks` and the reason
`Spam or advertising` deduct the flagging award from the author when a question is closed with them,
and restore it when the question is reopened; for the spam reason the deduction is multiplied by ten
when the question is the author's first question in that forum.

## 4.6 Blog

Reference page: [`blog.blog`](../../references/entities/blog.blog.md).

* Default ordering: `name`.
* Inherited mixins: discussion thread, Search Engine Metadata, Multi-site Restriction, Cover
  Properties, Searchable.
* Notification behaviour: the recipient header limit is zero, so recipients are never disclosed to
  each other.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `sequence` | Ordering | integer | no | the highest existing ordering plus one | stored | Order in the blog list and in the navigation. |
| `name` | Blog name | text | yes | none | stored, translated | Blog name. |
| `subtitle` | Subtitle | text | no | empty | stored, translated | Used as the description in the site search. |
| `active` | Active | boolean | no | true | stored | Archiving. |
| `content` | Free content | rich_text | no | empty | stored, never sanitised, translated with term translation | Content-block area of the blog landing page. |
| `blog_post_ids` | Posts | one_to_many to Blog Post | no | none | inverse of the blog link | The posts of this blog. |
| `blog_post_count` | Number of posts | integer | no | none | derived | Number of posts. |
| `website_id` | Site | many_to_one to Website | no | none | stored, from the multi-site mixin | Restricts the blog to one site. Deletion behaviour: restrict. |
| `cover_properties` | Cover configuration | long_text | no | the default cover record | stored, from the cover properties mixin | Cover picture, colour class, opacity and height class. |

Lifecycle rule: writing the active flag on a blog writes the same value on every post of that blog,
already archived ones included.

Comment routing rule: a comment posted as a reply to a message whose subtype is the publication
subtype is downgraded to an internal note, so that blog followers are not notified of every answer.

## 4.7 Blog Post

Reference page: [`blog.post`](../../references/entities/blog.post.md).

* Default ordering: `identifier` descending.
* Discussion thread access mode: read access on the post is enough to post a message.
* Inherited mixins: discussion thread, Search Engine Metadata, Multi-site Publication Flag, Page
  Visibility Options, Cover Properties, Searchable.
* Public address: `/blog/<slug of the blog>/<slug of the post>`.

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Title | text | yes | empty | stored, translated | renamed on copy | Post title. |
| `subtitle` | Subtitle | text | no | empty | stored, translated | yes | Used as the sharing description and the search description. |
| `author_id` | Author | many_to_one to Contact | no | the contact of the current user | stored, indexed when set | yes | The author. |
| `author_avatar` | Author picture | binary | no | none | derived from the author picture, writable | yes | Shown next to the post. |
| `author_name` | Author name | text | no | none | derived and stored from the author display name, writable | yes | Stored so that it survives contact changes. |
| `active` | Active | boolean | no | true | stored | yes | Archiving. |
| `blog_id` | Blog | many_to_one to Blog | yes | the first blog found | stored, indexed | yes | The blog the post belongs to. Deletion behaviour: cascade. |
| `tag_ids` | Tags | many_to_many to Blog Tag | no | empty | stored | yes | Labels of the post. |
| `content` | Article body | rich_text | no | a paragraph containing `Start writing here...` | stored, never sanitised, translated with term translation | yes | The body. |
| `teaser` | Teaser | long_text | no | none | derived from the content and the manual teaser, writable through an inverse, translated | yes | The short excerpt shown in listings. |
| `teaser_manual` | Manual teaser | long_text | no | empty | stored, translated | yes | The manually written excerpt; when empty the teaser is extracted from the content. |
| `website_message_ids` | Public comments | one_to_many to Message | no | none | filtered view of the thread | no | Messages of this post whose type is comment, that are not internal and whose subtype is not internal. |
| `create_date` | Created on | datetime | no | insertion time | stored, read only | no | Creation moment. |
| `published_date` | Publishing date | datetime | no | empty | stored | no | The moment the post was published, or a future moment for a scheduled post. |
| `post_date` | Effective publishing moment | datetime | no | none | derived and stored from the creation moment and the publishing date, writable through an inverse | no | The publishing date when set, otherwise the creation moment. Writing it writes the publishing date; clearing it falls back to the creation moment. |
| `create_uid` | Created by | many_to_one to User | no | the current user | stored, read only | no | Creator. |
| `write_date` | Last updated on | datetime | no | write time | stored, read only | no | Last change. |
| `write_uid` | Last contributor | many_to_one to User | no | the current user | stored, read only, relabelled Last Contributor | no | Last contributor. |
| `visits` | Views | integer | no | 0 | stored, read only, not copied | no | Number of views, counted once per session per post. |
| `website_id` | Site | many_to_one to Website | no | none | derived and stored from the blog, read only | no | The site the post belongs to. |
| `is_published` | Published | boolean | no | false | stored, from the publication mixin | no | Publication flag. |
| `header_visible`, `footer_visible` | Header and footer visibility | boolean | no | true | stored | yes | Page chrome visibility of the post page. |
| `cover_properties` | Cover configuration | long_text | no | the default cover record | stored | yes | Cover picture, colour class, opacity and height class. |

### Lifecycle

1. **Create.** The creation is silent in the discussion thread. When the values published the post,
   the publication notification runs.
2. **Update.** Writing the active flag as false also writes the publication flag as false. For each
   post separately: when the written values change the publication flag and carry no explicit
   publishing date, and the post either has no publishing date or its publishing date is already
   past, the publishing date is set to the current moment on publication and cleared on
   unpublication. After the write, the publication notification runs when the values published the
   post.
3. **Publication notification.** For every non-archived post concerned, a message is posted on the
   parent blog, rendered from the shipped new-post template, with the post title as subject and the
   publication subtype, so that blog followers are notified.
4. **Duplicate.** The copy's title becomes the original title followed by ` (copy)`.
5. **Access from a notification.** An external user is redirected to the public post address when the
   post is published; otherwise the ordinary record form is opened. For a published post every
   recipient group receives the access button.
6. **Inbox.** Comments are never pushed to the internal inbox; only outgoing electronic mail is used.

### Sharing metadata

The default sharing metadata sets the type to `article`, the description and the sharing description
to the subtitle, the published and modified times to the effective publishing moment and the last
change, the tag list to the tag names, the sharing image to the cover background picture extracted
from the cover record, and the title and sharing title to the post title.

## 4.8 Blog Tag

Reference page: [`blog.tag`](../../references/entities/blog.tag.md).

* Default ordering: `name`.
* Inherited mixin: Search Engine Metadata.
* Database constraint: `unique (name)`, message `Tag name already exists!`

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Tag name | text | yes | none | Tag label, translated. |
| `category_id` | Category | many_to_one to Blog Tag Category | no | none | Optional grouping, indexed. |
| `color` | Colour index | integer | no | 0 | Colour of the tag chip. |
| `post_ids` | Posts | many_to_many to Blog Post | no | empty | Posts carrying the tag. |

## 4.9 Blog Tag Category

Reference page: [`blog.tag.category`](../../references/entities/blog.tag.category.md).

* Default ordering: `name`.
* Database constraint: `unique (name)`, message `Tag category already exists!`

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Category name | text | yes | none | Category label, translated. |
| `tag_ids` | Tags | one_to_many to Blog Tag | no | none | Tags in the category. |

## 4.10 Partner Website Tag

Reference page: [`res.partner.tag`](../../references/entities/res.partner.tag.md).

The generated full name of this entity is a sentence; this specification calls it **Partner Website
Tag** throughout.

* Inherited mixin: Publication Flag, with a default publication state of published.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Tag name | text | yes | none | Tag label, translated. |
| `partner_ids` | Contacts | many_to_many to Contact | no | empty | Contacts carrying the tag. |
| `classname` | Colour class | selection: `info` = Info, `primary` = Primary, `success` = Success, `warning` = Warning, `danger` = Danger | yes | `info` | Colour class of the tag chip in the reference directory. |
| `active` | Active | boolean | no | true | Archiving. |
| `is_published` | Published | boolean | no | true | Whether the tag appears as a filter in the public reference directory. |

---

# Part 5: Storefront entities

## 5.1 Storefront fields of the Website

These fields are carried by the Website record of §1.1 and are grouped by purpose. Every one of them
appears in the settings screen listed in [configuration.md](configuration.md) §1.

### Order assignment and messages

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `salesperson_id` | Salesperson | many_to_one to User, internal users only | no | none | The salesperson assigned to orders placed on this site. Also the sender of back-in-stock messages and the recipient of feed notifications. |
| `salesteam_id` | Sales team | many_to_one to Sales Team | no | the shipped website sales team when it exists and is active | stored, indexed when set, deletion behaviour set to empty. Assigned to orders placed on this site. |
| `cart_recovery_mail_template_id` | Cart recovery template | many_to_one to Message Template restricted to the Sales Order entity | no | the shipped cart recovery template | The template used for abandoned-cart recovery messages. |
| `confirmation_email_template_id` | Order confirmation template | many_to_one to Message Template restricted to the Sales Order entity | no | the template named by the parameter `sale.default_confirmation_template` when set, otherwise the shipped order confirmation template | The template used for the confirmation message of orders placed on this site. |
| `send_abandoned_cart_email` | Send recovery messages | boolean | no | false | Whether the scheduled job sends recovery messages for this site. |
| `send_abandoned_cart_email_activation_time` | Recovery activation moment | datetime | no | computed, stored | Set to the present moment whenever the recovery flag is switched on. Carts created before this moment are never mailed. |
| `cart_abandoned_delay` | Abandoned-cart delay | decimal (hours) | no | 10.0 | How long a draft cart must remain untouched before it counts as abandoned. A value of zero is read as one hour. |

### Pricing, tax display and currency

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `show_line_subtotals_tax_selection` | Tax display mode | selection: `tax_excluded` = Tax Excluded, `tax_included` = Tax Included | no | computed, stored, editable; the computation sets `tax_excluded` whenever the company's fiscal country changes | Whether storefront prices and cart subtotals are shown with tax excluded or tax included. |
| `currency_id` | Display currency | many_to_one to Currency | no | computed, not stored | The currency of the current request's price list during a storefront request, otherwise the currency of the site's company. |
| `pricelist_ids` | Usable price lists | one_to_many to Price List | no | computed, not stored | Every price list usable on this site, computed with the site's company in context using the availability condition of §6.9. |
| `prevent_zero_price_sale` | Forbid zero-priced sales | boolean | no | false | When true, products whose computed price is zero cannot be added to the cart and their price is hidden. |

### Access and accounts

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `ecommerce_access` | Shop access | selection: `everyone` = All users, `logged_in` = Logged in users | yes | `everyone` | Whether anonymous visitors may reach shop pages. |
| `account_on_checkout` | Customer accounts | selection: `optional` = Optional, `disabled` = Disabled (buy as guest), `mandatory` = Mandatory (no guest checkout) | no | `optional` | Whether a shopper account is optional, refused or required to complete checkout. Writing it writes the sign-up policy of §1.1: free sign-up for `optional` and `mandatory`, invitation-only for `disabled`, because guest checkout and open registration are mutually exclusive. |

### Shop page presentation

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `shop_page_container` | Shop page width | selection: `regular` = Regular, `fluid` = Full-width | no | `regular` | Page width of the shop listing. |
| `shop_ppg` | Products per page | integer | no | 21 | Number of product tiles per page. A falsy value is read as 21 at render time and stored as 1 when written through the editor endpoint. |
| `shop_ppr` | Products per row | integer | no | 3 | Number of grid columns. A falsy value is read as 4 at render time. |
| `shop_gap` | Grid gap | text | no | `16px` | Grid gap expressed as a length. |
| `shop_opt_products_design_classes` | Product tile design | text | no | a fixed list of presentation tokens | Opaque presentation tokens describing the product tile: layout, thumbnail behaviour, hover behaviour, action placement, corner rounding, thumbnail aspect ratio, and the presence of the cart, wish list and comparison actions. |
| `shop_default_sort` | Default sort | selection, values in [storefront-catalogue.md](storefront-catalogue.md) §3.6 | yes | `website_sequence asc` | The order applied when the shopper has not chosen one. |
| `shop_extra_field_ids` | Product page extra fields | one_to_many to Storefront Extra Field | no | empty | The product-page extra field declarations of this site. |

### Product page presentation

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `product_page_container` | Product page width | selection: `unset` = Unset, `regular` = Regular, `fluid` = Full-width | no | `unset` | Page width; `unset` means "use the shop page width". |
| `product_page_cols_order` | Column order | selection: `regular` = Regular order, `inverse` = Inverse order | no | `regular` | Whether the media column precedes or follows the details column. |
| `product_page_image_layout` | Media layout | selection: `carousel` = Carousel, `grid` = Grid | yes | `carousel` | How media items are arranged. |
| `product_page_image_width` | Media width | selection: `none` = Hidden, `33_pc` = 33 %, `50_pc` = 50 %, `66_pc` = 66 %, `100_pc` = 100 % | yes | `50_pc` | Share of the page width taken by the media column. `none` hides media entirely and suppresses the media payload of the combination information service. |
| `product_page_image_spacing` | Media spacing | selection: `none` = None, `small` = Small, `medium` = Medium, `big` = Big | yes | `none` | Gap between grid media items. |
| `product_page_image_roundness` | Media rounding | selection: `none`, `small`, `medium`, `big`, with the same labels | yes | `none` | Corner rounding of grid media items. |
| `product_page_image_ratio` | Media ratio | selection: `auto` = Auto, `21_9` = Wider (21/9), `16_9` = Wide (16/9), `4_3` = Landscape (4/3), `6_5` = Horizontal (6/5), `1_1` = Default (1/1), `4_5` = Portrait (4/5), `2_3` = Vertical (2/3) | yes | `1_1` | Aspect ratio of media on wide screens. |
| `product_page_image_ratio_mobile` | Media ratio on narrow screens | the same selection | yes | `auto` | Aspect ratio of media on narrow screens. |
| `product_page_grid_columns` | Media grid columns | integer | no | 2 | Number of columns in the media grid. |

### Wish list page presentation

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `wishlist_opt_products_design_classes` | Wish list tile design | text | no | a fixed list of presentation tokens | Opaque presentation tokens for the wish list tiles. |
| `wishlist_grid_columns` | Wish list columns | integer | no | 5 | Columns on wide screens. |
| `wishlist_mobile_columns` | Wish list columns on narrow screens | integer | no | 2 | Meaningful values are 1 and 2. |
| `wishlist_gap` | Wish list gap | text | no | `16px` | Grid gap. |

### Cart behaviour, fulfilment and integrations

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `add_to_cart_action` | Action after adding | selection: `stay` = Stay on Product Page, `go_to_cart` = Go to cart | no | `stay` | What happens after a successful add to cart. Exposed to the storefront session payload. |
| `contact_us_button_url` | Contact button address | text, translatable | no | `/contactus` | The destination of the contact button that replaces the add-to-cart button when prices are hidden. |
| `warehouse_id` | Storefront warehouse | many_to_one to Warehouse | no | none | The warehouse whose quantity free to use drives storefront availability and which is assigned to carts of this site. Empty means every warehouse of the company counts on the shop, and the order's own rules apply at checkout. |
| `in_store_dm_id` | Collect-in-store method | many_to_one to Shipping Method | no | computed, not stored | The first published collect-in-store method that matches the site (generic or this site) and the company (generic or the site's company). |
| `newsletter_id` | Newsletter list | many_to_one to Mailing List | no | none | The list a shopper is subscribed to when they tick the newsletter box at checkout. |
| `google_places_api_key` | Address service access key | text, readable only by the administrator group | no | none | The access key of the Google Places address service used for address autocompletion. |
| `enabled_gmc_src` | Product feed enabled | boolean | no | true when the product feed group is enabled, otherwise false | Enables the feed endpoint and the feed menu. Writing the corresponding setting propagates the same value to every site. |

### Session values held for a storefront request

Three request-scoped values are resolved lazily on every storefront request and cached in the
browsing session; the session is internal state, not a published contract, so the names below are
this specification's own. Only the meaning, the lifetime and the reset triggers are binding.

| Session key | Holds | Reset by |
|---|---|---|
| `current_cart` | The identifier of the current cart, or false to record that the shopper has no cart. | The storefront session reset, payment validation, a missing or non-draft cart, a cart of another site, a cart with a pending, authorised or completed transaction. |
| `current_pricelist` | The identifier of the price list in force. | The reset, sign-in, the price list reset endpoint and the one-hour freshness rule of [calculations.md](calculations.md) §10. |
| `selected_pricelist` | The identifier of a price list the shopper explicitly selected. | The reset, sign-in, the price list reset endpoint and an address change that makes the selection unavailable. |
| `current_fiscal_position` | The identifier of the fiscal position in force. | The reset, sign-in and an address change that alters the fiscal position. |
| `cart_quantity_counter` | The number of units in the cart, used to render the header counter from a cached page. | The reset and every cart update. |
| `shop_layout_mode` | `grid` or `list`. | Toggling the list-view page option. |
| `attribute_values` | The last attribute filter selection on the shop page. | Browsing the shop page without an attribute filter. |
| `anonymous_wishlist_rows` | The identifiers of the anonymous wish list rows. | Sign-in. |
| `product_with_stock_notification_enabled` | The products for which an anonymous visitor requested a back-in-stock message. | Never inside the session. |
| `stock_notification_email` | The address the anonymous visitor last used for a back-in-stock request. | Never inside the session. |
| `last_order` | The identifier of the order being paid, kept to render the confirmation page after the cart key is cleared. | Overwritten at each checkout. |
| `last_payment_transaction` | The identifier of the last payment transaction created for the cart. | Overwritten at each transaction creation. |
| `affiliate` | The affiliate identifier read from the query string of any storefront request. The query parameter itself is the reproduced literal `affiliate_id`, spelled as the inbound marketing links already in circulation write it. | Overwritten when the query parameter is present. |
| `pending_coupon_code` | A promotional code the shopper opened before having a cart. | Applied and cleared once a cart exists. |
| `promotional_code_error`, `promotional_code_success` | The outcome message of the last promotional code attempt. | Read once, then cleared. |
| `donation_pay_values` | The amount, currency and option payload of a donation payment page. | Overwritten. |

### Operations added to the Website by the storefront

| Operation | Behaviour |
|---|---|
| Sellable-product condition | The condition that selects products sellable on this site; see §6.7. |
| Create cart, prepare cart values | The creation of a cart; see [storefront-checkout.md](storefront-checkout.md) §1.1. |
| Resolve the current price list, fiscal position and cart | The three lazy request values; see [storefront-catalogue.md](storefront-catalogue.md) §13 and [storefront-checkout.md](storefront-checkout.md) §1.2. |
| Storefront session reset | Clears the cart, cart quantity, price list, selected price list and fiscal position keys. |
| Price list availability helpers | See [storefront-catalogue.md](storefront-catalogue.md) §13.1. |
| Country from the network address | The country code resolved from the visitor's network address, or false. |
| Checkout navigation helpers | See [storefront-checkout.md](storefront-checkout.md) §5. |
| Shop access check | False when the current user is the public user and the shop access setting is `logged_in`; true otherwise. |
| Abandoned-cart job | See [storefront-checkout.md](storefront-checkout.md) §11.2. |
| Storefront quantity of a product | See [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §1.1. |
| Page geometry helpers | See [storefront-catalogue.md](storefront-catalogue.md) §7.3. |
| Canonical product address | Strips the category segment from a product page address so that one product has one canonical address. |
| Feed helpers | See §5.8. |
| Configurator helpers | See [configuration.md](configuration.md) §7. |
| Dashboard redirection | Redirects users holding the sales user group to the site dashboard instead of the generic destination. |
| Suggested pages | Adds the entries `("eCommerce", "/shop")` and `("Forum", "/forum")` to the list of pages an editor may link to. |

## 5.2 Website Product Category

Reference page:
[`product.public.category`](../../references/entities/product.public.category.md).

A storefront-facing product category, separate from the internal product category used for accounting
and costing: a product may belong to many storefront categories and to exactly one internal category.
Storefront categories drive shop navigation, the category page, the category filter, the category
content block and the feed filter.

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Category name | text, translated | yes | none | stored | yes | The label shown in navigation, breadcrumbs and filters. |
| `cover_image` | Cover picture | image | no | none | stored | yes | Used only by the category list content block. |
| `sequence` | Ordering | integer | no | derived default below | stored, indexed | yes | Display order; lower values first. |
| `parent_id` | Parent category | many_to_one to Website Product Category | no | none | stored, indexed, deletion behaviour cascade | yes | The parent in the hierarchy. Deleting a parent deletes its descendants. |
| `child_id` | Child categories | one_to_many to Website Product Category | no | empty | inverse of the parent link | no | The direct children. |
| `parent_path` | Materialised path | text | no | maintained by the platform | stored, indexed | no | The path of ancestor identifiers, slash separated and slash terminated. |
| `parents_and_self` | Ancestors and self | many_to_many to Website Product Category | no | computed | derived | no | This category and all of its ancestors, in root-to-leaf order. |
| `product_tmpl_ids` | Products | many_to_many to Product Template | no | empty | stored | yes | The products listed in this category. |
| `has_published_products` | Has published products | boolean | no | computed | derived, searchable | no | True when this category or one of its descendants contains at least one published, sellable product for the current site and company. |
| `website_description` | Category description | rich_text, translated | no | none | stored | yes | Free content shown at the top of the category page when the category is configured to show it. |
| `website_footer` | Category footer | rich_text, translated | no | none | stored | yes | Free content shown at the bottom of the category page. |
| `show_category_title` | Show the title | boolean | no | false | stored | yes | Whether the category title is printed on the shop page. |
| `show_category_description` | Show the description | boolean | no | true | stored | yes | Whether the description is printed on the shop page. |
| `align_category_content` | Centre the content | boolean | no | false | stored | yes | Whether title and description are centred on the shop page. |
| `website_id` | Site | many_to_one to Website | no | none | stored | yes | Restricts the category to one site. Empty means every site. |
| `website_meta_title`, `website_meta_description`, `website_meta_keywords`, `website_meta_og_image` | Search engine metadata | see §3.1 | no | none | stored | yes | Metadata of the category page. |
| `image_1920` and its derived sizes `image_1024`, `image_512`, `image_256`, `image_128` | Category picture | image | no | none | the largest stored, the others derived by downscaling | yes | The category picture. |

* Uniqueness: none beyond the surrogate key; two categories may carry the same name.
* Default ordering: `sequence` ascending, then `name` ascending, then `identifier` ascending.
* Display name rule: the names of the ancestors and self joined with the separator " / ". A category
  with no name yet displays the literal `New` for that segment. A category "Chairs" whose parent is
  "Furniture" displays as `Furniture / Chairs`.
* Site scoping: a category with a site is visible only on that site; the listing operations always
  apply the condition "site is empty or the current site".
* Archiving: not supported.

Derivations. The ancestors-and-self list is the categories whose identifier appears in the
materialised path, excluding the trailing empty segment; a category with no path is its own only
entry. The published-products flag is evaluated with elevated privileges, because the record rule
that hides empty categories itself reads it, and it depends on the current company and the current
site; its search condition is specified in [calculations.md](calculations.md) §25.

Default ordering value:

```formula
default ordering value = highest existing ordering value + 5
default ordering value = 10000   when no category exists
```

| Rule | Trigger | Message |
|---|---|---|
| No cycles in the hierarchy | create or write of the parent | `Error! You cannot create recursive categories.` |

Lifecycle: created by a sales administrator, a designer or the site configurator (which can generate a
starter set for an industry); changing the parent re-materialises the path of the category and of all
its descendants; deleting a category cascades to its children and removes the association rows to
products, and deletes no product.

Operations: the available-category condition (site is empty or the current site, plus the
published-products flag for users without the designer group), the grouped category list used by the
content block, the category list payload used by the block editor, and the search result address
`/shop/category/<identifier>`.

## 5.3 Product Ribbon

Reference page: [`product.ribbon`](../../references/entities/product.ribbon.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Caption | text, translated, at most 20 characters | yes | none | The ribbon caption. |
| `sequence` | Ordering | integer | no | 10 | Priority among automatically assigned ribbons; lower first. |
| `bg_color` | Background colour | text | yes | `#000000` | The ribbon background colour as a hexadecimal colour value. |
| `text_color` | Caption colour | text | yes | `#FFFFFF` | The caption colour as a hexadecimal colour value. |
| `position` | Corner | selection: `left` = Left, `right` = Right | yes | `left` | The corner of the picture on which the ribbon is drawn. |
| `style` | Shape | selection: `ribbon` = Ribbon, `tag` = Badge | yes | `ribbon` | A banner across the corner, or a small badge label. |
| `assign` | Assignment mode | selection: `manual` = Manually, `sale` = On Sale, `new` = When New, and `out_of_stock` = when out of stock when the storefront stock capability is installed | yes | `manual` | How the ribbon reaches a product. |
| `new_period` | New period | integer (days) | no | 30 | For the `new` mode, the number of days after publication during which the ribbon is shown. |

* Default ordering: `sequence` ascending, then `identifier` ascending. No site and no company
  scoping: a ribbon is global. Archiving is not supported.
* When the storefront stock capability is uninstalled, ribbons whose mode is `out_of_stock` are
  deleted.

| Rule | Trigger | Message |
|---|---|---|
| At most one ribbon per automatic mode | create or write of the mode, for every record whose mode is not `manual` | `Only one ribbon with the assign %s is allowed.` where the placeholder is the translated label of the mode. |

The reason is that automatic assignment always picks the first ribbon with a given mode, so a second
one would be unreachable.

Presentation derivation: the style class is the shape token (a ribbon that crosses the corner of the
product picture, or a flat badge) followed by a space and the corner token. The two tokens are a
presentation contract between this specification and whatever renders the storefront; a replacement
maps each token to its own styling.

Automatic applicability. A ribbon applies to a product when one of the following holds, and otherwise
does not:

1. the mode is `sale`, price data is present and at least one of: the shop-grid payload carries a
   base price strictly greater than the reduced price; the product-page payload carries a comparison
   price strictly greater than the price; the payload flag reporting a discount is true;
2. the mode is `new` and the number of whole days since the publication date is at most the new
   period;
3. the mode is `out_of_stock`, the product template forbids out-of-stock ordering and the variant is
   sold out.

Lifecycle: created from the ribbon list, from the product form or from the site editor; four ribbons
are shipped as data (see [configuration.md](configuration.md) §5.2); deleting a ribbon clears the
ribbon reference on the products that used it.

## 5.4 Product Image

Reference page: [`product.image`](../../references/entities/product.image.md).

An extra media item shown on the product page. Despite the entity name it may hold either a picture
or a reference to an external video.

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Caption | text | yes | none | stored | yes | The media caption, also used as the alternate text. |
| `sequence` | Ordering | integer | no | 10 | stored | yes | Display order inside the carousel or grid. |
| `image_1920` | Picture | image | no | none | stored | yes | The picture at full resolution; the mixin derives the four smaller sizes by downscaling. |
| `product_tmpl_id` | Product template | many_to_one to Product Template | no | none | stored, indexed, deletion behaviour cascade | yes | The template this media belongs to. |
| `product_variant_id` | Product variant | many_to_one to Product Variant | no | none | stored, indexed, deletion behaviour cascade | yes | The variant this media belongs to. |
| `video_url` | Video address | text | no | none | stored | yes | The address of an external showcase video. |
| `embed_code` | Player markup | rich_text | no | computed | derived, not stored | no | The embeddable player markup derived from the video address; empty when the address is empty or unsupported. |
| `can_image_1024_be_zoomed` | Zoomable | boolean | no | computed | derived, stored | no | True when the picture exists and is strictly larger than the 1024-pixel rendition, which enables the zoom interaction. |

* Default ordering: `sequence` ascending, then `identifier` ascending.
* A media row is attached either to a template or to a variant. With a variant it is shown only for
  that variant; with a template it is shown for every variant of the template.
* Archiving: not supported. Deleting the template or the variant deletes the media rows.
* On-change: when the editor fills the video address while the picture is still empty, the video
  thumbnail is fetched from the video service and stored as the picture, so that the carousel has a
  poster frame.

| Rule | Trigger | Message |
|---|---|---|
| The video address must be recognisable | create or write of the video address, when it is set and no player markup can be derived | `Provided video URL for '%s' is not valid. Please enter a valid video URL.` where the placeholder is the media name. The message is reproduced verbatim; the three capital letters in it abbreviate uniform resource locator, which this specification calls a web address. |

Creation rule: when media rows are created from a screen whose context carries a default template and
the row being created names a variant, the default template must not be applied to that row,
otherwise variant media would also appear as template media. Rows that do not name a variant keep the
default template, so the creation splits the incoming rows into two groups and creates each group in
its own context.

Ordering operations on the product media set: the ordered media list of a variant is the variant main
picture, then the variant extra media, then the template extra media; the ordered media list of a
template is the template main picture, then the template extra media. The resequencing operation
moves one entry first, left, right or last inside that list, then: when the main picture (the
template or variant record itself, not a media row) is no longer first, the entry that is now first
is swapped with it — the two records exchange positions, their picture payloads are exchanged and the
caption of the former main entry is copied onto the media row, while the product name is unchanged;
every media row is then renumbered with its index in the new list; and a video may never become the
main picture.

## 5.5 Base Unit Display

Reference page: [`website.base.unit`](../../references/entities/website.base.unit.md).

A named reference unit used only for the price-per-unit display on storefront pages. It is
independent from the unit of measure used for stock and invoicing. Default ordering: `name`
ascending. No scoping, no archiving.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Reference unit label | text, translated | yes | none | The label printed after the per-unit price, for example `100 g` or `750ml`. |

## 5.6 Storefront Extra Field

Reference page:
[`website.sale.extra.field`](../../references/entities/website.sale.extra.field.md).

A declaration that one field of the Product Template must be printed in the details block of the
product page of one site. Default ordering: `sequence` ascending. Deleting the field definition
deletes the declaration.

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `website_id` | Site | many_to_one to Website | no | none | stored, indexed when set | The site on whose product pages the field is printed. |
| `sequence` | Ordering | integer | no | 10 | stored | Print order. |
| `field_id` | Field | many_to_one to Field Definition | yes | none | stored, deletion behaviour cascade | The field of the Product Template to print. Only fields of that entity whose type is single-line text or binary may be chosen. |
| `label` | Printed caption | text | no | the field description | derived, read only | The printed caption. |
| `name` | Field name | text | no | the field name | derived, read only | The stored name used to read the value. |

Rendering rule: for each declaration of the current site, in ordering order, when the product has a
non-empty value, a single-line text value is printed as the caption, a colon, a space and the value,
and a binary value is printed as the caption, a colon and a download link to the field content of
that product. The whole block is omitted when no declared field has a value.

## 5.7 Website Checkout Step

Reference page:
[`website.checkout.step`](../../references/entities/website.checkout.step.md).

One step of the checkout flow. The generic rows (those without a site) are the templates; each site
receives its own copy at creation time.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Step label | text, translated | yes | none | The label shown in the progress indicator. |
| `sequence` | Ordering | integer | no | none | The order of the step in the flow. |
| `step_href` | Step path | text | yes | none | The path of the page that renders the step, for example `/shop/cart`. |
| `main_button` | Forward label | text, translated | no | none | The label of the button that moves forward from the previous step to this one. |
| `back_button` | Backward label | text, translated | no | none | The label of the link that returns to this step from the next one. |
| `website_id` | Site | many_to_one to Website | no | none | stored, deletion behaviour cascade. Empty identifies a generic template row. |
| `is_published` | Published | boolean | no | see below | Whether the step participates in the flow. Provided by the multi-site publication mixin. |

* Default ordering: by `identifier`; every navigation operation orders explicitly by `sequence`.
* A site's flow is the set of rows of that site whose publication flag is true, ordered by
  `sequence`.
* No archiving.
* Creation of the per-site copies: when a site is created, every generic step is duplicated with the
  new site. The copy is published, except the copy of the step whose path is `/shop/extra_info`,
  whose publication flag is set to the active state of the extra-information page option of that
  site.
* Navigation: the next step is the first allowed step whose ordering value is strictly greater,
  ordered ascending and limited to one; the previous step is the first allowed step whose ordering
  value is strictly smaller, ordered descending and limited to one. The allowed condition is "the
  step belongs to the current site and its publication flag is true".
* Translation propagation: when a language is installed or updated, the translations of the
  translatable fields of the generic steps are copied onto the per-site copies that share the same
  path; in overwrite mode the incoming translation wins, otherwise the existing translation of the
  copy wins, and a null value never overwrites a present one. The full rule is WS-467 in
  [business-rules.md](business-rules.md).

## 5.8 Product Feed

Reference page: [`product.feed`](../../references/entities/product.feed.md).

A published, token-protected syndication document that exposes the published catalogue of one site to
an external product syndication service. The record carries a discussion thread, so it can be
notified and followed.

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Feed label | text | yes | none | stored | yes | The feed label. |
| `website_id` | Site | many_to_one to Website | yes | none | stored | yes | The site whose catalogue is exposed. |
| `pricelist_id` | Price list | many_to_one to Price List | no | none | stored | yes | Localises prices and currency. Only price lists that are selectable and either generic or attached to the feed's site may be chosen. Empty means the site's default price list. |
| `lang_id` | Language | many_to_one to Language | yes | the site's default language, applied only while the field is empty | stored, precomputed | yes | The language used to translate names, descriptions and links. Only languages installed on the site may be chosen. |
| `website_language_ids` | Choice list | many_to_many to Language | no | the site's languages | derived | no | The choice list for the language. |
| `product_category_ids` | Categories | many_to_many to Website Product Category | no | empty | stored | yes | Restricts the feed to these categories and their descendants. Empty means the whole published catalogue of the site. |
| `target` | Target service | selection: `gmc` = Google Merchant Center | yes | `gmc` | stored | yes | The syndication service the document is formatted for. The stored value is reproduced; it names the Google Merchant Center product listing service, and the selection holds exactly this one value today. |
| `access_token` | Access token | text | yes | a newly generated 32-character hexadecimal value | stored, read only | no | The shared secret that must be presented to fetch the document. |
| `url` | Document address | text | no | computed | derived | no | The absolute address of the document, including the feed identifier and the access token. |
| `last_notification_date` | Last notification | date | no | none | stored | yes | The date of the last "too many products" notification, used to throttle it to one per week. |
| `feed_cache` | Cached document | binary | no | computed | derived, stored, read only | no | The compressed rendered document. |
| `cache_expiry` | Cache expiry | datetime | yes | the moment of creation | stored, read only | no | The moment after which the cache must be rebuilt. |

Address derivation: the document address is the site's base address joined with the fixed path
`/gmc.xml`, then the query string `?feed_id=<identifier>&access_token=<token>`. The path is a fixed
literal with no variable part: a slash, the three lower-case letters that abbreviate Google Merchant
Center, a dot, and the three lower-case letters that abbreviate extensible markup language. The two
query parameter names are reproduced exactly, because the external service stores the whole address
and replays it unchanged.

Cache derivation: any change to the site, the price list, the language or the categories invalidates
the cache by setting the expiry to one day before the present moment.

| Rule | Trigger | Message |
|---|---|---|
| Soft product limit | create or write of the categories or the site, counting the products the feed would contain, limited to the soft limit plus one | `A single feed cannot contain more than %(limit)s products. Please separate products with Categories.` where the limit is printed with thousands separators, that is `5,000`. |

A second, hard limit of 6000 products is applied when the document is rendered: only the first 6000
matching products are included.

| Operation | Behaviour |
|---|---|
| Invalidate the cache | Sets the expiry to one day before the present moment and returns a success notification carrying `Feed cache successfully reset.` |
| Render and cache | When the cache is empty or expired: an exclusive lock is taken on the record, the document is rendered, compressed and stored, the expiry is set to the start of tomorrow, and the compressed bytes are returned. Otherwise the decoded cached bytes are returned. |
| Feed condition | The site's basic feed condition (published, type goods or combo, site scope) narrowed by "the storefront categories are the selected ones or their descendants" when categories are selected. |
| Feed products | Searches the condition with the hard limit. When the number of returned products exceeds the midpoint between the soft and hard limits, that is 5500, and no notification was sent in the last week, the site salesperson is notified with the subject `GMC: Product Limit Exceeded`, reproduced verbatim (the three capital letters abbreviate Google Merchant Center), and the body `The feed %(feed_name)s contains more than %(limit)s products, which may not be fully updated. Consider refining the feed by adjusting the product categories.`, and the notification date is recorded. |

Lifecycle: created manually, or automatically when the feed feature is switched on and the site has
no feed yet — one feed for each site whose published product count is at most the soft limit, with
the shipped name `GMC 1`, reproduced verbatim; edited, each parameter change invalidating the cache;
fetched by the external service through the public endpoint, the first fetch of a day rebuilding the
cache; deleted manually. The document content is specified in
[storefront-catalogue.md](storefront-catalogue.md) §16.

## 5.9 Product Wishlist

Reference page: [`product.wishlist`](../../references/entities/product.wishlist.md).

One saved product for one shopper. A row belongs either to a contact (signed-in shopper) or to a
browsing session (anonymous shopper).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `partner_id` | Owner | many_to_one to Contact | no | none | stored, indexed when set | The owner. Empty for an anonymous session row. |
| `product_id` | Product | many_to_one to Product Variant | yes | none | stored | The saved variant. |
| `website_id` | Site | many_to_one to Website | yes | none | stored, deletion behaviour cascade | The site on which the product was saved. |
| `pricelist_id` | Price list | many_to_one to Price List | no | none | stored | The price list in force when the product was saved. |
| `currency_id` | Currency | many_to_one to Currency | no | the site currency | derived, read only | The display currency. |
| `price` | Saved price | monetary | no | none | stored | The price of the product at the moment it was saved, used to show a price drop. |
| `active` | Active | boolean | yes | true | stored | Archiving. |
| `stock_notification` | Back-in-stock subscription | boolean | yes | false | derived with an inverse | True when the owner is subscribed to the back-in-stock message of this product. Writing true subscribes the owner; writing false does not unsubscribe. |

| Constraint | Statement | Message |
|---|---|---|
| One row per product and owner | `UNIQUE(product_id, partner_id)` | `Duplicated wishlisted product for this partner.` |

Because the owner is empty for anonymous rows, the constraint does not restrain anonymous sessions in
storage engines that treat null values as distinct; the anonymous list is instead bounded by the
session.

* Default ordering: by `identifier`.
* Record rules: a portal or internal user sees only rows whose owner is their own contact; a sales
  manager sees every row. The public group has no access at all, so anonymous rows are handled with
  elevated privileges and tracked by a list of row identifiers held in the browsing session.
* Operations: read the current shopper's rows, add a row, migrate the session rows at sign-in and the
  scheduled cleanup; all four are specified in [storefront-engagement.md](storefront-engagement.md)
  §1.
* Lifecycle: created by the add-to-wish-list endpoint; deleted by the remove endpoint (an anonymous
  row only when its identifier is present in the session list), by the cleanup job, or when the site
  is deleted.

## 5.10 Product Attribute Category

Reference page:
[`product.attribute.category`](../../references/entities/product.attribute.category.md).

A grouping of product attributes used to structure the specification table on the product page and
the comparison table on the comparison page.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Section heading | text, translated | yes | none | The section heading. |
| `sequence` | Ordering | integer | no | 10 | Section order; lower first. |
| `attribute_ids` | Attributes | one_to_many to Product Attribute | no | empty | The attributes gathered in this section; when picking from the form, only attributes that have no category yet are offered. |

Default ordering: `sequence` ascending, then `identifier` ascending. No site or company scoping;
archiving is not supported. Attributes without a category are gathered into an implicit trailing
section with no heading.

## 5.11 Coupon Share Wizard

Reference page: [`coupon.share`](../../references/entities/coupon.share.md).

A transient dialogue that produces an address which applies a promotional code and lands on a chosen
page. It carries the program or coupon, the code, the target page path and the generated address; the
generated address is the site base address joined with `/coupon/<code>` and, when a landing page was
chosen, the query parameter that carries the return path. Applying the resulting address is specified
in [storefront-engagement.md](storefront-engagement.md) §5.1.

## 5.12 Product counters on the visitor records

| Entity | Identifier | Full name | Meaning |
|---|---|---|---|
| Website Visit Track | `product_id` | Product viewed | The product viewed during this visit entry; read only, indexed when set, deletion behaviour cascade. |
| Website Visitor | `visitor_product_count` | Product views | Total number of product views of this visitor. |
| Website Visitor | `product_count` | Distinct products | Number of distinct products viewed. |
| Website Visitor | `product_ids` | Products viewed | The distinct products viewed. |

All three counters are computed from one grouped read over the visit entries of the visitors,
restricted to entries that carry a product whose company is among the allowed companies. Recording a
view does nothing when the product is empty or when the variant is not a possible combination of its
template; otherwise a tracking entry is added for that product, reusing a recent entry as described in
[visitors-and-tracking.md](visitors-and-tracking.md) §2.4.

---

# Part 6: Fields added to entities owned by other folders

## 6.1 View

Reference page: [`ir.ui.view`](../../references/entities/ir.ui.view.md). Owned by
[platform foundation](../platform-foundation/README.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `website_id` | Site | many_to_one to Website | no | none | stored | Marks the template as specific to one site; empty means shared. Deletion behaviour: cascade. |
| `page_ids` | Pages | one_to_many to Website Page | no | none | inverse of the page's template link | Pages using this template as their content. |
| `controller_page_ids` | Model pages | one_to_many to Website Model Page | no | none | inverse of the model page's template link | Model pages using this template as their listing template. |
| `first_page_id` | First page | many_to_one to Website Page | no | none | derived, not stored | The first page linked to this template. |
| `track` | Track visits | boolean | no | false | stored | Serving content rendered from this template creates a visitor track. |
| `visibility` | Visibility | selection: empty = Public, `connected` = Signed In, `restricted_group` = Restricted Group, `password` = With Password | no | empty | stored | Who may open content rendered from this template as main content. |
| `visibility_password` | Visibility password | text | no | empty | stored, administrator only, not copied | The hashed password. |
| `visibility_password_display` | Visibility password shown | text | no | none | derived, writable, Editor and Designer only | `********` when a password is set, empty otherwise; writing a non-empty value stores its hash, writing an empty value clears it. Only plain templates are affected. |
| `theme_template_id` | Theme template | many_to_one to Theme Template | no | none | stored, not copied | The theme template this template was generated from. |
| Search engine metadata fields | see §3.1 | | | | | Templates carry the metadata mixin so that a page can hold its own metadata. |

The site-aware behaviour added to View — the creation guard, copy-on-write, copy-on-delete,
most-specific selection, inheritance filtering, template lookup, translation handling, content
saving, visibility enforcement and content-block saving — is specified in
[multi-site-and-languages.md](multi-site-and-languages.md) §§3–5 and
[content-management.md](content-management.md) §§1–2.

## 6.2 Asset

Reference page: [`ir.asset`](../../references/entities/ir.asset.md). Owned by
[platform foundation](../platform-foundation/README.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `key` | Asset key | text | no | none | Stable key used to pair a shared asset with its per-site copy. Not copied. |
| `website_id` | Site | many_to_one to Website | no | none | Marks the asset as specific to one site. Deletion behaviour: cascade. |
| `theme_template_id` | Theme template | many_to_one to Theme Asset | no | none | The theme template this asset was generated from. Not copied. |

## 6.3 Attachment

Reference page: [`ir.attachment`](../../references/entities/ir.attachment.md). Owned by
[platform foundation](../platform-foundation/README.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `key` | Attachment key | text | no | none | stored, not copied | Stable key of a theme attachment. |
| `website_id` | Site | many_to_one to Website | no | the current site when one is resolvable and the context does not forbid it | stored | The site the attachment belongs to. |
| `theme_template_id` | Theme template | many_to_one to Theme Attachment | no | none | stored, not copied | The theme template this attachment was generated from. |
| `local_url` | Local address | text | no | none | derived | The attachment's own address when it has one, otherwise `/web/image/<identifier>?unique=<checksum>`. |
| `image_src` | Picture source | text | no | none | derived from the media type, the address and the name | The address to use in a picture element; empty when the media type is not a supported picture type. |
| `image_width` | Picture width | integer | no | 0 | derived from the stored bytes | Pixel width, 0 when the bytes cannot be decoded as a picture. |
| `image_height` | Picture height | integer | no | 0 | derived from the stored bytes | Pixel height, 0 when the bytes cannot be decoded. |
| `original_id` | Original attachment | many_to_one to Attachment | no | none | stored, indexed when set | The unmodified, unresized attachment this one was derived from. |

Supported picture media types and their extensions: graphics interchange format `.gif`, joint
photographic experts group `.jpe`, `.jpeg` and `.jpg`, portable network graphics `.png`, scalable
vector graphics `.svg`, and web picture format `.webp`.

Picture source derivation:

1. When the attachment is of the address kind and its address starts with `/`, the source is that
   address.
2. When the attachment is of the address kind and its address is absolute, the source is
   `/web/image/<identifier>-redirect/<percent-encoded name>`.
3. Otherwise, with the marker being the first eight characters of the checksum: when the attachment
   also carries an address, the source is that address with `unique=<marker>` appended using `?` or
   `&`; otherwise the source is `/web/image/<identifier>-<marker>/<percent-encoded name>`.

Serving rule: attachments are served with the site dimension added to the lookup condition and with
the site as the first ordering key, so that a per-site attachment wins over a shared one at the same
address. The Editor and Designer group is added to the set of groups allowed to serve attachments.

## 6.4 Model Definition

Reference page: [`ir.model`](../../references/entities/ir.model.md). Owned by
[platform foundation](../platform-foundation/README.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `website_form_access` | Usable in site forms | boolean | no | false | The entity may be targeted by a public form. |
| `website_form_default_field_id` | Default free-text field | many_to_one to Field Definition | no | none | The plain text field that receives unmatched form values and request metadata. Restricted to plain text fields of this entity. |
| `website_form_label` | Form action label | text, translated | no | none | The label of the form submit action. |
| `website_form_key` | Form key | text | no | none | Key used to look up the form definition in the editor registry. |

## 6.5 Field Definition

Reference page: [`ir.model.fields`](../../references/entities/ir.model.fields.md). Owned by
[platform foundation](../platform-foundation/README.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `website_form_blacklisted` | Excluded from site forms | boolean | no | true | When true, the field may not be used in a public form. The default is true, so the mechanism is an allow list. On installation every existing unset value is set to true and the column default is set to true. |

Deletion guard: deleting a field fails when a form element bound to the field's entity contains an
input whose name is that field, in any stored rich text column where a form can survive sanitisation
(the template architecture column, plus every rich text column that is unsanitised, that allows form
elements, or that can be bypassed by the sanitisation override group). The message is:

```
The field '%(field)s' cannot be deleted because it is referenced in a website view.
Model: %(model)s
View: %(view)s
```

## 6.6 Server Action

Reference page: [`ir.actions.server`](../../references/entities/ir.actions.server.md). Owned by
[platform foundation](../platform-foundation/README.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `xml_id` | Shipped-document identifier | text | no | none | derived | The external identifier of the action when it comes from a shipped document. |
| `website_path` | Public path segment | text | no | none | stored | The path segment used to reach the action publicly. |
| `website_url` | Public address | text | no | none | derived from the kind, the publication flag, the path and the external identifier | The base address joined with `/website/action/` and the path, the external identifier or the numeric identifier, when the action is a code action and is published; empty otherwise. |
| `website_published` | Published | boolean | no | false | stored, not copied | Allows the action to be run from the site. |

Execution addition: for a code action, the request object and the structured-data helper are added to
the evaluation context, and a response placed in the evaluation context takes priority over a
returned action.

## 6.7 Product Template

Reference page: [`product.template`](../../references/entities/product.template.md). Owned by
[products and catalog](../products-and-catalog/README.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `website_description` | Storefront long content | rich_text, translated, similarity indexed | no | none | stored | yes | Long storefront content shown on the product page. |
| `description_ecommerce` | Storefront description | rich_text, translated, similarity indexed | no | none | stored | yes | Short storefront description shown under the product name. Stored empty when the supplied content is visually empty and contains neither an embedded video frame nor an embedded component. |
| `alternative_product_ids` | Alternative products | many_to_many to Product Template, company-checked | no | empty | stored | yes | Upsell suggestions shown at the bottom of the product page. |
| `accessory_product_ids` | Accessory products | many_to_many to Product Variant, company-checked | no | empty | stored | yes | Cross-sell suggestions shown in the cart. |
| `website_size_x` | Tile width | integer | no | 1 | stored | yes | Width of the product tile in grid cells. |
| `website_size_y` | Tile height | integer | no | 1 | stored | yes | Height of the product tile in grid cells. |
| `website_ribbon_id` | Ribbon | many_to_one to Product Ribbon | no | none | stored | yes | The manually assigned ribbon of the template. |
| `website_sequence` | Shop ordering | integer | no | derived default below | stored, indexed, not copied | no | Display order on the shop grid. |
| `public_categ_ids` | Storefront categories | many_to_many to Website Product Category | no | empty | stored | yes | The storefront categories the product belongs to. |
| `publish_date` | Publication date | datetime | yes | the moment of creation | derived and stored | yes | Set to the present moment each time the product becomes published. Drives the "when new" ribbon and the newest-arrivals sort. |
| `product_template_image_ids` | Extra media | one_to_many to Product Image | no | empty | stored | yes | Extra media of the template. |
| `base_unit_count` | Base unit count | decimal with unlimited precision | yes | 0 | derived from the single variant, stored, with an inverse | yes | The number of reference units contained in one sales unit. Zero hides the per-unit price. |
| `base_unit_id` | Reference unit | many_to_one to Base Unit Display | no | none | derived from the single variant, stored, with an inverse | yes | The reference unit label. |
| `base_unit_price` | Price per reference unit | monetary | no | computed, not stored | no | The sales price divided by the base unit count, or zero when the count is zero. |
| `base_unit_name` | Reference unit name | text | no | computed, not stored | no | The reference unit label when set, otherwise the name of the product's unit of measure. |
| `compare_list_price` | Comparison price | monetary | no | none | stored | yes | A strikethrough reference price shown when no price list discount applies and the comparison-price feature is enabled. |
| `variants_default_code` | Aggregated variant references | text, similarity indexed | no | computed, stored | no | The internal references of every variant joined by a rarely used separator character, so that the storefront search finds a product by any of its variant references. |
| `allow_out_of_stock_order` | Sell when out of stock | boolean | no | true | stored | yes | Whether the product may be ordered when no quantity is available. |
| `available_threshold` | Availability threshold | decimal | no | 5.0 | stored | yes | The quantity at or below which the remaining quantity is displayed. |
| `show_availability` | Show the remaining quantity | boolean | no | false | stored | yes | Whether the remaining quantity is displayed at all. |
| `out_of_stock_message` | Out-of-stock message | rich_text, translated | no | none | stored | yes | Shown in place of the availability when the product is sold out. |
| `is_published` | Published | boolean | no | false | stored | no | Publication flag from the multi-site publication mixin, also readable as the publication switch. |
| `website_id` | Site | many_to_one to Website | no | none | stored | yes | Restricts the product to one site. Empty means every site. |
| `website_url` | Public address | text | no | computed | derived | no | `/shop/<slug of the product>`. |
| `description`, `description_sale` | Internal and sales descriptions | rich_text and long_text | no | none | stored, similarity indexed | yes | Indexed for the storefront approximate search. |

Similarity indexes are created on the translatable fields `name`, `description`, `description_sale`
and `description_ecommerce` and on the internal reference, using an accent-insensitive expression
where the storage engine supports it. Their purpose is the approximate matching used by the
storefront search.

Derivations:

```formula
base unit count = the base unit count of the single variant, for a template with exactly one variant
base unit count = 0                                        for every other template
reference unit  = the reference unit of the single variant, for a template with exactly one variant
reference unit  = empty                                    for every other template
price per reference unit = sales price ÷ base unit count
price per reference unit = 0                               when the base unit count is 0
publication date = the present moment, for every template whose publication flag becomes true
```

Their inverses write the value back onto the single variant; templates with several variants ignore
the write.

Default shop ordering value:

```formula
default shop ordering value = highest existing shop ordering value + 5
default shop ordering value = 10000     when no product exists
```

When the storefront capability is installed on a database that already holds products, every existing
product receives a distinct ordering value computed as the highest value plus five times the row
index, written row by row, and the aggregated variant reference column is filled with one aggregate
statement rather than by recomputation, so that installation on a large catalogue stays bounded.

**The site sellable-product condition.** Used by every storefront listing, by the search, by the
feed, by accessory and alternative filtering and by the add-to-cart guard:

```formula
sellable on this site =
      the product may be sold
  AND ( the product's site is empty OR the product's site is the current site )
  AND ( the product's company is empty OR the product's company is the current site's company )
  AND ( the current user is internal
        OR ( the publication flag is true
             AND the service tracking value is one of the sellable tracking values ) )
```

Shop ordering operations: move to the top sets the ordering value to the smallest existing value
minus five; move to the bottom sets it to the largest existing value plus five; move up finds the
product with the greatest ordering value strictly below this one **and the same publication state**
and swaps the two values, or moves to the top when there is none; move down does the symmetric
operation.

Add-to-cart eligibility:

```formula
may be added to the cart = the template matches the site sellable-product condition
add to cart is possible  = the template is active
                           AND it may be added to the cart
                           AND at least one possible combination exists under the parent combination
quick add is offered     = the template matches the site product condition
                           AND ( the site allows zero-priced sales OR the contextual price ≠ 0 )
                           AND the template is not sold out
```

Media, picture holder and ribbon: the media list of a template is the template itself followed by its
extra media; the picture holder is the template when it carries a picture, otherwise the first
possible variant when that variant carries one, otherwise the template; the suitable picture size is
the 512-pixel rendition for a one-by-one tile on a grid of three columns or more and the 1024-pixel
rendition otherwise; the ribbon is the variant ribbon when set, otherwise the template ribbon when
set, otherwise the first automatic ribbon in ordering order whose applicability test passes.

Attribute display helpers: whether the template has attributes that create no variant; whether it has
attribute values accepting a free-text custom value; the possible variants ordered by attribute and
then by attribute value; the previewed attribute values of a product card (the first attribute line
whose attribute declares a preview mode other than hidden, its active values that actually have
variants, at most twenty of them with the address of the 512-pixel picture of the lowest-numbered
matching variant and the product page address carrying that value as a filter, plus the count of the
values beyond the twentieth); the informative single-value attribute lines; the same including
multiple-choice attributes; the attribute lines grouped by attribute category; and the same with the
single-value custom attributes removed.

Other operations: the accessory variants and the alternative templates filtered by the sellable
condition (and additionally by the publication flag for non-internal users); the contextual price
list falling back to the request price list; the product address builder of
[storefront-catalogue.md](storefront-catalogue.md) §1.2; the redirection of an external user to the
storefront page instead of the back-office form; the default sharing metadata (product name, sales
description, 1024-pixel picture); the restriction of the average rating and rating count to
non-internal ratings; the list of service tracking values exempt from the zero-price rule (empty by
default, extended with the course tracking value by the course capability); and the rule that an
external user may post a product review only when the review page option is active.

| Rule | Trigger | Message |
|---|---|---|
| Print images before publication | write of the publication flag for a product linked to a print-on-demand template that still lacks print images | `Print images must be set on products before they can be published.` |

## 6.8 Product Variant

Reference page: [`product.product`](../../references/entities/product.product.md). Owned by
[products and catalog](../products-and-catalog/README.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `variant_ribbon_id` | Variant ribbon | many_to_one to Product Ribbon | no | none | stored | The ribbon of this variant, which wins over the template ribbon. |
| `website_id` | Site | many_to_one to Website | no | the template's site | derived, writable | Site restriction, shared with the template. |
| `product_variant_image_ids` | Variant media | one_to_many to Product Image | no | empty | stored | Extra media specific to this variant. |
| `base_unit_count` | Base unit count | decimal with unlimited precision | yes | 1 | stored | Number of reference units in one sales unit of this variant. |
| `base_unit_id` | Reference unit | many_to_one to Base Unit Display | no | none | stored | The reference unit label of this variant. |
| `base_unit_price` | Price per reference unit | monetary | no | computed, not stored | The variant sales price divided by the base unit count; zero for an unsaved record. |
| `base_unit_name` | Reference unit name | text | no | computed, not stored | The reference unit label or the unit of measure name. |
| `website_url` | Public address | text | no | computed, not stored | The template address, with the query parameter `attribute_values` carrying the comma-separated attribute value identifiers when the variant carries attribute values. Depends on the display language. |
| `stock_notification_partner_ids` | Back-in-stock subscribers | many_to_many to Contact | no | empty | stored | The contacts to notify when the variant is back in stock. |
| `channel_ids` | Courses | one_to_many to Course | no | empty | inverse link | The courses whose paid enrolment is granted by buying this variant. |

| Rule | Trigger | Message |
|---|---|---|
| Base unit count not negative | create or write of the base unit count | `The value of Base Unit Count must be greater than 0. Use 0 to hide the price per unit on this product.` |

The check refuses values strictly below zero; zero itself is accepted and hides the per-unit price.

Write rule: archiving a variant first deletes every draft Sales Order Line that references it and
belongs to an order that has a site, which removes archived products from open carts.

On-change: on the simplified product creation form, selecting at least one storefront category
publishes the product and clearing the selection unpublishes it.

Operations: the media list (the variant, its own extra media, then the template extra media, the
variant entry falling back to the template picture); the combination information service for this
variant; the add-to-cart permission check (true for an administrator; false when the variant is
archived, unpublished, outside the sellable condition, or zero-priced while the site forbids it; then
the shop-access check); whether the variant is in the current shopper's wish list; whether a contact
is subscribed to its back-in-stock list; whether it may be subscribed to (active, sellable,
published); whether it is sold out; its maximum orderable quantity; the comparison matrix builder;
the structured description for search engines; the absolute media addresses used by the feed and the
comparison page; and the delegation of publication and navigation to the template with the variant
address.

## 6.9 Price List and Price List Rule

Reference pages: [`product.pricelist`](../../references/entities/product.pricelist.md) and
[`product.pricelist.item`](../../references/entities/product.pricelist.item.md). Owned by
[pricing and price lists](../pricing-and-pricelists/README.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `website_id` | Site | many_to_one to Website | no | the first site of the current company, or of the company named in the creation context | Attaches the price list to one site. Tracked; deletion behaviour restrict. |
| `code` | Promotional code | text, readable only by internal users | no | none | The code that activates the price list on the storefront. |
| `selectable` | Selectable | boolean | no | false | Whether the shopper may pick this price list from the storefront selector. |

| Rule | Trigger | Message |
|---|---|---|
| The site must belong to the price list company | create or write of the company or the site, for records that have both | `Only the company's websites are allowed.\nLeave the Company field empty or select a website from that company.` |

Availability:

```formula
available on a site =
      ( the price list company is empty OR it is the site's company )
  AND ( ( the price list is active AND its site is that site )
        OR ( its site is empty AND ( it is selectable OR it carries a code ) ) )

available in a country =
      true                                   when no country code is known
   OR true                                   when the price list has no country group
   OR the country code belongs to one of the price list's country groups
```

A price list with no site, not selectable and without a code is a back-office price list and never
reaches the storefront.

Cache invalidation: creating, writing or deleting any price list clears the memoised storefront
resolution of [calculations.md](calculations.md) §10.

Contact resolution hook: during a storefront request the search for the contact's applicable price
list is narrowed by the site availability condition, and the resulting candidates are filtered by the
availability test.

Price list rule addition — whether the shop shows a strikethrough base price:

```formula
show a discount on the shop =
      a rule was applied
  AND ( the rule computes a percentage
        OR ( the rule computes a formula
             AND its discount is not zero
             AND its base is the sales price or another price list ) )
```

This is deliberately broader than the sales rule: on the catalogue and the configurator, formula
rules also show a discount; on the cart and the checkout, they do not.

## 6.10 Product Attribute, attribute line and attribute value

Reference pages:
[`product.attribute`](../../references/entities/product.attribute.md),
[`product.template.attribute.line`](../../references/entities/product.template.attribute.line.md)
and
[`product.template.attribute.value`](../../references/entities/product.template.attribute.value.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `visibility` | Storefront visibility | selection: `visible` = Visible, `hidden` = Hidden | no | `visible` | Whether the attribute appears in the shop filter panel. |
| `preview_variants` | Preview on the product card | selection: `visible` = Visible, `hidden` = Hidden, `hover` = Hover | no | `hidden` | Whether the attribute's values are previewed on the product card, always or on hover. |
| `is_thumbnail_visible` | Preview with pictures | boolean | no | false | Whether the preview uses variant pictures instead of the attribute value swatches. |
| `category_id` | Attribute category | many_to_one to Product Attribute Category | no | none | The section in which the attribute is grouped on the specification and comparison tables. |

On-change: when the attribute's variant creation mode is not "always", or its display type is
multiple-choice, the preview mode is forced to hidden and the picture preview to false, because a
preview can only link to an already existing single variant.

Attribute value addition — the storefront extra price of one value:

```formula
extra price shown = 0                      when the value carries no extra price
extra price shown = 0                      when the payload reports that extra prices are hidden
extra price shown = the extra price, converted from the template currency to the payload currency
                    at the payload date, then passed through the tax treatment of the payload
```

## 6.11 Sales Order used as a cart

Reference page: [`sale.order`](../../references/entities/sale.order.md). Owned by
[sales](../sales/README.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `website_id` | Site | many_to_one to Website | no | none | stored, read only, not copied | The site through which the order was placed. Empty for back-office orders. |
| `cart_recovery_email_sent` | Recovery message sent | boolean | no | false | stored, not copied | True once a recovery message has been sent, or once the order has been ruled out for recovery. |
| `shop_warning` | Storefront warning | text | no | none | stored, not copied | A one-shot message shown to the shopper on the next storefront page, then cleared. |
| `website_order_line` | Displayed cart lines | one_to_many to Sales Order Line | no | computed, not stored | The lines that must appear in the cart display. Never used for amount computation. |
| `amount_delivery` | Delivery amount | monetary | no | computed, not stored | The total of the delivery lines, tax excluded or tax included according to the site tax display setting. |
| `cart_quantity` | Cart quantity | integer | no | computed, not stored | The sum of the displayed line quantities, truncated to a whole number. |
| `only_services` | Services only | boolean | no | computed, not stored | True when every displayed line carries a service product, and for an empty cart. |
| `is_abandoned_cart` | Abandoned cart | boolean | no | computed and searchable, not stored | True when the order is an abandoned cart. |
| `disabled_auto_rewards` | Disabled rewards | many_to_many to Loyalty Reward | no | empty | stored | Rewards the shopper removed from the cart, which must not be applied again automatically. |
| `pickup_location_data` | Pickup location | structured_data | no | empty | stored | The chosen pickup point payload. |

Computations:

```formula
delivery amount = sum of the subtotals of the delivery lines   when the site shows prices tax excluded
delivery amount = sum of the totals of the delivery lines      when the site shows prices tax included
delivery amount = 0                                            for an order without a site

cart quantity   = truncate to a whole number of the sum of the displayed line quantities
                  − the quantities of reward lines, when promotions are installed
services only   = every displayed line carries a service product
```

The displayed cart lines are every line whose line-level display rule is true; with promotions
installed, the discount lines generated by one program for several tax groups are replaced in the
display by a single unsaved aggregate line carrying the sum of the unit prices, the sum of the
subtotals and the sum of the totals, no taxes, quantity one and the short name of the first line.

Abandoned-cart detection:

```formula
abandoned cart =
      the order has a site
  AND the order state is draft
  AND the order date is set
  AND the order date ≤ the present moment − ( the site's abandoned-cart delay, or 1 hour when it is zero )
  AND the order contact is not the site's public contact
  AND the order has at least one line
```

The searchable form of the same rule, used by the abandoned-cart list and the scheduled job, reads
every site's delay and public contact and builds the union over sites of the same conjunction. Only
the operator "equals true" and its negation are supported.

Computations this folder overrides for orders that carry a site:

| Field | Behaviour for site orders |
|---|---|
| Signature required | Always false. |
| Payment terms | After the standard computation, an order that still has no payment terms receives the immediate-payment term when it exists and belongs to the order's company or to no company, otherwise the first payment term of the order's company. |
| Price list | When a country code is resolved from the visitor's network address, the standard computation runs with that country code in context, which lets country-group price lists win even when the contact has no address. |
| Salesperson | The standard assignment is skipped: a draft cart is left without a salesperson to avoid notifications. On confirmation, and when sending the payment-succeeded message, the assignment is forced and resolves to the site salesperson, else the contact's salesperson, else the parent contact's salesperson. |
| Sales team | Falls back to the site's sales team when the standard default yields nothing. |
| Warehouse | The site warehouse when set, otherwise the standard computation, and, when that still yields nothing, the current user's default warehouse. With collection in store, an order that already carries a pickup location keeps the warehouse of that location. |
| Fiscal position | With collection in store, an order with a pickup location takes the fiscal position resolved for the customer with the store address as delivery address. |
| Delivery address | With the external parcel-shop network, a delivery address that is a relay point is reset to the customer's own address when the chosen method is not that network's method. |
| Language | For a storefront request the request language wins over the contact language. |
| Note base address | The base address of the site in context, when a site is in context. |

Creation rule: when a cart is created with a site, a supplied company that differs from the site's
company is refused with `The company of the website you are trying to sell from
(%(website_company)s) is different than the one you want to use (%(company)s)`; when no company is
supplied, the site's company is written on the order.

Cart operations are specified in [storefront-checkout.md](storefront-checkout.md): add to the cart,
update a line quantity, find a matching line, create a line, prepare line values, verify the updated
quantity, equalise combo quantities, verify the cart after an update, clean the cart, compute
accessory suggestions, update an address, set and remove the delivery method, list delivery methods,
resolve the preferred method, set and list pickup locations, report an anonymous cart, report whether
a customer address is needed, report deliverable products, list zero-priced lines, report readiness,
check readiness before payment, recompute the cart, report express-checkout eligibility, compute the
amount excluding delivery, read and clear the storefront warning, report reorder eligibility, send
and filter recovery messages and resolve the recovery template.

Message and notification behaviour: after a mass mailing sent with the recovery flag, every order of
the batch that is still an abandoned cart and has not yet been mailed is marked as mailed; after a
single message posted with the recovery flag, the order is marked as mailed; the portal access button
of a recovery notification is relabelled `Resume Order` and points at the cart address carrying the
order identifier and the order access token; and the order preview action of a site order returns the
portal address prefixed with `/@`, which makes it open outside the back office.

## 6.12 Sales Order Line

Reference page: [`sale.order.line`](../../references/entities/sale.order.line.md). Owned by
[sales](../sales/README.md).

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `name_short` | Short name | text | no | computed, not stored | The product display name without the internal reference, used where space is scarce. |
| `shop_warning` | Line warning | text | no | none | stored | A one-shot warning shown on the cart line, then cleared. |

| Operation | Behaviour |
|---|---|
| Pricing date | For a draft line of a site order, the present moment instead of the order creation date, so that cart prices always use the current price list validity. |
| Shown in the cart | True when the line is not a delivery line, carries no display type (it is not a section or a note) and is not a combo item line. With promotions, discount reward lines are also excluded. |
| Sellable | True when the line's product is published and the line is not a delivery line. With promotions, reward lines are sellable only when the reward grants a free product. |
| Reorder allowed | True when the line has a product, the product may be added to the cart and the line is shown in the cart. With promotions, reward lines are excluded; with courses, course lines are excluded. |
| Displayed unit price | The unit price of the line, taken as the combo display price for a combo line, run through the line taxes for quantity one, and returned tax excluded or tax included according to the site setting. |
| Displayed quantity | The demanded quantity rounded to the product unit precision, returned as a whole number when it is integral. |
| Cart display price | The sum of the subtotals (tax-excluded display) or of the totals (tax-included display) over the line and its priced linked lines. |
| Strikethrough shown | True when the line has a discount, is sellable and its displayed unit price is not zero. |
| Line header | The product name when the line carries attribute values, otherwise the short name. With promotions, a reward line shows its own description. |
| Combination name | The combination name of the line's variant. |
| Description following lines | Every line of the line description after the first. |
| Selected combo items | For a combo line, the list of combo item, no-variant attribute values and custom values of its linked lines. |
| Set the stock warning | Writes and returns `You ask for %(desired_qty)s %(product_name)s but only %(new_qty)s is available`. |
| Maximum quantities and availability check | See [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §4. |
| Validity check | Raises `The given product does not have a price therefore it cannot be added to cart.` when the line is not a combo item, the sum of the unit prices of the line and its priced linked lines is zero, the site forbids zero-priced sales and the product's service tracking value is not exempt. |

## 6.13 Shipping Method and Warehouse

Reference pages: [`delivery.carrier`](../../references/entities/delivery.carrier.md) and
[`stock.warehouse`](../../references/entities/stock.warehouse.md). Owned by
[delivery and shipping](../delivery-and-shipping/README.md) and
[inventory operations](../inventory-operations/README.md).

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `is_published` | Published | boolean | no | false | Whether the method is offered on the storefront. Provided by the multi-site publication mixin, which also adds the site field. |
| `website_description` | Storefront description | long_text | no | the sales description of the delivery product | The description shown next to the method at checkout. |
| `warehouse_ids` | Stores | many_to_many to Warehouse | no | empty | For the collect-in-store type, the stores at which the order may be collected. |
| `delivery_type` | Integration kind | selection, extended with `in_store` = Pick up in store | yes | inherited | When the capability is uninstalled, methods with this value revert to the default kind. |
| `opening_hours_id` (Warehouse) | Opening hours | many_to_one to Working Schedule, company-checked | no | none | The schedule printed in the store selector. |

| Rule | Trigger | Message |
|---|---|---|
| A published collect-in-store method needs at least one store | create or write of the kind, the publication flag or the stores | `The delivery method must have at least one warehouse to be published.` |
| Stores must share the method's company | create or write of the kind, the company or the stores, when both the method and a store have a company | `The delivery method and a warehouse must share the same company` |

Creation and write rules for the collect-in-store kind: the integration level is forced to rate only,
cash on delivery is disabled, and the country, state and postal-code restrictions are cleared. On
creation the method additionally receives every warehouse of its company (the company of its delivery
product, or the current company when no company is given) and is published when at least one
warehouse was found.

Operations added: the rate of a collect-in-store method is the sales price of its delivery product,
with no error and no warning; and the close-locations operation geolocates the reference address,
builds the payload of each store, skips stores whose payload cannot be built, attaches the stock data
(per product when called from a product page, per cart when called from checkout) and the distance,
and returns the payloads sorted by increasing distance. The store payload and its opening hours are
specified in [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §6.2.

## 6.14 Payment Provider, Payment Token, Payment Transaction and Payment

Reference pages: [`payment.provider`](../../references/entities/payment.provider.md),
[`payment.token`](../../references/entities/payment.token.md),
[`payment.transaction`](../../references/entities/payment.transaction.md) and
[`account.payment`](../../references/entities/account.payment.md). Owned by
[payment providers](../payment-providers/README.md) and
[payments and bank reconciliation](../payments-and-bank-reconciliation/README.md).

| Entity | Addition | Behaviour |
|---|---|---|
| Payment Provider | `website_id` (many_to_one to Website, company-checked, not copied, deletion behaviour restrict) | Restricts the provider to one site. The compatible-provider filter keeps only providers with no site or with the requested site; the excluded ones are reported as unavailable with the reason "incompatible website". |
| Payment Provider | `custom_mode` extended with `on_site` = Pay on site | The pay-on-site mode; its default payment method code set is the single code `pay_on_site`. |
| Payment Provider | compatible-provider filter, with collection in store | Pay-on-site providers are removed unless the order's method is of the collect-in-store kind and the order contains at least one goods line; the excluded providers are reported with the reason `no in-store delivery methods available`. |
| Payment Provider | base address | The root address of the current request wins over the configured base address, converted to its plain seven-bit form: a host name written in a non-Latin script, or with diacritics or ligatures, is transcribed with the ascii-compatible encoding that the domain name system uses, and every remaining character outside the plain Latin set is percent-encoded. External payment services accept only that form, and using the request root rather than the configured base keeps multi-site redirections correct. |
| Payment Provider | duplication | A copy keeps the source site when the copy's company is the source company or one of its parents and the caller did not request another site. |
| Payment Token | available-token filter | During express checkout no stored token is offered. |
| Payment Transaction | `is_donation` (boolean) | Marks a transaction created from the donation page. On completion it triggers the donation message and logs the donor details on the payment. |
| Payment Transaction | post-processing with collection in store | Pending transactions of a pay-on-site provider confirm their draft orders and request the confirmation message, which creates the delivery work. |
| Payment | `is_donation` | Related to the transaction flag, read only. |

## 6.15 User, Contact, Company, Language and the portal access wizard

Reference pages: [`res.users`](../../references/entities/res.users.md),
[`res.partner`](../../references/entities/res.partner.md),
[`res.company`](../../references/entities/res.company.md),
[`res.lang`](../../references/entities/res.lang.md) and
[`portal.wizard.user`](../../references/entities/portal.wizard.user.md).

### User

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `website_id` | Site | many_to_one to Website | no | the contact's site | derived and stored, writable with the caller's own rights | The site the account is bound to; empty means the account works on every site. |

* Database constraint: `unique (login, website_id)`, message
  `You can not have two users with the same login!`
* Additional check: two accounts with the same login and no site are refused with the same message;
  the check uses an explicit query, because the unique constraint does not constrain rows whose site
  is empty.
* Sign-in scoping: the login lookup condition, the address lookup condition and the login ordering
  are extended with the current site dimension, so a site-specific account wins over a shared one.
* Sign-up: a self-created account receives the current site's company as its company and allowed
  company, and receives the current site when that site requires site-specific accounts. The sign-up
  invitation scope of the current site overrides the global one.
* Internal-user guard: an account bound to a site may not become an internal user; the message is
  `Remove website on related partner before they become internal user.`
* Authentication side effects: the visitor merge of [visitors-and-tracking.md](visitors-and-tracking.md)
  §3 and the wish list migration of [storefront-engagement.md](storefront-engagement.md) §1.5.
* Public profile: the reputation score becomes readable by the account itself; the country, the city,
  the personal site, the public description and the publication flag become writable by the account
  itself.
* Forum helper: the redirection list offered after a reputation change gains the entry
  `See our Forum` pointing at `/forum`.

### Contact

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `visitor_ids` | Visitors | one_to_many to Website Visitor | no | none | Browsing identities linked to this contact. |
| `website_description` | Public description | rich_text | no | empty | Long public description shown on the contact's public page. Inline styles are stripped; the sanitisation override group may store raw markup; translated with term translation. |
| `website_short_description` | Short public description | long_text | no | empty | Shown in the reference directory, translated. |
| `is_published` | Published | boolean | no | false | Whether the contact has a public page. Changes are tracked in the discussion thread. |
| `website_tag_ids` | Public tags | many_to_many to Partner Website Tag | no | empty | Labels used to filter the public customer reference directory. |
| `wishlist_ids` | Wish list | one_to_many to Product Wishlist | no | empty | The saved products of this contact, restricted to active rows. |

The public address of a contact is `/partners/<slug of the contact>`. Publishing or unpublishing a
contact posts a tracked message with the subtype Partner published or Partner unpublished. Two map
helpers are added: a static map picture address built from the street, city, postal code and country
with the configured mapping key, and an external map link built from the same address parts.

Storefront operations added to Contact: the storefront-writable field set gains every field declared
as writable by the site form definition of the Contact entity; the current-shopper resolution returns
the order's contact only when the cart is not anonymous, so that the public contact is never treated
as the shopper; the fiscal-position recomputation condition selects draft site orders whose customer
or delivery address is among the written contacts; writing the country, the tax identification number
or the postal code recomputes the fiscal position of those orders, then the taxes of the orders whose
fiscal position changed and then the prices of the draft ones; and changing the assigned price list
of a contact who has an open draft site cart shows a warning titled `Open Sale Orders` with the body
`This partner has an open cart. Please note that the pricelist will not be updated on that cart. Also, the cart might not be visible for the customer until you update the pricelist of that cart.`

### Company

| Identifier | Full name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|---|
| `website_id` | Site | many_to_one to Website | no | none | derived and stored | The first site whose company is this company, in site ordering order. |

Archive guard message:

```
The company “%(company_name)s” cannot be archived because it has a linked website “%(website_name)s”.
Change that website's company first.
```

The default price list values produced when a company is created, or when the price list feature is
switched on, are forced to carry no site, which prevents the current company's site from being
attached to every company's price list.

### Language

No stored field is added. Two behaviours are added: deactivating a language listed in the languages of
any site fails with `Cannot deactivate a language that is currently used on a website.`; and during a
frontend request the available languages are the languages of the current site sorted by name, each
enriched with an alternate-language code computed as described in
[multi-site-and-languages.md](multi-site-and-languages.md) §6, while outside a frontend request, or
when the caller explicitly asks for the installed language list, the platform list is used.

The language installation wizard gains a site list, defaulting to the site passed in the action
parameters; after the installation the selected languages are added to the languages of each selected
site, and, when the action parameters carry a return address, the placeholder `[lang]` inside it is
replaced by the code of the first installed language and the site editor is opened at that address.

### Portal access wizard line

No stored field is added. The duplicate-account detection is made site-aware: the search for accounts
with the same address is restricted to the sites of the portal users being granted access (their own
site, or, for a portal user with no site, both the empty site and the current site), the site is read
along with the other account fields, and two accounts are considered the same person only when their
sites are compatible.

## 6.16 Tracked Link

Reference pages: [`link.tracker`](../../references/entities/link.tracker.md),
[`link.tracker.code`](../../references/entities/link.tracker.code.md) and
[`link.tracker.click`](../../references/entities/link.tracker.click.md). Owned by
[marketing and mass mailing](../marketing-and-mass-mailing/README.md).

This folder adds the site-aware serving of short addresses:

* The host of a short address is the base address of the current site when the current site is the
  site of the acting company, otherwise the base address of that company, joined with `/r/`. Outside
  a frontend request the platform base address is used.
* The statistics action of a tracked link opens the short address followed by `+`, which is the
  statistics page of §7 of [interfaces.md](interfaces.md).
* The public redirection endpoint records one click — network address and country resolved from it —
  unless the caller is identified as a crawler, and then answers a permanent redirect to the target
  address, which may leave the site.

The uniqueness rule of a tracked link (target address, campaign, medium, source and label) and the
generation of a code are owned by the marketing folder and reproduced in
[calculations.md](calculations.md) §24 because the site is where codes are created interactively.

## 6.17 Other extended entities

| Entity | Addition | Behaviour |
|---|---|---|
| Product Document ([`product.document`](../../references/entities/product.document.md)) | `shown_on_product_page` (boolean, default false) | Whether the document is downloadable from the storefront product page. A published document may not target a single variant: `Documents shown on product page cannot be restricted to a specific variant`. A print image may not be emptied while its product is published: `Products must be unpublished before print images can be removed.` |
| Product Tag ([`product.tag`](../../references/entities/product.tag.md)) | the multi-site mixin | A tag is offered as a storefront filter only when it is marked visible to customers, is attached to at least one published template or variant, and matches the site condition. |
| Sales Team ([`crm.team`](../../references/entities/crm.team.md)) | `website_ids`, `abandoned_carts_count`, `abandoned_carts_amount` | The sites assigned to the team, and the number and summed total of the team's abandoned carts that have not been mailed, computed with a single grouped read; teams without a site report zero. The counter action opens the order list restricted to abandoned carts of that team with the recovery filter preselected and creation disabled. |
| Sales Analysis Report ([`sale.report`](../../references/entities/sale.report.md)) | `website_id`, `is_abandoned_cart`, `public_categ_ids` | The site of the source order, the abandoned-cart flag computed inside the report query, and the storefront categories of the product as a grouping axis. The query joins the site table on the order's site and groups additionally by the site and by the site's abandoned delay. |
| Journal Entry ([`account.move`](../../references/entities/account.move.md)) | `website_id` (computed, stored, read only, tracked) | The site of the single source order of the invoice lines; empty when the lines come from orders of several sites or from no order. The column is created directly when the storefront capability is installed, without recomputing history. The invoice preview action of a site invoice returns the portal address prefixed with `/@`. |
| Transfer ([`stock.picking`](../../references/entities/stock.picking.md)) | `website_id` (related to the source order's site, stored, read only) | The site through which the order that generated this transfer was placed. |
| Course ([`slide.channel`](../../references/entities/slide.channel.md)) | `enroll` extended with `payment` = On payment, `product_id`, `product_sale_revenues`, `currency_id` | Paid enrolment, the product whose purchase grants it, the summed line total of completed sales of that product and its currency. Constraint `CHECK( enroll != 'payment' OR product_id IS NOT NULL )` with message `Product is required for on payment channels.` Publishing a paid course publishes its product; unpublishing a course unpublishes its product unless another published course still uses it. |
| Loyalty Program and Loyalty Rule ([`loyalty.program`](../../references/entities/loyalty.program.md), [`loyalty.rule`](../../references/entities/loyalty.rule.md)) | `available_on_website` (default true), `website_id`, `show_non_published_product_warning`, and the site of a rule | Whether the program applies to storefront orders, the site restriction, the warning raised for an electronic wallet whose trigger products are not all published, and the site used by the code uniqueness rule: `The promo code must be unique.` and `A coupon with the same code was found.` Two programs may share a code only when they can never be reached from the same site. |
| Product Combo ([`product.combo`](../../references/entities/product.combo.md)) | maximum sellable quantity | With the storefront stock capability, the largest maximum quantity among the items of the combo, or unknown when any item has an unknown maximum; see [storefront-stock-and-pickup.md](storefront-stock-and-pickup.md) §2.2. |
| Digest Email ([`digest.digest`](../../references/entities/digest.digest.md)) | `kpi_website_sale_total` and its value | Whether the periodic digest includes the online sales measure, and the summed line subtotal of the sales analysis rows of the period whose state is not draft, cancelled or sent and whose site is set, per company. Reading it without the all-documents sales group raises an access error whose message is `Do not have access, skip this data for user's digest email`, which makes the digest skip the measure for that recipient. The measure's action link opens the site dashboard. |
| Badge ([`gamification.badge`](../../references/entities/gamification.badge.md)) | the publication mixin | A badge can be shown or hidden on public profiles. |
| Configuration Settings ([`res.config.settings`](../../references/entities/res.config.settings.md)) | every site and storefront setting | Listed in [configuration.md](configuration.md) §1. |
| Request routing | site resolution and storefront request values | Site resolution, language handling and the tracking hook of [multi-site-and-languages.md](multi-site-and-languages.md) §1 and [visitors-and-tracking.md](visitors-and-tracking.md) §2; plus, on every storefront request, the three lazy values of §5.1 and the reading of the reproduced query parameter `affiliate_id` into the session. |
| Theme Utilities | category-style exclusivity | Enabling one of the six category-style page options first disables the other five, because they are mutually exclusive presentations of the same category strip; the storefront footer template is prepended to the list of footer templates a theme may enable. |
| Capability package ([`ir.module.module`](../../references/entities/ir.module.module.md)) | `image_ids`, `is_installed_on_current_website`, and the checkout-step translation propagation | The theme preview pictures, whether the theme is the one applied to the current site, and rule WS-467 of [business-rules.md](business-rules.md). |

---

# Part 7: Naming choices

The names below differ from a mechanical expansion of the stored field name; they are used
consistently in this folder, and every table gives the stored identifier beside them.

| Stored identifier | Name used here | Reason |
|---|---|---|
| `public_categ_ids` | storefront categories | The mechanical plural of the stored name is not a word. |
| `product_category_ids` on Product Feed | feed categories | Same reason. |
| `child_id` on Website Product Category and on Website Menu | child categories, child entries | The stored name is singular for a list. |
| `contact_us_button_url` | contact button address | The mechanical expansion turns a pronoun into a country name. |
| `in_store_dm_id` | collect-in-store method | The mechanical expansion turns a preposition into a unit of length. |
| `google_places_api_key` | address service access key | Avoids an abbreviation; the value is the access key of the Google Places address service. |
| `kpi_website_sale_total` | online sales measure of the periodic digest | Avoids an abbreviation. |
| `enabled_gmc_src` | product feed enabled | Avoids an abbreviation; the flag enables the product syndication feed. |
| `shop_ppg`, `shop_ppr` | products per page, products per row | Avoids abbreviations. |
| `show_line_subtotals_tax_selection` | tax display mode | Shorter and unambiguous. |
| `shop_opt_products_design_classes` | product tile design | Removes a meaningless fragment. |
| `product_uom_qty` | demanded quantity | Folder-wide convention. |
| `sale_ok` | may be sold | Folder-wide convention. |
| `auth_signup_uninvited` | sign-up policy | The stored values are two-character market abbreviations that carry no meaning in prose; they are reproduced in the field table and described in words. |
| `website_url` | public address | Folder-wide convention: every address-valued field is named an address. |
| `arch` | architecture, or content markup on a page | The stored name abbreviates architecture. |
| `faq` on Forum | guidelines | The stored name abbreviates a phrase; the field holds the guidelines page. |
| `karma` and every `karma_*` field | reputation score and reputation thresholds | The stored name is a metaphor; the specification uses the business term. |
| `res.partner.tag` | Partner Website Tag | The generated full name of the entity is a sentence. |

---

# Part 8: Reconciliation notes

1. **Entity names.** The two source versions disagreed on eleven entity names; the resolutions are
   listed in [README.md](README.md) §11, and both spellings resolve to the same transport name in
   every table of this file.
2. **The storefront category identifier.** One version documented the entity as
   `website_product_category`, the other as `product.public.category`. The source tree stores it in
   the table `product_public_category` and transports it as `product.public.category`; the storage
   and transport names in §5.2 are therefore those, while the readable name stays Website Product
   Category.
3. **The page address field.** One version called the field `web_address` on every entity that has
   one. The stored name is `url` on Website Page, Website Menu, Website Rewrite, Theme Page, Theme
   Menu and Theme Attachment; this file reproduces the stored names and uses "address" in prose.
4. **Model page publication.** One version stated that a model page is created unpublished, the other
   did not mention a default. The source tree defines the publication flag of Website Model Page with
   a default of false, so §1.3 states it explicitly; a static page created through the New Page
   operation is published when the caller asks for it.
5. **Visitor page list.** One version placed the list of visited pages under the Editor and Designer
   group, the other left it unrestricted. The source tree restricts the field to that group and
   computes it with elevated rights; §1.7 keeps the restriction.
6. **Blog tag uniqueness.** One version stated uniqueness per blog. The stored constraint is on the
   tag name alone, so §4.8 states global uniqueness of the name; the forum tag, in contrast, is
   unique per forum (§4.4).
