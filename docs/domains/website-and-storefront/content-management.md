# Content management

The deep specification of everything a designer or an editor does to a site: editable regions and how they
are saved, architecture versioning and resetting, pictures and the media dialogue, style variables and asset
customisation, fonts and palettes, themes, the first-run configurator, the form builder, the anti-robot
verification services, dynamic content blocks and the revision history of rich text fields.

The per-site behaviour of templates — copy-on-write, most-specific selection — is in
[multi-site-and-languages.md](multi-site-and-languages.md); this file assumes it.

---

## 1. The editable page

### 1.1 What is editable

A rendered page carries three kinds of editable material.

| Kind | Marked by | Saved as |
|---|---|---|
| An editable region of a template | the editable-region marker on an element | a replacement of the addressed section of the template architecture |
| A record field rendered inside the page | a field binding on an element | a write on that field of that record |
| A content block dropped by the editor | the block marker on the root element of the block | part of the enclosing editable region |

Every element the editor can move, copy or configure carries a branding marker that ties it to a position in
the stored architecture. Branding markers and editing classes are removed before storage, so that the stored
architecture contains only the authored markup.

### 1.2 Saving a region

1. The editor sends, for each edited region, the region's location expression and its new markup.
2. When a location expression was supplied, the template has a key, a site is resolved and a site-specific
   template with that key already exists, the save is diverted to that specific template (WS-167). This
   matters when several regions of a shared template are saved in the same batch: the first save creates the
   specific template and the others must write on it.
3. When delayed translation is requested and the parameter that disables it is unset or false, the save is
   performed with delayed translation, which keeps the translations of unchanged terms (WS-172).
4. The platform save replaces the addressed section of the architecture, extracts and writes any embedded
   record field found in the section, and cleans the non-editing attributes before storing.
5. Writing a template while a site is in context performs copy-on-write.
6. Saving a content block region on a shared template writes the site into the created record, so that the
   block stays per site (WS-169).
7. The saved architecture is stored with the site's default language as the source language (WS-171).

### 1.3 Attributes that survive a save

In addition to the platform's allow list, the following attributes survive a save on the root element of a
block: the background video source, the shape, the scroll background ratio, the visibility mode, the
visibility identifier, the visibility selectors, and, for each of country, language, signed-in state,
campaign, medium and source, the visibility value and its comparison rule (WS-173). These carry the
conditional-visibility feature of content blocks, which shows or hides a block according to the visitor's
country, language, sign-in state or campaign parameters.

### 1.4 Resetting a template

| Mode | Effect |
|---|---|
| Soft | The architecture is restored to the previously stored architecture of the same template. |
| Hard | The architecture is re-read from the shipped file the template was loaded from, when the template records such a path. |

Both run with copy-on-write disabled, so that a broken shared template is repaired in place instead of being
forked per site. The theme options panel can also request a reset when a template is disabled, which
performs a hard reset before deactivation (WS-174, WS-175).

---

## 2. Page and template lifecycle from the editor

| Action | What it does |
|---|---|
| New Page | Copies a template, gives the copy a unique key, creates the page and optionally a menu entry (§5 of [workflows.md](workflows.md)). |
| Properties | Edits the address, the title, the menu membership, the home-page flag, the publication, the publishing date, the indexing flag, the visibility, the password, the authorised groups and the template flag, and optionally creates a redirect from the old address (§6 of [workflows.md](workflows.md)). |
| Clone | Duplicates the page, its template and, within the same site, its menu entry (§7 of [workflows.md](workflows.md)). |
| Delete | Deletes the page and the templates used only by it (WS-059). |
| Optimise for search engines | Edits the metadata of §3.1 of [entities.md](entities.md) and reports whether the record may still be edited. |
| Save as a new-page template | Sets the flag that makes the page appear in the custom group of the template picker. |

---

## 3. Pictures, attachments and the media dialogue

### 3.1 Uploading

1. Only the supported picture kinds are accepted: the graphics interchange format, the three joint
   photographic experts group extensions, the portable network graphics format, the scalable vector format
   and the web picture format. Anything else is refused with the message of WS-188.
