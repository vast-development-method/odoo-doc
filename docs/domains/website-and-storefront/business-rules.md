# Business rules

The complete rule catalogue of this folder. Every rule carries a stable identifier of the form
`WS-<number>`, used by the other documents of the folder to cite it. Every user-facing message is
reproduced verbatim, in code font; where a reproduced message contains an abbreviation, the words it
stands for are given beside it, because this specification's own prose never abbreviates. Conditions
are written in the language-neutral notation `field = "value" AND other field IN ("a", "b")`.

Two independently written catalogues were merged into this one. The mapping from their former
identifiers to the identifiers used here is in §Part 6 at the end of this file.

## Index of the numbering scheme

| Range | Subject | Part |
|---|---|---|
| WS-001 – WS-022 | Site configuration | 1 |
| WS-030 – WS-040 | Site resolution | 1 |
| WS-050 – WS-072 | Pages | 1 |
| WS-080 – WS-086 | Model pages | 1 |
| WS-090 – WS-106 | Menus | 1 |
| WS-110 – WS-125 | Rewrites and routing | 1 |
| WS-130 – WS-143 | Languages and addresses | 1 |
| WS-150 – WS-156 | Search engine metadata and indexing | 1 |
| WS-160 – WS-178 | Content editing | 1 |
| WS-180 – WS-196 | Assets, attachments and pictures | 1 |
| WS-200 – WS-218 | Public forms | 1 |
| WS-230 – WS-246 | Visitors | 1 |
| WS-250 – WS-264 | Content blocks | 1 |
| WS-270 – WS-278 | Site search | 1 |
| WS-280 – WS-287 | Publication and access | 1 |
| WS-290 – WS-469 | The storefront | 2 |
| WS-470 – WS-479 | Cookies and third-party content | 3 |
| WS-480 – WS-494 | Site index and crawler instructions | 3 |
| WS-500 – WS-509 | Themes | 3 |
| WS-510 – WS-519 | The site configurator | 3 |
| WS-520 – WS-544 | Blogs | 3 |
| WS-550 – WS-560 | Public contact references and profiles | 3 |
| WS-570 – WS-575 | Consistency and locking | 3 |
| WS-580 – WS-613 | Forums | 4 |
| WS-620 – WS-624 | Tracked links, public actions and portal access | 4 |
| WS-900 – WS-907 | Industry-standard completions | 5 |

---

# Part 1: Site, structure, content and visitors

## Site configuration rules

**WS-001 — Site domain uniqueness.** No two Website records may carry the same domain. Enforced at the
database level. Message: `Website Domain should be unique.`

**WS-002 — Site domain must be a parsable address.** When the domain is non-empty and cannot be parsed as
a web address, the save is refused. Message: `The provided website domain is not a valid URL.` The three
capital letters in the message abbreviate uniform resource locator, which this specification calls a web
address.

**WS-003 — Site domain must not contain relative path segments.** When the path part of the domain
contains `/./` or `/../`, the save is refused. Message:
`The domain path cannot contain relative path segments like '/./' or '/../'.`

**WS-004 — Site domain normalisation.** On create and on update, a non-empty domain that does not start
with `http` is prefixed with `https://`, and every trailing slash is removed. The normalisation runs
before WS-002 and WS-003.

**WS-005 — Home page address must be relative.** When the home page address is non-empty and does not
start with a slash, the save is refused. Message:
`The homepage URL should be relative and start with '/'.`

**WS-006 — Home page address normalisation.** On create and on update, trailing slash characters are
removed from the home page address.

**WS-007 — Site icon normalisation.** On create and on update, a supplied site icon is centre-cropped,
resized to 256 by 256 pixels and stored in icon format, whatever the uploaded size and format.

**WS-008 — Default language must be among the available languages.** When the user edits the language list
in the form and the current default language is not in the new list, the default language becomes the
first language of the new list. This is an on-change behaviour and is not enforced on direct writes.

**WS-009 — Public user is mandatory.** Every site must have a public user. When it is not supplied at
creation, the public user of the supplied company is taken; when no company is supplied, the platform
public user is taken.

**WS-010 — Public user follows the company.** When the company changes and the public user is not written
in the same operation, sites whose current public user belongs to a different company receive the public
user of the new company.

**WS-011 — The default site cannot be deleted.** Message:
`You cannot delete default website %s. Try to change its settings instead`, with the placeholder replaced
by the site name.

**WS-012 — Deleting a site deletes its site-scoped attachments only.** On deletion, the attachments of that
site that are theme attachments (they carry a key), customised asset attachments (their address starts
with `/_custom/`) or compiled bundle attachments (their address contains `.assets_`) are deleted. Any other
attachment of the site, in particular a business document, is kept.

**WS-013 — Multi-site group is granted automatically.** When a second site is created by a user who is not
in the multi-site group, that group is implied into the portal, internal user and public groups, so that
every user sees the site dimension.

**WS-014 — A company owning a site cannot be archived.** Message:

```
The company “%(company_name)s” cannot be archived because it has a linked website “%(website_name)s”.
Change that website's company first.
```

**WS-015 — The company site link is the first site of that company.** The derived site of a Company is the
first site whose company is that company, in site ordering order, that is by ordering value and then by
identifier. It is recomputed when a site is created or deleted, and when its ordering value or its company
changes.

**WS-016 — A language used by a site cannot be deactivated.** Message:
`Cannot deactivate a language that is currently used on a website.`

**WS-017 — Enabling the consent bar creates the cookie policy page.** When the consent bar is switched on
and no page of this site answers on `/cookie-policy`, the shipped cookie policy template is forked for this
site and a Website Page is created, published, not indexed, at that address, with the site and that
template.

**WS-018 — Disabling the consent bar deletes the cookie policy page.** When the consent bar is switched off,
the page of this site answering on `/cookie-policy` is deleted.

**WS-019 — Custom blocked domain list normalisation.** On save, each line is trimmed and lower-cased. Blank
lines are dropped. A line starting with `#` is kept verbatim as a comment. Any other line is reduced to its
host part by parsing it as a web address; a line that cannot be parsed is refused with the message
`The following domain is not valid:` followed by a line break and the offending line.

**WS-020 — Effective blocked domain list.** The effective list is the shipped list, then a line break, then
the non-comment lines of the custom list, unless the first custom line starts with `#ignore_default`, in
which case the effective list is the non-comment custom lines only.

**WS-021 — Content delivery network rewriting.** When the content delivery network is enabled, an asset
address is rewritten to the network base address joined with the address, when the address matches at least
one of the non-empty filter lines read as matching patterns anchored at the start. An empty address returns
an empty text.

**WS-022 — Content delivery network changes invalidate compiled templates.** Writing the network flag, the
network base address or the filters clears the registry cache, because asset addresses are baked into
compiled templates.

## Site resolution rules

**WS-030 — Resolution precedence.** The current site is, in order: the site forced in the session when it
still exists; the site in the execution context; when the request is a front-end request or a fallback was
requested, the site matching the request host; when a fallback was requested, the first site in ordering
order; otherwise no site.

**WS-031 — A stale forced site is discarded.** When the session forces a site identifier that no longer
exists, the identifier is removed from the session and resolution continues with the following steps.

**WS-032 — Host matching ignores the scheme.** Only the host and port part of the domain participates in
matching; the scheme is ignored.

**WS-033 — Host matching is case-insensitive.** Both sides are lower-cased before comparison.

**WS-034 — Host matching accepts both spellings of an internationalized domain.** The request host is
converted to the ascii-compatible encoding and back to the readable form, and both spellings are searched;
the stored domain may itself be written in either spelling.

**WS-035 — An exact host match wins over a containing match.** The candidate set is first produced with a
"contains" comparison for performance, then filtered to exact equality of the full host and port.
Subdomains therefore never match: a request for `www.example.com` does not match a site whose domain is
`example.com`.

**WS-036 — Port fallback.** When no site matches host and port exactly, the comparison is retried with the
port removed on both sides. A request on a host with an unusual port therefore reaches the site declaring
the same host with another port only through this fallback, and only when no site declares that port.

**WS-037 — Fallback site.** When no domain matches and a fallback was requested, the first site in ordering
order is used; this is also the site used when no site declares a domain at all.

**WS-038 — Back-office requests without fallback get no site.** When the request is not a front-end request
and the caller did not request a fallback, resolution returns no site rather than an arbitrary one.

**WS-039 — Site switching requires both groups.** Forcing another site through the switch endpoint requires
membership in both the multi-site group and the Restricted Editor group. A user lacking either is simply
redirected to the requested path without any forcing.

**WS-040 — Site switching hops to the target domain first.** When the target site declares a domain and the
current request host differs from it, the visitor is first redirected to the same endpoint on the target
domain, so that the forced identifier is written in the session belonging to that host.

## Page rules

**WS-050 — Page address slugification.** Every write of the page address replaces the value by a slash
followed by the path slug of the value, computed with a maximum length of 1024 characters per segment.

**WS-051 — Page address uniqueness per site.** When the slugified address differs from the current one, a
counter suffix `-1`, `-2` and so on is appended until no page of the same site, archived pages included,
answers on it. Pages of different sites may share an address.

**WS-052 — Menus follow a page address change.** Every menu entry linked to the page receives the new
address.

**WS-053 — The site home page setting follows a page address change.** When the site's home page address,
after the same trailing-slash normalisation, equals the old page address, it is updated to the new address.

**WS-054 — Page key regeneration.** When the page name changes, the template key is regenerated as the
unique key derived from the slug of the new name; uniqueness is evaluated against the templates of the
current site and the shared templates, archived ones included.

**WS-055 — Authorised groups are cleared unless the visibility is group-restricted.** Writing a visibility
with any value other than `restricted_group` empties the authorised group list.

**WS-056 — Routing-relevant page changes clear the template cache.** Writing the address, the visibility or
the authorised groups clears the template cache; so does deleting a page or a model page, and creating,
writing or deleting a menu entry.

**WS-057 — A page is visible when it is published and not embargoed.** The visibility flag is true when the
page is published on the current site and either the publishing date is empty or it is already past.

**WS-058 — A page is the home page when its address matches the setting.** The home-page flag is true when
the page address equals the site's home page address, or, when that setting is empty and the page belongs
to the current site, when the address is `/`.

**WS-059 — Deleting a page deletes its private template.** A template is deleted together with the page when
every page using it is in the deletion set and it has no inheriting children. Pages already removed by the
cascade from those templates are excluded from the deletion set, to avoid a double deletion.

**WS-060 — Duplicating a page duplicates its template.** Unless a template is explicitly supplied,
duplicating a page creates a copy of its template bound to the target site, sets the page key to the new
template key, and gives the copy a unique address derived from the original one.

**WS-061 — Cloning a page clones its menu entry only within the same site.** When the clone stays on the
same site and menu cloning was requested, the first menu entry targeting the original is duplicated with the
clone's address, name and page. A clone that changes site never clones the menu.

**WS-062 — The most specific page wins.** For a given address, the page bound to the current site is served;
the shared page is served only when no site-specific page exists for that address.

**WS-063 — A diverged forked page hides the shared page.** When a shared page was forked for a site and the
fork received a different address, the shared page is no longer served on that site at its own address
either. The rule is implemented by keeping, per address, only the first page in the ordering, specific
first, and, for a shared page, only when exactly one page in the whole scope carries that key.

**WS-064 — Page address matching is case-insensitive as a fallback.** An exact match on the request path is
attempted first; when it fails a case-insensitive match is attempted, and the visitor is then redirected to
the stored spelling.

**WS-065 — Trailing slashes are removed by redirection.** When no page answers on a path that is not `/` and
that ends with a slash, the visitor is redirected permanently to the path without the trailing slash,
re-prefixed with the language when it is not the default and carrying the original query string.

**WS-066 — Page publication permission.** A member of the Editor and Designer group may always publish or
unpublish a page. Any other user may do so only when the platform write check on the page succeeds.

**WS-067 — Creating a page already published without permission is refused.** Message:
`You do not have the rights to publish/unpublish`.

**WS-068 — Serving a page requires visibility or designer rights.** A page is rendered only when the current
user is in the Editor and Designer group, or the page is visible under WS-057. Additionally the page must be
bound to a site, or its template must be the most specific template for its key on the current site.

**WS-069 — Page response caching preconditions.** A page response is served from the cache only when the
request verb is a read verb, the request carries no parameters, the current user is the public user, and the
page has no authorised groups.

**WS-070 — Page cache key.** The cache key is the tuple made of the site identifier, the language code, the
request path, the diagnostic flag and the all-consents-granted flag.

**WS-071 — Page cache validity.** A cached response older than 3600 seconds is refreshed from a freshly
produced response before being served.

**WS-072 — Cached responses get a fresh forgery token.** Before a cached response is returned, the embedded
request forgery token is replaced by the token of the current session, both in the script variable and in the
hidden form field.

## Model page rules

**WS-080 — A model page must expose a concrete entity.** Exposing a transient, abstract or table-less entity
is refused. Message: `A page must be set to display a concrete model.`

**WS-081 — A model page requires read access to the exposed entity.** The creating or updating user must
pass the platform read check on the exposed entity; the platform message is raised when the check fails.

**WS-082 — Model page address uniqueness.** The address segment is unique across every model page. Message:
`url should be unique`.

**WS-083 — Model page address derivation.** The address segment is the slug of the name, recomputed whenever
the name or the exposed entity changes, and empty when no entity is exposed. Writing it directly
re-slugifies the written value.

**WS-084 — Menus follow a model page rename.** After every write, each menu entry targeting the model page
receives the address `/model/` followed by the address segment, and the name of the model page.

**WS-085 — Model pages are unpublished by default.** The default publication state of a model page is false,
unlike a static page created through the New Page operation, which is created published when the caller asks
for it.

