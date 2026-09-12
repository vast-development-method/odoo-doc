# Workflows

Every operational procedure of this folder, end to end. Each workflow names its actors, its
preconditions, its numbered steps with the decision taken at each one, the records created or changed
with the values written, the messages emitted and the postconditions. The state tables that these
procedures drive are in [state-machines.md](state-machines.md); the numbered rules they enforce are in
[business-rules.md](business-rules.md); the arithmetic they perform is in
[calculations.md](calculations.md).

The workflows are grouped:

| Part | Workflows | Subject |
|---|---|---|
| 1 | 1–18 | Serving requests, pages, menus, addresses, the site index and the site search |
| 2 | 19–26 | Editing content, styles, themes and the first-run configurator |
| 3 | 27–28 | Public forms and human verification |
| 4 | 29–33 | Visitors, tracking and cookie consent |
| 5 | 34–43 | Publication, blog, forum, public directories and profiles |
| 6 | 44–63 | The storefront: catalogue, cart, checkout, payment, fulfilment and syndication |

---

# Part 1: Serving the site

## 1. Serve an incoming public request

**Actors.** Public visitor, portal user, editor or search engine crawler.
**Precondition.** At least one Website record exists.

### 1.1 Resolve the site

1. When the browsing session carries a forced site identifier and that site still exists, it is the
   current site. When the site no longer exists, the forced identifier is removed from the session and
   resolution continues.
2. Otherwise, when the execution context carries a site identifier, that site is the current site.
3. Otherwise, when the request is not a front-end request and the caller did not ask for a fallback,
   the current site is empty and resolution stops.
4. Otherwise the request host, that is the host name together with its port, is matched against the
   configured domains by the algorithm of [calculations.md](calculations.md) §2. The result is cached
   per host and per fallback flag.
5. When no domain matches and a fallback was requested, the first site in ordering order is the
   current site. When no domain matches and no fallback was requested, the current site is empty.

**Postcondition.** The routing table used for the rest of the request is the routing table of the
resolved site, because rewrite rules are per site.

### 1.2 Match the path and resolve the language

1. When the request has already been rewritten once, the platform matching runs and the workflow
   continues at §1.3.
2. The platform matching is attempted on the raw path.
   * On success the matched endpoint declares whether it is a front-end endpoint and whether it is
     multi-language. A non-front-end endpoint is dispatched immediately with no language handling.
   * On failure the first path segment is provisionally read as a language prefix and the remainder is
     kept as the path without language.
3. Redirection is allowed only when the request verb is not a write verb and the endpoint is
   multi-language, or unknown, in which case it is assumed to be.
4. When redirection is allowed and the path contains a doubled slash, the browser is redirected
   permanently to the path with doubled slashes collapsed.
5. The requested language is resolved, in order, from the prefix in the path, then the front-end
   language cookie, then the language in the execution context, then the site's default language. Each
   candidate is mapped to the nearest available front-end language: an exact code match wins;
   otherwise the first available language whose code starts with the same two-letter prefix is used;
   otherwise there is no match.
6. One of nine outcomes applies.

| Case | Condition | Action |
|---|---|---|
| 1 | The endpoint is not a front-end endpoint | Dispatch as is. |
| 2 | No language prefix and the resolved language is the default language | Continue with the path as is. |
| 3 | No language prefix and the user agent is a crawler | Force the default language and continue with the path as is. |
| 4 | No language prefix and redirection is not allowed | Continue with the path as is. |
| 5 | No language prefix and another language was resolved | Redirect to the prefix followed by the path, keeping the query string, and set the front-end language cookie to the resolved language. |
| 6 | The prefix is the default language's prefix and redirection is allowed | Redirect to the path without the prefix, keeping the query string, and set the cookie to the default language. |
| 7 | The prefix is an alias of the resolved language's prefix and redirection is allowed | Redirect permanently to the resolved prefix followed by the path without language, keeping the query string, and set the cookie. |
| 8 | The path is exactly a prefix followed by a slash and redirection is allowed | Redirect permanently to the same path without the trailing slash and set the cookie to the default language. |
| 9 | The prefix equals the resolved language's prefix | Rewrite the request internally to the path without the prefix and continue. |

When none of the nine applies, the path is used as is and a warning is recorded.

7. Matching is attempted again on the possibly rewritten path. A second failure raises "not found",
   with the request marked as a front-end, multi-language request so that a styled error page can be
   produced.

### 1.3 Pre-dispatch

1. For every argument of the endpoint that is a record: when the record carries the multi-site
   accessibility helper and is bound to another site, the request answers "not found"; when reading
   the record's site is refused, the request answers "forbidden".
2. When the request is a front-end request, the front-end preparation runs: the language is put in the
   execution context; the front-end language cookie is refreshed when it differs; the time zone
   derived from the geolocation of the network address is put in the context when the context has
   none; the allowed company list is set to the site's company when the current user is the public
   user or when the site's company is among the user's companies, and otherwise to the user's own
   company; the site identifier and the editor context are put in the context; and the site record is
   bound to the request with that context.
3. When the endpoint is multi-language and the verb is a read verb, the canonical path is rebuilt from
   the endpoint and its arguments. When the rebuilt path differs from the requested path, the browser
   is redirected permanently to the rebuilt path, prefixed by the language when the language is not the
   default. This is what turns a numeric address such as `/blog/2/7` into the readable
   `/blog/news-2/my-post-7`.
4. Record-argument contexts are refreshed with the request context, so that slugs and names render in
   the right language.

### 1.4 Authentication

When the endpoint requires public authentication and the session carries no signed-in user, the
request runs as the site's public user. When no site is resolved, the platform public user is used.

### 1.5 Dispatch and post-dispatch

1. The endpoint runs and produces a response.
2. After dispatch the visitor tracking hook runs (§29).

### 1.6 Fallback when nothing matched

The fallback handler is tried for the status codes "not found" and "forbidden".

1. The platform attachment fallback is tried first; when an attachment answers on the path it is
   returned.
2. The front-end preparation and the diagnostic handling are run, so that a styled page can be
   produced.
3. The page fallback runs (§2). When it returns a response, the response is flattened and returned.
4. The redirect fallback runs (§4). When it finds a rule, a redirect response is returned.
5. Otherwise the error page is produced: for "not found" seen by a member of the Editor and Designer
   group, the designer-specific not-found page is rendered with the requested path, without its
   leading slash, so that a page can be created in one action; for "forbidden" raised with the password
   marker, the password prompt page is rendered with the requested path; otherwise the generic page for
   the status code is rendered, falling back to the generic client-error page for any client-error
   code. When even that fails, the bare error page is rendered with the status code 418.

## 2. Serve a page

**Actor.** Any visitor.
**Precondition.** The path matched no endpoint, or the home page endpoint delegated to this workflow.

1. Look up the page information for the request path, with elevated rights: search for a page of the
   current site, or a shared page, whose address equals the path, ordered by site ascending so that the
   shared page is preferred only when no specific page exists; when nothing is found, search again with
   a case-insensitive comparison. The lookup result contains the page identifier, its stored address,
   its template identifier and its authorised groups, and is cached by path and by site.
2. When a page was found and its stored address differs from the requested path — which happens only
   through the case-insensitive fallback — redirect to the stored address.
3. When no page was found, the path is not `/` and the path ends with a slash, redirect permanently to
   the path without the trailing slash, re-prefixed with the language when the language is not the
   default and with the query string appended when there is one.
4. When no page was found, return nothing so that the caller can try the redirect fallback.
5. Otherwise produce the response (§3).

## 3. Produce and cache a page response

**Actor.** Any visitor.

1. Decide whether the cache may be used: the verb must be a read verb, the request must carry no
   parameters, the current user must be the public user, and the page must have no authorised groups
   (WS-069).
2. When the cache may be used, look up the cached entry under the key of WS-070.
   * When the entry exists and is younger than 3600 seconds, a copy of the stored response is returned
     after post-processing: the request forgery token embedded in the markup is replaced by the token
     of the current session, the cart counter element is rewritten from the session, and the cached
     page and template identifiers are attached to the response so that tracking still works.
   * When the entry exists but is older than 3600 seconds, the raw response is produced again and, when
     it may be inserted, the cache entry is refreshed.
3. When the cache may not be used, the raw response is produced directly. A rendered page containing a
   cart marker is never inserted into the cache.
4. Raw response production:
   1. Prefetch every prefetchable field of the page and of its template in one read each.
   2. The page is rendered only when the current user is in the Editor and Designer group or the page is
      visible, **and** when either the page is bound to a site or the page's template is the most
      specific template for its key on the current site. The second condition prevents a shared page
      whose per-site copy has been given a different address from remaining reachable at the old
      address.
   3. The response media type is derived from the extension of the request path when the extension is a
      recognised web media type, and is rich text otherwise.
   4. The rendering context carries the page as the main object.
   5. The production moment is stored on the response for cache ageing.
   6. When the conditions of step 4.2 are not met, nothing is produced and the caller continues with the
      redirect fallback, which normally ends on the not-found page.

**Postcondition.** The template cache is invalidated whenever a page's address, visibility or
authorised groups change, whenever a page or model page is deleted, and whenever a menu entry is
created, changed or deleted.

## 4. Apply a redirect rule

**Actor.** Any visitor.
**Precondition.** No endpoint and no page matched.

1. Build four candidate source addresses from the request: the full request address with its query
   string; the request path without any trailing slash; the request path with a trailing slash; and the
   request path with the slug part of its last segment replaced by the bare numeric identifier.
2. Search the rewrite rules whose action type is `301` or `302`, whose source is one of the four
   candidates, and whose site is empty or the current site, ordered by source descending, taking the
   first.
3. When a rule is found: when its target starts with a slash and contains a slug pattern, the target is
   rebuilt for the current language so that the record name in the address is translated; the request
   parameters are re-appended to the target; the browser is redirected with the rule's status code. The
   redirect is allowed to leave the site, because only designers may create rules.