2. An uploaded picture is resized to the requested box and re-encoded at the requested quality, and its
   resolution is verified. A file the picture library cannot read is refused with the library's own message.
3. When no name is supplied, the name is built from the moment of upload and six random characters
   (WS-190).
4. A file whose name ends with the bitmap extension is stored without that suffix, so that the stored media
   type and the name agree (WS-191).
5. An attachment created by the editor is public when its related entity is the template entity, and its
   related identifier is then forced to empty (WS-192).
6. An attachment created while a site is resolvable receives that site (WS-184).

### 3.2 Modifying a picture

Cropping, resizing, rotating, applying a filter or converting the format produces a **derived** attachment
that records the source attachment as its original, is of the binary kind, carries the requested media type
and name, and is related either to the template entity with the identifier 0 or to the supplied record. When
an identical attachment already exists and carries no address, it is reused instead of creating a new one
(WS-193).

The caller must pass the read check on the source attachment's related record, when it has one, and the
write check on the target record; the copy itself is performed with elevated rights, and a media type forced
to plain text by the storage layer is corrected afterwards with elevated rights (WS-194). Converting to the
web picture format renames the file (WS-195).

### 3.3 Removing a picture

The remove operation deletes only the attachments whose local address appears in no template architecture,
in either quoting style. For each refused attachment the names of the blocking templates are returned, so
that the editor can tell the designer where the picture is still used (WS-187).

### 3.4 The external picture library

The media dialogue can search an external picture library. The library is called with the stored access key;
the chosen picture is downloaded and stored as an attachment of the site; a download notification is sent
back to the library, which its terms require. When no access key is stored, the library tab is not offered.

### 3.5 Alternate text

The editor can list every picture of the current page with its alternate text and rewrite them in one
operation, which is how a site is made accessible after the fact.

---

## 4. Style variables and asset customisation

A theme exposes its design decisions as a variable map inside a style source file. Editing a design option
in the theme panel writes a pair into that map.

### 4.1 The customisation procedure

1. The style variable file must contain a variable map with a line comment containing the word `hook` that
   marks where new pairs are written. A file without that marker cannot be customised.
2. The current content is read: the customised copy when one exists, and otherwise the shipped file.
3. The supplied key and value pairs are inserted at the hook, replacing any pair with the same key.
4. The result is saved as an asset customisation.

### 4.2 Saving an asset customisation

1. The customised address is the customisation prefix `/_custom/`, then the bundle name, then the original
   address.
2. When a customised attachment already exists at that address, its bytes are replaced and the asset cache
   is cleared.
3. Otherwise an Attachment is created with the original file name, the binary kind, the media type of a
   script for a script and of a style source for a style source, the new bytes, the customised address and
   the current site; and an Asset is created with the customised address as its path, the original address
   as its target, the replacement directive and the current site.
4. When an asset already targets the original address, the new asset takes that asset's name with the suffix
   ` override`, its bundle and its ordering value. Otherwise its name is the bundle, a colon, a space, the
   word `replace`, a space and the file name, and its bundle is derived from the original address.

Resetting a customisation deletes both the customised attachment and the replacing asset, which returns the
site to the shipped file.

### 4.3 Palettes

Setting the palette name additionally resets the user colour palette and the grey palette, sets the four
semantic theme colours to unset, and sets the five preset background gradients and the four chrome gradients
to unset. This is what makes a palette change a clean switch rather than an accumulation of overrides.

### 4.4 Fonts

Uploading a font stores the font files as public attachments and records their identifiers in the variable
map. A font file is accepted only when its bytes match its extension, and a compressed archive is accepted
with each contained font validated the same way (WS-196). Deleting a font deletes the attachment, every
attachment derived from it and every attachment whose name contains the font-provider marker, so that no
orphan rendition survives.

The editor also offers the catalogue of an external font service; choosing a font from it writes the
variable without uploading any file.

---

## 5. Content delivery network rewriting

