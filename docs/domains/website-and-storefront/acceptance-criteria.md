# Acceptance criteria

Numbered scenarios a rebuild must pass, written as Given, When and Then with concrete records, concrete
inputs and exact results. Messages, notification subjects and shipped record names quoted below are
reproduced verbatim, including any abbreviation the shipped text contains. The identifiers `WS-AC-nnn` are
stable; the rules they exercise are cited where it helps.

| Range | Subject |
|---|---|
| WS-AC-001 – WS-AC-160 | Sites, pages, menus, addresses, content, forms, visitors, search and themes |
| WS-AC-161 – WS-AC-200 | Forum, blog, profiles and the public directory |
| WS-AC-201 – WS-AC-523 | The storefront |

---

# Part 1: Sites, content and visitors

## 1. Site resolution

**WS-AC-001.** Given site 1 with the domain `https://shop.example.com` and site 2 with the domain
`https://example.com:8069`, when a request arrives on the host `shop.example.com`, then site 1 is resolved.

**WS-AC-002.** Given the same two sites, when a request arrives on the host `SHOP.EXAMPLE.COM`, then site 1
is resolved, because the comparison is case-insensitive.

**WS-AC-003.** Given the same two sites and a third site with no domain and the highest ordering value, when
a request arrives on the host `www.shop.example.com`, then site 3 is resolved, because a containing match is
not an exact match (WS-035).

**WS-AC-004.** Given the same sites, when a request arrives on the host `example.com` with no port, then site
2 is resolved through the port fallback (WS-036).

**WS-AC-005.** Given the same sites, when a request arrives on the host `example.com:8070`, then site 2 is
resolved, because both sides lose their port in the fallback pass.

**WS-AC-006.** Given the same sites, when a back-office request arrives on an unknown host without asking for
a fallback, then no site is resolved (WS-038).

**WS-AC-007.** Given a session that forces site 2 and site 2 is then deleted, when the next request is
served, then the forced identifier is removed from the session and resolution continues with the host match
(WS-031).

**WS-AC-008.** Given a user who is a multi-site user but not a Restricted Editor, when they call the
site-forcing endpoint, then they are redirected to the requested path and no identifier is written into the
session (WS-039).

**WS-AC-009.** Given a target site whose domain differs from the current request host, when the site-forcing
endpoint is called, then the visitor is first redirected to the same endpoint on the target domain, carrying
the path and the marker saying the domain hop already happened (WS-040).

**WS-AC-010.** Given a site whose domain is written without the scheme, when it is saved, then the stored
value is prefixed with `https://` and every trailing slash is removed (WS-004).

**WS-AC-011.** Given a domain that cannot be parsed as a web address, when the site is saved, then the save
fails with `The provided website domain is not a valid URL.`

**WS-AC-012.** Given a domain whose path contains `/../`, when the site is saved, then the save fails with
`The domain path cannot contain relative path segments like '/./' or '/../'.`

**WS-AC-013.** Given a home page address `about`, when it is saved, then the save fails with
`The homepage URL should be relative and start with '/'.`

**WS-AC-014.** Given two sites carrying the same domain, when the second is saved, then it fails with
`Website Domain should be unique.`

**WS-AC-015.** Given the default site, when its deletion is attempted, then it fails with
`You cannot delete default website %s. Try to change its settings instead` with the site name substituted.

**WS-AC-016.** Given a site of company A, when company A is archived, then the archiving fails with the
two-line message of WS-014 naming the company and the site.

**WS-AC-017.** Given a database with one site, when a second site is created by a user who is not in the
multi-site group, then that group is implied into the portal, internal user and public groups (WS-013).

**WS-AC-018.** Given a site with theme attachments, customised asset attachments, compiled bundle
attachments and one business document, when the site is deleted, then the first three kinds are deleted and
the business document is kept (WS-012).

## 2. Pages

**WS-AC-019.** Given a designer creating a page named `Our Team`, when the New Page operation runs, then the
page address is `/our-team`, a template is copied under a unique key, the page carries the current site and
tracking is enabled.

**WS-AC-020.** Given a site that already has `/contact` and `/contact-1`, when a third page named `Contact`
is created, then its address is `/contact-2` (WS-051).

**WS-AC-021.** Given a page at `/about` with a menu entry pointing at it and the site home page address set
to `/about`, when the page address is changed to `/about-us`, then the menu entry and the site home page
address both become `/about-us` (WS-052, WS-053).

**WS-AC-022.** Given a page whose name changes from `About` to `Company`, when the write is saved, then the
template key is regenerated from the slug of the new name with a uniqueness suffix (WS-054).

**WS-AC-023.** Given a page whose visibility is restricted to a group, when the visibility is written to
signed-in, then the authorised group list is emptied (WS-055).

**WS-AC-024.** Given a page, when its address, its visibility or its authorised groups are written, then the
template cache is cleared (WS-056).

**WS-AC-025.** Given a published page whose publishing date is tomorrow, when an anonymous visitor requests
it, then the answer is "not found"; when a member of the Editor and Designer group requests it, then the page
is rendered (WS-057, WS-068).

**WS-AC-026.** Given a page at `/team`, when the address `/team/` is requested and no page answers on it,
then the answer is a permanent redirect to `/team`, with the language prefix restored and the query string
preserved (WS-065).

**WS-AC-027.** Given a page whose stored address is `/Team`, when `/team` is requested, then the
case-insensitive fallback finds it and the visitor is redirected to `/Team` (WS-064).

**WS-AC-028.** Given a shared page at `/about` and a site-specific page at `/about` on site 2, when site 2
serves `/about`, then the site-specific page is served; when site 1 serves it, then the shared page is served
(WS-062).

**WS-AC-029.** Given a shared page forked for site 2 and the fork given the address `/about-us`, when site 2
serves `/about`, then the answer is "not found", because the diverged fork hides the shared page (WS-063).

**WS-AC-030.** Given a page whose template is used by no other page and has no inheriting children, when the
page is deleted, then the template is deleted with it (WS-059).

**WS-AC-031.** Given a page cloned with the new name `Team copy` on the same site, when the clone runs, then
a new template is created, the page key becomes the new template key, the address is `/team-copy` and the
menu entry targeting the original is duplicated (WS-060, WS-061).

**WS-AC-032.** Given the same page cloned onto another site, when the clone runs, then no menu entry is
duplicated.

**WS-AC-033.** Given a page whose properties dialogue changes the address from `/a` to `/b` with a permanent
redirect requested, when the dialogue is saved, then a Website Rewrite is created with the action type
`301`, the source `/a`, the target `/b` and the current site.

**WS-AC-034.** Given a page whose properties dialogue switches the in-menu flag on, when it is saved, then a
Website Menu entry is created with the page name, the page address, the site's root menu as parent and the
page; switching it off deletes every entry of that site that targets the page or matches the address.

**WS-AC-035.** Given a user who is not in the Editor and Designer group and has no write access on a page,
when they attempt to publish it, then the write fails with
`You do not have the rights to publish/unpublish` (WS-067).

**WS-AC-036.** Given a member of the Editor and Designer group with no write access on a page, when they
publish it, then the publication succeeds, because the group always carries the right (WS-066).

## 3. The page response cache

**WS-AC-037.** Given a public visitor requesting `/about` in French on site 2 with the diagnostic flag off
and all consents granted, when the page is served, then a cache entry is stored under the key made of the
site, the language, the path, the diagnostic flag and the consent flag (WS-070).

**WS-AC-038.** Given that entry and a second visitor with the same profile, when the page is served, then the
stored response is returned after its request forgery token has been replaced by the token of the current
session (WS-072).

**WS-AC-039.** Given a third visitor who refused optional cookies, when the page is served, then a separate
entry is stored, because the all-consents-granted flag differs (WS-474).

**WS-AC-040.** Given a cache entry stored 3601 seconds ago, when the page is requested, then the response is
produced again and the entry is refreshed (WS-071).

**WS-AC-041.** Given a request carrying a query parameter, or a signed-in user, or a page with authorised
groups, when the page is served, then no cache entry is used and none is stored (WS-069).

**WS-AC-042.** Given a page whose rendered markup contains a cart marker, when it is served, then it is not
inserted into the cache (WS-362).

**WS-AC-043.** Given a cached page served to a shopper whose session records three items in the cart, when
the page is rendered, then the header counter shows 3; with an empty cart the counter element is hidden.

## 4. Menus

**WS-AC-044.** Given a menu entry with two ancestors, when a third level is attached, then the save fails
with `Menus cannot have more than two levels of hierarchy.`

**WS-AC-045.** Given a mega menu entry, when a child is attached to it, then the save fails with
`A mega menu cannot have a parent or child menu.`

**WS-AC-046.** Given an entry that has children, when it is attached under an entry that itself has a parent,
then the save fails with `Menus with child menus cannot be added as a submenu.`

**WS-AC-047.** Given the shipped root menu, when its deletion is attempted, then it fails with
`You cannot delete this website menu as this serves as the default parent menu for new websites (e.g., /shop, /event, ...).`

**WS-AC-048.** Given three sites and a menu entry created with no site and no site in context, when the
creation runs, then three entries are created, one per site, each under that site's root menu, and the
operation returns the last one (WS-094).

**WS-AC-049.** Given an entry created with a site in the execution context and no site in the values, when it
is created, then the context site is written into the values (WS-095).

**WS-AC-050.** Given a direct child of the shipped root menu with the address `/shop` and per-site copies at
the same address, when the shared entry is deleted, then every per-site copy is deleted as well (WS-097).

**WS-AC-051.** Given an entry restricted to one group, when the groups are written, then the Editor and
Designer group is added to the group list (WS-098).

**WS-AC-052.** Given an entry that targets an unpublished page, when the navigation is rendered for an
anonymous visitor, then the entry is hidden; when the page's visibility is password-protected, the entry
stays visible (WS-100).

**WS-AC-053.** Given a menu editor payload carrying a temporary identifier for a new entry and a reference to
it as a parent, when the payload is saved, then the entry is created and both references are replaced by the
new numeric identifier.

**WS-AC-054.** Given a menu entry whose address becomes `#section-2` in the editor, when the payload is
saved, then the address is prefixed with the path of the page the designer came from, and no page link is
kept (WS-103).

**WS-AC-055.** Given a menu entry bound to a page whose address is changed in the editor to an address on
which an endpoint answers, when the payload is saved, then the page link is cleared and the page is **not**
renamed (WS-104).

**WS-AC-056.** Given a site with a menu entry whose address contains a numeric record segment, when the menu
cache flag is computed, then it reports that the menu cache is disabled for that site (WS-106).

## 5. Rewrites and the home page

**WS-AC-057.** Given a rewrite rule of type `301` from `/old` to `/new`, when `/old` is requested and no page
or endpoint answers on it, then the answer is a permanent redirect to `/new` with the original query string
re-appended (WS-120, WS-123).