4. When no rule is found, the caller produces the error page.

## 5. Create a page

**Actor.** Editor and Designer.
**Trigger.** The New Page action in the site editor, or the Create Page button on the designer
not-found page.

1. The caller supplies the page name, whether to add a menu entry, the template key to copy (by default
   the shipped blank page template), optional page values, optional menu values, optional section
   markup and an optional distinct page title.
2. The page address is computed as a slash followed by the path slug of the name, then made unique
   against the pages of the current site by appending `-1`, `-2` and so on
   ([calculations.md](calculations.md) §3).
3. The page key base is the plain slug of the name. When no name was supplied, the name becomes `Home`
   and the key base becomes `home`.
4. The source template is read and its architecture is taken. When section markup was supplied, it is
   appended inside the element identified as the page wrapper.
5. A unique template key is computed by prefixing the key base with the source template's package name,
   or with the supplied namespace, and appending `-1`, `-2` and so on until no template of the current
   site and no shared template uses it.
6. The source template is copied with the site taken from the execution context and with the new key.
   The copy is then written, with translation disabled, with the architecture in which every occurrence
   of the source key has been replaced by the new key, and with the name set to the supplied page title
   or to the page name. When the copy inherited a shipped file path for its architecture, that path is
   cleared so that the template is no longer considered file-backed.
7. When a page is wanted, which is the default, a Website Page is created with the computed address, the
   current site, the new template, tracking enabled, plus the supplied page values, which typically
   carry the publication flag.
8. When a menu entry is wanted, an entry with the computed address is looked up on the current site;
   when none exists, a Website Menu is created with the page name, the computed address, the site's root
   menu as parent, the new page and the site, merged with the supplied menu values.
9. The operation returns the address, the template identifier, and the page and menu identifiers when
   they were created.

**Postcondition.** A Website Page, a template and optionally a Website Menu exist. When the request came
from the not-found page, any pre-existing menu entry pointing at the raw path, with or without a leading
slash, and not yet bound to a page, is bound to the new page.

**Branch: pages that are not rich text.** When the supplied path ends with a recognised non-rich-text
web extension and no template was named, the shipped template for that extension is used when it
exists. The caller is then redirected to the technical template form rather than to the site editor.

## 6. Change the properties of a page

**Actor.** Editor and Designer.
**Trigger.** The page properties dialogue.

1. The dialogue is created for the current page with the current site. The current address is
   remembered as the old address.
2. The designer edits any of: title, address, in-menu flag, home-page flag, publication flag,
   publishing date, indexing flag, visibility, visibility password, authorised groups, new-page-template
   flag, and whether to create a redirect from the old address and with which action type.
3. On save the related fields are written through to the page, which triggers the page write rules:
   slugification, uniqueness, menu repointing, home-page synchronisation, key regeneration, group
   clearing and cache invalidation (WS-050 to WS-056).
4. The in-menu flag is applied: switching it on creates a Website Menu with the page name, the current
   address, the site's root menu as parent, the site and the page; switching it off deletes every menu
   entry of that site that targets the page or that matches the address.
5. The home-page flag is applied: switching it on writes the address into the site's home page address
   unless the address is `/`; switching it off clears the site's home page address.
6. The publication flag is applied through the publication mixin (§34).
7. After the write, when the address actually changed and the designer asked for a redirect, a Website
   Rewrite is created with the page name, or the supplied name, the chosen action type `301` or `302`,
   the old address as source, the new address as target, and the site. The remembered old address is
   then updated to the new one.

**Postcondition.** The page, its menu entries, the site's home page setting and possibly a rewrite rule
are consistent.

## 7. Clone a page

**Actor.** Editor and Designer.

1. The caller supplies the page identifier, optionally a new name, and whether to clone the menu entry,
   which defaults to yes.
2. The copy parameters are the new name, or the original name, and the current site.
3. When a new name was given, the address is recomputed as a slash plus the path slug of the new name,
   made unique among the pages of the current site.
4. The page is duplicated: its template is copied into a new template bound to the target site, the page
   key becomes the new template key, and the address is the computed one or, when no name was given, the
   original address made unique.
5. When the menu is to be cloned and the copy stayed on the same site, the first menu entry targeting the
   original page is duplicated with the new address, the new name and the new page. A clone that moved
   from the shared scope to a site scope never clones the menu, because the two entries would live in
   different trees.
6. The operation returns the new address.

## 8. Edit the navigation menu

**Actor.** Editor and Designer.
**Trigger.** Saving the menu editor dialogue.

1. The caller supplies the site identifier and a payload containing the full list of entries — identifier,
   parent identifier, name, address, ordering value, mega-menu flag — and a list of identifiers to
   delete.
2. Entries to delete are deleted first.
3. Entries whose identifier is a temporary text value are created with their name and the site; every
   reference to the temporary identifier in the payload, as identifier and as parent identifier, is
   replaced by the new numeric identifier.
4. Each entry is then reconciled with the page catalogue.
   * When the address contains a fragment marker, any page link is cleared. When the address starts with
     the fragment marker, is longer than one character and is not one of the two page-level anchors
     `#top` and `#bottom`, the address is prefixed with the path of the page the designer came from, so
     that the anchor keeps working from any page.
   * Otherwise a page of that site whose address equals the entry address, or equals a slash plus the
     entry address, is searched. When one is found, the entry is bound to it and the entry address is
     normalised to the page address; a parent identifier supplied as text is converted to a number. When
     no page is found but the entry was bound to a page, the entry address is tested against the routing
     table: when an endpoint answers on it, the page link is cleared, because a page must not shadow an
     endpoint; when no endpoint answers, the linked page is renamed to the new address.
5. Every entry is written with its payload values.
6. Creating, writing or deleting entries clears the template cache, which also invalidates the menu cache
   flag of the site.

**Postcondition.** The menu tree of the site matches the payload and satisfies the two-level, mega-menu
and container rules WS-090 to WS-092.

## 9. Create and apply a rewrite rule

**Actor.** Editor and Designer.

1. The designer opens the redirect list and creates a record. Choosing a registered endpoint path fills
   both the source and the target with that path.
2. On save the validation rules WS-110 to WS-118 run.
3. When the action type is `308` or `404`, the routing cache is cleared on every worker.
4. Routing table generation for a site then behaves as follows for every generated endpoint address.
   * When the address is not a rule source, the endpoint is registered unchanged.
   * When it is the source of a `308` rule, the endpoint is also registered at the rule target. When the
     source and the target differ, a duplicate endpoint is registered at the source that answers with a
     permanent redirect to the target, rebuilding the target with the original slug values and the
     original query string.
   * When it is the source of a `404` rule, the endpoint is not registered at all for this site.
5. Rules of type `301` and `302` never change the routing table; they are applied by the redirect
   fallback (§4).

## 10. Serve the home page

**Actor.** Any visitor requesting `/`.

1. When the site's home page address is set and is not `/`, the request is internally rerouted to that
   path.
2. The page fallback is tried (§2). When it answers, that answer is returned.
3. When the home page address is set and is not `/`, the routing table is consulted for that address and
   the matching endpoint is dispatched. Access failures, not-found failures and expired-session failures
   are ignored.
4. Otherwise the first reachable child of the root menu is used. Reachable means visible, with an address
   that is not `/`, not empty, not a bare fragment marker, and that does not start with a slash followed
   by a question mark, a slash followed by a fragment marker, or a space. The visitor is redirected to it.
5. Otherwise the request answers "not found".

## 11. Bootstrap the home page of a new site

**Actor.** The Website create operation.

1. When the shipped home page template does not exist, nothing happens.
2. The shipped home page template is given, for this site, an architecture containing only the site
   layout call and one empty editable region. This write goes through copy-on-write, so the shared home
   page template is not modified.
3. A page of this site whose key equals the home page template key is searched. When none exists, a
   Website Page is created, published, at the address `/`, pointing at the most specific home page
   template for this site.
4. The page address is then forced to `/`, which guards against the uniqueness suffix having produced
   `/-1`.
5. The shipped default main menu tree is copied for this site: the root entry is copied with the name
   `Top Menu for Website ` followed by the site identifier and with this site, then each descendant is
   copied recursively under its copied parent.
6. The copied entry whose address is `/` is bound to the home page.

## 12. Switch between sites in the editor

**Actor.** Multi-site user who is also a Restricted Editor.

1. The user picks another site. The site-forcing endpoint is called with the path to open.
2. A user who is not both a multi-site user and a Restricted Editor is simply redirected to the path,
   because forcing a site they cannot edit would be useless.
3. When the target site has a domain and the current request host differs from it, the visitor is
   redirected to the same endpoint on the target domain, carrying the path and a marker saying that the
   domain hop already happened. That second request creates the session on the right host.
4. The target site identifier is written into the session and the visitor is redirected to the path.

## 13. Change the display language

**Actor.** Any visitor.

1. The visitor selects a language; the language endpoint is called with the prefix and the
   address to go back to.
2. The special value `default` resolves to the site's default language prefix, and the return address is
   rebuilt as the prefix followed by the return address, or by `/` when there is none.
3. The language code is resolved from the prefix and put in the execution context, so that the redirect
   helper does not strip the prefix again.
4. The visitor is redirected to the return address, and the front-end language cookie is set to the
   resolved language.

## 14. Generate the crawler exclusion response

**Actor.** Any visitor or crawler requesting the crawler exclusion path.

1. The response media type is plain text and the response is never language-prefixed.
2. The body starts with the line `User-agent: *`, followed by one `Allow:` line per endpoint declared as
   allowed by an installed capability. The base list is empty and capabilities extend it.