When the network is enabled, an asset address is rewritten to the network base address joined with the
address, when the address matches at least one of the non-empty filter lines, read as matching patterns
anchored at the start (WS-021). An empty address returns empty text. Writing any of the three network fields
clears the registry cache, because asset addresses are baked into compiled templates (WS-022).

---

## 6. Themes

### 6.1 What a theme is

A theme is a capability package that ships template records: Theme Templates, Theme Assets, Theme
Attachments, Theme Pages and Theme Menus, specified in [entities.md](entities.md) Part 2. Installing it on a
site copies each of those into a real record bound to that site, and the real record keeps a pointer back to
its theme record, which is what allows the removal to find the copies again.

### 6.2 Loading

The load procedure, its fixed processing order, its deferred conversion and its orphan cleanup are §22 of
[workflows.md](workflows.md) and WS-500 to WS-505. Two details deserve emphasis:

* the order is templates, assets, pages, menus, attachments, because a page needs its template and a menu
  needs its page;
* a theme record whose dependency has not yet produced a copy for this site is postponed to a later pass
  over the same entity, which is what lets a theme ship a template whose parent is another theme template.

### 6.3 The theme utilities

The theme utilities service enables and disables named templates and assets and resets the default style
configuration.

| Operation | Effect |
|---|---|
| Enable templates | Activates the named templates; enabling a header template disables every other header template first, and the same for footers (WS-176). |
| Disable templates | Deactivates the named templates, optionally after a hard reset of their architecture (WS-175). |
| Enable and disable assets | The same for assets. |
| Reset the default configuration | Sets the nine style variables to unset — the font, the headings font, the navigation bar font, the buttons font, the palette number, the palette name, the ripple effect, the header template and the footer template — disables the two ripple assets, disables every non-default header and footer template, enables the default ones, and disables the footer scroll-to-top option (WS-506). |

A toggle that targets a shared template or asset, where no site-specific record exists yet and the requested
state already equals the current state, writes nothing, so that the record is not forked for nothing
(WS-177).

### 6.4 Removing and refreshing

Removing a theme resets the default configuration first, whether or not a theme is installed, then unloads
the stream in reverse installation order and clears the pointer. Refreshing upgrades the chain of packages
the theme depends on, which reloads the copies through the installation write. Both are §23 and §24 of
[workflows.md](workflows.md).

### 6.5 Translations of theme records

The translated fields of theme records — the template architecture and the menu name — have their
translations copied onto the generated records for every activated language, so that installing a theme on a
multi-language site does not lose the theme's own translations (WS-509).

---

## 7. The first-run configurator

The full procedure is §25 of [workflows.md](workflows.md). This section specifies the parts that a rebuild
most often has to re-derive.

### 7.1 The feature catalogue

Each Website Configurator Feature names either a page template or a capability package, never both
(WS-511). A feature carries its ordering value, its label, its one-line description, its icon, the page code
sent to the content suggestion service, the preselection list of site kinds and purposes, the address the
feature lives at, the menu ordering value and whether its menu entry belongs under the Company grouping.

### 7.2 Steps

The five steps are: initialisation, industry, purpose and kind, palette and theme, and features. Each step
answers with the values the next one needs; the sequence is resumable, because the completion flag is
written only at the end.

### 7.3 Applying the choices

The sixteen numbered actions of the apply step are in §25.7 of [workflows.md](workflows.md). The order
matters in three places:

1. the theme is installed **before** anything else is written, because the installation resets the style
   configuration;
2. the packages of the selected features are installed **before** their menu ordering values are applied,
   because a package creates its own endpoints and menu entries;
3. the pages are filled with content blocks **after** the packages are installed, because a package may
   contribute blocks.

### 7.4 Text generation

Generated text is requested only when the translation coverage of the rendered blocks exceeds 0.8
([calculations.md](calculations.md) §27). The text processor turns each rendered block into a set of
placeholder terms, sends the placeholders with the language name, the industry and the database identifier,
and re-applies the original formatting to the answers. Every occurrence of the marker `XXXX` in a generated
text is replaced by the site name (WS-514).