**WS-AC-058.** Given a rewrite rule of type `308` from `/a/<record>` to `/b/<record>`, when the routing table
is generated, then the endpoint is registered at both paths and the source answers a permanent redirect that
preserves the slug values.

**WS-AC-059.** Given a rewrite rule of type `404` on an endpoint path, when that path is requested on that
site, then the answer is "not found", and the endpoint still answers on every other site.

**WS-AC-060.** Given a `308` rule whose target does not start with a slash, when it is saved, then it fails
with `"URL to" must start with a leading slash.`

**WS-AC-061.** Given a `308` rule whose source carries a parameter placeholder absent from the target, when
it is saved, then it fails with `"URL to" must contain parameter %s used in "URL from".`

**WS-AC-062.** Given a `308` rule whose target is `/`, when it is saved, then it fails with the message of
WS-116.

**WS-AC-063.** Given a rule whose source equals its target once the fragment is removed, when it is saved,
then it fails with `base URL of 'URL to' should not be same as 'URL from'.`

**WS-AC-064.** Given the creation of a `308` rule, when it is saved, then the routing cache is cleared on
every worker; given the creation of a `301` rule, then it is not (WS-119).

**WS-AC-065.** Given a site whose home page address is `/welcome`, when `/` is requested, then the request is
internally rerouted to `/welcome`.

**WS-AC-066.** Given a site whose home page address is empty and whose root menu's first reachable child is
`/shop`, when `/` is requested and no page answers on it, then the visitor is redirected to `/shop`.

**WS-AC-067.** Given a new site, when it is created, then the shipped home page template is forked for it
with one empty editable region, a page is created at `/` and published, the default main menu tree is copied
with the root named after the site, and the copied entry whose address is `/` is bound to the home page
(§11 of [workflows.md](workflows.md)).

## 6. Languages

**WS-AC-068.** Given a site whose languages are, in name order, `English (US)`, `English (UK)`, `Español`,
`Español (Latinoamérica)` and `Français`, when the alternate-language codes are computed, then they are
`en`, `en-gb`, `es-es`, `es` and `fr` (WS-133).

**WS-AC-069.** Given a site whose default language is French and a visitor whose browser prefers Dutch, which
the site does not offer, when `/about` is requested, then the page is served in French without a redirect.

**WS-AC-070.** Given the same site offering Dutch as well, when `/about` is requested by that visitor, then
the answer is a redirect to `/nl/about` and the front-end language cookie is set to Dutch (WS-130).

**WS-AC-071.** Given the same site, when `/fr/about` is requested, then the answer is a redirect to `/about`,
because the default language carries no prefix (WS-134).

**WS-AC-072.** Given the same site and a request from a user agent identified as a crawler with no prefix,
then the page is served in the default language and no redirect is issued (WS-135).

**WS-AC-073.** Given a submission request with no language prefix, then no language redirection happens
(WS-136).

**WS-AC-074.** Given a site with exactly one language, when any page is requested, then no prefix is ever
inserted (WS-141).

**WS-AC-075.** Given a blog post reachable at `/blog/news-2/my-post-7`, when `/blog/2/7` is requested, then
the answer is a permanent redirect to the readable address (WS-139).

**WS-AC-076.** Given a language listed in the languages of a site, when it is deactivated, then the
deactivation fails with `Cannot deactivate a language that is currently used on a website.`

**WS-AC-077.** Given a multi-language site and a canonical address, when the page head is rendered, then one
alternate link per language is emitted plus a default-language link under the code `x-default`; given a
single-language site, then no alternate link is emitted (WS-142).

**WS-AC-078.** Given an address containing characters that require escaping, when the canonical test runs,
then the address is not canonical and no alternate link is emitted (WS-143).

## 7. Editing, copy-on-write and themes

**WS-AC-079.** Given a shared template used by site 1 and site 2, when a designer of site 2 saves an edited
region, then a copy of the template is created with the same key and site 2, the copy receives the edit, the
pages of the shared template are duplicated onto it, and site 1 still reads the shared template (WS-160).

**WS-AC-080.** Given that fork and a second region of the same template saved in the same batch, when the
second save runs, then it is applied to the existing fork and no second copy is created (WS-167).

**WS-AC-081.** Given a shared template with one inheriting child, when the parent is forked for site 2, then
the child is copied onto the new parent for site 2 and the shared child is untouched.

**WS-AC-082.** Given a shared template and a site in context, when the template is deleted, then the template
is first forked for every other site and only the current site loses it (WS-163).

**WS-AC-083.** Given a template written in a site context with no key, when the write runs, then a key is
generated before the copy-on-write logic (WS-161).

**WS-AC-084.** Given a template creation in a site context with an explicitly empty site, then the creation
is refused; with a different site, then it is refused as well (WS-162).

**WS-AC-085.** Given a shared template and an inactive site-specific template with the same key, when the
inheritance is resolved for that site, then the inactive specific template wins and the result is then
filtered to active templates, so the block disappears from that site only (WS-164).

**WS-AC-086.** Given a template translation written while a site is in context, then no fork is created
(WS-170).

**WS-AC-087.** Given a soft reset on a template, then its architecture returns to the previously stored one;
given a hard reset, then it is re-read from the shipped file; both run with copy-on-write disabled (WS-174).

**WS-AC-088.** Given a theme applied to site 2 and a second theme chosen, when the choice is applied, then
the first theme's stream is unloaded in reverse installation order, its copies are deleted, the default style
configuration is reset and the second theme's records are copied for site 2 only (WS-506, WS-507).

**WS-AC-089.** Given a theme load, when the records are converted, then the order is templates, assets,
pages, menus, attachments, and a record whose parent has no copy yet is postponed to a later pass (WS-500,
WS-501).

**WS-AC-090.** Given a theme record removed from the package, when the theme is loaded again, then the orphan
copies of the key-bearing entities are deleted while pages and menus are kept (WS-503).

**WS-AC-091.** Given a theme that enables a header template, when the post-copy hook runs, then every other
header template is disabled first (WS-176).

**WS-AC-092.** Given a toggle that requests the state a shared template already has, and no site-specific
copy exists, then nothing is written and no fork is created (WS-177).

**WS-AC-093.** Given a style variable file with a hook comment, when a palette pair is written, then the pair
is inserted at the hook, a customised attachment is created at the customisation address and a replacing
asset is created; resetting deletes both (§4.2 of [content-management.md](content-management.md)).

**WS-AC-094.** Given a font file whose bytes do not match its extension, when it is uploaded, then it is
refused with `File '%s' is not recognized as a font`; given an archive entry larger than 10485760 bytes, then
it is refused with `File '%s' exceeds maximum allowed file size` (WS-196).

**WS-AC-095.** Given a picture in the portable network graphics format converted to the web picture format,
when it is saved, then the name's suffix is replaced by `.webp` and a derived attachment records the source
attachment as its original (WS-193, WS-195).

**WS-AC-096.** Given an attachment whose local address appears in a template architecture, when its removal is
attempted, then it is not deleted and the names of the blocking templates are returned (WS-187).

**WS-AC-097.** Given a file of an unsupported kind, when it is uploaded, then it is refused with
`Uploaded image's format is not supported. Try with: ` followed by the list of extensions (WS-188).

## 8. The configurator

**WS-AC-098.** Given a caller who is not in the Editor and Designer group, when the configurator endpoint is
requested, then the answer is "not found"; given a site whose configurator is already done, then the answer
is a redirect to the site (WS-510).

**WS-AC-099.** Given a configurator feature carrying both a page template and a package, when it is saved,
then it fails with
`One and only one of the two fields 'page_view_id' and 'module_id' should be set`

**WS-AC-100.** Given six selected features that request a menu entry, of which two ask for the Company
grouping, when the configurator applies, then a menu entry named `Company` is created under the root menu at
the ordering value 40 (WS-512).

**WS-AC-101.** Given 60 placeholder terms of which 50 are translated in the site's default language, when the
coverage is computed, then it is 0.8333 and the text generation runs; with 48 translated terms the coverage
is 0.8000 and the shipped texts are kept (WS-513).

**WS-AC-102.** Given a generated text containing the marker `XXXX`, when it is applied, then every occurrence
is replaced by the site name (WS-514).

**WS-AC-103.** Given the text generation service unreachable, when the configurator applies, then the shipped
texts are kept and the run completes (WS-519).

**WS-AC-104.** Given a page filled with content blocks by the configurator, when the apply step ends, then the
page template is duplicated under the configurator template key and bound to the site (WS-515).

**WS-AC-105.** Given the configurator skipped, then the completion flag is set and the shipped default theme
is installed (WS-518).

## 9. Public forms

**WS-AC-106.** Given an entity that is not flagged as usable in forms, when a submission targets it, then the
answer carries the error `The form's specified model does not exist` (WS-200).

**WS-AC-107.** Given a form targeting the Contact entity with one input named after a field that is not in
the allow list, when it is submitted, then that value is kept as free text and written on the designated
field under the heading `Other Information:` (WS-208, WS-211).

**WS-AC-108.** Given a required writable field left empty and another field whose conversion failed, when the
submission is handled, then the answer lists both fields together (WS-209).

**WS-AC-109.** Given a required field left empty but no conversion failure, when the submission is handled,
then no error answer is produced, because the answer is produced only when at least one conversion actually
failed (WS-209).

**WS-AC-110.** Given an authenticated session and an invalid request forgery token, when a form is submitted,
then the request fails with `Session expired (invalid CSRF token)`; given an anonymous session, then the
token is not checked (WS-205).

**WS-AC-111.** Given a submission that violates a database constraint that cannot be attributed to a field,
then the answer is `false` (WS-216).

**WS-AC-112.** Given an uploaded file whose input name is a writable binary field, then the attachment is
linked to that field; given any other file, then it becomes an orphan attached to a comment message whose
body is the paragraph `Attached files: ` (WS-215).

**WS-AC-113.** Given a form targeting the outgoing mail entity, when it is submitted, then the reply address
becomes the submitted sender address, the sender address becomes the company name with the form-submission
suffix and the company address, and the message is sent immediately (WS-213).

**WS-AC-114.** Given a form targeting the outgoing mail entity whose signature does not match the keyed
digest of the recipient address, then the submission is refused with `invalid website_form_signature`
(WS-214).

**WS-AC-115.** Given the metadata parameter set, when a form is submitted, then the network address, the user
agent, the accepted languages and the referring address are appended under the heading `Metadata` (WS-210).

**WS-AC-116.** Given a field used by an input of a published form, when its deletion is attempted, then it
fails with the three-line message of WS-204 naming the field, the entity and the template.

**WS-AC-117.** Given the score-based verification enabled and an answer whose score is below the minimum, when
a form is submitted, then it is refused with `Suspicious activity detected by google reCAPTCHA.`

**WS-AC-118.** Given the score-based verification enabled with no private key stored, when a form is
submitted, then the check passes (§9.1 of [content-management.md](content-management.md)).

**WS-AC-119.** Given the challenge-widget verification enabled and a failed challenge, then the submission is
refused with `The CloudFlare human validation failed.`