3. Then one of two branches:
   * when the site has a domain and the request root does not match it — compared after removing the
     scheme, the leading `www.` and the trailing slash, and after encoding both in the ascii-compatible
     internationalized domain name encoding — the lines `Disallow: /` and a `Sitemap:` line pointing at
     the site index of the site's own domain, so that duplicate hosts are not indexed and crawlers are
     pointed at the canonical host;
   * otherwise a `Sitemap:` line pointing at the site index of the request root, then two blank lines,
     then the banner lines `##############`, `#   custom   #` and `##############`, then a blank line,
     then the site's custom crawler exclusion text.

## 15. Generate the site index

**Actor.** Any visitor or crawler requesting the site index path.

1. The index is cached as attachments whose address is `/sitemap-`, the site identifier, a hyphen, an
   eight-character digest of the request root, optionally a hyphen and a page number, and the extension
   of the extensible markup language.
2. When a cached attachment exists for the base address and is younger than twelve hours, its bytes are
   returned with the media type of the extensible markup language and the character set.
3. Otherwise every cached attachment of this site and this host is deleted and the index is regenerated:
   1. The address enumeration runs with the site's public user (§16).
   2. Addresses are written into chunks of at most 45000 entries; each chunk is rendered into an index
      document and stored as an attachment named with the base address, a hyphen and the chunk number.
   3. When no chunk was produced, the request answers "not found".
   4. When exactly one chunk was produced, it is renamed to the base address.
   5. When several chunks were produced, an index-of-indexes document listing each chunk address is
      stored at the base address.
4. Each entry carries the location and, when they are known, the last modification date and the priority.

## 16. Enumerate addresses

**Actor.** The site index generator, the link suggestion service of the editor and the address existence
check.

1. **Pages.** Website Pages of the current site, or shared pages, whose template is set and whose address
   is not `/`, are read in name order. When the caller did not force the full list, only pages that are
   indexed, whose visibility is public, that are published and whose publishing date is empty or already
   past are kept. The most specific page per address is kept, as described in
   [multi-site-and-languages.md](multi-site-and-languages.md) §4. For each page an entry is produced with
   the address, the identifier and the name; when the template priority is not the neutral value 16, a
   priority is added as computed in [calculations.md](calculations.md) §7; when the page or its template
   has a modification moment, the later of the two, reduced to a date, is added.
2. **Endpoints.** Every rule of the routing table is examined.
   * A rule whose index declaration is explicitly false is skipped.
   * A rule whose index declaration is a generator is called once per distinct generator with the site,
     in the default language, the rule and the search text; each produced location has its trailing slash
     removed, except for `/` itself, and is emitted once.
   * A rule with no generator is emitted only when it is enumerable: it answers the read verb, it is a
     rich-text endpoint, it requires no authentication or public authentication, it is declared as a
     front-end endpoint, every one of its converters can generate values, and every positional parameter
     without a default value has a converter. A rule that is enumerable but declares no index value at
     all is logged as a warning.
   * For an enumerable rule, the values of each converter are generated. Converters carrying a condition
     are processed last. A record converter whose entity carries a site dimension and that has no
     condition receives the condition "site is empty or the current site". When a search text is
     supplied, it is turned into a condition on the display field of the last converter.
   * Each combination of values is built into an address, its trailing slash is removed, and it is
     emitted once when it is not already emitted and, when a search text was supplied, when it matches
     the wildcard pattern built from that text.

## 17. Search the site

**Actor.** Any visitor.

1. The caller supplies the search scope, which is `all` or a scope name, the term, the ordering, the
   maximum number of results, the maximum number of characters for text fields and the display options.
2. The ordering is normalised to "publication flag descending, then the requested ordering or name
   ascending, then identifier descending", so that published records come first and the ordering is
   stable.
3. The scope is expanded into a list of search descriptors by asking every participating entity. This
   folder contributes the Website Page descriptor for the scopes `pages` and `all`, and the blog, forum
   and product descriptors for their own scopes; other folders contribute their own.
4. When approximate matching is allowed, which is the default, a closest term is computed
   ([calculations.md](calculations.md) §4). The exact search then runs with the closest term; when the
   closest term equals the original term ignoring case, no substitution is reported.
5. The exact search runs each descriptor: the condition is built from the base condition and the term,
   the records are fetched with the limit and the ordering, and the count is the length of the result
   when it is below the limit and a separate count otherwise.
6. The results are rendered: the declared fields are read, the fallback icon and the mapping are
   attached, and rich text fields are converted to plain text.
7. For the autocompletion answer, each mapped value is post-processed: an empty value becomes an empty
   string; a text value is shortened to the maximum number of characters with the ellipsis `...` unless
   truncation is disabled; a text value marked for matching has the term occurrences wrapped in the
   highlight template; a value whose type has a renderer is rendered by it, with the display currency for
   money; finally every value is escaped.
8. Results of the `all` scope are sorted by name, respecting a descending name ordering, and cut to the
   limit.
9. The answer contains the rendered results, the total count across entities, the set of mapped slot
   names present, and the substituted term when one was used.

**Postcondition.** No record is written. The public search listing reuses the same answer with a page
size of 50.

## 18. Read the search engine metadata of a record

**Actor.** Editor and Designer, or any user with write access to the record.

1. A user outside the Restricted Editor group must pass the write check on the record; a refusal answers
   "forbidden".
2. The metadata fields are read: meta title, meta description, meta keywords and sharing image. For a
   Website Page the indexing flag, the site and the publication state are added.
3. The edit permission is reported separately: it is false when the modification check on the record is
   refused.
4. For a record that is neither a page nor a template and that carries a slug name, the default slug —
   the slug of the display name — and the current slug are reported, so that the editor can offer a reset.

---

# Part 2: Content, styles, themes and the configurator

## 19. Edit page content and save it

**Actor.** Restricted Editor or Editor and Designer.

1. The editor opens the page in editing mode. Editable regions are the elements of the rendered markup
   that carry the editable-region marker or that are bound to a record field.
2. On save, each edited region is saved separately with the region's location expression and its new
   markup.
3. Saving a region of a template runs the following.
   1. When a location expression was supplied, the template key exists, a current site is resolved and a
      site-specific template with that key already exists for the site, the save is diverted to that
      specific template. This matters when several regions of a shared template are saved in the same
      batch: the first save creates the specific template and the others must write on it.
   2. When delayed translation is requested and the configuration parameter that disables it is unset or
      set to a false value, the save is performed with delayed translation, which keeps the translations
      of unchanged terms.
   3. The platform save then replaces the addressed section of the architecture, extracts and writes any
      embedded record field found in the section, and removes the non-editing attributes — branding
      markers and editing classes — before storing.
4. Writing a template while a site is in context performs copy-on-write, described in
   [multi-site-and-languages.md](multi-site-and-languages.md) §3.
5. Saving a content block region on a shared template writes the site into the created record, so that
   the block stays per site.
6. The saved architecture is stored with the site's default language as the source language.

**Postcondition.** The page content of the current site changed; other sites are untouched.

## 20. Reset the content of a broken template

**Actor.** Editor and Designer.

| Mode | Effect |
|---|---|
| Soft | The architecture is restored to the previously stored architecture of the same template. |
| Hard | The architecture is re-read from the shipped file the template was loaded from, when the template records such a path. |

The reset always runs with copy-on-write disabled, so that a broken shared template is repaired in place
instead of being forked per site. The theme options panel can also request a reset when a template is
disabled, which performs a hard reset before deactivating the template.

## 21. Customise a style variable file

**Actor.** Editor and Designer.

1. The style variable file must contain a variable map with a line comment containing the word `hook`
   that marks where new pairs are written.
2. The customisation operation reads the current content — the customised copy when one exists, otherwise
   the shipped file — inserts or replaces the supplied key and value pairs at the hook, and saves the
   result.
3. Saving an asset customisation:
   * the customised address is `/_custom/`, the bundle name, then the original address;
   * when a customised attachment already exists at that address, its bytes are replaced and the asset
     cache is cleared;
   * otherwise an Attachment is created with the original file name, the binary kind, the media type of a
     script for a script and of a style source for a style source, the new bytes, the customised address
     and the current site; and an Asset is created with the customised address as path, the original
     address as target, the replacement directive and the current site. When an asset already targets the
     original address, the new asset takes its name suffixed with ` override`, its bundle and its ordering
     value; otherwise its name is the bundle, a colon, a space, the word `replace`, a space and the file
     name, and its bundle is derived from the original address.
4. Resetting an asset customisation deletes both the customised attachment and the replacing asset.
5. Setting the palette name additionally resets the user colour palette and the grey palette, sets the
   four semantic theme colours to unset, and sets the five preset background gradients and the four
   chrome gradients to unset.
6. Uploading a font stores the font files as public attachments and records their identifiers in the
   variable map; deleting a font deletes the attachment, every attachment derived from it and every
   attachment whose name contains the font-provider marker.

## 22. Install a theme on a site

**Actor.** Editor and Designer.

1. The designer picks a theme from the theme gallery, which lists the packages of the theme category.
2. The current theme is removed first (§23), which also resets the default style configuration.
3. The site's theme pointer is set to the chosen theme **before** the package installation, because the
   load is triggered by the installation write.
4. The whole chain of packages the theme depends on is upgraded, which installs the theme when it is not
   installed yet.
5. During the installation write, for every theme package reaching the installed state from the
   to-install or to-upgrade state, the set of sites to update is computed as the sites whose own theme is
   in the same stream: the theme itself, the packages it depends on and the packages that extend it. When
   the upgrade was started from the interface, a configuration parameter restricts the set to the current
   site.
6. For each site to update, the theme load runs: for each template entity, in the fixed order templates,
   assets, pages, menus, attachments, the theme records are converted into real records.
   * A theme record whose converted values cannot be produced yet, because its parent template or its
     page template has no copy for this site, is postponed to a later pass over the same entity.
   * An existing copy for this site is updated with the converted values, except the fields the update
     must not overwrite; a missing copy is created.
   * Orphan copies, that is records generated for this site whose theme record no longer exists, are
     removed for the key-bearing entities — templates, assets and attachments. Pages and menus are not
     cleaned this way, because they carry no key.