**WS-086 — Public read of a model page requires the exposure group.** Reading a model page as a public or
portal user requires the group that grants public access to an arbitrarily exposed entity, which is implied
into the public and portal groups, and the record rule restricts the visible set to published model pages.

## Menu rules

**WS-090 — Two levels maximum.** A menu entry may not have more than two ancestors. Message:
`Menus cannot have more than two levels of hierarchy.`

**WS-091 — A mega menu is flat.** A mega menu may not have a parent or a child, and an entry may not be
attached under a mega menu. Message: `A mega menu cannot have a parent or child menu.`

**WS-092 — A container cannot become a submenu.** An entry that has children may not be attached under an
entry that itself has a parent, and its children may not have children. Message:
`Menus with child menus cannot be added as a submenu.`

**WS-093 — The shipped root menu cannot be deleted.** Message:
`You cannot delete this website menu as this serves as the default parent menu for new websites (e.g., /shop, /event, ...).`

**WS-094 — A menu created without a site is duplicated on every site.** Creating an entry with no site and no
site in the execution context creates one entry per existing site, each attached to that site's root menu
unless a site-specific parent was supplied. When the supplied parent is the shipped root menu, one entry is
additionally created under that root. The operation returns the last created record.

**WS-095 — A menu created with a site in context takes that site.** When no site is supplied in the values
but one is present in the execution context, it is written into the values.

**WS-096 — The shipped template root is created as is.** An entry whose address is exactly
`/default-main-menu` is created without duplication.

**WS-097 — Deleting a shared entry deletes its per-site copies.** Deleting a direct child of the shipped root
menu also deletes every site-specific entry with the same address.

**WS-098 — Restricting a menu always keeps designers.** Writing the authorised groups on an entry adds the
Editor and Designer group to the groups of every entry that now has groups.

**WS-099 — Menu address derivation.** The address is the fragment marker when the entry is a mega menu or has
children; otherwise the address of the linked page when one is linked; otherwise the stored value; otherwise
the fragment marker.

**WS-100 — Menu visibility for users who are not internal.** An entry is hidden when it targets a page or a
model page that is unpublished or embargoed, or whose template visibility check fails, unless the visibility
mode is `password`, in which case the entry stays visible so that the visitor can be prompted.

**WS-101 — Menu record rule.** A menu entry is readable only when it has no authorised groups or the reader
belongs to at least one of them.

**WS-102 — Menu address cleaning heuristic.** When an entry address does not start with a slash and is not
one of the two page-level anchors `#top` and `#bottom`: an address containing an at sign that does not
already start with `mailto` is prefixed with `mailto:`; any other address that does not start with `http` is
prefixed with a slash.

**WS-103 — Anchor entries are rebound to the referring page.** During the menu editor save, an address that
starts with the fragment marker, is longer than one character and is not `#top` or `#bottom` is prefixed with
the path of the page the designer came from.

**WS-104 — A menu address that matches an endpoint never renames the page.** During the menu editor save,
when the new address matches no page but the entry was bound to a page: when an endpoint answers on the new
address, the page link is cleared; otherwise the linked page is renamed to the new address.

**WS-105 — Menu activity detection.** Mega menus are never active. A leaf is active when its path equals the
request path after replacing the slug of the last segment by its numeric identifier, and, when it targets a
page, when the raw paths are also equal, and when every query parameter of the entry is present with the same
value in the request, and when the entry host, if any, equals the request host. A container is active when
one of its children is active.

**WS-106 — The menu cache flag.** A site disables the menu cache when any of its entries has an address
containing a numeric record segment — a path segment made of an optional slug prefix and digits — or has
authorised groups. The flag is cached per user and per site and recomputed when the template cache is
cleared.

## Rewrite and routing rules

**WS-110 — Target required for a redirect or rewrite.** Message: `"URL to" can not be empty.`

**WS-111 — Source required for a redirect or rewrite.** Message: `"URL from" can not be empty.`

**WS-112 — Neither address may start with a fragment marker.** Message: `URL must not start with '#'.`

**WS-113 — Source and target must differ.** Compared after removing the fragment part of each. Message:
`base URL of 'URL to' should not be same as 'URL from'.`

**WS-114 — A rewrite target must be absolute on this site.** Only for the action type `308`. Message:
`"URL to" must start with a leading slash.`

**WS-115 — Parameter placeholders must match.** Only for `308`. Every placeholder present in the source must
be present in the target and conversely. Messages:
`"URL to" must contain parameter %s used in "URL from".` and
`"URL to" cannot contain parameter %s which is not used in "URL from".`

**WS-116 — A rewrite target may not be the site root.** Only for `308`. Message:
`"URL to" cannot be set to "/". To change the homepage content, use the "Homepage URL" field in the website settings or the page properties on any custom page.`

**WS-117 — A rewrite target may not shadow an existing endpoint.** Only for `308`, compared ignoring a
trailing slash. Message: `"URL to" cannot be set to an existing page.`

**WS-118 — A rewrite target must be a valid routing pattern.** Only for `308`. Message:
`"URL to" is invalid: %s`, where the placeholder is the compilation error.

In WS-110 to WS-118 the three capital letters of the reproduced messages abbreviate uniform resource locator.

**WS-119 — Only rewrites and suppressions change the routing table.** Creating, writing or deleting a rule
whose action type is or was `308` or `404` clears the routing cache on every worker. The types `301` and
`302` never do.

**WS-120 — Rewrite candidate addresses.** The fallback handler looks up a `301` or `302` rule whose source
equals the full request address with query string, or the path without a trailing slash, or the path with a
trailing slash, or the path with the slug of its last segment replaced by its numeric identifier. The first
match in descending source order wins.

**WS-121 — Rewrite targets are re-slugified per language.** When the target starts with a slash and contains
a slug pattern, the target is rebuilt for the current language before redirecting, so that the record name in
the address is translated.

**WS-122 — Redirect targets may leave the site.** The redirect is not restricted to local addresses, because
only members of the Editor and Designer group may create rules.

**WS-123 — Request parameters survive a redirect.** The request parameters are re-appended to the target
address.

**WS-124 — Doubled slashes are collapsed.** When redirection is allowed and the request path contains a
doubled slash, the visitor is redirected permanently to the path with doubled slashes collapsed.

**WS-125 — Endpoint catalogue refresh.** The Website Route catalogue keeps exactly the endpoint paths that
answer the read verb. Paths that disappeared are deleted and new ones are created. A name search that returns
nothing triggers a refresh and is retried once.

## Language and address rules

**WS-130 — Language resolution order.** The requested language is, in order: the prefix present in the path,
then the front-end language cookie, then the language in the execution context, then the site's default
language. Each candidate is mapped to the nearest available front-end language.

**WS-131 — Nearest language.** An exact code match wins. Otherwise the first available front-end language
whose code starts with the same two-letter prefix is taken. Otherwise there is no match. An empty candidate
yields no match.

**WS-132 — Available front-end languages are the site's languages.** During a front-end request the list is
the language list of the current site, sorted by name. An active language that is not among them is never
served, never accepted as a prefix and never offered in the language selector.

**WS-133 — Alternate-language code assignment.** For each language of the site, in name order, the two-letter
prefix is assigned to the first language that claims it, and the remaining languages of that prefix receive
their full code lower-cased with the underscore replaced by a hyphen. Spanish is a special case: when
Latin-American Spanish is among the languages, it, and no other Spanish variant, receives the bare two-letter
prefix.

**WS-134 — The default language carries no prefix.** Addresses in the default language are served without a
prefix, and a request carrying the default prefix is redirected to the address without it.

**WS-135 — Crawlers are never redirected by browser language preference.** A request without a language
prefix from a user agent identified as a crawler is served in the default language without redirection.
Crawlers can still fetch any language by requesting its prefixed address explicitly.

**WS-136 — Write requests are never redirected for language.** When the request verb is a write verb, no
language redirection happens.

**WS-137 — Endpoints that are not multi-language are never redirected for language.** An endpoint that
declares itself non-multi-language, or that is not a front-end endpoint, never receives a language prefix.

**WS-138 — The language cookie is refreshed on every front-end request.** When the front-end language cookie
differs from the language in effect, it is rewritten.

**WS-139 — A slug address is redirected to its canonical spelling.** For a multi-language read request, the
canonical address is rebuilt from the endpoint and its record arguments. When the rebuilt address differs
from the requested one, the visitor is redirected permanently, with the language prefix added when the
language is not the default.

**WS-140 — Which addresses are translatable.** An address is translatable when its endpoint is a front-end
endpoint and either declares itself multi-language or is a rich-text endpoint. An address under the static
file path or starting with the platform client path is never translatable. An address that matches no
endpoint is treated as translatable, so that page addresses receive the prefix.

**WS-141 — Single-language sites carry no prefix.** When the site has exactly one language, no prefix is
inserted unless a language is explicitly forced by the caller.

**WS-142 — Alternate and canonical links.** Alternate-language links are emitted only when the endpoint is
multi-language, the current address is canonical, and the site has more than one language; a default-language
link is then also emitted under the code `x-default`. The canonical link is always emitted and points at the
current address rebuilt for the current language against the site's base address, without any query string.

**WS-143 — Canonical comparison.** The current address is canonical when the request root joined with the raw
request path and query equals the computed canonical address. An address containing characters that require
escaping is therefore never canonical, because the raw request is unescaped while the canonical address is
escaped.

## Search engine metadata and indexing rules

**WS-150 — Optimisation flag.** The metadata-complete flag is true only when the meta title, the meta
description and the meta keywords are all non-empty.

**WS-151 — Page title precedence.** The document title is the record's meta title when set; otherwise the
record name, a space, a vertical bar, a space and the site name, when the main object has a name; otherwise
the site name.

**WS-152 — No-index conditions.** A no-index instruction is emitted when the main object carries an indexing
flag that is false, when the site declares a domain and the request root does not match it, or when the
request is a paginated listing beyond the first page.

**WS-153 — Indexable host comparison.** The request root and the site domain are compared after removing the
scheme, the leading `www.`, the trailing slash, and after encoding both in the ascii-compatible
internationalized domain name encoding. Only equality makes the host indexable.

**WS-154 — Sharing image resolution.** The sharing image is the record's stored sharing image when set, its
scheme and host being stripped first; otherwise the site's dedicated sharing image when one exists; otherwise
the site logo. The resulting address is made absolute against the site domain, or against the request root
without its trailing slash when no domain is set.

**WS-155 — Slug human part.** When a record carries a slug name, the human part of its slug is that name;
otherwise it is the display name. The numeric identifier is always appended.

**WS-156 — Search engine verification token.** The verification endpoint answers with the line
`google-site-verification: ` followed by the stored token. It answers "not found" when no token is stored, or
when the requested key neither equals the stored token nor extends it as a prefix. When the requested key
starts with the stored token, the stored token is replaced by the requested key wrapped in the shipped
`google<key>.html` form and the answer is produced, which lets the site owner complete a partially entered
token.

## Content editing rules

**WS-160 — Copy-on-write for templates.** Writing a template while a site is present in the execution context
and copy-on-write is not disabled never modifies a shared template: an already existing site-specific template
with the same key receives the write; otherwise the shared template is copied with the same key and that
site, the copy receives the write, the pages of the shared template are duplicated onto the copy, and each
inheriting child is either copied onto the new parent, when it was already specific to this site, or
rewritten to point at the new parent, which forks it in turn.

**WS-161 — A template written in a site context always gets a key.** When a template has no key and none is
written, it receives a generated key made of the site package name, a dot, `key_` and six characters, before
the copy-on-write logic runs.

**WS-162 — Template creation guard.** Creating a template while a site is in context: a missing site in the
values is filled from the context; an explicitly empty site is refused with the internal message
`Trying to create a generic view from a website <identifier> environment`; a different site is refused with
`Trying to create a view for website <new identifier> from a website <identifier> environment`.

**WS-163 — Copy-on-delete for templates.** Deleting a shared template while a site is in context first forks
it for every **other** site, by writing its own name onto it in each of those contexts, so that only the
current site loses the template.

**WS-164 — The most specific template wins.** For a given key, the site-specific template of the current site
wins over the shared template. When no site is in context, only shared templates are considered. Inactive
site-specific templates win over active shared templates during inheritance resolution, and the result is then
filtered to active templates.

**WS-165 — Inheriting templates are looked up with the site dimension.** The inheritance condition is extended
with "site is empty or the current site" and, when a site is in context, the active filter is removed from the
condition, so that an inactive specific template can mask an active shared one.

**WS-166 — Template lookup ordering.** Template lookup is ordered by site ascending, so the shared template is
returned only when no specific one exists. A lookup failure reports the site identifier in the message,
appended as ` (website: <identifier>)`.

**WS-167 — Saving a region diverts to the existing fork.** When a location expression is supplied, the
template has a key, a site is resolved and a site-specific template with that key already exists, the save is
applied to that specific template.

**WS-168 — A save from the site never flags the shared template as protected from updates.** The no-update
flag is set only when no site is in context.

**WS-169 — Content blocks saved from the site are per site.** The values used to save a new content block and
to save an editable region carry the current site.

**WS-170 — Translations never fork a template.** Writing field translations on a template is performed with
copy-on-write disabled.

**WS-171 — The source language of a template is the site's default language.** For a template bound to a
site, the base language for translation is that site's default language; for any record carrying a site, the
same rule applies; otherwise the platform base language is used.

**WS-172 — Delayed translation on save.** When the caller asks for delayed translation and the configuration
parameter `website.disable_delay_translations` is unset or set to a false value — `False`, `0` or empty — the
save keeps the existing translations of unchanged terms. When the parameter is set to any other value, delayed
translation is disabled.