**WS-AC-120.** Given both verification services installed, when a submission passes the first and fails the
second, then it is refused.

**WS-AC-121.** Given a submission that carries a value in the decoy field, then the record is still created
and the value appears in the free-text block; a rebuild adopting WS-900 refuses it instead.

## 10. Visitors and consent

**WS-AC-122.** Given an anonymous visitor requesting a tracked page at 09:00, when the response is produced,
then one visitor row is inserted with the 32-character token, the visit count 1, the site, the language, the
country and the time zone, and one track row is inserted, both by one statement (WS-571).

**WS-AC-123.** Given that visitor browsing again at 09:05, 09:40 and 14:00, then the visit count stays 1, the
last connection moment is refreshed each time and three further track rows exist (WS-235).

**WS-AC-124.** Given that visitor returning at 23:30, then the visit count becomes 2, because the previous
moment is more than eight hours earlier.

**WS-AC-125.** Given that visitor at 23:33, then the connected flag is true; at 23:36 it is false (WS-236).

**WS-AC-126.** Given a response whose status is not 200, or a request carrying the tracking-disabled header
with the value `1`, or a crawler user agent, then no visitor and no track are written (WS-234).

**WS-AC-127.** Given a template whose tracking flag is false, when a page rendered from it is served, then
nothing is tracked.

**WS-AC-128.** Given an anonymous visitor who signs in as a contact that has no visitor yet, then the
anonymous visitor's token becomes the contact identifier and its last connection moment is refreshed
(WS-240).

**WS-AC-129.** Given an anonymous visitor who signs in as a contact that already has a visitor, then the
anonymous visitor's tracks are repointed to the existing one and the anonymous visitor is deleted.

**WS-AC-130.** Given a merge whose target has no contact, then the merge fails with the message of WS-241.

**WS-AC-131.** Given an anonymous visitor whose last connection moment is 61 days old, when the daily cleanup
runs, then the visitor and its tracks are deleted; given a visitor linked to a contact, then it is kept
(WS-242).

**WS-AC-132.** Given 1500 visitors matching the cleanup condition, when the job runs, then 1000 are deleted
and the job reports 500 still matching (WS-243).

**WS-AC-133.** Given a visitor with no contact, when the message composer is opened on it, then the action
fails with `There are no contact and/or no email linked to this visitor.`

**WS-AC-134.** Given a product view recorded twice within thirty minutes, then only one track row carries
that product; after thirty-one minutes a second row is created (WS-237).

**WS-AC-135.** Given a site whose consent bar is enabled and a visitor who has not answered, then the optional
category is refused and the measurement script is loaded with every category denied (WS-472, WS-479).

**WS-AC-136.** Given the same visitor pressing `Only essentials`, then the consent document maps the optional
category to false, is stored for 999 days, and third-party frames and scripts are neutralised.

**WS-AC-137.** Given the same visitor pressing `I agree`, then the optional category becomes true and the
one-shot handler grants the measurement categories.

**WS-AC-138.** Given a consent cookie containing a legacy plain flag, then the cookie is deleted with a zero
lifetime and the answer is refused (WS-473).

**WS-AC-139.** Given a site whose consent bar is disabled, then optional cookies are allowed and nothing is
neutralised (WS-471).

**WS-AC-140.** Given an embedded frame whose source host ends with a dot followed by a blocked domain, and a
visitor who refused optional cookies, then the element receives `data-need-cookies-approval` set to `true`
and its source is moved to `data-nocookie-src` (WS-476, WS-478).

**WS-AC-141.** Given a Restricted Editor viewing the same page, then nothing is neutralised (WS-475).

**WS-AC-142.** Given the consent bar switched on for a site with no page at `/cookie-policy`, then the
shipped cookie policy template is forked, a page is created at that address, published and not indexed;
switching the bar off deletes the page (WS-017, WS-018).

## 11. Site index, crawler instructions and search

**WS-AC-143.** Given a site with 46000 enumerable addresses, when the index is generated, then two index
documents are produced and a document listing them is written at the base address (WS-480, WS-484).

**WS-AC-144.** Given an index generated eleven hours ago, when it is requested, then the cached document is
returned; after thirteen hours it is regenerated (WS-481).

**WS-AC-145.** Given a site whose enumeration produced no location, then the request answers "not found"
(WS-483).

**WS-AC-146.** Given an unpublished page, an embargoed page, a page whose visibility is signed-in and a page
at `/`, then none of them is enumerated (WS-486).

**WS-AC-147.** Given a page whose template priority is 40, then the contributed priority is `1`; given the
priority 24, then it is `0.8`; given the priority 16, then no priority is contributed
([calculations.md](calculations.md) §7.3).

**WS-AC-148.** Given a site with a domain and a request arriving on another host, then the crawler exclusion
response contains `Disallow: /` and points at the index of the site's own domain (WS-493).

**WS-AC-149.** Given a request on the canonical host, then the response points at the index of the request
root and appends the site's custom text under the three banner lines (WS-494).

**WS-AC-150.** Given a search for `chiar` and a catalogue containing `chair` and `chart`, then `chair` is
chosen, the search is re-run with it, and the page reports both terms
([calculations.md](calculations.md) §4.4).

**WS-AC-151.** Given a search term of three characters, or containing a space, or made of 80 percent digits,
then approximate matching is skipped (WS-271).

**WS-AC-152.** Given a candidate word that contains the term as a substring, then the original term is used
and no substitution is reported (WS-272).

**WS-AC-153.** Given a chosen word that equals the term ignoring case, then no substitution is reported
(WS-273).

**WS-AC-154.** Given the public user searching pages, then unpublished, unindexed, password-protected and
signed-in pages are excluded and the results are post-filtered against the read rules (WS-274, WS-275).

**WS-AC-155.** Given a page forked for the current site, then it appears once in the search results, because
only the most specific page per address is kept (WS-276).

**WS-AC-156.** Given the autocompletion answer, then a text value longer than 999 characters is shortened
with `...` and the term occurrences are wrapped in the highlight template (WS-277, WS-278).

**WS-AC-157.** Given the `all` scope, then the merged results are sorted by name and cut to the limit.

## 12. Publication, access and server actions

**WS-AC-158.** Given a record bound to site 2 and published, when site 1 renders a listing, then the record
reads as unpublished there (WS-282).

**WS-AC-159.** Given a search for published records with site 2 in context, then the condition expands to
"the stored flag is true and the site is empty or site 2" (WS-283).

**WS-AC-160.** Given a published code action with the public path `hello`, when the public action address is
requested, then the action runs with the request object in its evaluation context and a response placed in
that context wins over a returned action (WS-623).

---

# Part 2: Forum, blog, profiles and the public directory

## 13. Forum

**WS-AC-161.** Given a forum whose asking threshold is 3 and a participant whose score is 2, when they submit
a question, then it is refused with `%d karma required to create a new question.` with 3 substituted.

**WS-AC-162.** Given a participant whose score is 50 and a forum whose validation threshold is 100, when they
ask a question, then the post is created in the pending state, no reputation is awarded and the shipped
validation message is posted as an internal note to the moderators and the tag followers (WS-604).

**WS-AC-163.** Given that pending question validated by a moderator, then the state becomes active, the
active flag is set, the moderator is recorded, the asking award of 2 points is granted and the shipped
new-question message is posted.

**WS-AC-164.** Given that pending question refused by a moderator, then only the moderator is recorded and
the post stays pending and invisible.

**WS-AC-165.** Given a question whose author has 100 points, when a participant up-votes it, then the author
reaches 105; when the same participant up-votes again, the vote is withdrawn and the author returns to 100
([calculations.md](calculations.md) §21.2).

**WS-AC-166.** Given an answer whose author has 100 points and a participant who has down-voted it, when that
participant up-votes it, then the author moves by `+10 − (−2) = +12` and reaches 110.

**WS-AC-167.** Given a participant attempting to vote on their own post, then it fails with
`It is not allowed to vote for its own post.`

**WS-AC-168.** Given a participant attempting to change another participant's vote, then it fails with
`It is not allowed to modify someone else's vote.`

**WS-AC-169.** Given a second vote row for the same participant and post, then it fails with
`Vote already exists!`

**WS-AC-170.** Given a participant whose score is 4 and an up-vote threshold of 5, then the up-vote is
refused with `%d karma required to upvote.`; a participant whose present vote is a down-vote may always
withdraw it.

**WS-AC-171.** Given Anne asking a question and Ben answering, when Anne accepts Ben's answer, then Ben gains
15 points, Anne gains 2, every other answer of the question loses its acceptance flag and the question's
answered flag becomes true.

**WS-AC-172.** Given Anne answering her own question and accepting her own answer, then no reputation moves
at all.

**WS-AC-173.** Given the accepted answer deleted, then the acceptance award is withdrawn from its author and
the acceptance bonus from the participant who accepted it (WS-610).

**WS-AC-174.** Given a closed question, when an answer is posted on it, then it fails with
`Posting answer on a [Deleted] or [Closed] question is not possible.`

**WS-AC-175.** Given a forum in questions mode and a participant who already answered, when they post a
second answer, then it is refused (WS-607).

**WS-AC-176.** Given an author whose score is below the no-follow threshold, when their content contains a
link, then the stored content carries the no-follow marker and the address is unchanged (WS-601).

**WS-AC-177.** Given an author whose score is below the editor threshold, when their content contains a
picture, then the write is refused with `%d karma required to post an image or link.`

**WS-AC-178.** Given a participant with 250 points whose first question in a forum is closed with the reason
`Spam or advertising`, then they lose 1000 points and end at −750; reopening gives the 1000 points back
(WS-609).

**WS-AC-179.** Given the same participant's second question closed with the same reason, then they lose 100
points.

**WS-AC-180.** Given a post archived, then every answer of the post is archived with it (WS-608).

**WS-AC-181.** Given a forum archived, then every post of the forum is archived, already archived ones
included.

**WS-AC-182.** Given a question with 5 votes created 3 days ago on a forum with the default parameters, then
its relevance is 0.1673050; created 30 days ago it is 0.0055733
([calculations.md](calculations.md) §22).

**WS-AC-183.** Given a question tagged `install`, `upgrade` and `database` and candidates tagged
identically, tagged `install` and `database`, and tagged `install` and `printing`, then the related questions
are those three in that order, with the similarities 1.0000, 0.6667 and 0.2500
([calculations.md](calculations.md) §23).

**WS-AC-184.** Given a participant whose score is below the tag-creation threshold submitting a question with
a new tag name, then the question is created and the new tag is silently dropped (WS-585).

**WS-AC-185.** Given two tags with the same name in one forum, then the second fails with
`Tag name already exists!`; the same name in another forum is accepted (WS-606).

**WS-AC-186.** Given a forum whose privacy is signed in, when an anonymous visitor requests it, then access is
refused; given the privacy "some users", then only members of the authorised group reach it (WS-580).

**WS-AC-187.** Given the privacy changed to public, then the authorised group is cleared.