7. The theme post-copy hook runs, letting the theme enable or disable specific templates and assets.
   Enabling a header template disables every other header template; enabling a footer template disables
   every other footer template.
8. The designer is redirected to the site.

**Postcondition.** The site has exactly one theme in its pointer, and the real templates, assets, pages,
menus and attachments of that theme exist for that site only.

## 23. Remove a theme from a site

**Actor.** Editor and Designer.

1. The default style configuration is reset first, whether or not a theme is installed: the font, the
   headings font, the navigation bar font, the buttons font, the palette number, the palette name, the
   ripple effect, the header template, the footer template and the footer scroll-to-top variables are all
   set to unset; the two ripple effect assets are disabled; every header template except the default one
   is disabled and the default one is enabled; the same is done for footer templates; and the footer
   scroll-to-top option is disabled.
2. When the site has no theme, the workflow stops.
3. The themes of the stream are unloaded in reverse installation order. Unloading deletes, for each
   template entity, the copies bound to this site, then cleans the orphans.
4. The site's theme pointer is cleared.

## 24. Refresh a theme

The chain of packages the current theme depends on is upgraded. The reload happens automatically through
the installation write described in §22.

## 25. Run the site configurator

**Actor.** Editor and Designer, on a site whose configurator has not been completed.

1. The configurator endpoint answers "not found" to a caller outside the Editor and Designer group,
   redirects to the site when the configurator is already done, and redirects to the default language
   prefix when the current language is not the site's default language, because the generated content is
   written in the default language.
2. **Step: initialisation.** The client receives the list of configurator features — identifier, name,
   description, kind `page` or `app`, icon, preselection list and package state — the company logo when it
   is not the shipped default one, the completion flag, and the industry list fetched from the content
   suggestion service, which is empty when the service is unreachable.
3. **Step: industry.** The designer picks an industry or types a free one; a free industry is reported to
   the service so that the catalogue can grow.
4. **Step: purpose and kind.** The designer states the purpose and the kind of the site; the answers
   preselect the features whose preselection list contains the chosen value.
5. **Step: palette and theme.** Recommended themes are requested from the service for the chosen
   industry, together with the locally available themes and their preview pictures; each returned theme's
   preview vector picture is recoloured with the chosen palette and its placeholder pictures are replaced
   by industry pictures.
6. **Step: features.** The designer confirms the features.
7. **Apply.**
   1. The chosen theme is installed (§22) and the completion flag is set.
   2. The guided tour asset of the configurator is copied for this site and activated.
   3. The logo is set: from the uploaded attachment when one was given, the attachment being rebound to
      the site logo field, and otherwise from the company logo when it is not the shipped default.
   4. The palette is applied: the palette name variable is set; when a custom colour list was given, the
      five palette colours are written as variables.
   5. When more than five selected features request a menu entry and more than one of them asks for the
      Company grouping, a Website Menu named `Company` is created under the root menu at the ordering
      value 40.
   6. For every selected feature: a feature backed by a package queues that package for installation when
      it is not installed and records its address and menu ordering value; a feature backed by a page
      template creates the page through the New Page operation with the feature address, published, and
      with a menu entry at the feature's ordering value under the Company menu or under the root menu.
   7. Queued packages are installed immediately.
   8. Menu ordering values of the endpoints created by the installed packages are applied, and packages
      may create additional entries, for example one blog per blog-kind feature, the first one reusing the
      shipped blog entry.
   9. The extension hook lets installed packages perform their own setup.
   10. The call-to-action data is read from the possibly extended site record; when a call-to-action label
       exists, a site-specific extension template is created that replaces the two call-to-action
       variables in the content-block template, and the header call-to-action link and label are rewritten
       in the header template.
   11. The footer link list is rewritten in every shipped footer template: the variable holding the
       configurator footer links is replaced by the current list, which by default contains a single entry
       labelled `Privacy Policy` pointing at `/privacy`.
   12. The custom resources for the chosen industry are fetched from the service.
   13. For the home page and for every created page, the theme's content block list is resolved, each
       block is rendered in the site's default language, its texts are replaced by placeholders, and the
       placeholder set is collected. The translation coverage is computed
       ([calculations.md](calculations.md) §27); when it exceeds 0.8 the placeholders are sent to the text
       generation service with the language name, the industry and the database identifier, and each
       returned text has the marker `XXXX` replaced by the site name. When the coverage is at or below
       0.8, no text is generated and the shipped texts are kept.
   14. Each page is then assembled: every block is re-rendered with the generated texts, marked with its
       block name, preconfigured with the block defaults and the theme customisations — filter identifier
       and result count, template key and its class, colour level class, class additions and removals
       possibly targeting a descendant, data attributes, inline style, background colour, background
       picture and background shape — stripped of its dialogue preview elements, and, for the first and
       the last block, given the shape classes that connect it to the header and footer colours. The
       assembled blocks are written into the last editable region of the page template, and the page
       template is duplicated under a key made of the index, an underscore, the original key and the
       suffix `_configurator_pages_landing`, so that the untouched page remains available as a template.
   15. Industry pictures are downloaded with a three-second timeout and stored as public attachments of
       the site, each registered under an external identifier made of `configurator_`, the site identifier,
       an underscore and the picture name, so that repeated runs do not duplicate them. A fixed fallback
       table maps pictures an industry did not provide onto another picture of the same industry.
   16. The designer is redirected to the site.

**Postcondition.** The site has a theme, a logo, a palette, a home page and feature pages filled with
content blocks, menu entries for the selected features, footer links, and the configurator is marked as
done.

**Skip branch.** Skipping the configurator sets the completion flag and installs the shipped default
theme.

## 26. Deactivate unused content block assets

**Actor.** Scheduled job runner, weekly.

1. Every asset whose path contains the content-block folder marker is read, inactive ones included,
   ordered by identifier.
2. Each path is parsed as the package name, any intermediate folders, the content-block folder, the block
   name, then a three-digit version optionally followed by an underscore and a variant, then the
   extension; a style source counts as a style asset and a script counts as a script asset.
3. For each distinct triple of block name, version and asset kind, usage is determined once:
   * the block's own template is rendered and its root element's opening tag is taken as one occurrence,
     because template definitions carry no block marker;
   * every stored rich text column of every concrete, non-abstract, stored entity that is not on the
     exclusion list — messages, activities and digest tips — is scanned for opening tags carrying the
     block marker for that block name;
   * an occurrence counts as using the version `000` when it carries no version attribute for that asset
     kind, and counts as using any other version when it carries that exact version.
4. An asset whose usage differs from its active flag is toggled.
5. A compatibility rule keeps the legacy quotation block style active when the carousel quotation block at
   version `000` or `001` is in use and the legacy block has no marker of its own.

---

# Part 3: Public forms

## 27. Submit a public form

**Actor.** Public visitor or portal user.
**Precondition.** A form whose action targets the public form endpoint for an entity is rendered on a
page, and the target entity is flagged as usable in forms.

1. The submission arrives on the form endpoint with the entity name in the path and the field values in
   the body. Request forgery protection is partial: the token is checked only when the session is
   authenticated, because a form embedded in a third-party page loses its session cookie (WS-205).
2. The submission runs inside a savepoint, so that a validation failure rolls back only the form
   handling.
3. The target entity is looked up with elevated rights among the entities flagged as usable in forms.
   When it is not found, the answer carries the error `The form's specified model does not exist`.
4. The human verification for the action name `website_form` runs (§28). A failure raises a validation
   error, which is returned as an error answer carrying the message.
5. The submitted values are extracted and classified:
   * the set of writable fields is computed as described in
     [content-management.md](content-management.md) §8.2;
   * a value that carries a file name is either decoded into a writable binary field, with the file name
     stored in the companion name field when one exists, or kept as an attachment;
   * a value whose name matches a writable field is converted with the filter for that field type; a
     conversion failure adds the field to the error list;
   * a value whose name matches a dynamic-property pseudo-field is appended to the property list of its
     parent field;
   * every other value, except the request forgery token and the form signature, is kept as a free-text
     pair;
   * when the target entity is the outgoing mail entity, the sender address is additionally kept as a
     free-text pair, so that it appears in the message body.
6. When the metadata parameter is enabled, the network address, the user agent, the accepted languages
   and the referring address are collected as request metadata.
7. When the target entity declares a value filter, it is applied to the extracted record values.
8. Required writable fields that received no value are collected. When any field failed conversion, the
   answer carries the list of failed fields together with the missing required fields.
9. The record is created with elevated rights and without subscribing the creator to its discussion
   thread. For the outgoing mail entity, the reply address is set to the submitted sender address and the
   sender address is replaced by the company name followed by ` form submission` in quotation marks and
   the company address in angle brackets.
10. When free text or metadata was collected, it is assembled as: the existing value of the designated
    field, a blank line, then the heading `Other Information:` — or, for the outgoing mail entity,
    `This message has been posted on your website!` — followed by a separator line and the free-text
    pairs, then the heading `Metadata` followed by a separator line and the metadata. When the entity
    designates a default field, the assembled text is written there, converted from line breaks to markup
    when that field is rich text or when the entity is the outgoing mail entity; otherwise it is logged as
    a comment message on the record.
11. Attachments are created for every kept file, related to the created record. When the file's field name
    is a writable field, the attachment is linked to that field, as a single link for a single-valued
    field and as an addition for a multi-valued one; otherwise it is collected as an orphan.
12. Orphan attachments are attached to a comment message on the record whose body is the paragraph
    `Attached files: `, except for the outgoing mail entity, where they are added to the message's own
    attachment list.
13. For the outgoing mail entity, the signature accompanying the form is verified before sending: it must
    equal the keyed digest of the recipient address, extended with `:email_cc` when a copy recipient field
    was present. A mismatch is refused with `invalid website_form_signature`. The message is then sent
    immediately instead of waiting for the queue.