The three service failures — the industry catalogue, the text generation and the picture download — are all
caught and leave the shipped content in place (WS-519).

### 7.5 Industry pictures

Each industry picture is stored as a public attachment of the site under a deterministic external
identifier, so that repeated runs do not duplicate them, and a fixed fallback table copies another picture of
the same industry under the missing name when an industry did not provide one (WS-516, WS-517).

---

## 8. The form builder

### 8.1 What a public form is

A public form is markup dropped into a page by the editor. Its action targets the public form endpoint with
the transport name of the target entity in the path, and each input's name is the name of a field of that
entity, or a free name. The target entity must be flagged as usable in forms (WS-200).

### 8.2 The writable field set

For any entity other than the outgoing mail entity, the writable set is the fields of that entity whose
form-exclusion flag is false, intersected with the authorised field set. For the outgoing mail entity the
writable set is fixed: the sender address, the recipient address, the copy recipients, the blind copy
recipients, the body, the reply address and the subject (WS-201).

The authorised field set is computed as follows (WS-202).

1. Start from the declared fields of the entity.
2. Remove the delegation link fields.
3. Mark every field that has a default value as not required.
4. Remove the fields that are read only, that are one of the platform's automatic columns, that are a
   polymorphic numeric reference or that are structured data.
5. Remove a field whose condition is expressed as text, because it would have to be evaluated.
6. Expand a dynamic-properties field into pseudo-fields read from its definition record:
   1. skip a property that is not fully defined — a record-valued property without a target entity, a choice
      property without choices, a tag property without tags, and a separator;
   2. mark every pseudo-field as not required;
   3. convert a textual condition into a structured condition, and drop the property when that conversion
      fails.

Only a member of the Editor and Designer group may remove a field from the exclusion list, and unknown field
names are refused (WS-203). A field that an existing published form writes cannot be deleted (WS-204).

### 8.3 Handling a submission

The complete procedure, with the classification of the submitted values, the conversions, the free-text
assembly, the attachments and the answers, is §27 of [workflows.md](workflows.md) and WS-205 to WS-218. The
three decisions that a rebuild must reproduce exactly are:

* the forgery token is verified **only** when the session is authenticated, because a form embedded in a
  third-party page loses its session cookie (WS-205);
* the whole handling runs in a savepoint, so a validation failure rolls back only the form handling
  (WS-206);
* unmatched values are never dropped: they become free text on the designated field, or a comment message
  when the entity designates none (WS-208, WS-211).

### 8.4 The outgoing mail form

A form whose target is the outgoing mail entity sends a message instead of creating a business record. Two
rules protect it: the sender is rewritten so that the message is sent from the company and answers to the
visitor (WS-213), and the recipient is signed, so that a visitor cannot rewrite the recipient in the markup
(WS-214). The signature is recomputed on the server every time a rich text value containing a form is
rendered, and a stale signature node is removed first.

---

## 9. Anti-robot verification

Two verification services can be installed. When both are installed both run, and the submission must pass
both. Each is called with a two-second timeout, and each removes its own token from the request parameters
before the values are classified, so that the token never becomes free text.

### 9.1 The score-based service

The submission token, the private key and the network address are sent to the verification endpoint. The
answer is classified into: no private key configured; the request succeeded with a score at or above the
minimum and a matching action; the score is below the minimum; the action does not match; the token is
missing or invalid; the private key is missing or invalid; the token is expired or already used; or the
request was malformed.

No private key configured and a human answer both pass. Otherwise:

| Outcome | Message |
|---|---|
| Invalid private key | `The reCaptcha private key is invalid.` |
| Invalid token | `The reCaptcha token is invalid.` |
| Timeout or reused token | `Your request has timed out, please retry.` |
| Malformed request | `The request is invalid or malformed.` |
| Anything else, including a score below the minimum | `Suspicious activity detected by google reCAPTCHA.` |

### 9.2 The challenge-widget service

The same shape, with its own token parameter, its own secret parameter and its own verification endpoint.