**WS-AC-188.** Given a question by an author whose reputation is zero and who is not the current user, then
the question is not viewable, does not appear in the listing and is absent from the site index (WS-605).

## 14. Blog

**WS-AC-189.** Given a draft post published with no explicit publishing date, then the publishing date
becomes the current moment and a message with the publication subtype is posted on the parent blog (WS-524,
WS-526).

**WS-AC-190.** Given a post whose publishing date is tomorrow, when it is published, then the date is kept,
the message is still posted and a visitor who is not a designer does not see the post (WS-535).

**WS-AC-191.** Given a published post unpublished, then the publishing date is cleared when it was empty or
already past, and no message is posted.

**WS-AC-192.** Given a post archived, then the publication flag is forced to false (WS-523).

**WS-AC-193.** Given a blog archived, then every post of the blog is archived; unarchiving the blog
unarchives them (WS-522).

**WS-AC-194.** Given a post whose content renders as 640 characters of plain text and that has no manual
teaser, then the teaser is the first 200 characters followed by `...`
([calculations.md](calculations.md) §6.1).

**WS-AC-195.** Given a post duplicated, then the copy's title is the original title followed by ` (copy)`
(WS-533).

**WS-AC-196.** Given a post reached through the wrong blog, then the answer is a permanent redirect to the
correct address; reached through the legacy address form, then it is redirected as well (WS-534).

**WS-AC-197.** Given a post opened twice in one session, then the view counter is increased once (WS-537).

**WS-AC-198.** Given a blog index requested on a site with exactly one blog, then the answer is a temporary
redirect to that blog (WS-538); given a read request carrying two tags, then it is redirected to the first
(WS-539).

**WS-AC-199.** Given a feed requested with no limit, then 15 entries are returned; requested with 80, then 50
are returned (WS-542).

**WS-AC-200.** Given an account whose reputation score is zero consuming a valid address-validation token,
then the score becomes 3; given a score of 40, then it is unchanged (WS-560).

---

# Part 3: The storefront

## 15. Access to the shop

**WS-AC-201.** Given a site whose shop access is "All users", when an anonymous visitor requests the shop
listing, then the listing page is returned with a success status, even when no storefront category exists.

**WS-AC-202.** Given a site whose shop access is "Logged in users", when an anonymous visitor requests the
shop listing, then the answer is a redirect to the sign-in page carrying the shop path as the return path.

**WS-AC-203.** Given the same site, when a signed-in portal user requests the listing, then the listing page
is returned with a success status.

**WS-AC-204.** Given the same site, when the navigation menu is rendered for an anonymous visitor, then no
menu entry whose address starts with the shop path is present.

**WS-AC-205.** Given the same site, when the site index is generated without a query string, then it contains
no shop, category or product entry.

**WS-AC-206.** Given a site whose shop access is "All users" and a published product, when the site index is
generated, then it contains the shop path, one entry per storefront category and one entry per published
product.

## 16. Catalogue listing, search and filters

**WS-AC-207.** Given three published products with the shop ordering values 10000, 10005 and 10010 and the
default sort "featured", when the listing is requested, then the products appear in ascending ordering-value
order.

**WS-AC-208.** Given a product published today and a product published last month, when the newest-arrivals
sort is chosen, then the product published today appears first.

**WS-AC-209.** Given products priced 10.00, 50.00 and 80.00 and the price filter active, when the listing is
requested with a minimum of 40 and a maximum of 100, then only the products priced 50.00 and 80.00 are
listed and the slider reports the available bounds 10.00 and 80.00.

**WS-AC-210.** Given the same catalogue, when the shopper navigates to a category whose prices run from
300.00 to 900.00 while the maximum of 100 is still in the address, then the maximum is replaced by 900.00 and
the whole category is listed instead of an empty page.

**WS-AC-211.** Given a product whose only variant carries the internal reference `E-COM12`, when the shopper
searches for `E-COM12`, then the product is found, because the aggregated variant reference field is
searched.

**WS-AC-212.** Given a catalogue containing `Chair`, when the shopper searches for `Chiar`, then the
corrected term `Chair` is searched and the page reports both terms.

**WS-AC-213.** Given the shop listing and the site-wide search box, when the same term is submitted to each,
then both search exactly the same product fields.

**WS-AC-214.** Given an attribute `Color` with the values `Red` and `Blue` and an attribute `Size` with the
value `M`, when the address carries the attribute filter entries for both attributes, then the listing
contains the products that offer `M` and offer `Red` or `Blue`.

**WS-AC-215.** Given an attribute whose storefront visibility is hidden, when the filter panel is built, then
that attribute is absent from it.

**WS-AC-216.** Given a category `Furniture` with the child `Chairs` and a product attached only to `Chairs`,
when the listing of `Furniture` is requested, then the product is listed, because the category condition
includes the descendants.

**WS-AC-217.** Given a category `Chairs` that contains no published product and an anonymous visitor, when
the category list is read, then `Chairs` is absent (WS-294).

**WS-AC-218.** Given a category `Furniture` whose only published products live in its child `Chairs`, when
the published-products flag is evaluated for `Furniture`, then it is true
([calculations.md](calculations.md) §25).

**WS-AC-219.** Given a category with no published product anywhere below it, then its flag is false, and a
search for categories without published products returns it.

**WS-AC-220.** Given a selected category with no accessible children, when the listing is rendered, then the
category strip shows the siblings of the selected category instead of an empty strip.

**WS-AC-221.** Given the tag filter page option active and a tag marked visible to customers attached to a
published product, when the listing is requested, then that tag is offered as a filter; a tag not marked
visible to customers is not offered.

**WS-AC-222.** Given products of the sizes two by two, one by one, one by one, one by one, one by one, two by
one and one by one, four columns and a page size of six, when the grid is packed, then the rows are
[two by two, one by one, one by one], [one by one, one by one] and [two by one, one by one].

**WS-AC-223.** Given an unpublished product and an internal user, when the listing is requested, then the
unpublished product appears after every published product, because the order starts with the publication flag
descending.

## 17. Product page and combination information

**WS-AC-224.** Given a template with a sales price of 100.00, an attribute value carrying an extra price of
5.00, one sales tax of 21 percent not included in the price, a site displaying tax-included prices and a
price list applying a 10 percent percentage discount, when the combination information is requested for that
value with quantity 1, then the price is 114.35, the reference price is 127.05, the discount flag is true,
the extra-price flag is true and the attribute value shows an extra of 6.05.

**WS-AC-225.** Given the same setup but a site displaying tax-excluded prices, then the price is 94.50, the
reference price is 105.00 and the attribute value shows an extra of 5.00.

**WS-AC-226.** Given the same setup but a fixed-price rule at 94.50, then the price is 114.35, the discount
flag is false and the extra-price flag is false, therefore the attribute value shows no extra.

**WS-AC-227.** Given a product priced 2222.00, a price list with a 10 percent discount in a currency whose
rate is 2, and a tax of 15 percent, when the combination information is requested with tax-excluded display,
then the price is 3999.60 and the reference price is 4444.00; with tax-included display, the price is
4599.54 and the reference price is 5110.60.

**WS-AC-228.** Given a product priced 2000.00 with a tax of 15 percent, a price list fixing the price at
500.00 and an attribute value carrying an extra price of 200.00, when the combination information is
requested with tax-included display, then the price and the reference price are both 575.00.

**WS-AC-229.** Given the same product and a fiscal position mapping the 15 percent tax to 0 percent for the
visitor's country, then the price and the reference price are both 500.00.

**WS-AC-230.** Given the same product whose tax of 15 percent is defined as included in the price and the
same fiscal position mapping it to 0 percent, then the price is 434.78.

**WS-AC-231.** Given the same product and a fiscal position mapping the 15 percent price-included tax to a 5
percent price-included tax, then the price is 456.52.

**WS-AC-232.** Given a template with two variants, when the structured product description is generated, then
it uses the product-group form and carries one nested description per variant; with a single variant it uses
the product form directly.

**WS-AC-233.** Given a site displaying tax-excluded prices, when the structured description of a variant
priced 100.00 with a tax of 21 percent is generated, then the published price is 100.00; with tax-included
display it is 121.00.

**WS-AC-234.** Given a site whose price list currency differs from the company currency at a rate of 2, when
the structured description is generated, then the published price is expressed in the display currency and
the published currency code is that one.

**WS-AC-235.** Given an attribute value that has no matching variant, when the product card previews are
computed, then that value is not previewed.

**WS-AC-236.** Given a template whose attribute offers only one value, when the product page is rendered,
then that attribute appears in the informative block below the configurator rather than as a choice control.

**WS-AC-237.** Given an attribute line with a single value marked as accepting a free-text custom value, when
the specification table is built, then that line is excluded.

**WS-AC-238.** Given a product with a base unit count of 2 and the reference unit `L`, a price of 60.00 and
the per-unit price feature enabled, when the product page is rendered, then it prints `30.00 / L`; with a
base unit count of 0 it prints nothing.

**WS-AC-239.** Given a base unit count of 0.0001, when it is stored and read back, then the exact value is
preserved, because the field keeps unlimited decimal places.

**WS-AC-240.** Given a product whose price becomes 0.00 on a site that forbids zero-price sales, then the
zero-price flag is true, the comparison price is forced to 0 and the add-to-cart action is not offered.

**WS-AC-241.** Given a product with a comparison price of 75.00, a price of 60.00 and no price list discount,
when the comparison-price feature is enabled, then the shop tile shows 60.00 struck through with 75.00; when
the feature is disabled, no strikethrough is shown.

**WS-AC-242.** Given a product priced 61.98 excluding a tax of 21 percent, a site displaying tax-included
prices and a price list applying 20 percent, when the catalogue price payload is computed, then the price is
60.00 and the reference price is 75.00.

**WS-AC-243.** Given a price list rule whose price depends on the quantity, when the shopper raises the
quantity on the product page past the rule threshold, then the combination information returned for the new
quantity carries the discounted price.

**WS-AC-244.** Given a price list rule computed from the product cost, when the cost is 5.00 and the rule
applies a 10 percent reduction, then the catalogue price payload reports 4.50; when the cost is 0.00 the
payload reports 0.00.

**WS-AC-245.** Given a cart created before a price list rule's validity period starts, when a line is priced,
then the rule applies as soon as the period is current, because a draft storefront line prices itself at the
present moment (WS-339).

## 18. Media, documents and extra fields

**WS-AC-246.** Given a template with a main picture and two extra media, when the ordered media list is read
for a variant with one variant-specific medium, then the order is the variant picture, the variant medium,
then the two template media.

**WS-AC-247.** Given a template whose own picture is empty and whose first variant has one, when the picture
holder is resolved, then the first variant is returned; when neither has one, the template is returned.

**WS-AC-248.** Given four media where the first is the main picture, when the third is moved to the first
position, then the third becomes the main picture, the former main picture takes the third position, the
picture payloads are exchanged, the caption of the former main entry is copied onto the media row and the
product name is unchanged.