14. A constraint violation that cannot be attributed to a field answers `false`.
15. The submitting session remembers the created entity name, its label and the created identifier, so
    that the success page can show the created record.
16. The answer carries the created identifier.

**Postcondition.** One business record exists, with its attachments, and the session can display the
confirmation.

## 28. Verify that a submission comes from a human

**Actor.** The form endpoint, or any endpoint declaring a verification action name.

Two verification services can be installed. When both are installed both run, and the submission must
pass both. Their outcomes and messages are specified in
[content-management.md](content-management.md) §9. In summary:

1. When the parameter that enables a service is false, that service's check passes.
2. The token supplied with the submission is removed from the request parameters and sent, together with
   the private key and the network address, to the verification endpoint of the service with a two-second
   timeout.
3. The answer is classified and either passes or raises a validation error carrying the message of the
   matching outcome.
4. Independently of the two services, the shipped form template includes a decoy field that no human
   fills; a submission that carries a value in it is treated as an unmatched free-text value.

---

# Part 4: Visitors and consent

## 29. Track a visitor

**Actor.** The request pipeline, after a response was produced.

1. When the user agent is a crawler, nothing happens.
2. When the response status is not 200, or the request carries the tracking-disabled header with the
   value `1`, nothing happens.
3. The served template is determined: from a cached response, the stored page and template identifiers
   are used; from a normally rendered response, the main object is used as the page when it is a Website
   Page, and the response template name is used as the template, prefixed with the site package name when
   it carries no package prefix.
4. When a template was determined, the database is writable and the template's tracking flag is set, the
   dispatch hook runs.
5. The dispatch hook builds the tracking values: the full request address, plus the page identifier when
   the served content was a page.
6. The visitor is fetched from the request with forced creation and with those tracking values, which
   performs one insert-or-update statement:
   * the browsing token is the contact identifier of the signed-in user, or the 32-character digest of the
     network address, the user agent and the session identifier for an anonymous visitor;
   * on insert, the row receives the token, the request language, the country resolved from the
     geolocation code — empty when the code is unknown — the site, the resolved time zone, the acting user
     as creator and last writer, the current moment as creation, write and last connection moments, and a
     visit count of 1;
   * on conflict with an existing token, the last connection moment is set to the current moment and the
     visit count is increased by one only when the previous last connection moment was more than eight
     hours ago;
   * in the same statement, a Website Visit Track row is inserted with the visitor identifier, the
     address, the page identifier and the current moment.

**Postcondition.** Exactly one Website Visitor exists per token, and one Website Visit Track exists per
tracked page view.

**Variant: tracking outside the page pipeline.** Other folders record views, for example a product view,
by calling the add-tracking operation with a lookup condition and the tracking values. That operation
reads the most recent matching track of the visitor and creates a new track only when there is none or
when the most recent one is older than thirty minutes; it then refreshes the visitor's last connection
moment, increasing the visit count by one when the previous moment was more than eight hours old.

## 30. Link visitors when a user signs in

**Actor.** Any visitor who authenticates.

1. Before authentication, the visitor of the current request is read, without forcing creation.
2. Authentication runs.
3. When authentication succeeded and a visitor existed before it:
   * search for a visitor already linked to the contact of the authenticated user;
   * when one exists and it is not the pre-authentication visitor, merge the pre-authentication visitor
     into it: every Website Visit Track of the anonymous visitor is repointed to the target visitor and the
     anonymous visitor is deleted. Merging into a target that has no contact is refused. The target's last
     connection moment is then refreshed;
   * when none exists, the pre-authentication visitor's browsing token is replaced by the contact
     identifier of the authenticated user, which makes the derived contact link resolve to that contact,
     and its last connection moment is refreshed.

**Postcondition.** A signed-in person has exactly one visitor record, holding the merged history of every
device and session they browsed from.

## 31. Delete inactive visitors

**Actor.** Scheduled job runner, daily.

1. The inactivity condition is "the contact is empty and the last connection moment is older than the
   present moment minus the retention period", the retention period coming from the configuration
   parameter `website.visitor.live.days` and defaulting to 60 days.
2. At most one batch, by default 1000 visitors, is read and deleted. Deleting a visitor cascades to its
   Website Visit Track rows.
3. The job reports the number processed and, when the batch was full, the number still matching, so that
   the runner schedules another pass.

**Postcondition.** Anonymous visitors older than the retention period, together with their tracking
history, no longer exist. Visitors linked to a contact are never deleted by this job.

## 32. Manage cookie consent

**Actor.** Any visitor, when the consent bar is enabled.

1. The consent bar is rendered after the footer. It offers `Only essentials` and `I agree`.
2. The visitor's answer is stored in the consent cookie as a structured document mapping cookie
   categories to booleans, with a retention of 999 days.
3. The permission check for a cookie category behaves as follows: required cookies are always allowed; for
   optional cookies, when the site's consent bar is disabled the answer is "allowed", because the site is
   assumed to implement its own consent; otherwise the stored document is read and the value for the
   optional category is returned, defaulting to "refused" when absent. A stored value that is not a
   structured document, that is a legacy plain flag, causes the cookie to be deleted and the answer to be
   "refused", so that the visitor is asked again.
4. When optional cookies are refused, the audience measurement script is loaded with every consent
   category denied and registers a one-shot handler that grants them when the visitor later accepts.
5. When optional cookies are refused, the consent bar is enabled, third-party blocking is enabled and the
   viewer is not a Restricted Editor, third-party content is neutralised (§33).

## 33. Neutralise third-party content

**Actor.** The rendering pipeline, for every rich text value and for the page markup.

1. Every script element and frame element of the markup is examined.
2. An element is watch-listed when its source host equals a blocked domain with the `www.` prefix removed,
   or ends with a dot followed by that domain. Container elements are watch-listed when one of their
   classes is in the blocked container class set, which contains the map block, the picture-network page
   block, the social-network page block, the background video block and the embedded video frame.
3. A watch-listed element receives the marker attribute `data-need-cookies-approval` set to `true`; when
   it carries a source, the source is moved to `data-nocookie-src` and the source is replaced by the blank
   document address.
4. In the browser, a watcher script receives the effective block list and applies the same treatment to
   frames created after the page has loaded.

---

# Part 5: Publication, blog, forum and public directories

## 34. Publish or unpublish a record

**Actor.** Any user whose write access on the record is granted, subject to the entity's own override.

1. The publication right is evaluated: the platform write check is attempted on the record and a refusal
   makes the right false. Website Page overrides this: a member of the Editor and Designer group always
   has the right.
2. Toggling flips the publication switch and returns the new value.
3. Creating a record already published, or writing the publication flag, without the right is refused with
   `You do not have the rights to publish/unpublish`.
4. For an entity carrying the multi-site publication mixin, the switch is evaluated for the site in
   context: a record bound to another site reads as unpublished there even when its stored flag is true.
5. For a Blog Post, publishing also sets the publishing date and posts the publication message (§35). For
   a Website Page, the visibility of every menu entry targeting it is recomputed on the next render. For a
   Product Template, the publication date is set to the present moment.

## 35. Publish a blog post

**Actor.** Editor and Designer, or any user with write access to the post.

1. The post is created or written with the publication flag true.
2. When the values carry no explicit publishing date, and the post has no publishing date or its
   publishing date is already past, the publishing date is set to the current moment.
3. For every non-archived post concerned, a message is posted **on the parent blog**, rendered from the
   shipped new-post template with the post as render value, the post title as subject and the publication
   subtype. Blog followers receive it.
4. Unpublishing sets the publication flag false and, under the same date condition, clears the publishing
   date.
5. Archiving a post forces the publication flag to false. Archiving a blog archives every post of that
   blog; unarchiving a blog unarchives them.

**Postcondition.** The post is reachable at its public address for visitors once its publishing date is
past, and appears in the blog listing, the site index and the site search.

## 36. Read a blog post

**Actor.** Any visitor.

1. The post is resolved from the two slugs. When the post does not belong to the blog in the address, the
   visitor is redirected permanently to the correct address.
2. The set of posts of the blog is read; a non-designer sees only posts whose publishing date is past.
   When the requested post is not in that set, the visitor is redirected to the blog page.
3. The next post is the following identifier in that set, wrapping around to the first; when the blog has
   one post there is no next post.
4. The page is rendered with the post as main object, the blog list, the tag list, the archive navigation
   and the next post.
5. The view counter is increased by one, without locking, the first time the post is seen in a session;
   the session then remembers the post identifier, so that reloads do not count again.

## 37. Browse a blog listing

**Actor.** Any visitor.

1. The listing endpoint accepts an optional blog, an optional comma-separated tag list, a page number and
   a search term.
2. When no blog was addressed and exactly one blog exists, the visitor is redirected temporarily to that
   blog's address.
3. When several tags were addressed and the verb is a read verb, the visitor is redirected temporarily to
   the first tag only.
4. The tag slugs are normalised: unknown identifiers are dropped and the remaining ones are re-slugified;
   when the normalised list differs from the requested one, the visitor is redirected permanently to the
   corrected address.
5. The condition is the site dimension, plus the blog, plus the date interval when both bounds were given,
   plus the tags. A designer additionally receives the published and unpublished counts and may filter on
   either state; a non-designer only sees posts whose publishing date is past.
6. The site search runs on the blog-post scope with the ordering "publication flag descending, publishing
   moment descending, identifier ascending" and a page size of 12.
7. The tag cloud is the set of tags used by the posts of the shown blogs, split into categorised tags,
   sorted by category name in upper case, and uncategorised tags, sorted by tag name in upper case.
8. The archive navigation groups the posts by publishing month and formats the month and year names in the
   visitor's language and time zone.

## 38. Produce a blog subscription feed

**Actor.** Any visitor or feed reader requesting the feed address of a blog.