**WS-173 — Allowed root attributes of an editable block.** In addition to the platform list, the following
attributes survive a save on the root element of a block: the background video source, the shape, the scroll
background ratio, the visibility mode, the visibility identifier, the visibility selectors, and, for each of
country, language, signed-in state, campaign, medium and source, the visibility value and its comparison rule.

**WS-174 — Soft and hard reset.** A soft reset restores the previously stored architecture of the template. A
hard reset re-reads the architecture from the shipped file the template records. Both run with copy-on-write
disabled.

**WS-175 — Disabling a template may reset it.** When the theme options panel disables templates and asks for
an architecture reset, a hard reset is performed on each before deactivation.

**WS-176 — Enabling one header disables the others.** Enabling a header template disables every other header
template first. The same rule applies to footer templates. The default header and the default footer are the
last entries of their lists.

**WS-177 — Theme toggles avoid useless forks.** When a toggle targets a shared template or asset, no
site-specific record exists yet, and the requested state already equals the current state, nothing is written,
so that the record is not forked for nothing.

**WS-178 — Theme copies are not marked as user-edited.** During a package installation, a copy whose
architecture equals its theme record's architecture is written with the user-edited flag cleared.

## Asset, attachment and picture rules

**WS-180 — Copy-on-write for assets.** The forking rule of WS-160 applies to Asset records, keyed on the asset
key.

**WS-181 — The most specific asset wins.** For a given key, the asset of the current site wins over the shared
one; an asset with no key is always kept; outside a site context, assets bound to a site are dropped.

**WS-182 — Asset bundles are per site.** The address of a compiled bundle carries the site identifier when a
site is resolved and omits it otherwise.

**WS-183 — Inactive themes are excluded from asset resolution.** When a site is resolved, every theme package
other than that site's theme is removed from the list of packages contributing assets.

**WS-184 — Attachments take the current site.** An attachment created while a site is resolvable receives that
site, unless the values already carry one or the execution context forbids it.

**WS-185 — Attachment serving is site-aware.** Serving an attachment by address adds the site dimension to the
lookup and orders by site first, so that a per-site attachment wins.

**WS-186 — Theme attachment lookup by key.** When the current site has a theme, a lookup by external
identifier first searches an attachment of that site whose key equals the identifier; a caller who is not
internal additionally requires the attachment to be public, and the search runs with elevated rights.

**WS-187 — Removing an editor picture is refused while it is used.** The remove operation deletes only the
attachments whose local address appears in no template architecture, in either quoting style. For each refused
attachment the names of the blocking templates are returned.

**WS-188 — Uploaded pictures must be of a supported kind.** Only the graphics interchange format, the three
joint photographic experts group extensions, the portable network graphics format, the scalable vector format
and the web picture format are accepted. Message:
`Uploaded image's format is not supported. Try with: ` followed by the comma-separated list of extensions.

**WS-189 — Uploaded pictures are processed.** An uploaded picture is resized to the requested box and
re-encoded at the requested quality, and its resolution is verified. A file that the picture library cannot
read is refused with the library's own message.

**WS-190 — Generated picture names.** When no name is supplied, the name is the moment of upload written as
year, month, day, hour, minute and second, a hyphen, six characters and the extension of the detected kind.

**WS-191 — Files named with the bitmap extension lose their extension.** A file whose name ends with `.bmp` is
stored without that suffix, so that the stored media type and the name agree.

**WS-192 — Editor attachments default to the template scope.** An attachment created by the editor is public
when its related entity is the template entity, and its related identifier is forced to empty in that case.

**WS-193 — Modifying a picture creates a derived attachment.** The modified copy records the source attachment
as its original, is of the binary kind, carries the requested media type and name, and is related either to
the template entity with the identifier 0 or to the supplied record. When an identical attachment already
exists and carries no address, it is reused instead of creating a new one.

**WS-194 — Modifying a picture requires access on both sides.** The caller must pass the read check on the
source attachment's related record, when it has one, and the write check on the target record. The copy itself
is performed with elevated rights, and a media type forced to plain text by the storage layer is corrected
afterwards with elevated rights.

**WS-195 — Web picture conversion renames the file.** When the requested media type is the web picture format,
a `.jpg`, `.jpeg` or `.png` suffix in the name is replaced by `.webp`.

**WS-196 — Font uploads are validated by content.** A font file is accepted only when its bytes match its
extension: the open type signature for `.otf`, the web open font signature for `.woff`, the second-generation
web open font signature for `.woff2`, and, for `.ttf`, the presence of all nine mandatory tables in the table
directory, whose declared size must fit inside the file. A compressed archive is accepted and each contained
font is validated the same way; archive entries larger than 10485760 bytes are refused with
`File '%s' exceeds maximum allowed file size`; entries under the system metadata folder and hidden entries are
skipped. When nothing valid was found, the message is `File '%s' is not recognized as a font`.

## Public form rules

**WS-200 — Only opted-in entities may be targeted.** The form endpoint accepts only entities whose
usable-in-forms flag is true. The answer when it is not is the error
`The form's specified model does not exist`.

**WS-201 — Only opted-in fields are writable.** For any entity other than the outgoing mail entity, the
writable set is the fields of that entity whose form-exclusion flag is false, intersected with the authorised
field set. For the outgoing mail entity the writable set is fixed: sender address, recipient address, copy
recipients, blind copy recipients, body, reply address and subject.

**WS-202 — Authorised field computation.** Starting from the declared fields of the entity: the delegation
link fields are removed; fields that have a default value are marked as not required; fields that are read
only, that are one of the platform's automatic columns, that are a polymorphic numeric reference or that are
structured data are removed; a condition expressed as text is removed, because it would have to be evaluated;
a dynamic-properties field is expanded into pseudo-fields read from its definition record, skipping properties
that are not fully defined — a record-valued property without a target entity, a choice property without
choices, a tag property without tags, and a separator — marking every pseudo-field as not required and
converting a textual condition into a structured condition, and dropping the property when that conversion
fails.

**WS-203 — Only designers may extend the allow list.** The operation that removes fields from the exclusion
list requires the Editor and Designer group, refuses unknown field names with
`Unable to whitelist field(s) %r for model %r.`, and answers false for an empty list.

**WS-204 — A field used in a published form cannot be deleted.** Message:

```
The field '%(field)s' cannot be deleted because it is referenced in a website view.
Model: %(model)s
View: %(view)s
```

The scan covers the template architecture column and every stored rich text column where a form can survive,
that is columns that are not sanitised, that allow form elements, or that can be bypassed by the sanitisation
override group.

**WS-205 — Partial forgery protection.** The request forgery token is verified only when the session is
authenticated. Failure raises a bad request with the message `Session expired (invalid CSRF token)`; the four
capital letters abbreviate cross-site request forgery.

**WS-206 — A form submission is atomic.** The handling runs inside a savepoint; a validation failure rolls
back only the form handling, not the rest of the request.

**WS-207 — Value conversion per field type.** Single-line text, plain text, date and date-and-time values and
choice values are taken verbatim; rich text values are converted from plain text with line breaks turned into
markup; record links and whole numbers are parsed as integers; decimal numbers and money values are parsed as
decimals; booleans are the truth value of the submitted value; binary values are the base-64 encoding of the
uploaded bytes; a multiple-record link is the comma-separated list of integers; a many-to-many link is that
list wrapped in a replace command; a tag property value is the comma-separated list where `\,` means a literal
comma and `\/` means a literal backslash. A conversion failure adds the field name to the error list.

**WS-208 — Unmatched values become free text.** Every submitted value that is not a writable field, and that
is not the request forgery token or the form signature, is kept as a pair of the submitted name, a space, a
colon, a space and the value.

**WS-209 — Missing required fields are reported with the conversion errors.** The answer carries the fields
that failed conversion followed by the required writable fields with no value. The answer is produced only
when at least one conversion actually failed.

**WS-210 — Metadata collection is opt-in.** When the configuration parameter `website_form_enable_metadata` is
set, the network address, the user agent, the accepted languages and the referring address are appended to the
record under the heading `Metadata`.

**WS-211 — Free-text placement.** When the entity designates a default field, the assembled free text is
written there, converted from line breaks to markup when that field is rich text or when the entity is the
outgoing mail entity. Otherwise the assembled text is logged as a comment message on the record. The free-text
heading is `Other Information:` in general and `This message has been posted on your website!` for the
outgoing mail entity.

**WS-212 — Records are created with elevated rights and without automatic subscription.** The created record's
author is not automatically subscribed to its discussion thread.

**WS-213 — Mail sender rewriting.** For the outgoing mail entity, the reply address becomes the submitted
sender address and the sender address becomes the company name followed by ` form submission`, in quotation
marks, and the company address in angle brackets.

**WS-214 — Mail recipient signature.** For the outgoing mail entity a signature accompanies the form. Its
value is the keyed digest of the recipient address, with `:email_cc` appended when a copy recipient field is
present; the same suffix is appended to the signature itself. A mismatch refuses the submission with
`invalid website_form_signature`. The signature is recomputed on the server every time a rich text value
containing a form is rendered, and a stale signature node is removed first. When the recipient address is
empty, or when it is the shipped example address and the page is the shipped contact form, the company address
is used instead.

**WS-215 — Attachment placement.** An uploaded file whose field name is a writable field is linked to that
field, as the single value for a single-valued field and as an addition for a multi-valued one. Any other file
becomes an orphan. Orphans are attached to a comment message whose body is the paragraph `Attached files: `,
except for the outgoing mail entity, where they are added to the message's own attachment list.

**WS-216 — Constraint violations answer false.** A database constraint violation that cannot be attributed to
a field answers `false`.

**WS-217 — The session remembers the created record.** The entity name, its label and the created identifier
are stored in the session, so that the confirmation page can display them.

**WS-218 — Human verification runs before the record is created.** The verification for the action
`website_form` runs at the start of the handling; a failure answers with the error message of the failing
service.

## Visitor rules

**WS-230 — Browsing token derivation.** The token is the decimal contact identifier of the signed-in user, or,
for the public user, the first 32 characters of the hexadecimal digest of the remote network address, the
browser user agent and the session identifier.

**WS-231 — Token uniqueness.** No two visitors may share a token. Message: `Access token should be unique.`

**WS-232 — Contact derivation from the token.** A token of exactly 32 characters means an anonymous visitor
with no contact. Any other token is read as a contact identifier.

**WS-233 — Visitors are created only from the front end.** Attempting to derive a token outside a request
fails with `Visitors can only be created through the frontend.`

**WS-234 — Tracking preconditions.** Tracking happens only when the user agent is not a crawler, the response
status is 200, the request does not carry the tracking-disabled header with the value `1`, the database is
writable, and the served template has its tracking flag set.

**WS-235 — Visit counting.** A visit is counted at creation, with the count 1, and increased by one on a page
view whose previous last connection moment is more than eight hours old.

**WS-236 — Connected state.** A visitor reads as connected when the last page view happened less than five
minutes ago.

**WS-237 — Track deduplication outside the page pipeline.** The add-tracking operation creates a track only
when no matching track exists for the visitor, or when the most recent matching one is older than thirty
minutes.

**WS-238 — Country resolution.** The country is resolved at creation from the geolocation code of the network
address; an unknown code leaves the country empty.

**WS-239 — Time zone resolution.** The time zone is the browser time zone cookie when its value is a
recognised time zone name; otherwise the signed-in user's time zone; otherwise empty. It is written only once,
with a statement that skips locked rows.

**WS-240 — Merge on sign-in.** After a successful authentication, when a visitor already exists for the
authenticated contact and differs from the pre-authentication visitor, the pre-authentication visitor's tracks
are repointed to it and the pre-authentication visitor is deleted. When no such visitor exists, the
pre-authentication visitor's token becomes the contact identifier.

**WS-241 — The merge target must be identified.** Merging into a target that has no contact fails with
``The `target` visitor should be linked to a partner.``

**WS-242 — Cleanup condition.** The scheduled cleanup deletes visitors whose contact is empty and whose last
connection moment is older than the present moment minus the retention period, which comes from the parameter
`website.visitor.live.days` and defaults to 60 days. Deleting a visitor cascades to its tracks. Visitors linked
to a contact are never deleted by the job.

**WS-243 — Cleanup batching.** The job processes at most one batch, by default 1000 visitors, and reports the
remaining count when the batch was full.

**WS-244 — Contacting a visitor requires an address.** Message:
`There are no contact and/or no email linked to this visitor.`

**WS-245 — The visitor page list is restricted.** The list of visited pages is readable only by the Editor and
Designer group; it is computed with elevated rights and written back through an elevated write.

**WS-246 — Visitors are never created by an editor.** Model access grants read, update and delete on Website
Visitor to the Editor and Designer group and to the administrator group, but never create. Visitors are
created only by the tracking statement.

## Content block rules

**WS-250 — Exactly one data source.** A content block filter must carry either a saved filter or a
record-returning action, not both and not neither. Message:
`Either action_server_id or filter_id must be provided.`

**WS-251 — Result cap.** The limit must be strictly greater than 0 and at most 16. Message:
`The limit must be between 1 and 16.`

**WS-252 — Field names must be non-empty.** Every comma-separated part of the field list must be non-blank.
Message: `Empty field name in “%s”`.

**WS-253 — Template key gate.** The rendering template key must contain `.dynamic_filter_template_`, otherwise
the render is refused with `You can only use template prefixed by dynamic_filter_template_ `.

**WS-254 — Site gate.** A block filter bound to another site renders nothing.

**WS-255 — Entity and template must agree.** When the filter exposes an entity, the template key must contain
that entity name with dots replaced by underscores; otherwise the render produces nothing.