**WS-AC-249.** Given media where the second entry is a video, when it is moved to the first position, then
the move is refused with `You can't use a video as the product's main image.`

**WS-AC-250.** Given media where a video sits in the second position and the third is moved to the first,
when the resequencing runs, then the video is not promoted to the first position by the subsequent shift.

**WS-AC-251.** Given a move that would not change the index, when it is requested, then nothing is written.

**WS-AC-252.** Given a user without the Restricted Editor group, when any media endpoint is called, then the
answer is "not found".

**WS-AC-253.** Given a video address from which no player markup can be derived, when the media row is saved,
then the save is refused with `Provided video URL for '%s' is not valid. Please enter a valid video URL.`

**WS-AC-254.** Given a video address and an empty picture, when the address is entered in the form, then the
video thumbnail is fetched and stored as the picture.

**WS-AC-255.** Given a template with dynamic attributes and no existing variant for the chosen combination,
when media are added with that combination, then the variant is created first and the media are attached to
it.

**WS-AC-256.** Given a product with variant media and a call to clear the media, then the variant media are
deleted and the template media are kept; when the variant has no media, the template media are deleted.

**WS-AC-257.** Given a document attached to a Product Template with the product-page flag on, when its
download address is requested, then the file is served as an attachment; when the flag is off, or the
document belongs to another product, then the answer is a redirect to the shop.

**WS-AC-258.** Given a document whose target is a variant, when the product-page flag is switched on, then
the write is refused with
`Documents shown on product page cannot be restricted to a specific variant`

**WS-AC-259.** Given a site declaring the Product Template field "Manufacturer reference" as a storefront
extra field and a product whose value is `AX-12`, when the product page is rendered, then the details block
contains that caption and that value; when the product has no value, the block is absent.

## 19. Ribbons

**WS-AC-260.** Given a ribbon assigned by hand to a template and another assigned by hand to its variant,
when the ribbon is resolved for that variant, then the variant ribbon wins.

**WS-AC-261.** Given no manual ribbon, a ribbon whose mode is the sale mode, and a catalogue payload whose
reference price is 120.00 and whose price is 99.00, then the sale ribbon is returned.

**WS-AC-262.** Given no manual ribbon, a ribbon whose mode is the new mode with a period of 30 days, and a
product published 12 days ago, then the new ribbon is returned.

**WS-AC-263.** Given both the sale ribbon at the ordering value 3 and the new ribbon at 4 applicable, then
the sale ribbon is returned, because the candidates are evaluated in ordering order.

**WS-AC-264.** Given no manual ribbon, no discount and a product published 60 days ago with a new period of
30, then no ribbon is returned.

**WS-AC-265.** Given an existing ribbon whose mode is the sale mode, when a second ribbon is saved with the
same mode, then it fails with `Only one ribbon with the assign %s is allowed.`

**WS-AC-266.** Given the storefront stock capability, a ribbon whose mode is the out-of-stock mode and a
tracked product with out-of-stock ordering disabled and no quantity free to use, then the out-of-stock
ribbon is returned.

## 20. Price lists on the storefront

**WS-AC-267.** Given price lists attached to the current site, generic and selectable, generic with a code,
and generic without either, when the available price lists are read without the visible-only flag, then the
first three are returned and the fourth is not.

**WS-AC-268.** Given the same set, when they are read with the visible-only flag, then only the selectable
ones, plus the one currently in force, are returned.

**WS-AC-269.** Given a price list with the code `BLACKFRIDAY`, when the shopper submits that code, then the
price list becomes both the session price list and the explicitly selected price list, and the cart prices
are recomputed.

**WS-AC-270.** Given an explicitly selected price list and an empty code submitted afterwards, then the
selection is cleared, the price list is resolved again and the cart prices are recomputed.

**WS-AC-271.** Given a visitor whose network country resolves to Belgium and a price list carrying the
country group Europe, then that price list is used even though the visitor has no address.

**WS-AC-272.** Given the same visitor and a signed-in contact whose assigned price list carries the country
group United States, then the assigned price list is not offered, because it is not available in the
visitor's country.

**WS-AC-273.** Given a signed-in contact whose assigned price list is generic and selectable, then the
assigned price list is used.

**WS-AC-274.** Given a country group removed from every price list, then the generic site price lists are
used for a visitor from that country.

**WS-AC-275.** Given a cached price list stored more than 3600 seconds ago, when a listing request is served,
then the cache is dropped and the price list is resolved again.

**WS-AC-276.** Given a signed-in session with a cached price list, when the shopper signs out and signs in as
another user, then the cached price list, the selected price list and the cached fiscal position are cleared
before the redirect.

**WS-AC-277.** Given a price list attached to company A and a site of company B, when the price list is
saved, then it fails with
`Only the company's websites are allowed.\nLeave the Company field empty or select a website from that company.`

**WS-AC-278.** Given a price list in use on a site, when it is archived, then it stops being available and the
memoised resolution no longer returns it.

**WS-AC-279.** Given a contact whose assigned price list is read in two different companies, then the
company-specific assignment is returned in each, and the storefront resolution respects it.

**WS-AC-280.** Given an inactive contact, when their assigned price list is read, then a price list is still
returned.

**WS-AC-281.** Given a multi-company database with price lists of another company, when an anonymous visitor
opens the shop, then the page is served without an access failure.

## 21. Cart

**WS-AC-282.** Given an empty session, when a published product is added with the quantity 2, then a Sales
Order is created with the site, the site's company, the public contact, the request price list and the site
sales team, one line is created with the quantity 2, and the session records the order and the cart quantity
2.

**WS-AC-283.** Given a product that has been deleted, when it is added, then the call fails with
`The given product does not exist therefore it cannot be added to cart.`

**WS-AC-284.** Given an unpublished product, when an anonymous visitor adds it, then the call fails with the
same message.

**WS-AC-285.** Given an archived product, when it is added, then the call fails with the same message.

**WS-AC-286.** Given a product already in the cart with the quantity 1, when the same product with the same
unit and no custom values is added with the quantity 2, then the existing line reaches 3 and no second line
is created.

**WS-AC-287.** Given a product added twice with different free-text custom values, then two separate lines
exist.

**WS-AC-288.** Given a product with a no-variant attribute offering two values, when it is added once with
each value, then two separate lines exist.

**WS-AC-289.** Given a site that forbids zero-price sales and a product priced 0.00, when it is added, then
the add fails with `The given product does not have a price therefore it cannot be added to cart.`

**WS-AC-290.** Given the same site, a product priced 0.00 and a no-variant attribute value carrying an extra
price of 10.00, when it is added with that value, then the add succeeds, because the line price is not zero.

**WS-AC-291.** Given a cart whose line price becomes 0.00 after a price list change, when the shopper opens
the checkout, then the line receives `This product is not available for purchase in your country.`, the order
receives
`Some products in your cart are not available for purchase in your country. Please remove them or contact us.`
and the shopper is redirected to the cart.

**WS-AC-292.** Given a cart with a delivery line priced 0.00 because of a free-shipping threshold, when the
shopper proceeds to payment, then the delivery line does not block the checkout.

**WS-AC-293.** Given a combo product with two choices, when it is added with one item of each, then one combo
line and two item lines are created, all with the same quantity, and the combo line's validity is checked
only after every item line exists.

**WS-AC-294.** Given a combo whose second item can only be supplied twice while three were requested, when
the combo is added with the quantity 3, then the combo line and both item lines end at 2 and the combo line
carries the shortage warning.

**WS-AC-295.** Given a combo whose item cannot be supplied at all, when the combo is added, then the combo
line and its already created item lines are deleted, the answer reports the quantity 0 and it carries the
item's warning.

**WS-AC-296.** Given a line with an optional product attached, when the parent line quantity is set to 0,
then both lines are deleted.

**WS-AC-297.** Given a cart update for a line that no longer exists, then the answer carries
`We weren't able to update your cart. Please refresh your page before trying again.` and nothing is written.

**WS-AC-298.** Given a product added twice in one request, when the cart notification is built, then it lists
only the lines whose added quantity is strictly positive, with the newly added quantity and the amount for
that quantity, not the whole line total.

**WS-AC-299.** Given a site displaying tax-included prices, when the cart notification is built for one unit
priced 100.00 with a tax of 21 percent, then the notification amount is 121.00; with tax-excluded display it
is 100.00.

**WS-AC-300.** Given a product archived while it sits in a draft storefront cart, when the archiving is
written, then the cart line is deleted.

**WS-AC-301.** Given a signed-in shopper with a draft cart on this site and an empty session, when they open
the shop, then their existing cart is adopted, its addresses are propagated again and its archived-product
lines are removed.

**WS-AC-302.** Given a session cart whose last transaction is pending, when the next storefront request is
served, then the session cart is released and a new cart will be created on the next add.

**WS-AC-303.** Given an anonymous cart, when the shopper signs in with a different contact, then the cart's
contact is rewritten and the price list, fiscal position, taxes and prices are recomputed.

**WS-AC-304.** Given a site of company A, when an order is created for that site with company B, then it is
refused with
`The company of the website you are trying to sell from (%(website_company)s) is different than the one you want to use (%(company)s)`

**WS-AC-305.** Given a cart containing a service product and a goods product, when the goods line is removed,
then the cart becomes services-only, the delivery line is deleted and the pickup location is cleared.

**WS-AC-306.** Given a cart with a delivery method whose rate later fails, when a line is updated, then the
delivery line is removed.

**WS-AC-307.** Given a cart with a delivery method whose rate changes, when a line is updated, then the
delivery line price is rewritten with the new rate.

**WS-AC-308.** Given a cart with three lines, when the clear endpoint is called, then every line is deleted.

## 22. Accessory suggestions and reordering

**WS-AC-309.** Given a product with two published accessory products, one of which is already in the cart,
when the cart page is rendered, then only the other accessory is suggested.

**WS-AC-310.** Given an accessory product that is unpublished, when the suggestions are built for an
anonymous visitor, then it is not suggested; for an internal user it is.

**WS-AC-311.** Given a confirmed order with two ordinary lines, one section line and one delivery line, when
the reorder endpoint is called, then only the two ordinary lines are added to the cart.

**WS-AC-312.** Given a confirmed order whose only line carries an unpublished product, then reordering is not
allowed and the endpoint raises `Nothing can be reordered in this order`.

**WS-AC-313.** Given a confirmed combo order line with two item children, when it is reordered, then the
combo line and both item lines are recreated with the same choices, custom values and no-variant values.

**WS-AC-314.** Given a reorder of a tracked product with only 1 unit free while 3 were ordered, then the cart
line ends at 1 and the cart carries the shortage warning.

**WS-AC-315.** Given two confirmed orders placed today and yesterday, when the quick reorder history is
built, then it contains one group labelled `Today` and one labelled `Yesterday`, and a product already in the
cart appears in neither.

## 23. Checkout, addresses and accounts

**WS-AC-316.** Given an anonymous cart, when the checkout page is requested, then the answer is a redirect to
the address form.