The most recent posts of that blog, ordered by publishing date descending, are rendered into a syndication
document whose media type is that of the syndication format. The number of entries is the requested limit
capped at 50 and defaulting to 15. The base address of the blog is used to build absolute links, and the
post content is converted to plain text.

## 39. Ask a question on a forum

**Actor.** Any participant with a reputation at or above the asking threshold of the forum.

1. The participant opens the ask form of the forum. A forum whose privacy is `connected` refuses an
   anonymous visitor; a forum whose privacy is `private` refuses anybody outside its authorised group.
2. The participant fills the title, the content and the tags. Tags are submitted as one text in which an
   existing tag is its identifier and a new tag is an underscore followed by the name.
3. On submission the tag text is parsed: identifiers are kept; each new name reuses an existing tag of the
   same forum when one already carries that name, and otherwise creates one only when the participant's
   reputation reaches the tag-creation threshold and the name is not empty (WS-585).
4. An empty title is refused with `Title should not be empty.` and an empty content with
   `Question should not be empty.`, both rendered as a bad-request page.
5. The content is rewritten: when the author's reputation is below the no-follow threshold, every link
   element receives the no-follow marker; when the author's reputation is below the editor threshold and
   the content carries a picture, a link or a background picture, the write is refused with
   `%d karma required to post an image or link.` with the threshold substituted.
6. The post is created. When the author's reputation is below the validation threshold, the state is
   forced to `pending`; otherwise the state is `active` and the author's reputation increases by the
   asking award.
7. The state notification runs: an active question posts the shipped new-question message on itself with
   the question title as subject; a pending question posts the shipped validation message as an internal
   note addressed to every follower of the post and of its tags whose reputation reaches the moderation
   threshold. Followers of the tags of the post are added as recipients.
8. When the forum allows sharing, the participant is offered the social sharing actions after posting.

**Postcondition.** The question is reachable at its public address once it is active; a pending question is
visible only to its author and to the moderators.

## 40. Answer, comment, vote and accept

**Actor.** Any participant.

1. **Answer.** The reply form creates a post whose parent is the question, whose title is `Re: ` followed
   by the question title, and whose state is `active`. The answering threshold applies. Answering a closed
   or archived question is refused with
   `Posting answer on a [Deleted] or [Closed] question is not possible.` In a forum whose mode is
   `questions`, a participant may post only one answer per question. The shipped new-answer message is
   posted on the question and the question's last activity moment is refreshed.
2. **Comment.** A comment is a message on the discussion thread of a post, not a post. The commenting
   threshold applies, distinguishing own posts from other people's posts.
3. **Convert.** A comment may be converted into an answer, and an answer into a comment, subject to the
   conversion thresholds; the messages are listed in [business-rules.md](business-rules.md).
4. **Vote.** Casting a vote creates or updates a Forum Post Vote and moves the author's reputation by the
   difference between the new and the old award ([calculations.md](calculations.md) §21). A participant may
   not vote on their own post and may not modify somebody else's vote. Voting the same way twice withdraws
   the vote by storing the value `0`.
5. **Accept.** Accepting an answer sets its acceptance flag, clears the flag on every other answer of the
   question, awards the acceptance award to the answer's author and the acceptance bonus to the accepting
   participant, unless the two are the same person, and sets the question's answered flag. Withdrawing the
   acceptance reverses all of it.
6. **Favourite.** A participant may mark a question as a favourite, which adds them to the favourite list
   and increases the favourite counter.
7. Each of these operations refreshes the last activity moment of the question, which is what the default
   ordering "Last Updated" reads.

## 41. Moderate a forum

**Actor.** Any participant whose reputation reaches the moderation threshold of the forum.

1. The moderation queues are the pending posts, the flagged posts, the closed posts and the posts with
   fewer than the configured number of votes; each is a filtered listing of the forum.
2. **Validate.** A pending post becomes active, its active flag is set, the moderator is recorded, the
   asking award is granted and the state notification runs.
3. **Refuse.** Only the moderator is recorded; the post stays pending and invisible.
4. **Flag.** Any participant whose reputation reaches the flagging threshold may flag an active post. The
   answer distinguishes a moderator from an ordinary participant; a post already flagged answers with an
   "already flagged" marker and a post in any other state with a "not flaggable" marker.
5. **Mark as offensive.** A flagged or active post becomes offensive with an offensive reason: the active
   flag is cleared, the moderator, the closing moment and the reason are recorded, and the author loses the
   flagging award.
6. **Close.** A question is closed with a basic reason: the closing user, moment and reason are recorded.
   With the offensive reason or the spam reason the author loses the flagging award, multiplied by ten for
   the spam reason when the question is the author's first question in that forum.
7. **Reopen.** The state returns to active and, when the question had been closed with the offensive or the
   spam reason, the deduction is given back with the same tenfold rule.
8. **Delete.** Archiving a post archives every answer of the post; deleting an accepted answer withdraws the
   acceptance award from its author and the acceptance bonus from the participant who accepted it.

All the reputation thresholds and their refusal messages are in
[business-rules.md](business-rules.md) Part 3, and the state table is in
[state-machines.md](state-machines.md) §7.

## 42. Publish a contact reference page

**Actor.** Salesperson or Editor and Designer.

1. A contact is published by setting its publication flag; the change is tracked and posts a message with
   the subtype Partner published or Partner unpublished.
2. The public page is the contact path followed by the contact slug. A visitor reaching a stale slug is
   redirected to the current slug. A contact that is not published is reachable only by a Restricted
   Editor; otherwise the request answers "not found".
3. The customer reference directory lists published contacts that have an assigned partner, faceted by
   industry and by country, filterable by a published Partner Website Tag and by a free-text search over
   the name, the public description and the industry name. When the addressed country has no matching
   contact but other countries do, the country filter is dropped and the listing reports that it fell back
   to every country. The page size is 20.
4. The map frame renders at most the requested number of published contacts, 80 by default, with their
   name, address, latitude and longitude, and links each marker to the contact's reference page.

## 43. View and edit a public user profile

**Actor.** Any visitor for viewing; the account owner for editing.

1. Viewing another account's profile requires that the account is published and that the viewer's
   reputation score reaches the site's minimum, which defaults to 150. A private profile answers with the
   denial page and the reason `This profile is private!`; an insufficient score answers with the reason
   `Not have enough karma to view other users' profile.`; a non-existing account answers "not found". An
   account always sees its own profile.
2. The avatar endpoint serves only the four permitted picture sizes and elevates its rights only when the
   account is published and has a strictly positive reputation score.
3. Editing writes the name, the personal site, the electronic mail address, the city, the country and the
   public description; the publication flag is writable only by the owner, and an administrator may edit
   another account.
4. Address validation: a token is derived from the current day, a stored secret, the account identifier and
   the address; the validation message is sent with a link carrying the token; consuming a valid token for
   an account whose score is zero raises the score to 3.
5. The profile page shows the ranks and the published badges of the account, its reputation history and its
   forum activity.

---

# Part 6: The storefront

## 44. Publish a product in the online shop

**Actors.** Sales Administrator or Restricted Editor.
**Preconditions.** A Product Template exists and is marked as sellable.

1. The actor opens the product and fills the storefront fields: the storefront description, the extra
   media, the storefront categories, the ribbon, the comparison price, the base unit count and reference
   unit, the alternative and accessory products, the availability settings and the out-of-stock message.
2. The actor sets a site restriction, or leaves it empty for every site.
3. The actor switches the publication flag on, either from the back-office form, from the storefront page
   toggle, or by selecting records in a list and ticking the published column.
4. On the write of the publication flag the publication date is set to the present moment. When the product
   belongs to a print-on-demand template with missing print pictures, the write is refused with
   `Print images must be set on products before they can be published.`
5. When the product was created from the simplified storefront creation form, selecting at least one
   storefront category publishes it automatically, and clearing the selection unpublishes it.
6. The product now matches the site sellable-product condition and appears in the listing, in the search, in
   the feed and in the site index.

**Postconditions.** The product is reachable at its storefront address; it is counted by the
published-products flag of its categories and of every ancestor of those categories; the "when new" ribbon
applies for the configured number of days after the publication date.

**Variant.** Publishing a paid course publishes its linked product; unpublishing the course unpublishes the
product unless another published course uses it.

## 45. Browse, filter and search the catalogue

**Actors.** Anonymous visitor or registered customer.
**Precondition.** The site allows shop access for that visitor.

1. The visitor opens the shop listing. The listing pipeline of
   [storefront-catalogue.md](storefront-catalogue.md) §2 runs.
2. The visitor selects a category. The address becomes the category listing address; the condition gains
   "the storefront categories are the category or one of its descendants"; the category strip switches to
   the children of that category, or to its siblings when it has no children.
3. The visitor ticks two values of the same attribute. The address gains one attribute filter entry holding
   both value identifiers; the condition gains one clause requiring any of them; the session records the
   selection.
4. The visitor drags the price slider. The address gains a minimum and a maximum price in the display
   currency; the condition converts them to the company currency with today's rate; the bounds are clamped
   against the available bounds ([calculations.md](calculations.md) §13).
5. The visitor types a search term. The approximate search runs; when the exact term matches nothing, the
   closest term is searched and both terms are reported.
6. The visitor changes the sort order. The address gains the order, and the search is re-run with
   "publication flag descending, the chosen order, identifier descending".
7. The visitor moves to the second page. The address gains the page segment and the slice offset becomes the
   page size.

**Records written.** None, except the session keys for the attribute filter, the layout mode and the price
list cache.

**Postcondition.** The address alone reproduces the listing: a rebuild must keep every filter in the
address, not in the session, except the attribute memory used by the "back to the shop" link.

## 46. Configure a variant and add it to the cart

**Actors.** Anonymous visitor or registered customer.
**Precondition.** The product page is open and the product passes the add-to-cart permission check.

1. The visitor selects attribute values. Each change calls the combination information service with the
   template, the current variant, the chosen values, the quantity and the unit of measure.