**WS-256 — Effective result count.** The number of records fetched is the smaller of the requested limit and
the larger of the configured limit and 16, and defaults to the larger of the configured limit and 16 when no
limit is requested. The floor of 16 exists because the editor allows up to 16.

**WS-257 — Only registered entities may be queried.** The exposed entity must be the entity of at least one
existing content block filter; any other entity yields no records.

**WS-258 — Implicit conditions.** When the exposed entity carries a site dimension, the current site condition
is added. When it carries a company dimension, the condition "company is empty or the site's company" is
added. When it carries a publication flag, the condition "published" is added.

**WS-259 — Single-record mode.** When the requested limit is 1 and both an entity and a record identifier are
supplied, exactly that record is fetched, bypassing the filter.

**WS-260 — Sample fallback.** When the selection is empty and samples were requested, a fixed sample is
repeated up to the requested length and its missing fields are filled: picture and binary fields with nothing,
money fields with a random value between 10.0 and 1000.0 with one decimal, whole and decimal numbers with the
sample index, and anything else with `Sample ` followed by the sample index plus one.

**WS-261 — Money conversion in blocks.** A money value is converted from the record's currency to the site
company's currency at today's rate. When the record has neither a money currency field nor a currency link,
the raw value is used.

**WS-262 — Picture values in blocks.** A picture or binary field is rendered as the site picture address of
that field on that record, which carries a cache marker derived from the record's modification moment. For a
sample record, the value is used directly or the bare picture endpoint is used as a placeholder.

**WS-263 — Field presentation derivation.** A field name may carry a forced presentation after a colon.
Without one, the presentation is the field's declared type; for a name that is not a field of the entity, a
name containing `image` becomes a picture presentation, a name containing `price` becomes a money
presentation, and anything else becomes a text presentation.

**WS-264 — Block option listing requires an editor.** Listing the available block filters requires the
Restricted Editor group; otherwise the request answers "not found". Any supplied extra condition must mention
only fields of the content block filter entity.

## Site search rules

**WS-270 — Result ordering.** The effective ordering is "publication flag descending, then the requested
ordering or name ascending, then identifier descending".

**WS-271 — Approximate matching preconditions.** Approximate matching is skipped, and the term is used as is,
when the term is shorter than four characters, contains a space, or is composed of at least 80 percent digits.

**WS-272 — Approximate matching candidate set.** Only words that start with the same first character as the
term are compared. A word that contains the term as a substring short-circuits the search, and the original
term is used.

**WS-273 — Substitution reporting.** When the chosen word equals the original term ignoring case, no
substitution is reported to the caller.

**WS-274 — Page search visibility.** For a user outside the Editor and Designer group, the page search
condition adds: published, indexed, visibility is not `password`, and, for the public user, visibility is not
`connected`; plus "no authorised groups, or the user belongs to one of them". The search runs with elevated
rights, so these conditions restate the record rules.

**WS-275 — Page search post-filtering.** Every candidate page is re-checked against the read rules of the page
entity and of the template entity. When descriptions are searched, the term must additionally appear in the
concatenation of the page name, the page address and the plain text of the page architecture, matched
case-insensitively.

**WS-276 — Page search uses the most specific pages.** The candidate set is the most specific page per
address, so a forked page never appears twice.

**WS-277 — Highlighting.** A mapped value declared as matchable has every occurrence of any
whitespace-separated part of the term wrapped in the highlight template; the value is then treated as markup.

**WS-278 — Truncation.** A mapped text value is shortened to the requested maximum number of characters with
the ellipsis `...`, unless the mapping disables truncation. The default maximum is 999 characters for
autocompletion and 200 for the public listing.

## Publication and access rules

**WS-280 — Publication right by default.** The right is the result of the platform write check on the record.

**WS-281 — Publication refusal message.** `You do not have the rights to publish/unpublish`.

**WS-282 — Per-site publication reading.** With a site in context, a record reads as published only when its
stored flag is true and it is shared or bound to that site.

**WS-283 — Per-site publication searching.** Only the equality-to-true search is supported. With a site in
context it expands to "the stored flag is true and the site is empty or the current site"; without a site in
context it expands to "the stored flag is true".

**WS-284 — Public and portal record rules.** Public and portal users may read only: pages whose per-site
publication is true; model pages whose per-site publication is true, through the exposure group; blogs that
are active; blog posts whose per-site publication is true; published Partner Website Tags; templates whose
visibility is empty or public for public users, and additionally `connected` for portal users; and every
template that is not a page template.

**WS-285 — Designer template rules.** The Editor and Designer group may create, read, update and delete page
templates, and may only read other templates. The administrator group may do everything on every template.

**WS-286 — Base address of a record.** The base address of a record is, in order: for a Website record, its own
domain, otherwise the platform base address; for a record carrying a site whose site has a domain, that
domain; for a record carrying a company whose company's site has a domain, that domain; otherwise the platform
base address.

**WS-287 — Record rules are cached per site.** The site identifier participates in the cache key of computed
record rule conditions, and the current site is available in the rule evaluation context. Outside a front-end
request the context receives an empty site, so that back-office queries are not silently filtered by a site.

---

# Part 2: The storefront (WS-290 – WS-469)

Messages reproduced in this part are quoted character for character, including any abbreviation the shipped
text contains, because a rebuild must show equivalent text; such an abbreviation belongs to the message and
not to this specification's naming, and its full words are given beside it.

## Access and visibility

**WS-290 — Shop access.** A shop page, a product page or a cart page may be served to the public user only
when the site's shop access setting is `everyone`. Otherwise a listing or product page is redirected to the
sign-in page carrying the requested path as the return path, and the cart page is redirected to the sign-in
page without a return path.

**WS-291 — Menu visibility.** A site menu whose address begins with the shop path is hidden from the
navigation of the public user when WS-290 would refuse access.

**WS-292 — Site index.** When the shop is restricted to signed-in users, the site index generator emits no
shop, category or product entry, unless a query string is supplied, which only happens for the address
autocompletion of the editor.

**WS-293 — Product visibility.** A public or portal user may read a Product Template only when it is published
and sellable. A product whose site is set is visible only on that site. A product whose company is set is
visible only on sites of that company.

**WS-294 — Category visibility.** A public or portal user may read a Website Product Category only when the
category or one of its descendants contains at least one published, sellable product for the current site and
company.

**WS-295 — Custom attribute values.** A public, portal or internal user may read only the custom attribute
values they created. A user holding the sales user group or the product manager group may read all of them.

**WS-296 — Catalogue read access.** The public, portal and internal access groups have read-only access to
Product Variant, Product Template, internal product category, Product Tag, Website Product Category, Price
List, Price List Rule, Product Ribbon, Product Attribute, Product Attribute Value, Product Template Attribute
Value, custom attribute value, attribute exclusion, Product Template Attribute Line, Product Image, unit of
measure, Storefront Extra Field, Base Unit Display and Product Attribute Category. The portal group
additionally reads fiscal positions and payment terms; the public group additionally reads taxes.

**WS-297 — Catalogue write access.** Website Product Category, Product Ribbon, Base Unit Display and Product
Attribute Category may be created, written and deleted by the sales manager group. Website Product Category
may also be created and written, but not deleted, by the Editor and Designer group. Product Image and
Storefront Extra Field may be created, written and deleted by the Restricted Editor group; Product Image also
by the sales manager group. Website Checkout Step may be created, written and deleted by the Editor and
Designer group. Product Feed may be created, written and deleted by the administrator group and by the Editor
and Designer group.

**WS-298 — Editor endpoints.** The endpoints that change product ordering, product tile size, attribute
display type, page style settings and category cover pictures require the Restricted Editor group. A caller
without it receives "not found", except for the cover picture endpoint, which receives "forbidden".

**WS-299 — Implied groups.** Every internal user implicitly receives the delivery and invoice address group.
Every sales manager implicitly receives the Restricted Editor group.

## Publication and catalogue integrity

**WS-300 — Publication date.** Whenever a product's publication flag is written to true, the publication date
is set to the present moment. It is never reset when the product is unpublished.

**WS-301 — Category publication link.** On the simplified storefront product creation form, selecting at least
one storefront category publishes the product; clearing the selection unpublishes it.

**WS-302 — Recursive categories.** A Website Product Category may not be its own ancestor. Message:
`Error! You cannot create recursive categories.`

**WS-303 — One automatic ribbon per mode.** At most one Product Ribbon may carry each automatic assignment
mode. Message: `Only one ribbon with the assign %s is allowed.`, where the placeholder is the translated label
of the mode.

**WS-304 — Ribbon precedence.** The ribbon shown for a product is the variant ribbon when set, else the
template ribbon when set, else the first ribbon in ordering order whose automatic applicability test passes.

**WS-305 — Video address.** A Product Image whose video address is set but from which no embeddable player
markup can be derived is refused. Message:
`Provided video URL for '%s' is not valid. Please enter a valid video URL.`, where the placeholder is the
media name; the three capital letters abbreviate uniform resource locator.

**WS-306 — A video is never the main picture.** A video may never occupy the first position of the product
media list. Message: `You can't use a video as the product's main image.`

**WS-307 — Media reorder integrity.** The record to reorder must belong to the product's media list. Message:
`Invalid image`. When neither a variant nor a template can be resolved: `Product not found`.

**WS-308 — Variant media isolation.** A media row created with a variant must not inherit a default template
from the creating screen, otherwise the variant media would also be shown as template media.

**WS-309 — Published document scope.** A Product Document flagged for the product page may not be restricted
to a single variant. Message: `Documents shown on product page cannot be restricted to a specific variant`

**WS-310 — Document download guard.** The document download endpoint serves a document only when the caller
may read the product, the document exists and is active, the document is flagged for the product page, and the
document belongs to that Product Template. Otherwise the caller is redirected to the shop.

**WS-311 — Base unit count.** A negative base unit count is refused. Message:
`The value of Base Unit Count must be greater than 0. Use 0 to hide the price per unit on this product.` The
value zero is accepted and hides the per-unit price.

**WS-312 — Template base unit mirror.** The base unit count and the reference unit of a template mirror those
of its single variant; a template with several variants reports zero and no unit, and writing those fields on
it has no effect.

**WS-313 — Storefront extra field scope.** Only single-line text fields and binary fields of the Product
Template may be declared as storefront extra fields.

**WS-314 — Archived product cleanup.** Archiving a variant deletes every draft Sales Order Line referencing it
that belongs to an order with a site. The cart page and the cart revival additionally delete lines whose
product is archived.

**WS-315 — Empty storefront description.** A storefront description whose markup is visually empty is stored
as an empty value, unless it contains an embedded video frame or an embedded component.

**WS-316 — Print pictures before publication.** A product of a print-on-demand template that still lacks print
pictures may not be published. Message:
`Print images must be set on products before they can be published.`

**WS-317 — Print picture removal.** A print picture may not be emptied while its product is published.
Message: `Products must be unpublished before print images can be removed.`

**WS-318 — Print-on-demand resynchronisation.** Synchronising a print-on-demand template that produces new
print pictures unpublishes the product.

**WS-319 — Paid course product.** A course whose enrolment mode is on payment must name a product. The stored
check is that the enrolment mode is not the payment value or the product is not null. Message:
`Product is required for on payment channels.`

**WS-320 — Course publication synchronisation.** Publishing a paid course publishes its product; unpublishing
a course unpublishes its product unless another published course uses the same product.

## Prices, taxes and price lists

**WS-321 — Display tax mode.** Every storefront price is rendered in the site's tax display mode, tax excluded
or tax included. The mode is a site setting, not a company setting, therefore two sites of one company may
display differently.

**WS-322 — Fiscal position detection.** For the public user with a resolvable network country, the fiscal
position is the one automatically detected for a contact carrying only that country. For every other case it
is the one detected for the current contact. The result is cached in the session and refreshed whenever an
address change alters it.

**WS-323 — Tax mapping before display.** A price is restated to a tax-excluded base when the product taxes are
price-included and the mapped taxes differ; then the mapped taxes are computed for exactly one unit, and the
requested side of the computation is returned.

**WS-324 — Price list availability on a site.** A price list is usable on a site when its company is empty or
equals the site's company, and either it names that site and is active, or it names no site and is selectable
or carries a promotional code.

**WS-325 — Price list availability in a country.** A price list with no country group is available everywhere.
A price list with country groups is available only for a visitor whose country code belongs to one of them,
and for a visitor with no resolvable country.

**WS-326 — Price list selection order.** When a country is known, the country-group price lists win. When none
matches, the site price lists that have no country group are used. The signed-in contact's assigned price list
is added when it is usable on the site and available in the country.

**WS-327 — Selectable filter.** The price list selector offers only price lists that are selectable, plus the
one currently in force.

**WS-328 — Explicit selection persistence.** An explicitly selected price list is remembered separately. After
an address change it is re-applied when it is still usable on the site and in the new country, and forgotten
otherwise.

**WS-329 — Price list freshness.** A cached price list older than 3600 seconds is discarded on the next shop
listing request and resolved again.

**WS-330 — Price list reset at sign-in.** Signing in clears the cached price list, the explicitly selected
price list and the cached fiscal position before the redirect.

**WS-331 — Promotional code lookup.** A promotional code is matched exactly against the price list code, with
elevated privileges, and is applied only when the price list is available on the site. A failure answers with
a redirect carrying the code-not-available marker.

**WS-332 — Site of a price list.** A price list that names both a company and a site must name a site of that
company. Message:
`Only the company's websites are allowed.\nLeave the Company field empty or select a website from that company.`

**WS-333 — Price list cache invalidation.** Creating, writing or deleting a price list clears the memoised
storefront price list resolution.

**WS-334 — Default price list of a new company.** A price list created automatically for a company carries no
site.