**WS-AC-317.** Given a site whose account policy is mandatory and an anonymous visitor with a cart, when
the checkout page is requested, then the answer is a redirect to the sign-in page carrying the checkout
page as its return path.

**WS-AC-318.** Given the account policy written to optional, then the site sign-up policy becomes free
sign-up; written to disabled, then it becomes invitation-only.

**WS-AC-319.** Given a country that requires a state and a postal code, when an address is submitted without
them, then it fails with `Some required fields are empty.` and the two inputs are highlighted.

**WS-AC-320.** Given an order containing only services and the billing-details parameter left at its default,
when an address is submitted with only a name and an address, then it fails, because the telephone and the
postal address are required.

**WS-AC-321.** Given the same order and the parameter set to a false value, then the same submission
succeeds.

**WS-AC-322.** Given an order that would not require an address, when the shopper fills the street but leaves
the city empty, then the submission fails, because filling any address field makes the whole address
mandatory.

**WS-AC-323.** Given an existing contact with a country and an issued document, when the country is changed
at checkout, then it fails with
`Changing your country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`

**WS-AC-324.** Given a contact linked to an internal user, when its name is changed at checkout, then it
fails with
`If you are ordering for an external person, please place your order via the backend. If you wish to change your name or email address, please do so in the account settings or contact your administrator.`

**WS-AC-325.** Given a child address of a company contact, when a commercial field is submitted with a
different value, then it fails with `The %(field_name)s is managed on your company account.`; submitted with
the same value, then the field is silently dropped and the submission succeeds.

**WS-AC-326.** Given a malformed electronic mail address, then the submission fails with
`Invalid Email! Please enter a valid email address.`

**WS-AC-327.** Given an anonymous cart and a successful address submission, then a contact of the kind
"contact" is created with the site's company and the site salesperson, the order's customer, billing and
delivery contacts are set, the public contact is unsubscribed from the order followers, and the redirect
target is the checkout page with the skip flag.

**WS-AC-328.** Given a site that keeps accounts separate per site, when a delivery address is created at
checkout, then the created contact records that site.

**WS-AC-329.** Given a signed-in shopper choosing an address that belongs to another commercial contact, when
the address selection endpoint is called, then the answer is "forbidden".

**WS-AC-330.** Given a cart with a selected delivery method and a delivery address in country A, when the
delivery address is changed to country B where the method is unavailable, then the method is replaced by the
first available one and the delivery line is rewritten at the new rate.

**WS-AC-331.** Given a contact whose country is changed in the back office, then the fiscal position of every
draft storefront order of that contact is recomputed, and the taxes and prices of the orders whose fiscal
position changed are recomputed.

**WS-AC-332.** Given a shopper updating one address used as both billing and delivery, when the form is
submitted without a telephone number, then it fails, because the telephone is mandatory for both roles.

**WS-AC-333.** Given a site with the newsletter option active and a newsletter list, when an address is
submitted with the newsletter box ticked and an address, then that address is subscribed to the list.

**WS-AC-334.** Given a site whose extra-information page option is switched on, then the corresponding
checkout step is published; switched off, the step is unpublished and its address redirects to the payment
page.

**WS-AC-335.** Given a site created after the generic checkout steps exist, when the site record is created,
then it owns one copy of each generic step, and the extra-information copy is published only when the page
option is active.

**WS-AC-336.** Given a site-specific checkout step whose generic counterpart is translated into a language,
when that language is installed, then the site-specific step carries the same translation (WS-467).

**WS-AC-337.** Given the four checkout steps, when the address page is rendered, then the forward target is
the extra-information step and the backward target is the cart step; when the extra-information step is
unpublished, the forward target is the payment step.

## 24. Delivery

**WS-AC-338.** Given a fixed-price method at 7.50 offering free shipping above 100.00 and a cart of 95.00,
when the method is rated, then the price is 7.50 and the order total is 102.50.

**WS-AC-339.** Given the same method and a cart of 105.00, then the price is 0.00, the warning
`The shipping is free since the order amount exceeds 100.00.` is returned, the delivery line still exists at
the price 0.00 and the order total is 105.00.

**WS-AC-340.** Given the same method and a cart that falls back to 95.00 after a line removal, when the cart
is verified, then the delivery line returns to 7.50.

**WS-AC-341.** Given a delivery product carrying a tax of 21 percent and a site displaying tax-excluded
prices, when a rate of 10.00 excluding tax is displayed, then 10.00 is shown; with tax-included display,
12.10 is shown.

**WS-AC-342.** Given a cart with a transaction in the pending state, when the shopper tries to change the
delivery method, then it is refused with
`It seems that there is already a transaction for your order; you can't change the delivery method anymore.`

**WS-AC-343.** Given a cart with a transaction in the draft state, when the shopper changes the delivery
method, then the change succeeds.

**WS-AC-344.** Given two methods, one restricted to another country, when the available methods are computed
for a delivery address in the shopper's country, then only the unrestricted method is returned.

**WS-AC-345.** Given a method belonging to another company, when the available methods are computed for an
order of the site's company, then that method is excluded.

**WS-AC-346.** Given a request for the rate of a method that is not among the available ones, then it fails
with
`It seems that a delivery method is not compatible with your address. Please refresh the page and try again.`

**WS-AC-347.** Given a cart that does not exist, when a rate is requested, then it fails with
`Your cart is empty.`

**WS-AC-348.** Given an order with goods and no available delivery method, when the payment step is rendered,
then the payment form is hidden and the page shows `Sorry, we are unable to ship your order.` with
`No shipping method is available for your current order and shipping address. Please contact us for more information.`

**WS-AC-349.** Given a cart re-priced by the payment step, when the delivery method is still set, then its
rate is recomputed and the delivery line rewritten, with the pickup location preserved.

## 25. Payment and confirmation

**WS-AC-350.** Given a cart with a selected delivery method, when a transaction is created, then the order row
is locked, the transaction carries the order's billing contact, the order currency and the order total, and
the session records the transaction.

**WS-AC-351.** Given a transaction request carrying an amount different from the order total, then it fails
with `The cart has been updated. Please refresh the page.`

**WS-AC-352.** Given an order already fully paid, when a transaction is requested, then it fails with
`The cart has already been paid. Please refresh the page.`

**WS-AC-353.** Given a transaction request carrying an argument that is not part of the accepted set, then
the request is rejected.

**WS-AC-354.** Given a cart with goods and no delivery method, when a transaction is requested, then it fails
with `No shipping method is selected.`

**WS-AC-355.** Given an order whose delivery method is no longer available, when a transaction is requested,
then it fails with `The delivery method is not compatible with your delivery address.`

**WS-AC-356.** Given an order with a zero total and no transaction, when the payment validation endpoint is
reached, then the readiness check runs and the order is confirmed with message sending enabled.

**WS-AC-357.** Given a transaction reaching the completed state on an order needing no signature, then the
order is confirmed, a salesperson is assigned, and the assignment notification is authored by the platform
system user rather than by the shopper.

**WS-AC-358.** Given a site carrying its own order confirmation template, then that template is used on
confirmation; without it, the platform default template is used.

**WS-AC-359.** Given a transaction reaching the pending state with a wire-transfer provider, then the order
moves to the sent state, the payment reference is written on the order, the payment-status message is sent,
and no invoice is created.

**WS-AC-360.** Given automatic invoicing enabled and a transaction reaching the completed state, then an
invoice is created, posted and sent, and the invoice carries the site of the source order.

**WS-AC-361.** Given an invoice whose lines come from orders of two different sites, then the invoice's site
field is empty.

**WS-AC-362.** Given a transaction still in the draft state when the shopper returns, then the payment
validation endpoint redirects to the shop rather than to the confirmation page.

**WS-AC-363.** Given a confirmed order, when the confirmation page is requested with the session's last order
identifier, then the page is rendered with the analytics payload carrying the order identifier, the company
name, the total, the tax amount, the currency code, one entry per line that is not a delivery line and the
delivery amount.

**WS-AC-364.** Given a portal user and an order paid with a transaction in the draft, pending or error state,
when the unpaid order list is read, then the order is returned; an order whose transaction is authorised or
completed is not.

**WS-AC-365.** Given a provider restricted to another site, when the payment step of this site is rendered,
then the provider is not offered and the availability report records the incompatible-site reason.

**WS-AC-366.** Given express checkout, when the payment values are built, then no stored payment token is
offered and the amount offered excludes the delivery amount.

**WS-AC-367.** Given an anonymous cart and an express delivery-address callback, then a new contact is
created, it is written as the order's customer, the price list is not recomputed, and the available methods
are returned sorted by increasing price with the cheapest preselected.

**WS-AC-368.** Given a registered shopper whose submitted express address matches an existing descendant
contact, then that contact is reused rather than duplicated.

**WS-AC-369.** Given a registered shopper whose submitted express address matches none, then a new descendant
contact is created and used as the delivery contact.

**WS-AC-370.** Given a placeholder contact created by a previous express callback, when a full address
arrives, then the placeholder is completed rather than duplicated.

## 26. Abandoned carts

**WS-AC-371.** Given a site with a delay of 1 hour and a cart whose order date is 1 hour and 1 minute ago
with a customer that is not the public contact and one line, then the cart is an abandoned cart; a cart whose
order date is 59 minutes ago is not.

**WS-AC-372.** Given a site with a delay of 30 minutes and another with 24 hours, when abandoned carts are
searched, then each site's own delay is applied.

**WS-AC-373.** Given a cart whose customer is the site's own public contact, then it is never an abandoned
cart; a cart whose customer is the public contact of **another** site is an abandoned cart.

**WS-AC-374.** Given a site with a delay of 10 hours, the feature switched on at 07:00 and a cart created at
08:00, when the job runs at 18:01, then the recovery message is sent once and the cart is marked as mailed;
when the job runs again at 19:00, nothing is sent.

**WS-AC-375.** Given the same site and a cart created at 06:00, that is before the activation moment, then no
recovery message is ever sent for it.

**WS-AC-376.** Given an abandoned cart whose customer has no electronic mail address, then no message is sent
and the cart is marked as mailed.

**WS-AC-377.** Given an abandoned cart every line of which is priced 0.00, then no message is sent and the
cart is marked as mailed.

**WS-AC-378.** Given an abandoned cart with a transaction in the error state, then no message is sent.

**WS-AC-379.** Given a customer who confirmed an order after their abandoned cart was created, then no
message is sent for that cart.

**WS-AC-380.** Given the storefront stock capability and an abandoned cart containing a sold-out product,
then no message is sent.

**WS-AC-381.** Given the recovery message, when it is sent, then it uses the record's default recipients
rather than an explicit address, and the portal button is labelled `Resume Order` and points at the cart
address carrying the order identifier and the order access token.

**WS-AC-382.** Given several abandoned carts of one site selected in the back office, when the recovery
composer is opened, then the site's own template is proposed; given carts of two sites, then the shipped
template is proposed.

