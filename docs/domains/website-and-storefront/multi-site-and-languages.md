# Multiple sites and languages

The deep specification of the machinery that lets one database serve several public sites in several
languages: how a request is mapped onto exactly one site, what "belongs to a site" means for a record, how a
shared template becomes a site-specific one, how the most specific record is chosen, how a language appears
in an address, how canonical and alternate links are produced, and how the crawler instructions, the site
index and the site-wide search are built.

---

## 1. Site resolution

### 1.1 The algorithm

The ordered procedure, the two spellings of an internationalized host, the exact-match rule, the port
fallback and the fallback site are [calculations.md](calculations.md) §2, with a worked example covering the
seven interesting cases. The rules themselves are WS-030 to WS-038.

### 1.2 What the decision governs

Once the site is resolved, it decides:

| What | How |
|---|---|
| The routing table | Rewrite rules are per site, so the set of registered addresses differs per site (§9 of [workflows.md](workflows.md)). |
| The public user | Unauthenticated requests run as that site's public user. |
| The company | The allowed company list becomes the site's company when the current user is the public user or when the site's company is among the user's companies, and the user's own company otherwise. |
| The languages | Only the site's languages are served, accepted as prefixes and offered in the selector (WS-132). |
| The templates, assets and attachments | The most specific record per key wins (§4). |
| The record rules | The site identifier is in the rule evaluation context and in the rule cache key (WS-287). |
| The base address | Absolute addresses are built from the site domain, falling back to the platform base address (WS-286). |
| The theme | Only that site's theme contributes assets (WS-183). |

### 1.3 Forcing a site

A user who is both a multi-site user and a Restricted Editor may force another site into the session. When
the target site declares a domain and the current request host differs from it, the visitor is first
redirected to the same endpoint on the target domain, so that the forced identifier is written in the
session belonging to that host (WS-039, WS-040). A stale forced identifier is discarded silently (WS-031).

---

## 2. What "belongs to a site" means

Every site-aware record carries an optional site reference.

| Value | Name used in this folder | Meaning |
|---|---|---|
| empty | generic record | Shared by every site. |
| a site | specific record | Only that site sees it. |

The single accessibility test is "the record's site is empty or the current site". It is applied:

* by the pre-dispatch guard, which answers "not found" for a record argument bound to another site and
  "forbidden" when reading its site is refused (§1.3 of [workflows.md](workflows.md));
* by every listing and every search, through the record rules of [configuration.md](configuration.md) §4.2;
* by the publication reading and searching of WS-282 and WS-283, which make a record bound to another site
  read as unpublished;
* by the address enumeration, which scopes record converters to the site (WS-492);
* by the content block conditions (WS-258).

A record that is generic and a record that is specific may coexist under the same key; §4 decides which one
is used.

---

## 3. Copy-on-write

### 3.1 The rule

Writing a template while a site is present in the execution context, and while copy-on-write is not
disabled, never modifies a shared template.

1. When a site-specific template with the same key already exists for that site, the write is applied to it
   and nothing else happens.
2. Otherwise the shared template is copied with the same key and that site.
3. The copy receives the write.
4. The pages of the shared template are duplicated onto the copy, so that the site's page now points at its
   own template.
5. Each template that inherits from the shared template is handled in turn: a child that was already
   specific to this site is re-pointed to the new parent; a child that is shared is itself copied onto the
   new parent, which forks it in turn.

The same rule applies to Asset records, keyed on the asset key (WS-180).

### 3.2 Preconditions

* A template written in a site context always receives a key first, generated when it has none, because the
  rule is keyed on the key (WS-161).
* Creating a template in a site context fills a missing site from the context, refuses an explicitly empty
  site and refuses a different site, each with its own internal message (WS-162).
* Writing field translations never forks a template (WS-170).
* A save from the site never flags the shared template as protected from updates; that flag is set only when
  no site is in context (WS-168).

### 3.3 Copy-on-delete

Deleting a shared template while a site is in context first forks it for every **other** site, by writing
its own name onto it in each of those contexts, so that only the current site loses the template (WS-163).