2. The service resolves the variant, creating it when the template uses dynamic variant creation and the
   combination is possible, computes the price, the reference price, the discount flag, the extra prices,
   the availability and the media, and returns the payload of
   [storefront-catalogue.md](storefront-catalogue.md) §8.4.
3. The page updates the price, the strikethrough price, the per-unit price, the availability line, the media
   block, the tag block and the add-to-cart button.
4. The visitor types a quantity. When the payload reports a maximum, the input is capped at it.
5. The visitor presses the add-to-cart action.
   * **Decision.** When the configurator dialogue must be shown — optional products exist, or the product is
     not yet fully configured, or several units of measure are offered — the dialogue opens and collects the
     final combination, the custom values, the optional products and the combo choices.
   * Otherwise the add proceeds directly.
6. The add endpoint runs the steps of [storefront-checkout.md](storefront-checkout.md) §2.1.
   * Records created: a Sales Order when none existed, carrying the site, the company, the contact — the
     public contact for an anonymous visitor — the price list, the fiscal position, which is written
     explicitly only for the public user, and the sales team; one Sales Order Line per product, carrying the
     product, the demanded quantity, the unit of measure, the order, the parent line, the combo item, the
     no-variant attribute values and the custom attribute values.
   * Session written: the cart identifier and the cart quantity counter.
7. The availability hook may cap the quantity and produce a warning; the warning is written on the line, or
   on the order when no line was created.
8. The post-update verification re-rates or removes the delivery line and re-applies the promotions.
9. **Decision.** When the site's add-to-cart action is "go to cart", the browser is sent to the cart page;
   otherwise a notification panel lists what was added, with the quantity and the amount per line.

**Postcondition.** The cart holds the requested lines; the header counter shows the new cart quantity; the
cart becomes eligible to be an abandoned cart once the delay elapses, provided that its customer is not the
public contact.

## 47. Update or empty the cart

**Actors.** Anonymous visitor or registered customer.

1. The visitor changes a line quantity on the cart page. The update endpoint runs the operation of
   [storefront-checkout.md](storefront-checkout.md) §2.4.
2. **Decision.** A quantity of zero or less deletes the line; its linked lines — optional products and combo
   items — are deleted with it.
3. **Decision.** For a combo line, the requested quantity is checked against every combo item and lowered to
   the smallest available one; every child line is aligned; the warning is stored on the combo line.
4. The post-update verification runs once.
5. The response carries the re-rendered cart lines, the re-rendered totals, the re-rendered reorder history,
   the new cart quantity, the readiness flag and the amounts.
6. The visitor may press the clear-cart action, which deletes every line.

**Postcondition.** The cart quantity in the session matches the order; the delivery line is either re-rated
or gone; the promotions are consistent with the new content.

## 48. Recover an abandoned cart from the message link

**Actor.** Registered or guest customer who received a recovery message.
**Precondition.** The order is an abandoned cart with a portal access token.

1. The customer opens the recovery address, which is the cart page carrying the order identifier and the
   order access token as query parameters.
2. The token is compared in constant time. A mismatch or a missing order answers "not found".
3. **Decision.**
   * The order is no longer in draft: the page is rendered with the "already completed" flag.
   * The revival method is "squash", or it is "merge" and the session has no cart: the session cart becomes
     the old order and the browser is redirected to the cart page.
   * The revival method is "merge" and a session cart exists: every line of the old order is re-parented onto
     the session cart and the old order is cancelled.
   * No revival method and the old order is not already the session cart: the page shows the choice between
     merging and replacing.
4. The lines whose product is archived are deleted before rendering.

**Records modified.** The Sales Order Lines, through their order link; the old Sales Order, which reaches the
cancelled state on a merge; the session cart key.

**Postcondition.** Exactly one draft cart remains attached to the session.

## 49. Complete a checkout as a guest

**Actor.** Anonymous visitor.
**Preconditions.** A non-empty cart; the customer account policy is `optional` or `disabled`.

1. The visitor presses the checkout action on the cart page. The forward label comes from the next published
   checkout step.
2. The checkout page runs the cart guard, then the address guard. The cart is anonymous, therefore the
   visitor is redirected to the address form.
3. The address form is rendered with no contact, the address kind `billing`, the country defaulted from the
   visitor's network location, and the discard link pointing at the cart page.
4. The visitor fills the form and submits it.
5. The values are parsed into contact values and extra form data. Validation runs
   ([storefront-checkout.md](storefront-checkout.md) §6.4). A failure returns the invalid field list and the
   messages, and the page highlights them.
6. On success a Contact is created with the submitted fields, the kind "contact", the site's company, the
   site salesperson as its salesperson, and the site when accounts are site-specific. A language outside the
   site's languages is dropped.
7. Extra form data is handled: a ticked newsletter box subscribes the submitted address to the site
   newsletter list.
8. The order is updated with the customer, the billing address and, when the order is services-only or the
   address is new, the delivery address. The propagation of
   [storefront-checkout.md](storefront-checkout.md) §6.6 recomputes the fiscal position, the price list, the
   taxes and the prices, and re-selects the delivery method when one was already chosen.
9. The public contact is unsubscribed from the order's followers.
10. The browser is redirected to the checkout page with the skip flag set.
11. The checkout page lists the delivery methods, preselects one, rates it and writes the delivery line.
    **Decision.** When the order has no deliverable products and the skip flag is set, the page immediately
    redirects to the next step.
12. The visitor may open the address form again to add a distinct billing address; the "same as delivery"
    switch controls whether one address serves both roles.
13. The visitor presses the forward button. **Decision.** When the extra-information step is published, the
    extra-information page is rendered and its form writes fields, a message and attachments on the order.
    Otherwise the payment page is rendered.
14. The payment page recomputes the cart, computes the blocking errors and renders the payment form with the
    compatible providers and the express buttons.
15. The visitor chooses a payment method and confirms. The transaction endpoint locks the order, runs the
    readiness check, creates the Payment Transaction with the order's billing contact, the order currency and
    the order total, and returns the provider's processing values.
16. The provider processes the payment and calls back. On success the transaction reaches the completed or
    the authorised state.
17. The order is confirmed: a salesperson is assigned, the state becomes confirmed, the confirmation message
    is sent using the site template when one is set, delivery work is created, and, when automatic invoicing
    is enabled, the invoice is created, posted and sent.
18. The browser returns to the payment validation endpoint, which resets the storefront session and redirects
    to the confirmation page.
19. The confirmation page renders the order summary and the analytics payload. The visitor may print the
    order document.

**Records written.** One or two Contacts; the Sales Order, with its confirmed state, addresses, price list,
fiscal position, delivery line and salesperson; a Payment Transaction; optionally a Journal Entry, that is the
invoice; optionally a Transfer, that is the delivery; a mailing contact for the newsletter; messages on the
order.

**Postcondition.** The session no longer holds a cart; the last-order key still names the order, which is what
allows the confirmation page and the printed document to be served.

## 50. Complete a checkout as a registered customer with separate addresses

**Differences from §49.**

1. The cart was resolved from the session or, on a first visit, by searching for the customer's existing draft
   order on this site. In the latter case the address propagation is run for the customer's contact and the
   archived-product cleanup is applied.
2. The checkout page lists the customer's saved billing and delivery addresses. Addresses of the commercial
   contact are offered only when they are complete, because a child contact may not edit them.
3. Selecting an address calls the address selection endpoint, which verifies that the chosen contact is the
   customer, their commercial contact or one of its descendants of the kind invoice, delivery or other, and
   then runs the propagation.
4. Editing an address opens the address form with that contact; the validation additionally refuses changes to
   commercial fields, to the country once documents exist, and to the tax identification number once documents
   exist.
5. Changing the delivery address re-rates the delivery method: the available methods are recomputed, the
   preferred one is selected and the delivery line is rewritten.

## 51. Express checkout

**Actor.** Any visitor, with a payment provider that supports an express wallet.

1. The cart page renders the express button with the payload of
   [storefront-checkout.md](storefront-checkout.md) §9.1. The amount offered excludes delivery.
2. The wallet returns a partial delivery address. The storefront creates or matches a contact, writes it as
   the delivery contact while protecting the price list from recomputation, computes the available methods and
   their rates in express mode — excluding methods that need a pickup point and collect-in-store methods —
   sorts them by price, preselects the cheapest and returns the list.
3. The wallet may ask for a tax recomputation; the storefront recomputes the taxes in express mode and returns
   the total excluding delivery, or an error flag when an external tax service refuses the partial address.
4. The wallet returns the final billing address, delivery address and chosen shipping option. The storefront
   creates or matches the contacts, writes them on the order, records the order identifier in the session, sets
   the chosen method and returns the customer identifier.
5. The transaction is created through the ordinary transaction endpoint and the flow continues as in §49 from
   step 16.

**Postcondition.** The order carries real contacts, a delivery method and a delivery line; the placeholder
contacts created during the flow carry the order name in their own name, which is how a later call recognises
and completes them.

## 52. Buy and collect in a store

**Actor.** Any visitor.
**Preconditions.** The collect-in-store capability is installed, a collect-in-store method is published with
at least one store, and the site names a warehouse.

1. On the product page the availability block shows both the quantity available for delivery, taken from the
   site warehouse, and the quantity available for collection, taken from the best stock across the stores or
   from the already chosen store.
2. The visitor presses the store selection action. The location endpoint runs: with no cart, an unsaved order
   carrying the collect-in-store method is used, which avoids creating a cart; with a cart that has another
   method, the collect-in-store method and its delivery line are written on the cart first.
3. The stores are returned sorted by distance, each with its address, its opening hours and the availability of
   the product in that store.
4. The visitor picks a store. The store selection endpoint resolves or creates the cart, writes the
   collect-in-store method and its delivery line when needed, writes the pickup location, sets the order
   warehouse to that store and recomputes the fiscal position; when the fiscal position changed, the taxes are
   recomputed.