**WS-AC-383.** Given the recovery composer sent in mass-mailing mode, then only the orders that are still
abandoned carts and not yet mailed are marked as mailed.

**WS-AC-384.** Given a recovery link opened with the revival method "squash", then the session cart becomes
the old order; with "merge" and an existing session cart, then the old order's lines are moved onto the
session cart and the old order is cancelled; with "merge" and no session cart, then the old order becomes the
session cart.

**WS-AC-385.** Given a recovery link with an incorrect access token, then the answer is "not found".

**WS-AC-386.** Given a recovery link for an order that is no longer in draft, then the cart page is rendered
with the "already completed" flag.

**WS-AC-387.** Given a sales team with two unmailed abandoned carts totalling 200.00, then the team reports a
count of 2 and an amount of 200.00; after one is marked as mailed, a count of 1.

## 27. Availability

**WS-AC-388.** Given a tracked product with out-of-stock ordering disabled, a quantity free to use of 3 and a
cart already holding 2, when the shopper asks for 5 on that line, then the line ends at 3 and the warning is
`You ask for 5 <product> but only 3 is available`.

**WS-AC-389.** Given the same product with a second cart line holding 1 unit, when the first line is raised
to 5, then it ends at 2, keeping the total cart quantity at 3.

**WS-AC-390.** Given a tracked product with out-of-stock ordering disabled and no quantity free to use, when
it is added, then the answer reports the quantity 0 with
`%(product_name)s has not been added to your cart since it is not available.`

**WS-AC-391.** Given a cart line whose product becomes unavailable, when the line is updated, then it is
deleted and the warning is
`Some products became unavailable and your cart has been updated. We're sorry for the inconvenience.`

**WS-AC-392.** Given a product with out-of-stock ordering allowed, when a quantity of 999 is requested, then
it is accepted unchanged.

**WS-AC-393.** Given a quantity free to use of 20 units and a product also sold in boxes of 6, when the
shopper asks for 4 boxes, then the line ends at 3 boxes.

**WS-AC-394.** Given a site with no warehouse and stock in two warehouses of 4 and 6, then the storefront
quantity free to use is 10; given a site warehouse holding 4, then it is 4.

**WS-AC-395.** Given a tracked variant with a quantity free to use of 7.8 units, when the combination
information is built, then the reported quantity is 7.

**WS-AC-396.** Given a product whose availability display is enabled with a threshold of 5 and a quantity free
to use of 3, then the remaining quantity is shown; with 9, nothing is shown.

**WS-AC-397.** Given a sold-out product with an out-of-stock message, then the message is shown and the
back-in-stock form is offered; the quick-add action is not offered.

**WS-AC-398.** Given a combo whose items have the maximum quantities 4 and unknown, then the combo maximum is
unknown; given 4 and 7, then it is 7; given a product with two combos whose maximums are 7 and 3, then the
product maximum is 3.

**WS-AC-399.** Given a cart line group whose free-minus-cart quantities are 5 and 2, then the line's maximum
available quantity is 2 and its maximum line quantity is the current quantity plus 2.

**WS-AC-400.** Given a cart whose line quantity exceeds the availability, when the payment is attempted, then
it fails with the concatenated per-line stock warnings.

**WS-AC-401.** Given an order with a storable product and no available delivery method, when the payment is
attempted, then it fails with the delivery error rather than with a stock error.

**WS-AC-402.** Given a zero-priced out-of-stock product on a site that forbids zero prices, when the payment
is attempted, then it fails.

**WS-AC-403.** Given a kit made of 1 table top and 4 legs with 7 tops and 30 legs free, then the storefront
quantity free to use of the kit is 7.

**WS-AC-404.** Given a template all of whose variants were deleted, when the shop page is opened, then the
page is rendered without failure.

## 28. Back-in-stock notifications

**WS-AC-405.** Given a sold-out published product and an anonymous visitor submitting a valid address, then a
contact is found or created, it is added to the product's subscriber list, and the session records the
product and the address.

**WS-AC-406.** Given an anonymous visitor submitting an address that already belongs to a user account, then
it fails with `Please sign in to proceed.`

**WS-AC-407.** Given an address that is not syntactically valid, then it fails with `Invalid Email`.

**WS-AC-408.** Given an unpublished or unsellable product, then it fails with
`This product is not eligible for stock notifications.`

**WS-AC-409.** Given a subscribed contact and a product that becomes available, when the hourly job runs,
then one message is created with the subject `The product '%(product_name)s' is now available` in the
contact's language and the contact is removed from the subscriber list.

**WS-AC-410.** Given a product that is still sold out, when the job runs, then nothing is sent and the
subscriber list is unchanged.

**WS-AC-411.** Given a wish list row whose owner is subscribed to the product, then the row reports the
subscription; writing the subscription flag to true subscribes the owner.

## 29. Collection in store

**WS-AC-412.** Given a collect-in-store method with no store, when it is published, then it fails with
`The delivery method must have at least one warehouse to be published.`

**WS-AC-413.** Given a collect-in-store method of company A and a store of company B, then it fails with
`The delivery method and a warehouse must share the same company`

**WS-AC-414.** Given a new method whose kind is set to collect-in-store, then its integration level becomes
rate only, cash on delivery is disabled, its country, state and postal-code restrictions are cleared, it
receives every warehouse of its company and it is published when at least one exists.

**WS-AC-415.** Given a method whose kind changes from collect-in-store to a carrier kind on an order, then
the order warehouse and fiscal position are recomputed and, when the fiscal position changed, the taxes are
recomputed.

**WS-AC-416.** Given a product page with no cart, when the pickup locations are fetched, then no cart is
created.

**WS-AC-417.** Given a store whose address cannot be geolocated, when its payload is built, then the
coordinates 1000 and 1000 are stored and the store sorts last; a later call does not attempt geolocation
again.

**WS-AC-418.** Given a reference address at latitude 50.8503 and longitude 4.3517 and a store at latitude
51.2194 and longitude 4.4025, then the computed distance is 41.19 kilometres and the stores are returned by
increasing distance.

**WS-AC-419.** Given a selected store, when a pickup location is set, then the order warehouse becomes that
store, the fiscal position is recomputed from the store address and, when it changed, the taxes are
recomputed.

**WS-AC-420.** Given an order collected in store whose selected store lacks stock for one line, when the
payment step is rendered, then the page shows `Some products are not available in the selected store.` and
the line carries `%(available_qty)s/%(line_qty)s available at this location`.

**WS-AC-421.** Given a collect-in-store method with exactly one store, when the checkout page is rendered,
then that store is offered as the default pickup location.

**WS-AC-422.** Given a site with a warehouse and a published collect-in-store method and a cart with no
delivery method, then the storefront quantity free to use of a product is the greater of the site warehouse
quantity and the best in-store quantity.

**WS-AC-423.** Given the same site and a cart with a collect-in-store method and a selected store, then the
storefront quantity free to use is the quantity in that store.

**WS-AC-424.** Given the same site and a cart with an ordinary carrier, then the storefront quantity free to
use is the site warehouse quantity.

**WS-AC-425.** Given an unpublished collect-in-store method, then the collection availability block is not
shown on the product page.

**WS-AC-426.** Given a product carrying a tag excluded by the collect-in-store method, then the collection
availability block is not shown for it.

**WS-AC-427.** Given an order whose delivery method is a collect-in-store method, then the pay-on-site
provider is offered; with any other method it is not, and the availability report records
`no in-store delivery methods available`.

**WS-AC-428.** Given a pay-on-site transaction reaching the pending state, then the order is confirmed and
the transfer is created.

**WS-AC-429.** Given an attempt to pay on site without a collect-in-store method, then the transaction is
refused with `You can only pay on site when selecting the pick up in store delivery method.`

**WS-AC-430.** Given a method with cash on delivery enabled, when it is switched to the collect-in-store
kind, then cash on delivery is disabled.

**WS-AC-431.** Given express checkout, then collect-in-store methods are never among the returned delivery
options.

**WS-AC-432.** Given a public shopper completing a collection purchase with more than one store available,
then the order carries the chosen store's warehouse, the fiscal position of that store and a delivery line at
the method's product price.

## 30. External relay points

**WS-AC-433.** Given an anonymous cart, when a relay point is submitted, then it fails with
`Customer of the order cannot be the public user at this step.`

**WS-AC-434.** Given a relay point in a country the method does not serve, then it fails with
`%s is not allowed for this delivery carrier.`

**WS-AC-435.** Given a valid relay point, then a relay contact is found or created under the customer and
written as the delivery address, and the re-rendered address block is returned.

**WS-AC-436.** Given a relay delivery address and a method that is not the relay method, when the payment is
attempted, then it fails with `Point Relais® can only be used with the delivery method Mondial Relay.`

**WS-AC-437.** Given the relay method and a delivery address that is not a relay point, then the payment
fails with `Delivery method Mondial Relay can only ship to Point Relais®.`

**WS-AC-438.** Given a relay contact, when the address form is opened for it, then the request fails with
`You cannot edit the address of a Point Relais®.`

## 31. Wish list and comparison

**WS-AC-439.** Given an anonymous visitor adding a product priced 114.35, then a wish list row is created
with no owner, the price 114.35, the current price list, the display currency and the site, and its
identifier is appended to the session list.

**WS-AC-440.** Given that visitor signing in as a contact who already saved the same product, then the
session row is deleted and the contact keeps one row.

**WS-AC-441.** Given that visitor signing in as a contact who has not saved it, then the session row is
assigned to the contact and the session list is cleared.

**WS-AC-442.** Given a saved product that is later unpublished, when the wish list is read, then the row is
not listed but is not deleted.

**WS-AC-443.** Given a portal user, when they read the wish list entity, then only their own rows are
returned; a sales manager reads every row; the public group has no access at all.

**WS-AC-444.** Given an anonymous row whose identifier is not in the session list, when removal is attempted,
then nothing is deleted.

**WS-AC-445.** Given a wish list row without an owner created six weeks ago, when the cleanup runs, then it is
deleted.

**WS-AC-446.** Given a second attempt to save the same product for the same contact, then it fails with
`Duplicated wishlisted product for this partner.`

**WS-AC-447.** Given the comparison page requested with no valid identifier, then the answer is a redirect to
the shop.

**WS-AC-448.** Given two products whose attributes belong to two attribute categories with the ordering
values 1 and 2 and one attribute without a category, when the comparison matrix is built, then the sections
appear in category order with the uncategorised section last.

**WS-AC-449.** Given a product whose attribute does not create variants, when the comparison matrix is built,
then the cell lists every value offered by that product's attribute line.

**WS-AC-450.** Given a product with a discount, when the comparison data is requested, then the entry carries
the discounted price and the reference price as the strikethrough price; with a comparison price greater than
the price and no discount, the comparison price is the strikethrough price.

## 32. Content blocks and visitor tracking on the storefront

**WS-AC-451.** Given one confirmed order containing one computer and three other products, when the recently
sold block is rendered, then the four products are listed with the most frequently sold first.