### 3.4 Consequences a rebuild must accept

* A site that has never edited a template keeps reading the shared one, and therefore receives the shipped
  improvements of an update.
* A site that has edited a template is frozen at the moment of the fork for that template, and an update
  changes only the shared original.
* Forking a parent forks its children, which is why the first edit of a layout template can create several
  records at once.
* A forked page that receives a different address hides the shared page on that site, by WS-063.

---

## 4. Most-specific selection

### 4.1 Templates

For a given key, the site-specific template of the current site wins over the shared template. When no site
is in context, only shared templates are considered. During inheritance resolution an inactive site-specific
template wins over an active shared one, and the result is then filtered to active templates, which is how a
site switches a shipped block off without deleting it (WS-164, WS-165).

Template lookup is ordered by site ascending, so that the shared template is returned only when no specific
one exists; a lookup failure reports the site identifier in its message (WS-166).

### 4.2 Assets

For a given key, the asset of the current site wins over the shared one; an asset with no key is always
kept; outside a site context, assets bound to a site are dropped (WS-181). Compiled bundles carry the site
identifier in their address (WS-182), and every theme package other than the site's own theme is removed
from the list of contributing packages (WS-183).

### 4.3 Attachments

Serving an attachment by address adds the site dimension to the lookup and orders by site first, so that a
per-site attachment wins over a shared one at the same address (WS-185). A lookup by external identifier, on
a site that has a theme, first searches an attachment of that site whose key equals the identifier
(WS-186).

### 4.4 Pages

For a given address, the page bound to the current site is served; the shared page is served only when no
site-specific page exists (WS-062). For the address enumeration and the page search, the candidate set is
reduced to the most specific page per address, which is implemented by keeping, per address, only the first
page in the ordering — specific first — and, for a shared page, only when exactly one page in the whole scope
carries that key (WS-063, WS-276).

---

## 5. Languages in the address

### 5.1 The prefix

The default language of a site is served without a prefix; every other language of the site is served under
its alternate-language code as the first path segment. A request carrying the default prefix is redirected to
the address without it (WS-134). A site with exactly one language never inserts a prefix (WS-141).

### 5.2 Resolution and redirection

The resolution order, the nearest-language rule and the nine outcomes of the matching step are §1.2 of
[workflows.md](workflows.md) and WS-130 to WS-138. The three rules that most often surprise a rebuild are:

* a crawler is never redirected by browser language preference, so that a crawler indexes the default
  language of an unprefixed address (WS-135);
* a write request is never redirected for language (WS-136);
* an endpoint that declares itself non-multi-language never receives a prefix (WS-137).

### 5.3 Which addresses are translatable

An address is translatable when its endpoint is a front-end endpoint and either declares itself
multi-language or is a rich-text endpoint. An address under the static file path or under the platform
client path is never translatable. An address that matches no endpoint is treated as translatable, so that
page addresses receive the prefix (WS-140).

---

## 6. Alternate-language codes

Each language of a site is given a code used as its path prefix and in the alternate links.

1. Take the site's languages in name order.
2. For each language, the two-letter prefix of its code is claimed by the first language that asks for it.
3. A language whose two-letter prefix is already taken receives its full code, lower-cased, with the
   underscore replaced by a hyphen.
4. Spanish is a special case: when Latin-American Spanish is among the languages, it, and no other Spanish
   variant, receives the bare two-letter prefix.

**Worked example.** A site offers, in name order, `English (US)`, `English (UK)`, `Español`,
`Español (Latinoamérica)` and `Français`. The English variant that comes first in name order claims `en`;
the other becomes `en-gb`. The Spanish rule applies: `Español (Latinoamérica)` claims `es` and `Español`
becomes `es-es`. `Français` claims `fr`. The site therefore serves `/en-gb/about`, `/es-es/about`,
`/fr/about`, and `/about` in the default language.

---

## 7. Canonical and alternate links

* The canonical link is always emitted. It points at the current address rebuilt for the current language
  against the site's base address, without any query string (WS-142).