| Outcome | Message |
|---|---|
| Invalid private key | `The Cloudflare turnstile private key is invalid.` |
| Failed challenge | `The CloudFlare human validation failed.` |
| Timeout | `Your request has timed out, please retry.` |
| Malformed request | `The request is invalid or malformed.` |
| Anything else | `Suspicious activity detected by Turnstile CAPTCHA.` |

### 9.3 The decoy field

Independently of the two services, the shipped form template includes a field that no human fills. A
submission that carries a value in it is treated as an unmatched free-text value and, by convention, the
record is still created but the value appears in the free-text block, so that the submission can be
recognised as automated. This is the observed behaviour; refusing such a submission outright is the
industry-standard default WS-900, and a rebuild that adopts it must answer with the same shape as a
validation failure.

---

## 10. Dynamic content blocks

A dynamic content block renders records chosen by a Website Content Block Filter
([entities.md](entities.md) §1.9).

### 10.1 Rendering

1. The block names a filter, a limit, a template key and, for a cross-selling block, a reference product.
2. The filter is refused when it is bound to another site (WS-254).
3. The template key must contain the dynamic block marker, and, when the filter exposes an entity, the
   entity name with dots replaced by underscores (WS-253, WS-255).
4. The effective number of records is computed by WS-256.
5. The records come from the saved filter or from the record-returning action of the filter, narrowed by the
   implicit conditions of WS-258: the current site, the company, and the publication flag when the entity
   carries one.
6. Only entities registered by at least one existing filter may be queried (WS-257).
7. When the requested limit is 1 and both an entity and a record are supplied, exactly that record is
   fetched, bypassing the filter (WS-259).
8. Each record is reduced to the exposed field list; a money value is converted to the site company's
   currency at today's rate (WS-261); a picture value becomes the site picture address with a cache marker
   (WS-262); the presentation of each field is derived by WS-263.
9. When the selection is empty and samples were requested, the sample fallback of WS-260 fills the block, so
   that a designer sees a realistic layout before any record exists.

### 10.2 Offering the filters in the editor

Listing the available filters requires the Restricted Editor group; otherwise the request answers "not
found". Any supplied extra condition must mention only fields of the filter entity, which prevents the
listing from being turned into a general query endpoint (WS-264).

### 10.3 Unused block assets

Each content block ships its own style and script assets, versioned by a three-digit number. A weekly job
activates exactly the assets that some stored content actually uses and deactivates the others; the scan,
the parsing of the asset paths and the compatibility rule for the legacy quotation block are §26 of
[workflows.md](workflows.md).

---

## 11. Revision history of rich text fields

An entity may declare that some of its rich text fields are versioned. Every declared field must be a
sanitised rich text field; a record whose declared fields are not all sanitised fails with
`Ensure all versioned fields ( %s ) in model %s are declared as sanitize=True`.

| Operation | Effect |
|---|---|
| Create | An empty history is initialised for each declared field. |
| Write | The reverse patch between the old and the new content is prepended to that field's history, the revision identifier is increased by one, and the history is truncated to the configured depth. |
| Restore | The field is rebuilt at a chosen revision by applying the stored reverse patches in order. |
| Compare | A marked-up comparison between the current content and a revision is produced. |
| Difference | A line-oriented difference between them is produced. |

The metadata view of the history lists, per field, the revision identifiers with their author and moment,
without the patch payloads, so that a revision list can be shown without transferring the content.

---

## 12. Cookie consent and third-party content

The consent bar, the stored consent document, the permission check and the neutralisation of third-party
frames and scripts are §32 and §33 of [workflows.md](workflows.md), WS-470 to WS-479, and the state table of
[state-machines.md](state-machines.md) §16. Two consequences for content management deserve emphasis:

* the all-consents-granted flag participates in the page cache key, so a page containing a neutralised block
  and the same page with its block restored are two separate cache entries (WS-474);
* neutralisation never applies to a Restricted Editor, so a designer always sees the real blocks while
  editing (WS-475).