**WS-AC-452.** Given a visitor who viewed two products and then a computer, when the recently viewed block is
rendered, then the three products are listed with the computer first.

**WS-AC-453.** Given a product already in the cart, when the recently viewed block is rendered, then that
product is excluded.

**WS-AC-454.** Given a confirmed order containing a computer and two other products, when the sold-with block
is rendered for the computer, then the two other products are listed and the computer is not.

**WS-AC-455.** Given a product that is unpublished, when the newest products block is rendered, then it
disappears from the block.

**WS-AC-456.** Given a visitor whose company changes, when the recently viewed block is rendered, then only
products of the allowed companies are listed.

**WS-AC-457.** Given a product page view, when the tracking endpoint is called, then a visitor is created
when needed and a visit entry carrying the product is recorded; a variant that is not a possible combination
records nothing.

**WS-AC-458.** Given a category block and a visitor without shop access, then the block renders nothing.

**WS-AC-459.** Given the category block with no parent, then the top-level categories matching the
availability condition are returned, each with its identifier, name, published-products flag and cover
picture, falling back to the placeholder picture.

**WS-AC-460.** Given the category block for a parent, then the parent is returned first, followed by its
children.

**WS-AC-461.** Given a member of the Editor and Designer group, when the available block categories are read,
then categories without published products are included; for any other user they are excluded.

**WS-AC-462.** Given a category cover picture set through the editor endpoint by a user without the
Restricted Editor group, then the answer is "forbidden".

## 33. Promotions on the storefront

**WS-AC-463.** Given a promotion program not marked available on the site, then it never applies to a
storefront order.

**WS-AC-464.** Given a coupon code opened without a cart, then the code is remembered and the redirect
carries `The coupon will be automatically applied when you add something in your cart.`; after the first add,
the code is applied.

**WS-AC-465.** Given a discount program producing discount lines for two tax groups, when the cart is
displayed, then a single aggregate discount line is shown with the summed amounts.

**WS-AC-466.** Given a reward line removed by the shopper, then the reward is recorded in the order's disabled
list and is not applied again automatically.

**WS-AC-467.** Given a free-shipping reward and a method with a free-shipping threshold, when the reward is
applied, then the reward total is excluded from the amount compared with the threshold.

**WS-AC-468.** Given a reward that expires between the payment page and the transaction creation, then the
transaction is refused with
`Cannot process payment: applied reward was changed or has expired.\nPlease refresh the page and try again.`

**WS-AC-469.** Given an anonymous visitor, then nominative programs are not offered.

**WS-AC-470.** Given two active promotional rules with the same code, one generic and one attached to a site,
then the second save fails with `The promo code must be unique.`

**WS-AC-471.** Given an active coupon carrying the code `SUMMER`, when a program rule with that code is
saved, then it fails with `A coupon with the same code was found.`

## 34. Product feed

**WS-AC-472.** Given a site with the feed feature enabled and a valid feed identifier and token, when the feed
address is fetched, then the compressed document is returned with the extensible markup language content type
and a compressed content encoding.

**WS-AC-473.** Given the feature disabled, then the same request answers "not found".

**WS-AC-474.** Given a wrong access token, then the request answers "forbidden"; given an unknown identifier,
"not found"; given an identifier that is not a number, "bad request"; given a feed of another site, "bad
request" with `Website does not match.`

**WS-AC-475.** Given a feed whose language is set to a second site language, then the item names, descriptions
and links are produced in that language.

**WS-AC-476.** Given a feed naming a price list, then every item price uses that price list and every item
link carries the price list query parameter, and opening that link shows the same price.

**WS-AC-477.** Given a product with an internal reference, then the item identifier is the internal reference;
without one, it is the variant identifier.

**WS-AC-478.** Given a product with a barcode, then the item carries the barcode as the global trade item
number and reports that an identifier exists; without a barcode, it reports that none exists.

**WS-AC-479.** Given a site displaying tax-included prices, then the feed prices are tax included; with
tax-excluded display, they are tax excluded.

**WS-AC-480.** Given a price list rule with a discount and a validity window, then the item carries a sale
price and the effective-date range built from the two moments in coordinated universal time at minute
precision.

**WS-AC-481.** Given a product with a base unit count of 6 and the reference unit `750ml`, then the item
carries the reference measure `4500ml` and the base measure `750ml`.

**WS-AC-482.** Given a template with two variants, then every item carries the template identifier as the
group identifier.

**WS-AC-483.** Given the storefront stock capability and a sold-out variant, then the item availability is the
out-of-stock value; otherwise it is the in-stock value.

**WS-AC-484.** Given the storefront stock capability, a site warehouse and stock only in another warehouse,
then the feed availability reflects the site warehouse, not the total stock.

**WS-AC-485.** Given a feed whose configured categories would contain 5001 products, then the save fails with
`A single feed cannot contain more than %(limit)s products. Please separate products with Categories.` with
the limit printed as `5,000`.

**WS-AC-486.** Given a rendering returning 5600 products and no warning sent in the last week, then the site
salesperson receives the subject `GMC: Product Limit Exceeded` and the notification date is recorded.

**WS-AC-487.** Given a feed rendered once today, when it is fetched again the same day, then the cached
document is returned without re-rendering; when a feed parameter is changed, the next fetch re-renders.

## 35. Settings and configuration

**WS-AC-488.** Given the account policy written to optional and then to disabled, when the site is read after
each write, then the sign-up policy is free sign-up and then invitation-only.

**WS-AC-489.** Given the product feed setting switched on for one site, then every site receives the same flag
and each site without a feed and within the soft limit receives one feed carrying the shipped name `GMC 1`.

**WS-AC-490.** Given the page settings endpoint called with a products-per-page value of 0, then 1 is stored.

**WS-AC-491.** Given the page settings endpoint called with a field outside the allow list, then that field is
not written.

**WS-AC-492.** Given a category style page option enabled, then the other five category style options are
disabled.

**WS-AC-493.** Given a new company created while the price list feature is enabled, then the automatically
created price list carries no site.

**WS-AC-494.** Given the company's fiscal country changed, then the site tax display mode is reset to tax
excluded and remains editable.

**WS-AC-495.** Given a product created from the storefront with a storefront category, then it is published;
created without one, it is unpublished.

**WS-AC-496.** Given the contact-button option and a hidden price, then the product page shows the contact
button pointing at the site's configured address instead of the add-to-cart button.

**WS-AC-497.** Given an anonymous visitor, when they try to react to a product review, then the action is
refused, because posting requires the review page option and write access otherwise.

**WS-AC-498.** Given a member of the Editor and Designer group, when they edit the search engine metadata of
a product, then the write succeeds.

**WS-AC-499.** Given a product page opened in a language that is not the default through a category address,
then the advertised canonical address carries the language prefix and no category segment.

**WS-AC-500.** Given a contact with an open storefront cart, when their assigned price list is changed in the
back office, then the form shows the warning titled `Open Sale Orders`.

## 36. Courses and print on demand

**WS-AC-501.** Given a paid course whose product is published, when the course is unpublished and no other
published course uses that product, then the product is unpublished.

**WS-AC-502.** Given a paid course with no product, when it is saved, then it fails with
`Product is required for on payment channels.`

**WS-AC-503.** Given a course product already in the cart, when it is added again, then the quantity stays 1
and the warning is `You can only add a course once in your cart.`

**WS-AC-504.** Given a free course product and a site that forbids zero-price sales, when it is added, then
the add succeeds, because the course tracking value is exempt.

**WS-AC-505.** Given a confirmed order containing a paid course product, then the customer becomes a member of
that course and the confirmation page links to it.

**WS-AC-506.** Given a print-on-demand product and an ordinary goods product, when the second is added to a
cart holding the first, then the add returns the quantity 0 with
`The product %(product_name)s cannot be added to the cart as it requires separate shipping. Please place your order for the current cart first.`

**WS-AC-507.** Given a print-on-demand product without print pictures, when it is published, then the write
fails with `Print images must be set on products before they can be published.`

**WS-AC-508.** Given a published print-on-demand product, when one of its print pictures is emptied, then the
write fails with `Products must be unpublished before print images can be removed.`

## 37. Donations

**WS-AC-509.** Given the donation page submitted by form with an amount and a currency, then the values are
stored in the session and the answer is a redirect to the same address, after which the page renders with
those values.

**WS-AC-510.** Given a donation transaction requested below the configured minimum, then it fails with
`Donation amount must be at least %.2f.`

**WS-AC-511.** Given an anonymous donor without a name, an address or a country, then the transaction fails
with `Name is required.`, `Email is required.` or `Country is required.` respectively.

**WS-AC-512.** Given an anonymous donation, then the transaction is attached to the site's public contact,
tokenisation is disabled, and the donor name, address, country and language are written on the transaction.

**WS-AC-513.** Given a completed donation transaction, then the donor receives the confirmation message and
the resulting payment carries the donation flag and a message listing the donor details.

**WS-AC-514.** Given an anonymous donation page, then the option that saves the payment details is not
offered.

**WS-AC-515.** Given a site with two published providers offering card and wallet payments, when the
advertised payment methods block is requested, then the card brands and the wallet method are returned, the
answer is not cached for internal users, and it is cached for one week for everybody else.

## 38. Redirects and robustness on the storefront

**WS-AC-516.** Given a product page address carrying a category the product does not belong to, then the
answer is a permanent redirect to the canonical product address.

**WS-AC-517.** Given a category segment naming a category that does not exist, then the answer is "not
found".

**WS-AC-518.** Given an unpublished product requested by an anonymous visitor through a category address,
then the answer is a redirect to the category listing; without a category, the answer is "not found".

**WS-AC-519.** Given the compatibility product address with a category query parameter, then the answer is a
permanent redirect to the canonical category-qualified address with that parameter removed.

**WS-AC-520.** Given a product page requested with a price list parameter that is not a number, then the
request fails with ``Wrong format: got `pricelist=abc`, expected an integer``.

**WS-AC-521.** Given a site rewrite rule that changes the path of a checkout step, then the forward and
backward navigation still resolves the step, because the paths are compared after rewriting.

**WS-AC-522.** Given a cached page served to a shopper with three items in the cart, then the rendered header
counter shows 3 and carries the order identifier attribute; with an empty cart, the counter element is
hidden.

**WS-AC-523.** Given a page whose markup contains a cart marker, then it is not inserted into the page cache.

---

## Reconciliation notes

1. **Two scenario catalogues.** The storefront catalogue numbered its scenarios `SHOP-AC-001` to
   `SHOP-AC-323`; they are `WS-AC-201` to `WS-AC-523` here, in the same order, so that the identifier here is
   the former number plus 200. The site half of the folder had no scenarios at all; `WS-AC-001` to
   `WS-AC-200` are written here for the first time from the entities, the rules and the state machines.
2. **Message wording.** Where a scenario quotes a message, the message is the one reproduced in
   [business-rules.md](business-rules.md); where the two former catalogues quoted the same message with
   different placeholders, the placeholder spelling of the rule catalogue is used.
