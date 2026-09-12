# Visitors and tracking

The deep specification of the browsing identity a site keeps for every visitor: how the identity is derived,
how a page view is recorded, how an anonymous identity becomes an identified one at sign-in, what statistics
are derived from the history, how the data is deleted, and how cookie consent gates all of it.

The entity fields are in [entities.md](entities.md) §1.7 and §1.8, the state table in
[state-machines.md](state-machines.md) §6, and the arithmetic in
[calculations.md](calculations.md) §26.

---

## 1. The browsing identity

### 1.1 The token

```formula
browsing token = the decimal contact identifier of the signed-in user
browsing token = the first 32 characters of the hexadecimal digest of
                 ( the remote network address, the browser user agent, the session identifier )
                 for the public user
```

The token is unique across visitors (WS-231). A token of exactly 32 characters means an anonymous visitor
with no contact; any other token is read back as a contact identifier, which is how the contact link is
derived rather than stored (WS-232).

Deriving a token outside a request fails with `Visitors can only be created through the frontend.`
(WS-233), which is why no background job may create a visitor.

### 1.2 What is stored at creation

The site on which the visitor was first seen; the front-end language in effect; the country resolved from
the geolocation code of the network address, left empty when the code is unknown (WS-238); the time zone
resolved from the browser time zone cookie when its value is a recognised name, otherwise from the
signed-in user's time zone, otherwise empty (WS-239); the creation moment; the last connection moment; and
a visit count of 1.

### 1.3 What a visitor is not

A visitor is not a user account and not a contact. One person browsing from three devices while signed out
has three anonymous visitors; the same person signing in on each device ends with one identified visitor,
because §3 merges them.

---

## 2. Recording a page view

### 2.1 Preconditions

Tracking happens only when the user agent is not a crawler, the response status is 200, the request does not
carry the tracking-disabled header with the value `1`, the database is writable, and the served template has
its tracking flag set (WS-234). The tracking flag is a field of the template, so a designer decides which
pages are tracked; pages created through the New Page operation are created with tracking enabled.

### 2.2 Determining the served template

* From a cached response, the page and template identifiers stored with the cache entry are used, which is
  what keeps tracking working for a page served from the cache.
* From a normally rendered response, the main object is used as the page when it is a Website Page, and the
  response template name is used as the template, prefixed with the site package name when it carries no
  package prefix.

### 2.3 The single statement

Creating or refreshing a visitor and recording its track is performed by **one** insert-or-update statement
(WS-571), which does the following at once.

1. Insert a visitor row with the token, the language, the country, the site, the time zone, the acting user
   as creator and last writer, the current moment as creation, write and last connection moments, and a
   visit count of 1.
2. On a conflict with an existing token, set the last connection moment to the current moment and increase
   the visit count by one **only** when the previous last connection moment was more than eight hours ago.
3. Insert a Website Visit Track row with the visitor identifier, the full request address, the page
   identifier when the served content was a page, and the current moment.

Two concurrent first requests of the same visitor therefore cannot both insert. The two statements that
refresh a visitor's time zone and last connection moment select the row for update while skipping locked
rows, so a concurrent request never blocks and never fails (WS-570).

### 2.4 Tracking outside the page pipeline

Other folders record views — a product view, a talk view, a course view — by calling the add-tracking
operation with a lookup condition and the tracking values.

1. Read the most recent track of the visitor matching the lookup condition.
2. Create a new track only when there is none, or when the most recent one is older than thirty minutes
   (WS-237).
3. Refresh the visitor's last connection moment, increasing the visit count by one when the previous moment
   was more than eight hours old.

The thirty-minute window is what keeps a shopper who reloads a product page from producing dozens of
entries.

---

## 3. Linking identities at sign-in