**WS-335 — Strikethrough on the catalogue.** A strikethrough price is shown when the applied price list rule
is a percentage rule, or a formula rule with a non-zero discount whose base is the sales price or another
price list, and the resulting base price is strictly greater than the price. Otherwise, when the
comparison-price feature is enabled and the product carries a comparison price, that price is shown instead.
The two are never shown together.

**WS-336 — The comparison price is never taxed.** The comparison price is converted into the display currency
but never passed through the tax computation, because it is meant to be printed exactly as entered.

**WS-337 — Attribute extra price display.** An attribute value's extra price is shown only when the applied
price list rule is not a fixed-price rule. It is converted into the display currency and passed through the
same tax treatment as the price.

**WS-338 — A zero price hides the price.** When the site forbids zero-price sales and the computed price is
zero, the price is hidden, the comparison price is forced to zero, the quick-add action is suppressed and the
add-to-cart permission check fails.

**WS-339 — Cart line pricing date.** A draft storefront order line prices itself at the present moment, not at
the order creation date, therefore a price list rule that starts or ends during the shopper's session takes
effect immediately.

## Cart

**WS-340 — Cart ownership.** A session cart is dropped when it no longer exists, when its state is not draft,
when its last portal transaction is pending, authorised or completed, or when it belongs to another site.

**WS-341 — Cart adoption at sign-in.** When a signed-in user's contact differs from the cart's contact, the
cart's contact is rewritten and the price list, fiscal position, taxes and prices are recomputed.

**WS-342 — Abandoned cart adoption.** A signed-in user with no session cart adopts their first draft order on
this site, provided their contact is allowed to trade with the site's company. The adopted cart has its
addresses re-propagated and its archived-product lines deleted.

**WS-343 — Company consistency.** A storefront order may not be created with a company different from the
site's company. Message:
`The company of the website you are trying to sell from (%(website_company)s) is different than the one you want to use (%(company)s)`

**WS-344 — Whole quantities.** The storefront truncates every requested quantity to a whole number before
applying it.

**WS-345 — Add-to-cart permission.** A product may be added to the cart when the caller holds the
administrator group, or when the variant is active, published, matches the site sellable-product condition,
has a non-zero price whenever the site forbids zero-price sales, and the site grants shop access. Failure
message: `The given product does not exist therefore it cannot be added to cart.`

**WS-346 — Unit of measure.** A unit other than the product's own may be used only when the product supports
several units and the requested unit is among them. Message:
`This product is not available (anymore) in this unit of measure.`

**WS-347 — Line merging.** A new add merges into an existing line only when the product, the unit and the
parent line match, the line has no custom values, the line is not a combo item line, and, when the product has
a no-variant attribute offering a real choice, the no-variant values match exactly. A combo product never
merges.

**WS-348 — Combination normalisation.** A requested combination is normalised to the closest possible
combination of the template before the variant is resolved or created. When no variant results:
`The given combination does not exist therefore it cannot be added to cart.`

**WS-349 — Parent line ownership.** A parent line supplied for a linked product must belong to the same order.
Message: `Invalid request parameters.`

**WS-350 — Missing parent mapping.** A linked product whose parent template is not among the lines created
earlier in the same request is a hard failure of the whole add.

**WS-351 — Combo atomicity.** When a combo item cannot be added at all, the combo line and its already created
children are deleted and the response reports a quantity of zero with the child's warning.

**WS-352 — Combo quantity equality.** A combo line and all of its item lines always carry the same quantity,
equal to the smallest quantity available among the items.

**WS-353 — Line deletion.** A quantity of zero or less deletes the line; its linked lines are deleted with it.

**WS-354 — Stale line.** Updating a line that no longer exists answers with the warning
`We weren't able to update your cart. Please refresh your page before trying again.` and changes nothing.

**WS-355 — Zero-priced line.** A line whose priced group totals zero is refused when the site forbids
zero-price sales and the product's service tracking value is not exempt. Message:
`The given product does not have a price therefore it cannot be added to cart.` Combo item lines are exempt,
because a combo is priced through its parent.

**WS-356 — One verification per request.** The post-update verification — delivery re-rating, promotion
refresh, session counter — runs once per request, not once per line.

**WS-357 — Delivery consistency.** A cart that becomes services-only loses its delivery line and its pickup
location. A cart that keeps deliverable products has its delivery line re-rated; a failing rate removes the
line.

**WS-358 — Accessory suggestions.** An accessory is suggested only when it is not already in the cart, may be
quick-added, belongs to the line's company, is a possible variant under the line's combination and, when the
site forbids zero-price sales, has a non-zero price. The suggestions are returned in random order.

**WS-359 — Course quantity.** A course product may appear at most once in a cart. Message:
`You can only add a course once in your cart.`

**WS-360 — Print-on-demand separation.** A goods product whose print-on-demand nature differs from a goods
product already in the cart is refused with quantity zero and the message
`The product %(product_name)s cannot be added to the cart as it requires separate shipping. Please place your order for the current cart first.`

**WS-361 — Cart counter.** The session cart counter is rewritten after every cart change and is the value used
to render the header counter of a cached page.

**WS-362 — Page caching.** A rendered page containing a cart marker attribute may not enter the page cache. A
cached page served to a shopper with a cart has its counter element rewritten from the session.

## Checkout guards

**WS-363 — Cart guard.** A checkout page requires a draft cart with at least one line. A missing or non-draft
cart clears the cart and transaction session keys and redirects to the shop. An empty cart redirects to the
cart page.

**WS-364 — Mandatory account.** When the customer account policy is `mandatory`, the public user is redirected
to the sign-in page with the checkout page as the return path.

**WS-365 — Zero-priced cart.** When the cart contains zero-priced lines while the site forbids them, each such
line receives `This product is not available for purchase in your country.`, the order receives
`Some products in your cart are not available for purchase in your country. Please remove them or contact us.`
and the shopper is redirected to the cart page.

**WS-366 — Address guard.** An anonymous cart is redirected to the address form. An incomplete delivery address
of an order that is not services-only, or an incomplete billing address, is redirected to the address form for
that contact and that address kind, but only when the current customer is allowed to edit that contact.

**WS-367 — Services-only address requirement.** An order containing only services still requires a customer
address, unless the parameter `website_sale.require_billing_details_for_services` is set to a false value.

**WS-368 — Mandatory address fields.** The mandatory set is the name and the electronic mail address, plus,
when an address is required, the telephone, the street, the city, the country, the state when the country
requires a state, and the postal code when the country requires one.

**WS-369 — Partial address completion.** When the submission fills any common address field, the whole address
becomes mandatory even for an order that would not have required one. Message when something is missing:
`Some required fields are empty.`

**WS-370 — Country change.** The country of a contact for which documents have been issued may not be changed.
Message:
`Changing your country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`

**WS-371 — Internal user identity.** The name or the electronic mail address of a contact linked to an
internal user may not be changed from the storefront. Message:
`If you are ordering for an external person, please place your order via the backend. If you wish to change your name or email address, please do so in the account settings or contact your administrator.`

**WS-372 — Commercial fields.** A commercial field submitted on a child address with a different value is
refused. Message: `The %(field_name)s is managed on your company account.` when the commercial contact is a
company, and otherwise `The %(field_name)s is managed on your main account address.` A commercial field
submitted with an unchanged value is silently dropped, as is a company name submitted on a contact that is not
the shopper's own.

**WS-373 — Tax identification number.** The tax identification number of a commercial contact for which
documents have been issued may not be changed. Message:
`Changing VAT number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.` The three capital letters abbreviate value-added tax. A submitted number is also
validated against the country rules, and the validation message is returned verbatim.

**WS-374 — Address format of the electronic mail address.** Message:
`Invalid Email! Please enter a valid email address.`

**WS-375 — Address ownership.** A contact may be edited or selected at checkout only when it is the order's
customer, the order's billing or delivery contact, or a descendant of the customer's commercial contact of the
kind invoice, delivery or other. Otherwise the response is "forbidden".

**WS-376 — New address defaults.** A contact created on an anonymous cart has the kind "contact"; every created
contact receives the site's company, the site salesperson and, when the site keeps accounts separate, that
site. A language outside the site's languages is dropped.

**WS-377 — Public follower removal.** Once a cart stops being anonymous, the site's public contact is
unsubscribed from the order's followers.

**WS-378 — Address propagation.** Writing an address on an order recomputes, in this order: the fiscal position
and, when it changed, the taxes; the explicitly selected price list; the price list and, when it or the fiscal
position changed, the prices; and finally the delivery method when the delivery address changed and the order
has deliverable products.

**WS-379 — Relay address immutability.** An external relay point address may not be edited. Message:
`You cannot edit the address of a Point Relais®.` Its completeness check always succeeds.

## Delivery

**WS-380 — Offered methods.** A method is offered when it is published for this site, belongs to the order's
company or to no company, matches the destination address restrictions, matches the product tag requirements
and exclusions, matches the weight and volume bounds and, for a rule-based method, produces a successful rate.

**WS-381 — Preferred method.** The order keeps its current method when it is still available; otherwise the
delivery contact's preferred method when it is available; otherwise the first available method.

**WS-382 — Delivery line lifecycle.** Setting a method always removes the previous delivery line first. A new
line is created only when the rate succeeds and the order has deliverable products.

**WS-383 — Method change during payment.** The delivery method may not be changed once a transaction exists
whose state is not draft, cancelled or in error. Message:
`It seems that there is already a transaction for your order; you can't change the delivery method anymore.`

**WS-384 — Rate for an unavailable method.** Requesting the rate of a method that is not in the available list
answers
`It seems that a delivery method is not compatible with your address. Please refresh the page and try again.`
Requesting a rate without a cart answers `Your cart is empty.`

**WS-385 — Rate taxation.** A displayed rate is passed through the taxes of the delivery product mapped by the
order's fiscal position, and returned tax excluded when the site displays tax-excluded prices and tax included
otherwise. In express mode the tax-included value is always used.

**WS-386 — Free shipping threshold.** When the method offers free shipping above a threshold, is not a
rule-based method, and the order total excluding delivery, converted to the company currency, is at or above
the threshold, the rate becomes zero with the warning
`The shipping is free since the order amount exceeds %.2f.`, whose placeholder prints the threshold with two
decimal places.

**WS-387 — Pickup location.** A pickup location is stored only when the current method declares that it uses
locations. Clearing it recomputes the warehouse. Setting it on a collect-in-store method sets the order
warehouse to the store and recomputes the fiscal position and, when it changed, the taxes.

**WS-388 — Collect-in-store publication.** A published collect-in-store method must have at least one store.
Message: `The delivery method must have at least one warehouse to be published.`

**WS-389 — Collect-in-store company.** A collect-in-store method and its stores must share the same company.
Message: `The delivery method and a warehouse must share the same company`

**WS-390 — Collect-in-store defaults.** A collect-in-store method is forced to rate-only integration, cash on
delivery disabled and no country, state or postal-code restriction; on creation it receives every warehouse of
its company and is published when at least one exists.

**WS-391 — Store payload completeness.** A store whose address cannot produce a complete payload is omitted
from the selector. A store address that cannot be geolocated receives the impossible coordinates 1000 and 1000,
which prevents repeated lookups and sorts it last.

**WS-392 — No pickup point available.** When no pickup location can be returned:
`No pick-up points are available for this delivery address.`

**WS-393 — Temporary order for locations.** Fetching pickup locations from a product page without a cart must
not create a cart; an unsaved order is used instead.

**WS-394 — Express checkout exclusions.** Methods that require a pickup point, and collect-in-store methods,
are never offered in express checkout.

**WS-395 — Relay method pairing.** A relay delivery address requires the relay method
(`Point Relais® can only be used with the delivery method Mondial Relay.`) and the relay method requires a
relay delivery address (`Delivery method Mondial Relay can only ship to Point Relais®.`).

**WS-396 — Relay country.** A relay point in a country the method does not serve is refused:
`%s is not allowed for this delivery carrier.` A relay selection on an anonymous cart is refused:
`Customer of the order cannot be the public user at this step.`

## Availability

**WS-397 — Unlimited products.** A product that is not tracked, or whose out-of-stock ordering is allowed, is
never limited and never declared sold out.

**WS-398 — Storefront quantity free to use.** Before checkout, the quantity free to use is evaluated with the
site warehouse in context; with no site warehouse, every warehouse of the company counts.

**WS-399 — Whole-unit availability.** A displayed or enforced quantity free to use is rounded down to a whole
number.

**WS-400 — Quantity cap.** A requested quantity that would make the total quantity of that product across the
whole cart exceed the quantity free to use is capped at what the edited line may hold, with one of these
messages: `You ask for %(desired_qty)s %(product_name)s but only %(new_qty)s is available` for an existing
line; `You ask for %(desired_qty)s products but only %(available_qty)s is available.` for a new line;
`Some products became unavailable and your cart has been updated. We're sorry for the inconvenience.` when the
line must disappear; and
`%(product_name)s has not been added to your cart since it is not available.` when no line could be created.

**WS-401 — Availability before payment.** Every line is re-checked before a transaction is created; the
collected warnings are raised joined by single spaces.

**WS-402 — In-store availability.** For a collect-in-store method, every storable product must be available at
the selected store. Message: `Some products are not available in the selected store.` The per-line warning is
`%(available_qty)s/%(line_qty)s available at this location`.

**WS-403 — Availability threshold.** The remaining quantity is shown only when the product asks for it and the
remaining quantity, minus what is already in the cart, is at or below the product threshold and above zero.

**WS-404 — Back-in-stock eligibility.** A subscription requires a syntactically valid address
(`Invalid Email`), an active, sellable, published product
(`This product is not eligible for stock notifications.`) and, for an anonymous visitor, an address that does
not already belong to an account (`Please sign in to proceed.`).