5. The visitor adds products. The availability cap now uses the store's quantity free to use.
6. At checkout the address step shows the store selector again; a single-store method is preselected
   automatically.
7. The payment step verifies that a store is selected and that every storable product is available there;
   otherwise it shows `Please choose a store to collect your order.` or
   `Some products are not available in the selected store.` and hides the payment form.
8. **Decision.** The visitor may pay online, or choose to pay on site when that provider is offered. Paying on
   site creates a transaction that stays pending; the order is nevertheless confirmed, which creates the
   transfer for the store.

**Records written.** The Sales Order, with its shipping method, pickup location payload, warehouse, fiscal
position and delivery line; a Payment Transaction; a Transfer.

## 53. Pay later by wire transfer

**Actor.** Any visitor; a wire-transfer provider is enabled and published.

1. The visitor selects the wire-transfer method on the payment step and confirms.
2. A transaction is created and immediately reaches the pending state.
3. The order is marked as sent; the payment reference is written on the order; the payment-status message is
   sent to the customer with the transfer instructions.
4. The order appears in the unpaid order list of the back office.
5. No stock is reserved and no invoice is produced.
6. When the funds arrive, a salesperson confirms the order by hand, which creates the delivery work and sends
   the confirmation message.

**Postcondition.** Until the manual confirmation, the order is in the sent state and is not an abandoned cart,
because its state is no longer draft.

## 54. Notify a shopper when a product is back in stock

**Actors.** Any visitor, then the scheduled job runner.

1. The product page of a sold-out product shows the subscription form, pre-filled with the user's address or
   with the address held in the session.
2. The visitor submits it. The endpoint validates the address format, the product eligibility and, for an
   anonymous visitor, that the address does not belong to an existing account.
3. The contact is found or created and added to the product's subscriber list. For an anonymous visitor the
   product identifier and the address are recorded in the session, which makes the page show the subscribed
   state on the next visit.
4. The hourly job scans the products that have subscribers. For a product that is no longer sold out, it
   renders the availability message in each subscriber's language, sends it from the company contact or, when
   there is none, from the site salesperson, and removes the subscriber.

**Records written.** A Contact, possibly created; the product's subscriber list; one outgoing message per
subscriber.

## 55. Save a product for later and buy it afterwards

**Actor.** Any visitor.

1. The visitor presses the wish list action on a product card or a product page. A wish list row is created
   with the current price, price list, currency and site; for an anonymous visitor the row has no contact and
   its identifier is appended to the session list.
2. The visitor opens the wish list page. The rows are filtered to published, addable products.
3. The visitor may remove a row, or move it to the cart, which performs an ordinary add.
4. The visitor signs in. The session rows are merged into their contact: duplicates are deleted, the rest are
   re-assigned, and the session list is cleared.
5. From the cart, a line may be moved back to the wish list.

**Postcondition.** The wish list survives the session for a signed-in shopper and is deleted after five weeks
for an anonymous one.

## 56. Compare products

**Actor.** Any visitor.

1. The visitor presses the comparison action on several product cards or product pages. The selection is held
   by the page.
2. The visitor opens the comparison panel and presses the compare action. The browser goes to the comparison
   page with the identifiers in the query string.
3. The page reads the variants through a search, which drops the ones the visitor may not see, and renders the
   matrix of [storefront-engagement.md](storefront-engagement.md) §2.3, grouped by attribute category.
4. The data endpoint supplies the price, the strikethrough price and the media for each column.

## 57. Send the abandoned cart messages

**Actors.** Scheduled job runner, hourly, then the customer.
**Precondition.** At least one site has the recovery feature switched on.

1. For each site with the feature on, the job selects the draft orders that are abandoned carts, have not been
   mailed and were created at or after the activation moment.
2. The eligibility filter of [storefront-checkout.md](storefront-checkout.md) §11.3 is applied; with the stock
   capability, carts containing a sold-out product are also removed.
3. Every rejected candidate is marked as mailed, which permanently removes it from future runs.
4. Every remaining order receives the recovery message and is marked as mailed.
5. The customer opens the link and lands in §48.

**Records written.** The recovery flag on the orders; one outgoing message per mailed order.

## 58. Send a recovery message by hand

**Actor.** Salesperson.

1. The salesperson opens the abandoned cart list, which is pre-filtered on the abandoned-cart flag with the
   recovery filter active and record creation disabled.
2. The salesperson selects one or several orders and presses the recovery action.
3. Every selected order receives a portal access token when it lacks one.
4. The message composer opens in mass-mailing mode for several orders and in single-message mode for one,
   pre-loaded with the site template when all the selected orders share one site and with the shipped template
   otherwise.
5. On send, the recovery flag in the composer context marks the orders as mailed: after a mass mailing, only
   the orders that are still abandoned carts and not yet mailed; after a single message, the order
   unconditionally.

## 59. Reorder a previous order

**Actor.** Registered customer.

1. From the portal order page the customer presses the reorder action. The reorder endpoint resolves the order
   through the portal access check.
2. The reorderable lines are selected: a product exists, it may be added to the cart, and the line is shown in
   the cart. Sections, notes, delivery lines, rewards, event tickets and courses are skipped. An order with
   nothing reorderable answers `Nothing can be reordered in this order`.
3. Each line is added to the cart with its quantity, its custom values, its no-variant values and, for a combo,
   the complete description of every combo item.
4. Warnings produced by adds that ended at quantity zero are joined with line breaks and written on the cart.
5. The customer is taken to the cart, where they may adjust or remove lines.

**Alternative.** From the cart, the quick reorder block offers the products of the last ten confirmed orders,
grouped by `Today`, `Yesterday` and the number of days ago, excluding what is already in the cart and what
cannot be sold.

## 60. Apply a promotional code

**Actor.** Any visitor; nominative programs require a signed-in shopper.

1. The visitor enters a code in the cart's promotional code box and submits it to the price list endpoint.
2. The promotion engine is asked first.
   * It reports "not found": the code is tried as a price list promotional code. A match that is available on
     the site is applied as the session price list and the selected price list, and the cart prices are
     recomputed. No match answers with a redirect carrying the code-not-available marker.
   * It reports an error: the message is kept in the session and shown once.
   * It succeeds: when exactly one reward results and it is not a multi-product reward, it is applied
     immediately; the code is kept in the session as the successful code.
3. Applying a reward refreshes the programs and, when the delivery method offers free shipping above a
   threshold and the reward is not a payment program, re-rates the delivery line.
4. The cart page shows the reward lines, with the discount lines of one program aggregated into a single
   displayed line.

**Alternative entry.** A visitor opening the coupon address before having a cart has the code stored in the
session; it is applied automatically on the first add to cart.

## 61. Create a site and its checkout flow

**Actor.** Administrator.

1. A Website record is created. For every generic checkout step, a copy is created with that site. The copy of
   the extra-information step is published only when that page option is active for the site.
2. The site configurator may apply a shop page style and a product page style, each of which enables and
   disables a set of page options, writes a set of presentation fields and writes the three category page
   options on every category.
3. The configurator may also generate a starter set of storefront categories for the chosen industry.
4. The administrator sets the storefront settings of [configuration.md](configuration.md) §1.
5. Switching the product feed feature on creates one feed per site whose published product count is within the
   soft limit, with the shipped name `GMC 1`, reproduced verbatim — the three capital letters abbreviate the
   name of the Google Merchant Center product listing service — and propagates the flag to every site.

## 62. Serve the product feed

**Actor.** The product syndication service, unauthenticated and token-bearing.

1. The service fetches the feed path with the feed identifier and the access token as query parameters. The
   path is a fixed literal with no variable part: a slash, the three lower-case letters that abbreviate Google
   Merchant Center, a dot, and the three lower-case letters that abbreviate extensible markup language. The two
   query parameter names are external wire literals, spelled as the external service stored them when the
   address was handed to it.
2. The site must have the feature enabled, the identifier must parse as a number, the feed must exist, the
   token must match in constant time, and the feed's site must be the requested one; otherwise the
   corresponding failure response is returned (WS-436).
3. **Decision.** When the cached document is missing or expired, the feed row is locked, the document is
   rendered in the feed's language and price list, compressed and stored, and the cache expiry is set to the
   start of tomorrow. Otherwise the cached bytes are returned.
4. The rendering may notify the site salesperson when the product count is above the warning threshold and no
   notice was sent in the last week.

**Records written.** The cached document, the cache expiry, and possibly the last notification date and a
notification message.

## 63. Change the tax display mode of a site

**Actor.** Sales Administrator.

1. The administrator switches the tax display mode between tax excluded and tax included.
2. Every storefront price computation now returns the other side of the tax computation; no stored amount
   changes.
3. Existing carts are unaffected in their stored line values; only the displayed subtotals, the displayed
   delivery amount and the cart notification amounts change.
4. Changing the company's fiscal country resets the setting to tax excluded, after which it may be set again.

---

## Reconciliation notes

1. **Two workflow catalogues.** One version numbered the site workflows 1 to 40 and the storefront workflows 1
   to 20. They are merged here into one sequence of 63; the mapping is: site workflows 1–14 keep their numbers
   1–14 except that the former workflow 5 "create a page" is now §5 and the former 21 "run the configurator" is
   now §25; the storefront workflows 1–20 are now §44–§63 in the same order, with the former workflow 18
   "create a website and its checkout flow" now §61.
2. **State tables.** One version repeated the state tables at the end of each workflow file. They are stated
   once, in [state-machines.md](state-machines.md), and are not duplicated here.
3. **Forum workflows.** Neither version carried an operational procedure for the forum; §39 to §41 are written
   from the entity behaviour of [entities.md](entities.md) §§4.1–4.5 and the state machine of
   [state-machines.md](state-machines.md) §§7–9.
4. **Pseudo-code.** One version expressed several procedures as code-like blocks. They are restated here, and
   in the topic files, as numbered procedures and as formulas, which is the form this repository uses.