* Alternate-language links are emitted only when the endpoint is multi-language, the current address is
  canonical, and the site has more than one language. A default-language link is then also emitted under the
  code `x-default`.
* The current address is canonical when the request root joined with the raw request path and query equals
  the computed canonical address. An address containing characters that require escaping is therefore never
  canonical, because the raw request is unescaped while the canonical address is escaped (WS-143).
* A multi-language read request whose canonical address differs from the requested one is redirected
  permanently to the canonical one, which is what turns a numeric address into a readable one (WS-139).
* A no-index instruction is emitted when the record is not indexed, when the site declares a domain and the
  request root does not match it, or when the request is a paginated listing beyond the first page
  (WS-152).

---

## 8. The crawler exclusion response

The response is plain text and is never language-prefixed. Its two branches — the non-canonical host branch,
which disallows everything and points at the canonical host's index, and the ordinary branch, which points
at the request root's index and appends the site's custom text under its banner — are §14 of
[workflows.md](workflows.md), WS-493 and WS-494.

The host comparison removes the scheme, the leading `www.` and the trailing slash and encodes both sides in
the ascii-compatible internationalized domain name encoding; only equality makes the host canonical
(WS-153).

---

## 9. The site index

### 9.1 Generation and caching

The generation procedure, the chunking at 45000 locations, the index of indexes and the twelve-hour cache
are §15 of [workflows.md](workflows.md) and WS-480 to WS-484. The enumeration runs with the site's public
user, so that the index never leaks an address a visitor cannot reach (WS-485).

### 9.2 What is enumerated

Two sources.

**Pages.** Pages of the current site or shared pages, whose template is set and whose address is not the
site root, are read in name order and kept when they are indexed, publicly visible, published and not
embargoed. The most specific page per address is kept. Each contributes its address, its identifier, its
name, optionally a priority ([calculations.md](calculations.md) §7.3) and optionally a last modification
date (WS-486 to WS-488).

**Endpoints.** Every rule of the routing table is examined; the eligibility test, the generators, the
converter value generation and the deduplication are §16 of [workflows.md](workflows.md) and WS-489 to
WS-492.

### 9.3 What is never enumerated

An unpublished or embargoed page; a page whose visibility is not public; the site root, which the root
endpoint enumerates separately; an endpoint that declares its index value as false; an endpoint that is not
enumerable; and, on a site whose shop access is restricted, every shop, category and product address
(WS-292). The industry-standard default WS-903 adds the sources of redirect rules.

---

## 10. Site search

### 10.1 The contract

Every entity that participates implements the searchable contract of [entities.md](entities.md) §3.5: it
describes its search, fetches its matches and renders them. A search descriptor names the entity, the base
condition, the fields the term is matched against, the fields to read, the mapping from template slot to
field, the fallback icon and, optionally, an elevated-read flag, an extra condition builder and a forced
ordering.

### 10.2 The procedure

The ordered procedure, the approximate matching, the rendering, the truncation and the highlighting are §17
of [workflows.md](workflows.md), WS-270 to WS-278 and [calculations.md](calculations.md) §4 and §6.

### 10.3 The scopes

| Scope | Contributed by | Ordering |
|---|---|---|
| Pages | this folder | name ascending |
| Blog posts | this folder | publishing moment descending |
| Forums, questions and tags | this folder | last activity descending |
| Products and storefront categories | this folder | the shop default sort |
| Contacts of the public directory | this folder | name ascending |
| Events, courses, job positions and any other published entity | the owning folder | its own |
| `all` | every scope above | name, after merging |

The page scope adds, for a user outside the Editor and Designer group, the conditions of WS-274 and
post-filters the candidates against the read rules of the page and template entities (WS-275). The result of
the `all` scope is sorted by name and cut to the limit.

### 10.4 Two entry points, one answer

The autocompletion endpoint and the public search listing use the same answer; the listing merely asks for a
page size of 50 and a shorter truncation. The shop listing uses the same field set as the site-wide search
box for products; a rebuild must keep them identical or the two entry points will disagree.