1. Before authentication, read the visitor of the current request, without forcing creation.
2. Run the authentication.
3. When it succeeded and a visitor existed before it:
   1. Search for a visitor already linked to the contact of the authenticated user.
   2. When one exists and it is not the pre-authentication visitor, **merge**: every track of the anonymous
      visitor is repointed to the target and the anonymous visitor is deleted. Merging into a target that has
      no contact is refused with the message of WS-241. The target's last connection moment is then
      refreshed.
   3. When none exists, the pre-authentication visitor's token is replaced by the contact identifier, which
      makes the derived contact resolve to that contact, and its last connection moment is refreshed.

A signed-in person therefore has exactly one visitor record, holding the merged history of every device and
session they browsed from. Signing out and browsing again creates a new anonymous visitor; the identified one
is untouched.

The same sign-in also migrates the anonymous wish list rows
([storefront-engagement.md](storefront-engagement.md) §1.5) and clears the storefront session keys
(WS-330).

---

## 4. Statistics derived from the history

| Value | Derivation |
|---|---|
| Number of visits | Stored; increased by one on a page view whose previous last connection moment is more than eight hours old (WS-235). |
| Tracked page views | The number of track rows of the visitor. |
| Distinct pages | The number of distinct pages among the track rows. |
| Pages visited | The distinct pages themselves, readable only by the Editor and Designer group, computed with elevated rights and written back through an elevated write (WS-245). |
| Last page | The page of the most recent track that carries one. |
| Time since the last action | A human phrasing of the elapsed time since the last connection moment. |
| Connected | True when the last page view happened less than five minutes ago (WS-236). |
| Product views, distinct products, products viewed | Computed from one grouped read over the track rows that carry a product, restricted to products of the companies the reader may see ([entities.md](entities.md) §5.12). |

Other folders add their own counters to the same record: leads, event registrations, wish-listed talks, live
chat sessions.

**Worked example.** The counting rules are exercised end to end in
[calculations.md](calculations.md) §26.

---

## 5. Deletion and retention

### 5.1 The scheduled cleanup

A daily job deletes the visitors whose contact is empty and whose last connection moment is older than the
present moment minus the retention period, which comes from the retention parameter and defaults to 60 days.
Deleting a visitor cascades to its track rows. Visitors linked to a contact are never deleted by the job
(WS-242).

The job processes at most one batch, by default 1000 visitors, and reports the remaining count when the
batch was full, so that the runner schedules another pass (WS-243).

### 5.2 What a rebuild should add

The observed behaviour keeps identified visitors for ever. The industry-standard default WS-904 asks for an
explicit retention for identified visitors as well, and for a per-visitor erasure operation that deletes the
visitor and its tracks, so that a data-protection request can be honoured without a manual query.

### 5.3 Access

Model access grants read, update and delete on Website Visitor to the Editor and Designer group and to the
administrator group, but never create: visitors are created only by the tracking statement (WS-246). Tracks
are fully writable by the Editor and Designer group, because a deletion of a visitor must cascade.

---

## 6. Contacting a visitor

The visitor form offers a message composer. It requires the visitor to be linked to a contact that has an
electronic mail address; otherwise the action fails with
`There are no contact and/or no email linked to this visitor.` (WS-244).

---

## 7. Consent gates the whole mechanism

| Consent state | Effect on tracking |
|---|---|
| The consent bar is disabled on the site | Optional cookies are allowed, on the assumption that the site implements its own consent mechanism; tracking runs (WS-471). |
| The consent bar is enabled and the visitor has not answered | The optional category is refused; the audience measurement script is loaded with every category denied and registers a one-shot handler that grants them when the visitor later accepts (WS-479). |
| The visitor accepted only the essentials | The optional category stays refused and third-party content is neutralised. |
| The visitor accepted everything | The optional category is allowed and the measurement categories are granted. |
| The stored consent value is a legacy plain flag | The cookie is deleted with a zero lifetime and the answer is refused, so the visitor is asked again (WS-473). |

The visitor row itself is created by the page pipeline and is essential to the site's own operation, so it
is not gated by the optional category; what the optional category gates is the third-party measurement and
the third-party embedded content of WS-475 to WS-478.

The all-consents-granted flag participates in the page cache key (WS-070), so a page rendered with
neutralised blocks and the same page rendered with its blocks restored are two separate cache entries.