**WS-405 — Single-shot notification.** A subscriber is removed from the product's list as soon as the
notification has been created, whether or not it was delivered.

**WS-406 — Combo availability.** A combo's maximum is the largest maximum among its items; a product's maximum
across several combos is the smallest of the combo maximums. An unknown maximum anywhere makes the result
unknown.

## Payment and confirmation

**WS-407 — Payment readiness.** Payment requires a non-empty cart with no zero-priced line
(`Your cart is not ready to be paid, please verify previous steps.`) and, for an order that is not
services-only, a delivery method (`No shipping method is selected.`) that is still among the available ones
(`The delivery method is not compatible with your delivery address.`).

**WS-408 — Concurrent payment.** The order row is locked without waiting while a transaction is created. A
conflict answers `Payment is already being processed.`

**WS-409 — Access token.** A transaction may only be created with a valid order access token. Message:
`The access token is invalid.`

**WS-410 — Cancelled order.** Message: `The order has been cancelled.`

**WS-411 — Amount agreement.** The requested amount must equal the order total at currency precision. Message:
`The cart has been updated. Please refresh the page.` An order already fully paid answers
`The cart has already been paid. Please refresh the page.`

**WS-412 — Transaction argument allow list.** Unexpected arguments in a transaction creation request are
rejected.

**WS-413 — One transaction reference.** The session keeps only the last transaction identifier for the cart.

**WS-414 — Site-scoped providers.** Only providers with no site, or with the current site, are compatible.
Excluded providers are reported with the reason "incompatible website".

**WS-415 — Pay on site.** A pay-on-site provider is offered only for a collect-in-store method with at least
one goods line; otherwise it is excluded with the reason `no in-store delivery methods available`. Using it
without a collect-in-store method is refused:
`You can only pay on site when selecting the pick up in store delivery method.`

**WS-416 — Promotion revalidation.** When re-evaluating the promotions at transaction time changes the order
total, the payment is refused:
`Cannot process payment: applied reward was changed or has expired.\nPlease refresh the page and try again.`

**WS-417 — No stored tokens in express checkout.** Express checkout never offers a stored payment token.

**WS-418 — Blocking payment errors.** When no delivery method is available for a deliverable order, the payment
form is hidden and the shopper sees `Sorry, we are unable to ship your order.` with
`No shipping method is available for your current order and shipping address. Please contact us for more information.`
For a collect-in-store method without a store: `Please choose a store to collect your order.`

**WS-419 — Salesperson assignment.** A draft storefront cart never receives a salesperson. On confirmation, and
before the payment-succeeded message, the assignment is forced and resolves to the site salesperson, else the
contact's salesperson, else the parent contact's salesperson, and it is performed as the platform system user.

**WS-420 — Signature.** A storefront order never requires a signature.

**WS-421 — Payment terms.** A storefront order with no payment terms receives the immediate-payment term when
it belongs to the order's company or to no company, and otherwise the first payment term of the order's
company.

**WS-422 — Confirmation message.** The site's confirmation template is used when set; otherwise the platform
default applies.

**WS-423 — Free order.** An order whose total is zero and that has no transaction is confirmed directly by the
payment validation endpoint, after the readiness check.

**WS-424 — Draft transaction return.** Returning from a provider with a transaction still in draft sends the
shopper back to the shop rather than to the confirmation page.

**WS-425 — Confirmation page source.** The confirmation page and the printed order document are served from the
session key holding the last order identifier, which survives the cart reset.

**WS-426 — Automatic invoicing.** When the platform parameter `sale.automatic_invoice` is true, a completed
transaction also creates, posts and sends the invoice, including for a partial payment. Sending is deferred to
the invoice sending job when asynchronous messages are enabled.

**WS-427 — Wire transfer.** A wire-transfer transaction stays pending; the order only reaches the sent state,
no stock is reserved and no invoice is produced until a person confirms the order.

## Abandoned carts

**WS-428 — Definition.** A cart is abandoned when it has a site, is in draft, has at least one line, its
customer is not the site's public contact, and its order date is at or before the present moment minus the
site's abandoned-cart delay. A delay of zero is read as one hour.

**WS-429 — Activation window.** Only carts whose order date is at or after the moment the recovery feature was
switched on are mailed.

**WS-430 — Eligibility.** A cart is mailed only when its customer has an electronic mail address, no
transaction of the order is in error, at least one line has a non-zero unit price, the customer has not placed
a confirmed order since, and, with the stock capability, no product of the cart is sold out.

**WS-431 — Single message.** Every examined cart is marked as mailed, whether it was mailed or rejected, and is
never examined again.

**WS-432 — Template resolution.** The site template is used when every order of the batch belongs to one site;
otherwise the shipped template; otherwise no template.

**WS-433 — Recovery link.** The recovery notification button is labelled `Resume Order` and points at the cart
address carrying the order identifier and the order access token.

**WS-434 — Recovery token.** The token is compared in constant time; a mismatch or an unknown order answers
"not found".

**WS-435 — Merge.** Merging moves the lines of the old order onto the current cart and cancels the old order.

## Product feed

**WS-436 — Feature flag and token.** The feed endpoint answers "not found" when the site has the feature
disabled. The access token is compared in constant time; a mismatch answers "forbidden"; an unknown feed
answers "not found"; a non-numeric feed identifier answers "bad request"; a feed of another site answers "bad
request" with `Website does not match.` Writing the feature setting propagates the same value to every site.

**WS-437 — Soft limit.** A feed may not be configured to contain more than 5000 products. Message:
`A single feed cannot contain more than %(limit)s products. Please separate products with Categories.`, with
the limit printed as `5,000`.

**WS-438 — Hard limit.** At most 6000 products are rendered, whatever the configuration.

**WS-439 — Warning.** When more than 5500 products are rendered and no warning was sent in the last week, the
site salesperson is notified with the subject `GMC: Product Limit Exceeded`, reproduced verbatim — the three
capital letters abbreviate the name of the Google Merchant Center product listing service — and the body
`The feed %(feed_name)s contains more than %(limit)s products, which may not be fully updated. Consider refining the feed by adjusting the product categories.`

**WS-440 — Daily cache.** The document is rendered at most once per day per feed, under an exclusive row lock.
Any parameter change invalidates the cache; a manual reset answers with `Feed cache successfully reset.`

**WS-441 — Feed price list.** Only a selectable price list that is generic or belongs to the feed's site may be
chosen. When one is chosen, product addresses in the feed carry it, which keeps the advertised price and the
landing page price identical.

**WS-442 — Feed language.** Only a language installed on the site may be chosen; the default is the site's
default language.

**WS-443 — Feed exclusions.** A product with no price payload — that is, a zero-priced product on a site that
forbids zero-price sales — and a variant that is not a possible combination are excluded. A product with no
picture publishes an empty picture address rather than a placeholder.

## Wish list and comparison

**WS-444 — Uniqueness.** One wish list row per product and contact. Message:
`Duplicated wishlisted product for this partner.`

**WS-445 — Anonymous ownership.** An anonymous row may be read or deleted only when its identifier is present
in the session list.

**WS-446 — Visibility filter.** A wish list listing hides rows whose product is unpublished or no longer
addable to the cart, without deleting them.

**WS-447 — Merge at sign-in.** Session rows whose product the contact already owns are deleted; the rest are
re-assigned; the session list is cleared.

**WS-448 — Cleanup.** Rows without a contact older than five weeks are deleted, archived rows included.

**WS-449 — Price capture.** The price and the price list in force at the moment of saving are stored on the row.

**WS-450 — Comparison access.** The comparison page reads the requested variants through a search, which
silently drops the ones the caller may not see; an empty or invalid selection redirects to the shop.

## Promotions on the storefront

**WS-451 — Program scope.** A program applies to a storefront order only when it is marked available on the
site and is generic or attached to that site.

**WS-452 — Nominative programs.** Nominative programs are not offered to anonymous visitors.

**WS-453 — Automatic rewards.** A reward is applied automatically only when its program carries exactly one
reward, the program is not nominative, the reward is not a multi-product free-product reward, it is not in the
order's disabled list and it is not already on a line.

**WS-454 — Reward removal.** Removing a reward line records the reward in the order's disabled list.

**WS-455 — Reward display.** Reward lines do not count in the cart quantity, never show a strikethrough price,
are excluded from reordering, and only free-product rewards are treated as sellable. Discount lines of one
reward split across tax groups are displayed as one aggregate line.

**WS-456 — Free shipping and thresholds.** Free-shipping reward lines are excluded from the amount used by the
free-shipping threshold.

**WS-457 — Pending coupon.** A coupon opened without a cart is stored in the session and applied on the first
cart update; the shopper is told
`The coupon will be automatically applied when you add something in your cart.`

**WS-458 — Code uniqueness.** Two promotional codes may coexist only when they can never be reached from the
same site. Messages: `The promo code must be unique.` and `A coupon with the same code was found.`

**WS-459 — Coupon cleanup.** Coupons attached to carts untouched for longer than the configured number of days
are released and the carts are re-evaluated.

## Consistency and technical guarantees of the storefront

**WS-460 — One canonical product address.** The canonical address of a product page never contains a category
segment. A product page opened with a category the product does not belong to redirects permanently to the
canonical address.

**WS-461 — Access failures on catalogue pages.** An access failure on a catalogue page produces a redirect or
"not found", never "forbidden", which prevents the existence of unpublished products from being probed.

**WS-462 — Variant creation opacity.** The variant creation endpoint returns the variant identifier whether the
variant already existed or was just created, and zero for every failure.

**WS-463 — Sort stability.** Every catalogue search is ordered by publication flag descending, then the chosen
order, then identifier descending, which makes pagination stable.

**WS-464 — Shop ordering value uniqueness.** A new product receives a shop ordering value five above the
current maximum, or 10000 for the first product. Moving a product up or down swaps its ordering value with the
neighbour that has the same publication state.

**WS-465 — Checkout step copies.** Every site owns a full copy of the generic checkout steps; the
extra-information copy is published exactly when the corresponding page option is active.

**WS-466 — Category style exclusivity.** Enabling one category-style page option disables the other five.

**WS-467 — Checkout step translations.** Installing or updating a language copies the newly loaded translations
of the generic Website Checkout Step rows onto the per-site copies that carry the same step path. The fields
copied are the three translatable ones: the step label, the forward label and the backward label. In overwrite
mode the incoming translation wins, otherwise the existing one does, and a null value never overwrites a
present one. The rule does nothing when no capability package was loaded, or when the only loaded language is
the source language of the shipped terms.

**WS-468 — Settings allow list.** The storefront page settings endpoint writes only the fields of its explicit
allow list; a products-per-page value of zero is stored as 1; the media grid column count is coerced to a whole
number.

**WS-469 — Session payload.** Every storefront page receives, in its session payload, the site's add-to-cart
action, which tells the page whether to stay on the product page or to navigate to the cart after an add.

---

# Part 3: Consent, indexing, themes, configurator, blogs and profiles

## Cookie and third-party content rules

**WS-470 — Required cookies are always allowed.** The permission check answers true for the required category
unconditionally.

**WS-471 — Optional cookies without a consent bar are allowed.** When the current site's consent bar is
disabled, optional cookies are allowed, on the assumption that the site implements its own consent mechanism.

**WS-472 — Optional cookies with a consent bar follow the stored answer.** The consent cookie stores a
structured document mapping categories to booleans. The optional category is allowed only when the document
contains it with the value true. A missing entry means refused.

**WS-473 — A legacy consent value is discarded.** When the consent cookie does not contain a structured
document, the cookie is deleted with a zero lifetime and the answer is refused, so that the visitor is asked
again.

**WS-474 — All consents granted.** All consents are considered granted when the consent bar is disabled or when
the optional category is allowed. This value participates in the page cache key of WS-070.

**WS-475 — Neutralisation preconditions.** Neutralisation applies when the consent bar is enabled, third-party
blocking is enabled, optional cookies are refused and the viewer is not in the Restricted Editor group.

**WS-476 — Domain matching for blocking.** A source host matches a blocked domain when it equals the domain
with a leading `www.` removed, or when it ends with a dot followed by that domain. A domain therefore blocks
its own subdomains but not another domain that merely ends with the same letters.

**WS-477 — Blocked container classes.** In addition to source hosts, an element is neutralised when one of its
classes is the map block, the picture-network page block, the social-network page block, the background video
block or the embedded video frame.

**WS-478 — Neutralisation transformation.** A neutralised element receives `data-need-cookies-approval` set to
`true`; when it carries a source, the source is moved to `data-nocookie-src` and replaced by the blank document
address.

**WS-479 — Consent-aware audience measurement.** When the measurement identifier is set and the page is not in
editing mode, the measurement script is loaded with the four consent categories denied and grants them either
immediately, when all consents are granted, or on the one-shot acceptance event.

## Site index and crawler rules

**WS-480 — Site index page size.** At most 45000 locations per index document.

**WS-481 — Site index cache lifetime.** A cached index document is reused for twelve hours; after that every
cached document of that site and host is deleted and the index is regenerated.

**WS-482 — Site index cache key.** The cached document address carries the site identifier and an
eight-character digest of the request root, so that a site served on several hosts keeps one index per host.

**WS-483 — An empty index answers not found.** When the enumeration produced no location at all, the request
answers "not found" rather than an empty document.

**WS-484 — Index of indexes.** When more than one document was produced, a document listing them is written at
the base address and served.

**WS-485 — The index runs as the public user.** Enumeration is performed with the site's public user, so that
the index never leaks addresses a visitor cannot reach.

**WS-486 — Page eligibility for the index.** A page is enumerated when it is indexed, its visibility is public,
it is published, its publishing date is empty or already past, and its address is not `/`. The address `/` is
excluded because the root endpoint is enumerated separately.

**WS-487 — Index priority.** A page whose template priority differs from the neutral value 16 contributes a
priority computed as in [calculations.md](calculations.md) §7.

**WS-488 — Index last modification.** The contributed date is the later of the page modification moment and the
template modification moment, reduced to a date; when neither exists, no date is contributed.

**WS-489 — Endpoint eligibility for the index.** An endpoint is enumerated only when it answers the read verb,
is a rich-text endpoint, requires no authentication or public authentication, declares itself a front-end
endpoint, every converter can generate values, and every positional parameter without a default has a
converter.

**WS-490 — Explicit index declarations win.** An endpoint that declares its index value as false is never
enumerated. An endpoint that declares a generator is enumerated by calling the generator once per distinct
generator.

**WS-491 — Locations are deduplicated and de-slashed.** Every emitted location has trailing slashes removed,
except the root, and is emitted at most once per generation.

**WS-492 — Record converters are scoped to the site.** A record converter whose entity carries a site dimension
and that declares no condition receives the condition "site is empty or the current site".

**WS-493 — Crawler exclusion when the host is not canonical.** When the site declares a domain and the request
root does not match it, the crawler exclusion response contains `Disallow: /` and points at the site index of
the canonical domain.

**WS-494 — Crawler exclusion custom block.** Otherwise the response points at the site index of the request
root and appends the site's custom crawler exclusion text under a banner block.

## Theme rules

**WS-500 — Theme processing order.** Template entities are processed in the fixed order templates, assets,
pages, menus, attachments, because pages need templates and menus need pages.

**WS-501 — Deferred conversion.** A theme record whose dependency has not yet produced a copy for this site is
skipped in the current pass and retried in a later pass over the same entity.

**WS-502 — Copies are updated, not duplicated.** When a copy already exists for this site, it is updated with
the converted values except the fields the update protects; otherwise a copy is created.

**WS-503 — Orphan cleanup.** For key-bearing entities — templates, assets, attachments — records generated for
this site whose theme record no longer exists are deleted. Pages and menus are never cleaned this way, because
they carry no key.

**WS-504 — Cleanup requires editor rights.** The cleanup operation refuses a caller outside the Restricted
Editor group with a forbidden error.

**WS-505 — Scope of a theme load.** A package installation loads the theme for every site whose own theme is in
the same stream, that is the theme itself, the packages it depends on and the packages that extend it. When the
operation was started from the interface, the scope is narrowed to the current site through a configuration
parameter.

**WS-506 — Changing theme resets the configuration first.** Choosing or removing a theme always resets the
default style configuration: the nine style variables are unset, the two ripple assets are disabled, every
non-default header and footer template is disabled, the default ones are enabled, and the footer scroll-to-top
option is disabled.

**WS-507 — Unload order.** Themes of the stream are unloaded in reverse installation order.

**WS-508 — Deleting a theme record deletes its copies.** When a theme package is uninstalled or a theme record
is removed, the copies of that record are deleted; when a site restriction parameter is set, only the copies of
that site are deleted.

**WS-509 — Theme translations follow the copies.** Translated fields of theme records — the template
architecture and the menu name — have their translations copied onto the generated records for every activated
language.

## Configurator rules

**WS-510 — Configurator access.** The configurator endpoint answers "not found" to a caller outside the Editor
and Designer group, redirects to the site when the site's configurator is already done, and redirects to the
default language prefix when the current language is not the site's default language.

**WS-511 — Exactly one backing for a feature.** A configurator feature must carry either a page template or a
package, not both and not neither. Message:
`One and only one of the two fields 'page_view_id' and 'module_id' should be set`

**WS-512 — Company grouping menu threshold.** A grouping entry named `Company` is created only when more than
five selected features request a menu entry and more than one of them asks for the grouping.

**WS-513 — Text generation threshold.** Generated text is requested only when the translation coverage of the
rendered blocks exceeds 0.8. At or below that value the shipped texts are kept and the decision is recorded.

**WS-514 — Generated text personalisation.** Every occurrence of the marker `XXXX` in a generated text is
replaced by the site name, escaped for use in a matching pattern.

**WS-515 — Configurator pages are preserved as templates.** After the blocks are written into a page template,
that template is duplicated under a key made of the index, an underscore, the original key and the suffix
`_configurator_pages_landing`, and bound to the site, so that the untouched page remains available in the
new-page template picker.

**WS-516 — Industry pictures are downloaded once.** Each industry picture is stored as a public attachment of
the site registered under an external identifier made of `configurator_`, the site identifier, an underscore
and the picture name; a picture whose identifier already exists is skipped. The download timeout is three
seconds and a failure is recorded without failing the run.

**WS-517 — Picture fallbacks.** When an industry did not provide a picture that a block needs, a fixed mapping
copies the bytes of another picture of that industry under the missing name.

**WS-518 — Skipping the configurator installs the default theme.** Skipping sets the completion flag and
installs the shipped default theme.

**WS-519 — Service failures never block the configurator.** A failure to reach the industry catalogue yields an
empty industry list; a failure to reach the text generation service leaves the shipped texts in place; a
failure to download a picture is recorded and skipped.

## Blog rules

**WS-520 — Tag name uniqueness.** Message: `Tag name already exists!`

**WS-521 — Tag category name uniqueness.** Message: `Tag category already exists!`

**WS-522 — Archiving a blog archives its posts.** Writing the active flag on a blog writes the same value on
every post of that blog, already archived ones included.

**WS-523 — Archiving a post unpublishes it.** Writing the active flag as false also writes the publication flag
as false.

**WS-524 — Publishing date maintenance.** When the publication flag is written without an explicit publishing
date and the post has no publishing date or its publishing date is already past, the publishing date is set to
the current moment on publication and cleared on unpublication. A future publishing date is never overwritten.

**WS-525 — Effective publishing moment.** The effective publishing moment is the publishing date when set and
otherwise the creation moment. Writing it writes the publishing date; clearing it falls back to the creation
moment.

**WS-526 — Publication notification.** Publishing a non-archived post posts a message on the parent blog,
rendered from the shipped new-post template, with the post title as subject and the publication subtype.

**WS-527 — Replies to a publication notification are notes.** A comment posted as a reply to a message whose
subtype is the publication subtype is downgraded to an internal note, so that blog followers are not notified
of every answer.

**WS-528 — Comments are not pushed to the inbox.** Notifications for messages of the comment kind on a blog
post are not recorded as inbox items; only outgoing electronic mail is used.

**WS-529 — Comments visible on the site.** The public comment list of a post contains the messages of that post
whose kind is comment, that are not internal and whose subtype is not internal.

**WS-530 — Post thread access.** Read access on a post is sufficient to post a message on it.

**WS-531 — Teaser derivation.** When a manual teaser exists it is used verbatim; otherwise the teaser is the
first 200 characters of the whitespace-collapsed plain text of the content, followed by the ellipsis `...`.

**WS-532 — Writing the teaser writes the manual teaser.** When the source-language manual teaser is empty, it
is first explicitly cleared in the source language, so that writing a translation does not also write the
source value; then the manual teaser receives the written teaser.

**WS-533 — Duplicating a post renames it.** The copy's title is the original title followed by ` (copy)`.

**WS-534 — Post address.** The post address is the blog path, the blog slug and the post slug. A post reached
through the wrong blog is redirected permanently to the correct address. A post reached through the legacy
address form that inserts the segment `post` between the two slugs is redirected permanently to the current
form.

**WS-535 — Post visibility for non-designers.** A user outside the Editor and Designer group sees only posts
whose publishing date is past; a request for any other post redirects to the blog listing.

**WS-536 — Next post wrap-around.** The next post is the following identifier in the visible set of the blog,
wrapping to the first. A blog with a single visible post has no next post.

**WS-537 — View counting.** The view counter is increased by one, without locking, the first time a post is
opened in a session; the session then remembers the post identifier.

**WS-538 — Single blog redirection.** A request to the blog index when exactly one blog exists redirects
temporarily to that blog's address.

**WS-539 — Multiple tags are reduced to one.** A read request carrying several tags redirects temporarily to
the first one.

**WS-540 — Tag slugs are normalised.** Unknown tag identifiers are dropped and the remaining ones are
re-slugified; when the result differs from the requested list the visitor is redirected permanently.

**WS-541 — Blog access action.** Opening a post from a notification redirects an external user to the public
post address when the post is published; otherwise the record form is opened. When the post is published, every
recipient group gets the access button.

**WS-542 — Feed size.** The subscription feed contains the requested number of the most recent posts, capped at
50 and defaulting to 15, ordered by publishing date descending.

**WS-543 — Blog listing page size.** Twelve posts per page, chosen as a common multiple of 2, 3 and 4 so that
the grid is always complete.

**WS-544 — Blog recipient privacy.** Blogs never disclose recipients to each other; the recipient header limit
is zero.

## Public contact reference and profile rules

**WS-550 — Contact reference visibility.** A contact's public page is reachable when the contact is published,
or when the viewer is in the Restricted Editor group. Any other case answers "not found".

**WS-551 — Contact slug redirection.** A request on a stale contact slug redirects to the current slug.

**WS-552 — Reference directory condition.** The directory lists contacts that are published and have an
assigned partner, optionally narrowed by industry, country and a published Partner Website Tag, and optionally
filtered by a term matched against the name, the public description and the industry name.

**WS-553 — Country fallback.** When the requested country yields no contact but other countries do, the country
filter is dropped and the listing reports the fallback.

**WS-554 — Reference directory page size.** Twenty contacts per page.

**WS-555 — Map limit.** The map frame renders at most the requested number of contacts, defaulting to 80, and
only contacts that are published; when neither an explicit contact list nor a named condition is supplied, no
contact is rendered.

**WS-556 — Map contact list mode.** When an explicit contact list is supplied, only company contacts of that
list are rendered.

**WS-557 — Profile access.** Viewing another account's profile requires the account to be published and the
viewer's reputation score to reach the site's minimum. Refusals render the denial page with
`This profile is private!` or `Not have enough karma to view other users' profile.` An account always sees its
own profile. A non-existing account answers "not found".

**WS-558 — Profile avatar access.** Only the four permitted picture sizes are served; elevated reading is used
only when the account is published and its reputation score is strictly positive. Any other requested field
answers "forbidden".

**WS-559 — Profile editing scope.** The name, the personal site, the electronic mail address, the city, the
country and the public description are writable; the publication flag is writable only by the owner; an
administrator may edit another account.

**WS-560 — Profile validation token.** The token is the digest of the current day at midnight, a stored secret,
the account identifier and the address, so that it is valid for one day. Consuming a valid token for an account
whose reputation score is zero raises the score to 3; any other case leaves the score unchanged.

## Consistency and locking rules

**WS-570 — Visitor updates avoid lock contention.** The two statements that refresh a visitor's time zone and
last connection moment select the row for update while skipping locked rows, so that a concurrent request never
blocks and never fails.

**WS-571 — Visitor creation is a single statement.** Creating or refreshing a visitor and recording its track
is performed by one insert-or-update statement, so that two concurrent first requests of the same visitor
cannot both insert.

**WS-572 — View counting avoids locking.** The blog post view counter is increased with a lock-free increment
that skips locked rows.

**WS-573 — The routing cache is cleared on every worker.** Rules that change the routing table clear the
routing cache registry-wide, not only in the current process.

**WS-574 — The template cache is cleared on every structural change.** Creating, writing or deleting a menu
entry, deleting a page or a model page, and writing a page address, visibility or authorised groups clear the
template cache.

**WS-575 — The whole registry cache is cleared when a site changes.** Any write on a Website clears the registry
cache, because site settings participate in compiled templates, in routing and in record rules.

---

# Part 4: Forums, tracked links and public actions

## Forum access and reputation gates

Every reputation threshold below is a field of the Forum record, so two forums may demand different scores for
the same operation; the defaults are listed in [entities.md](entities.md) §4.1. In every message the
placeholder is replaced by the numeric threshold, and the word reproduced in the message for the reputation
score is the one the shipped text uses.

**WS-580 — Forum privacy gate.** A forum whose privacy is `public` is reachable by anyone; a forum whose privacy
is `connected` refuses the public user; a forum whose privacy is `private` is reachable only by members of its
authorised group. Setting the privacy to `public` or `connected` clears the authorised group.

**WS-581 — Reputation to ask.** Creating a question below the asking threshold is refused with
`%d karma required to create a new question.`

**WS-582 — Reputation to answer.** Creating an answer below the answering threshold is refused with
`%d karma required to answer a question.`

**WS-583 — Reputation to edit.** Writing any field outside the trusted set below the editing threshold is
refused with `%d karma required to edit a post.` The threshold is the own-post threshold when the current user
is the author and the all-posts threshold otherwise. The trusted set is the active flag, the acceptance flag
and the tag list, plus the state and its closing fields when the state is written to `active` or `close`, plus
the state and the flagging user when the state is written to `flagged`.

**WS-584 — Reputation to close or reopen.** Writing the state to `active` or `close` below the closing
threshold is refused with `%d karma required to close or reopen a post.`

**WS-585 — Reputation to create a tag.** Creating a Forum Tag below the tag-creation threshold is refused with
`%d karma required to create a new Tag.` During tag parsing, a new name is silently dropped instead when the
participant may not create tags.

**WS-586 — Reputation to retag.** Changing the tag list below the retagging threshold is refused with
`%d karma required to retag.`

**WS-587 — Reputation to comment.** Posting a comment below the commenting threshold is refused with
`%d karma required to comment.`

**WS-588 — Reputation to convert.** Converting a comment into an answer below the conversion threshold is
refused with `%d karma required to convert your comment to an answer.` when the participant is the comment
author and the own threshold is lower, and with `%d karma required to convert a comment to an answer.`
otherwise. Converting an answer into a comment is refused with
`%d karma required to convert an answer to a comment.`

**WS-589 — Reputation to delete a comment.** Deleting a comment below the matching threshold is refused with
`%d karma required to delete a comment.`

**WS-590 — Reputation to flag.** Writing the state to `flagged` below the flagging threshold is refused with
`%d karma required to flag a post.`

**WS-591 — Reputation to moderate.** Validating, refusing and marking offensive below the moderation threshold
are refused with `%d karma required to validate a post.`, `%d karma required to refuse a post.` and
`%d karma required to mark a post as offensive.` respectively.

**WS-592 — Reputation to delete or reactivate.** Writing the active flag below the deletion threshold is
refused with `%d karma required to delete or reactivate a post.`, and deleting a post outright with
`%d karma required to unlink a post.` The threshold is the own-post threshold for the author and the all-posts
threshold otherwise.

**WS-593 — Reputation to accept.** Writing the acceptance flag below the acceptance threshold is refused with
`%d karma required to accept or refuse an answer.` The threshold is the own-question threshold when the current
user asked the question and the all-questions threshold otherwise.

**WS-594 — Reputation to vote.** Up-voting below the up-vote threshold is refused with
`%d karma required to upvote.` and down-voting below the down-vote threshold with
`%d karma required to downvote.` A participant whose present vote is the opposite one may always withdraw it,
whatever their score.

**WS-595 — No self-voting.** A participant may not vote on their own post:
`It is not allowed to vote for its own post.`

**WS-596 — No voting for somebody else.** A participant may not change another participant's vote:
`It is not allowed to modify someone else's vote.` A caller who is not an administrator may never write the
voter or the beneficiary; both are dropped from the submitted values.

**WS-597 — One vote per participant and post.** Database constraint on the pair of post and voter. Message:
`Vote already exists!`

**WS-598 — No cycles among posts.** The answer chain of a post may not contain the post itself. Message:
`You cannot create recursive forum posts.`

**WS-599 — Answering a closed or deleted question.** Message:
`Posting answer on a [Deleted] or [Closed] question is not possible.`

**WS-600 — Empty title and empty content.** A blank title is refused with `Title should not be empty.`; a
visually empty content is refused with `Question should not be empty.` for a question and
`Reply should not be empty.` for an answer. All three are rendered as a bad-request page.

**WS-601 — Links are not followed below the threshold.** When the author's reputation is below the no-follow
threshold, every link element in the content is rewritten with the no-follow marker, keeping its address.

**WS-602 — Pictures and links require the editor threshold.** When the author's reputation is below the editor
threshold and the content contains a picture element, a link element or an element whose style declares a
background picture, the write is refused with `%d karma required to post an image or link.`

**WS-603 — Biography visibility.** The author's biography is shown on a post only when the author's reputation
reaches the biography threshold and the author's profile is published.

**WS-604 — Questions below the validation threshold wait.** A question created by an author whose reputation is
below the validation threshold is forced into the pending state and awards no reputation until it is validated.

**WS-605 — Post visibility.** A post may be viewed when the current user may close posts, or the post is active
and its author has a strictly positive reputation, or the current user is the author. The rule is searchable, so
that listings, the site search and the site index apply the same filter.

**WS-606 — Forum tag uniqueness.** A tag name is unique per forum. Message: `Tag name already exists!`

**WS-607 — One answer per question in questions mode.** In a forum whose answering mode is `questions`, a
participant may post only one answer per question; in `discussions` mode several are allowed.

**WS-608 — Archiving cascades.** Writing the active flag on a post writes the same value on every answer of the
post. Writing the active flag on a forum writes the same value on every post of the forum, archived posts
included.

**WS-609 — Closing reasons with a reputation consequence.** Closing a question with the reason
`Contains offensive or malicious remarks` or the reason `Spam or advertising` deducts the flagging award from
the author, and reopening restores it; for the spam reason the deduction is multiplied by ten when the question
is the author's first question in that forum.

**WS-610 — Deleting an accepted answer.** Deleting an accepted answer withdraws the acceptance award from its
author and the acceptance bonus from the participant who accepted it.

**WS-611 — Forum count refresh.** The forum counter of every site is refreshed whenever a forum is created,
deleted, archived or re-scoped.

**WS-612 — Forum notification behaviour.** The recipient header limit of a forum post is zero, so recipients are
never disclosed to each other, and comments are never pushed to the internal inbox; only outgoing electronic
mail is used.

**WS-613 — Default ordering of questions.** The listing order is the forum's default ordering unless the visitor
chooses another; the five offered orders are newest, last updated, most voted, relevance and answered, and the
relevance value is computed as in [calculations.md](calculations.md) §22.

## Tracked links, public actions and portal access

**WS-620 — Host of a short address.** The host of a short address is the base address of the current site when
the current site is the site of the acting company, otherwise the base address of that company, joined with the
short-address path. Outside a front-end request the platform base address is used.

**WS-621 — Click recording.** The public redirection endpoint records one click, with the network address and
the country resolved from it, unless the caller is identified as a crawler, and then answers a permanent
redirect to the target address, which may leave the site.

**WS-622 — Statistics address.** The statistics action of a tracked link opens the short address followed by a
plus sign.

**WS-623 — Public execution of a server action.** A code action may be executed from the site only when it is
published; the public address is the base address joined with the public action path and the action's path
segment, its external identifier or its numeric identifier. The request object and the structured-data helper
are added to the evaluation context, and a response placed in the evaluation context takes priority over a
returned action.

**WS-624 — Site-aware duplicate account detection.** When portal access is granted, the search for accounts with
the same address is restricted to the sites of the portal users being granted access — their own site, or, for a
portal user with no site, both the empty site and the current site — the site is read along with the other
account fields, and two accounts are considered the same person only when their sites are compatible.

---

# Part 5: Industry-standard completions

The rules below are not observable in the behaviour this specification describes; they are stated as
**industry-standard default** resolutions so that a rebuild can distinguish them from observed behaviour.

**WS-900 (industry-standard default) — Decoy-field submissions should be refused.** A submission that carries a
value in the decoy field of a public form is by construction automated. A rebuild should refuse it with the same
answer shape as a validation failure and should not create the record. The observed behaviour records the value
as free text instead.

**WS-901 (industry-standard default) — Redirect loops must be broken.** A rewrite rule whose target, after
applying the rule table again, leads back to the source produces an endless redirect. A rebuild should detect a
cycle in the rule table at save time and refuse the rule, or cap the number of consecutive fallback redirects at
ten, matching the rerouting limit already applied to internal rewrites.

**WS-902 (industry-standard default) — Deleted pages should leave a redirect offer.** Deleting a page that had a
published address leaves the address answering "not found". A rebuild should offer, at deletion time, to create a
permanent redirect from the deleted address to a chosen target, using the same dialogue shape as the page
properties dialogue.

**WS-903 (industry-standard default) — The site index should exclude addresses answering a redirect.** An
address that is the source of a `301` or `302` rule should not be listed in the site index, because the index
would then advertise an address that always redirects.

**WS-904 (industry-standard default) — Retention of visitor data.** Personal data protection practice requires a
documented retention period for browsing data. The observed retention is 60 days for anonymous visitors and
unlimited for identified visitors. A rebuild should also offer an explicit retention for identified visitors and
a per-visitor erasure operation that deletes the visitor and its tracks.

**WS-905 (industry-standard default) — Anonymous wish list rows need a session bound.** The uniqueness constraint
of a wish list row does not constrain rows whose owner is empty, because a storage engine treats null values as
distinct. The observed bound is the session list alone. A rebuild should additionally cap the number of anonymous
rows per session, a value of 100 being the common choice, so that an automated client cannot grow the table
without limit.

**WS-906 (industry-standard default) — The variant creation endpoint should be rate-limited.** The endpoint that
creates a variant from a combination is reachable by anonymous visitors and deliberately returns the same answer
whether the variant existed or was created (WS-462), which means an automated client can create unused variants.
A rebuild should keep the answer opaque and add a per-session rate limit rather than an error channel, because an
error channel would leak which templates use dynamic variant creation.

**WS-907 (industry-standard default) — Feed tokens should be rotatable.** A product feed's access token is
generated once and never expires. A rebuild should offer an explicit token rotation operation that regenerates
the token and invalidates the cached document, so that an address handed to a syndication service can be revoked
without deleting the feed.

---

# Part 6: Mapping of the former rule identifiers

Two catalogues were merged. `WCM-RULE-nnn` identifiers come from the site and content management catalogue and
`SHOP-RULE-nnn` identifiers from the storefront catalogue. Rules that exist here but in neither catalogue are
marked "new"; they were written from the entity behaviour of [entities.md](entities.md) and the state machines
of [state-machines.md](state-machines.md).

| Former identifier | Identifier here |
|---|---|
| `WCM-RULE-001` … `WCM-RULE-022` | WS-001 … WS-022, unchanged number |
| `WCM-RULE-030` … `WCM-RULE-040` | WS-030 … WS-040, unchanged number |
| `WCM-RULE-050` … `WCM-RULE-072` | WS-050 … WS-072, unchanged number |
| `WCM-RULE-080` … `WCM-RULE-086` | WS-080 … WS-086, unchanged number |
| `WCM-RULE-090` … `WCM-RULE-106` | WS-090 … WS-106, unchanged number |
| `WCM-RULE-110` … `WCM-RULE-125` | WS-110 … WS-125, unchanged number |
| `WCM-RULE-130` … `WCM-RULE-143` | WS-130 … WS-143, unchanged number |
| `WCM-RULE-150` … `WCM-RULE-156` | WS-150 … WS-156, unchanged number |
| `WCM-RULE-160` … `WCM-RULE-178` | WS-160 … WS-178, unchanged number |
| `WCM-RULE-180` … `WCM-RULE-196` | WS-180 … WS-196, unchanged number |
| `WCM-RULE-200` … `WCM-RULE-218` | WS-200 … WS-218, unchanged number |
| `WCM-RULE-230` … `WCM-RULE-246` | WS-230 … WS-246, unchanged number |
| `WCM-RULE-250` … `WCM-RULE-264` | WS-250 … WS-264, unchanged number |
| `WCM-RULE-270` … `WCM-RULE-278` | WS-270 … WS-278, unchanged number |
| `WCM-RULE-280` … `WCM-RULE-287` | WS-280 … WS-287, unchanged number |
| `WCM-RULE-290` … `WCM-RULE-299` | WS-470 … WS-479, in the same order |
| `WCM-RULE-300` … `WCM-RULE-314` | WS-480 … WS-494, in the same order |
| `WCM-RULE-320` … `WCM-RULE-329` | WS-500 … WS-509, in the same order |
| `WCM-RULE-340` … `WCM-RULE-349` | WS-510 … WS-519, in the same order |
| `WCM-RULE-360` … `WCM-RULE-384` | WS-520 … WS-544, in the same order |
| `WCM-RULE-390` … `WCM-RULE-400` | WS-550 … WS-560, in the same order |
| `WCM-RULE-410` … `WCM-RULE-415` | WS-570 … WS-575, in the same order |
| `WCM-RULE-900` | WS-900 |
| `WCM-RULE-901` | WS-901 |
| `WCM-RULE-902` | WS-902 |
| `WCM-RULE-903` | WS-903 |
| `WCM-RULE-904` | WS-904 |
| `SHOP-RULE-001` … `SHOP-RULE-180` | WS-290 … WS-469, in the same order: the identifier here is the former number plus 289 |
| (new) | WS-580 … WS-613, the forum rules |
| (new) | WS-620 … WS-624, tracked links, public server actions and site-aware portal access |
| (new) | WS-905, WS-906, WS-907, three further industry-standard defaults |

Individual storefront rules, for readers checking a citation: `SHOP-RULE-001` is WS-290, `SHOP-RULE-050` is
WS-339, `SHOP-RULE-100` is WS-389, `SHOP-RULE-150` is WS-439 and `SHOP-RULE-180` is WS-469.

---

## Reconciliation notes

1. **Two numbering schemes.** The site catalogue used a `WCM-RULE` prefix with topical blocks and the storefront
   catalogue a dense `SHOP-RULE` sequence. The merged scheme keeps the topical block numbers of the first
   catalogue up to WS-287, gives the whole storefront one contiguous block WS-290 to WS-469, and moves the
   remaining site blocks above it. The mapping above is exhaustive.
2. **Duplicate subjects.** Both catalogues stated the page cache rules; the site catalogue's wording is kept
   (WS-069 to WS-072) and the storefront's cart-marker exclusion is added to it as WS-362, because the two are
   complementary rather than contradictory.
3. **Publication permission.** The site catalogue stated that a member of the Editor and Designer group may
   always publish a page (WS-066); the storefront catalogue stated only the platform write check (WS-280). The
   source tree carries both: the override exists on Website Page only, so WS-066 is stated as an override of
   WS-280 rather than as a contradiction.
4. **Zero-priced lines.** One catalogue said that a zero-priced line is refused, the other that it is refused
   "unless the product's service tracking is exempt". The exempt list is empty by default and gains the course
   tracking value when the course capability is installed, so WS-355 states the exemption explicitly.
5. **The abandoned-cart delay of zero.** One catalogue said "a delay of zero disables the feature", the other
   "a delay of zero is read as one hour". The searchable form of the detection substitutes one hour for a zero
   or missing delay, so WS-428 keeps the second statement.
6. **Tag uniqueness.** The blog tag name is unique globally (WS-520) and the forum tag name is unique per forum
   (WS-606); one catalogue stated the blog rule as "per blog". The stored constraints decide, and both are stated
   in their own scope.
