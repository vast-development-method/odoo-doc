# Attachments and the file store

Every piece of binary content the system holds, whether an uploaded document, a generated printable document, a product photograph, a company logotype or a compiled asset bundle, is an **Attachment** record. This document specifies the Attachment entity field by field, the two storage strategies and the operation that moves content between them, the content-addressed layout of the file store with its collision detection, the write path with its media-type detection, its plain-text forcing and its automatic image reduction, the access rules that govern reading and writing an attachment, the deferred deletion protocol and the collection that empties it, the derivation of image variants, the streaming contract with its validators and cache directives, and the upload paths.

The request paths that expose these behaviours, and the response hardening applied to every response, are in [`request-lifecycle.md`](request-lifecycle.md). The cache directives placed on the responses are catalogued in [`caching.md`](caching.md), section 14. The lock the collection takes is specified in [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 7.

## 1. The Attachment entity

The transport name of the entity is `ir.attachment` and its full name is Attachment. Records are ordered by key descending, so that the newest attachment of a record is listed first. Attachments are not archivable: the entity declares no active field, and the search operation asserts that none is present.

| Identifier | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | Text | yes | none | yes | The file name shown to the user and proposed when downloading. |
| `description` | Description | Long text | no | none | yes | A free description. |
| `res_model` | Resource Model | Text | no | none | yes | The transport name of the entity the attachment belongs to. |
| `res_id` | Resource Identifier | Reference key into the entity named by `res_model` | no | none | yes | The key of the record the attachment belongs to. |
| `res_field` | Resource Field | Text | no | none | yes | When set, the attachment **is** the content of that binary field of that record, rather than a document attached to it. |
| `res_name` | Resource Name | Text, computed, not stored | no | none | no | The display name of the referenced record, or nothing when there is no reference. |
| `company_id` | Company | Many-to-one to Company, participates in default propagation | no | the acting company | yes | Company scoping. |
| `type` | Type | Selection: `url`, `binary` | yes | `binary` | yes | Whether the content is a remote address or stored bytes. |
| `url` | Url | Text of at most 1 024 characters, indexed when not empty | no | none | yes | For the type `url`, the remote address. Also used to make an attachment serve a request path, as specified in section 11. |
| `public` | Is public document | Boolean | no | false | yes | When true the attachment is readable by anyone and the response may be stored by shared caches. |
| `access_token` | Access Token | Text, readable only by internal users | no | none | yes | A random token that grants read access without any other permission. |
| `raw` | File Content (raw) | Binary, computed with an inverse | no | none | no | The content as bytes. Reading it returns the bytes from wherever they are stored; writing it stores them according to the strategy. |
| `datas` | File Content (base64) | Binary, computed with an inverse | no | none | no | The same content in its text-encoded form. |
| `db_datas` | Database Data | Binary that is itself never stored as an attachment | no | none | yes | The content when the strategy is database storage. |
| `store_fname` | Stored Filename | Text, indexed | no | none | yes | The path of the content inside the file store when the strategy is file storage. |
| `file_size` | File Size | Integer, read-only | no | 0 | yes | The size of the content in bytes. |
| `checksum` | Checksum | Text of at most 40 characters, read-only | no | none | yes | The content digest, as specified in section 3. |
| `mimetype` | Mime Type | Text, read-only | no | none | yes | The media type of the content, as specified in section 5.2. |
| `index_content` | Indexed Content | Long text, read-only, never prefetched | no | none | yes | The searchable text extracted from the content, as specified in section 5.6. |

The label of the checksum field, reproduced as it appears on the form, is `Checksum/SHA1`; the second part names the digest algorithm. The two selection labels of the type field are `URL` for a remote address and `File` for stored bytes, and the field's help text reads `You can either upload a file from your computer or copy/paste an internet link to your file.`

**Index.** One composite index over the pair (`res_model`, `res_id`), which is the index every "the attachments of this record" query uses.

**Computed reads.**

| Field | Rule |
|---|---|
| `res_name` | When both `res_model` and `res_id` are set, the display name of that record; otherwise nothing. |
| `raw` | The bytes read from the file store when `store_fname` is set, otherwise the value of `db_datas`. |
| `datas` | Under the size-only reading mode, a human-readable rendering of `file_size`; otherwise the text encoding of `raw`. |

Reading `datas` under the size-only mode never touches the file store, which is what lets a list view show hundreds of attachments without reading hundreds of files.

**Computed writes.** Writing `raw` or `datas` runs the content write path of section 5.

**Validations.**

| Condition that fails | Message |
|---|---|
| An attachment points at its own record: `res_model` is `ir.attachment` and `res_id` equals the attachment's own key. | `You cannot attach an attachment to itself.` followed by a line break and `Attachment <record> cannot have res_id: <res_id>`, in which both placeholders render the attachment itself. |
| A user who is not an administrator creates or writes an attachment whose type is `binary` and which also carries a web address, without belonging to one of the groups declared as allowed to publish served attachments. | `Sorry, you are not allowed to write on this document` |

The declared group list contains exactly one group as shipped: the settings administration group, whose external identifier is `base.group_system`. The second rule exists because such an attachment is served by the request fallback of section 11: being able to create one means being able to publish arbitrary content at an arbitrary path.

**Derived content of a printable document.** An attachment exposes its bytes as a paginated document only when its type is `binary` and its media type begins with the Portable Document Format type; otherwise the request for that form returns nothing rather than raising.

## 2. Storage strategies

The strategy of a database is the value of the system parameter `ir_attachment.location`, the attachment storage location, defaulting to `file`.

| Value | Where content lives | Consequences |
|---|---|---|
| `file` | In the file store, at a path derived from the content digest. `store_fname` is set and `db_datas` is empty. | Identical content is stored once, whatever the number of attachments referencing it. A backup of the database alone does not contain the content. |
| `db` | In `db_datas`. `store_fname` is empty. | Content is included in a database backup. Identical content is stored once per attachment. |

Any other value is a configuration error: the operation that lists the attachments to migrate resolves the strategy through a fixed two-entry map and fails on an unknown value.

### 2.1 Migrating between strategies

A privileged operation moves every attachment to the currently configured strategy.

1. If the acting user is not an administrator, refuse with `Only administrators can execute this action.`
2. Select every attachment whose type is `binary` and whose content is in the other strategy: those with a stored file name when the target is database storage, those with database content when the target is file storage. The selection explicitly includes the attachments that are the content of a field, by naming a condition that is true for both a set and an unset field name, which suppresses the implicit filter of section 6.2.
3. For each selected attachment, in order, log at debug level `Migrate attachment <n>/<total> to <STRATEGY>`, in which the placeholders are the one-based position, the total count and the target strategy in capital letters.
4. Rewrite its content with its own current content and its own current media type.

Rewriting the content re-runs the write path, which stores the bytes according to the current strategy and clears the other location. Passing the media type explicitly avoids detecting it a second time.

An attachment whose type is `url` is never migrated to local storage. An attempt to do so is refused with `URL attachment (%s) shouldn't be migrated to local.`, in which the placeholder is the attachment's key; an attachment whose type is `binary` is accepted without doing anything.

## 3. The file store layout

```formula
digest        = content digest of the content, 40 hexadecimal characters
relative path = first 2 characters of the digest + "/" + digest
absolute path = file store root of this database + "/" + relative path
```

The two-character first level scatters files across at most 256 directories, which keeps any one directory small. Directories are created on demand.

**Path sanitizing.** Every relative path is sanitized before use: every full stop and every colon is removed, then leading and trailing path separators of both kinds are stripped. This is what prevents a crafted stored file name from escaping the file store, because a path that cannot contain a full stop cannot contain the parent-directory reference.

**Collision detection.** Before writing, if a file already exists at the target path, its content is compared with the content being written, block by block in blocks of 1 024 bytes. The comparison walks the existing file to its end, so a file that is a strict prefix of the new content is detected as different. If the two differ, the write is refused with `The attachment collides with an existing file.` If they are identical, nothing is written and the existing file is reused.

**Deduplication.** Because the path is the digest, two attachments with identical content share one file. Deleting one of them must therefore not delete the file while the other still references it, which is what the deferred deletion of section 7 guarantees.

**Empty content.** The digest of empty content is computed like any other, therefore an empty attachment has a checksum and its response can be validated and cached like any other.

**Marking before writing.** A file is added to the collection checklist *before* it is written, and the marker is left in place. If the transaction that was writing the file aborts, the file exists on disk with no attachment referencing it, and the marker is what makes the next collection remove it. A successful transaction leaves the marker too; the collection then finds the file referenced and only removes the marker.

## 4. Reading content

1. If the stored file name is set, open the sanitized absolute path and read it, optionally only its first bytes when a size limit was given.
2. On any failure of that read, log at informational level `_read_file reading <absolute path>` with the traceback, and return empty content.
3. Otherwise return the database content.

A missing file is deliberately **not** an error: it yields empty content. This keeps a database whose file store has been partially lost usable, and it makes the loss visible as empty documents rather than as failing pages.

## 5. Writing content

### 5.1 The order of operations

On creation:

1. Remove `file_size`, `checksum` and `store_fname` from the supplied values. They are derived and are never accepted from a caller.
2. Normalize the content: take the raw bytes when they were supplied, encoding them when text was given instead of bytes; otherwise decode the text-encoded form; otherwise use empty content. The text-encoded form is removed from the values in every case, so that the inverse of that field is not triggered a second time.
3. Run the content checks of sections 5.2 to 5.4, which set the media type, may replace it with the plain text type, and may replace the content with a reduced image.
4. Compute the derived values of section 5.5 and remove the raw content from the values to be stored.
5. Group the values by the pair of referenced entity and referenced record, and verify that the acting user may write on every distinct referenced record. Otherwise refuse with `Sorry, you are not allowed to access this document.`
6. Create the records.
7. If the strategy is file storage, write each distinct content to the file store, indexed by its digest, so that a batch creating ten attachments with the same content writes one file.
8. Run the served-attachment validation of section 1.

On write:

1. Verify write access on the attachments themselves.
2. When the referenced entity or the referenced record is being changed, build the set of pairs the attachments will have after the write and verify write access on each. Otherwise refuse with the same message.
3. Remove `file_size`, `checksum` and `store_fname` from the supplied values.
4. If the media type or either content field is supplied, run the content checks.
5. Apply the write.
6. If the address or the type changed, run the served-attachment validation.

Writing content through one of the two content fields takes a different path, because the acting user usually may not write the derived columns:

1. For each attachment, compute the derived values from the new content and write them **with elevated rights**.
2. Remember the previous stored file name of each attachment.
3. Collect the distinct new contents by digest.
4. When the strategy is not database storage: flush the checksum and stored file name of the changed attachments **before** touching the file store, mark every previous file for collection, and write every new content.

Flushing first is required. The collection of section 7 must be able to see the new stored file name; without the flush it could conclude that a file which has just become referenced is unreferenced and delete it.

### 5.2 Media type detection

1. Start from the supplied media type.
2. If there is none, guess from the file name.
3. If there is still none, guess from the web address with its query part removed.
4. If there is still none, or if the result is the generic binary type `application/octet-stream`, guess from the content itself, taking the raw bytes when supplied or decoding the text-encoded form.
5. The result is the lowercase form of whatever was found, or the generic binary type when nothing was.

### 5.3 Forcing plain text

Content whose media type is markup-like is forced to plain text unless the acting user may write view definitions.

A media type is markup-like when it contains the two letters `ht`, which covers the markup type and its variants, or when it contains `xml` and does not begin with the office document container prefix `application/vnd.openxmlformats`. The office exception exists because modern office documents are containers whose type names end in a markup marker but which browsers never execute.

The forcing applies when the media type is markup-like **and** either the caller asked for plain markup or the acting user does not have write access on the view definition entity. When it applies, the media type becomes `text/plain`.

This is the defence against stored scripting: an ordinary user who uploads a markup document gets a file that browsers will render as text rather than execute.

### 5.4 Automatic image reduction

Reduction applies when the media type belongs to the image family, its subtype is in the allowed list, and content was supplied. It is skipped entirely when the caller sets the no-post-processing marker in the execution context.

| System parameter | Full name | Default | Meaning |
|---|---|---|---|
| `base.image_autoresize_extensions` | Image automatic resize subtypes | `png,jpeg,bmp,tiff` | The image subtypes that may be reduced, as a comma-separated list. |
| `base.image_autoresize_max_px` | Image automatic resize bounding box | `1920x1920` | The bounding box, as width, the letter `x`, and height. A value that reads as false disables reduction entirely. |
| `base.image_autoresize_quality` | Image automatic resize quality | `80` | The quality applied when re-encoding, used only for the lossy subtype. |

1. If the bounding box reads as false, stop.
2. Decode the image without verifying its resolution.
3. If it decoded to nothing, which happens for empty content, for vector images and for the modern web format, log the line "Post processing ignored : Empty source, SVG, or WEBP" and stop.
4. Take the image width and height, and the bounding width and height from the parameter.
5. If the width exceeds the bounding width or the height exceeds the bounding height, resize the image to fit inside the bounding box, preserving the aspect ratio and never enlarging.
6. Choose the quality: the configured quality when the subtype is the lossy one, and zero otherwise. Zero means that the encoder's own default is used and that a palette is not touched, which is what keeps a palette image lossless.
7. Replace the content with the re-encoded image.
8. On a decoding failure, log `Post processing ignored : <reason>` with the failure text and keep the original content.

**Worked example.** A photograph of 4 000 by 3 000 pixels in the lossy format is uploaded with the default settings.

```formula
bounding width  = 1 920 pixels
bounding height = 1 920 pixels
aspect ratio    = 4 000 ÷ 3 000 = 1.3333
new width       = 1 920 pixels          (the width exceeds the bound)
new height      = 1 920 ÷ 1.3333 = 1 440 pixels
```

The image is reduced to 1 920 by 1 440, re-encoded at quality 80, and the reduced bytes are what is stored, digested and served. The original is not kept. A photograph of 1 600 by 1 200 is stored unchanged, because neither dimension exceeds the bound.

### 5.5 The derived values

```formula
checksum      = content digest of the content
file size     = number of bytes of the content
index content = the extractable text of section 5.6
```

Then, when the content is non-empty and the strategy is not database storage, the stored file name is the relative path of section 3 and the database content is emptied. Otherwise the stored file name is emptied and the database content is the content.

### 5.6 Text extraction

Only content whose media type begins with `text/` is indexed. The extraction finds every run of four or more consecutive printable characters in the byte range 32 to 126 and joins those runs with line breaks. Every other media type yields no indexed content at all, which is why searching the text of a Portable Document Format document finds nothing unless a capability package adds an extractor.

### 5.7 Duplication

Duplicating an attachment copies its content unless the caller supplies content explicitly. The raw content of the source is placed in the values of the copy, which makes the write path recompute the checksum and the stored file name. The copy therefore shares the same file in the file store, because the digest is the same.

### 5.8 Creating only when absent

An operation creates an attachment only when no attachment with the same digest, the same size and the same media type already exists. For each supplied value set:

1. Decode the text-encoded content. A malformed encoding refuses with `Attachment is not encoded in base64.`
2. Compute the digest of the decoded content.
3. Search, with elevated rights, for an attachment whose key is set, which is the condition that disables the implicit field-content filter of section 6.2, and whose checksum, file size and media type all match.
4. If any exist, return their keys. Otherwise create one and return its key.

The search is not restricted to a referenced record, so this operation deduplicates across the whole database.

## 6. Access

### 6.1 The rule set

Access to an attachment is decided in the following order. Each step applies only to the attachments not already refused by an earlier step.

1. Apply the ordinary permissions and record rules of the attachment entity itself. Attachments already refused there are refused, and if every attachment of the set is refused the decision is final.
2. Creating and deleting an attachment are checked as **writing**, both on the attachment and on the referenced record. Attaching a document to a record is therefore permitted exactly to those who may change that record.
3. For reading, a public attachment is always allowed, whatever the rest of the rules say.
4. If the acting user is not a settings user:
   1. An attachment with no referenced record is allowed only to the user who created it.
   2. An attachment that is the content of a field is allowed only when the acting user has the required access to that field of that entity. A field that no longer exists on the entity denies access.
5. For an attachment that references a record, the required access on the referenced record decides.
6. A referenced entity that is not in the registry denies access, because the permissions to apply cannot be determined.
7. An attachment that references the acting user's own user record is exempt from the restriction that a user may not write on their own user record. This is what lets a user upload their own signature image.

The refusal message is:

```
Sorry, you are not allowed to access this document. Please contact your system administrator.

(Operation: <operation>)

Records: <up to six attachments>, User: <acting user key>
```

The placeholders are the operation, at most six of the refused attachments and the key of the acting user. The refused attachments have their security fields invalidated before the message is built, so that the failed check does not leave them in the cache.

### 6.2 Searching

Two filters are applied to every search, unless the caller opts out.

| Filter | Rule |
|---|---|
| Field-content filter | Unless the condition already constrains the key or the field name, or the caller set the skip marker in the execution context, the condition that the field name is unset is added. Attachments that are the content of a binary field are therefore invisible to ordinary searches. |
| Security filter | Built as the union of: public attachments; attachments with no referenced record that the acting user created, or any such attachment when the acting user is a settings user; and, for each distinct referenced entity named in the condition, the attachments whose referenced record the acting user may read. |

The security filter is built in the precise form above only when the condition names between one and five distinct referenced entities. For each of them, the sub-condition additionally restricts the field name, when the field-content filter was not applied and the acting user is not a settings user, to the empty value together with the names of the fields of that entity that the acting user may read and that are either binary or relational fields pointing back at the attachment entity.

When the condition does not restrict the referenced entity to at most five entities, the search degrades to a filtered scan. The condition is widened to the security filter or any attachment with a referenced entity, and then:

- with no limit, every candidate is read with elevated rights, restricted to the security fields, and the access rule of section 6.1 filters the result;
- with a limit, the candidates are read in batches whose size is the prefetch maximum, each batch is filtered, and the scan stops when enough keys have accumulated or when a batch comes back short.

The batch order, when the caller asked for no particular order, is by referenced entity with the empty value first, then by key. Grouping the candidates by entity is what makes the access checks cheap, because one permission evaluation then serves a whole batch.

### 6.3 Access tokens

| Token | Shape | Grants |
|---|---|---|
| Attachment access token | A random universally unique value stored on the attachment | Read access to that attachment's content, compared in constant time. Generated on demand; an attachment that already has one keeps it, so that addresses already handed out keep working. |
| Scoped field access token | The keyed digest, then the letter `o`, then the expiry in hexadecimal | Read access to one field of one record within one named scope. The attachment entity uses the scope `binary` for its raw content field. |

The keyed digest of a scoped token is computed over the entity name, the record key, the field name and the expiry, with the platform secret.

The expiry, when it was not supplied, is deterministic within a fourteen-day period, which is what lets a browser cache the address instead of requesting a new token on every page.

```formula
period        = 1 209 600 seconds (14 days)
start         = current time in seconds ÷ period, truncated, × period
spread        = period × checksum of the entity name, record key and field name ÷ 4 294 967 295
expiry        = start + 2 × period + spread
```

The spread is at least zero and at most one period, so the expiry is at least 28 days and at most 42 days after the start of the current period, which is at least 14 and at most 42 days in the future. Deriving the spread from the record identity prevents every token of a deployment from expiring at the same instant.

**Worked example.** The current time is 1 700 000 000 seconds. The period is 1 209 600 seconds. The start of the period is 1 700 000 000 ÷ 1 209 600 = 1 405.42, truncated to 1 405, multiplied by 1 209 600 = 1 699 488 000. Suppose the checksum of the record identity is 2 147 483 647, which is very close to half of 4 294 967 295, so the spread is 1 209 600 × 2 147 483 647 ÷ 4 294 967 295 = 604 800 seconds. The expiry is 1 699 488 000 + 2 419 200 + 604 800 = 1 702 512 000, which is 2 512 000 seconds, about 29 days, after the current time.

Verification recomputes the digest from the expiry embedded in the token, compares it in constant time, and additionally requires the expiry to be in the future.

### 6.4 Content-return exemptions

Independently of the rules above, an entity may declare that the content of one of its fields may be returned even to a user who could not read the record. The platform's default answer is no. The attachment entity answers yes when:

- a valid attachment access token was presented; an invalid one raises `Invalid access token`;
- or the attachment is public;
- or the acting user is a portal user **and** passes an ordinary read check on the attachment, which is what lets a customer download a document from a portal page.

The record resolution used by the content and image paths applies these rules in order: resolve the record by external identifier or by the pair of entity and key, raising `No record found for xmlid=<external identifier>, res_model=<entity>, id=<key>` when nothing matches; then, if a scoped field token verifies, act with elevated rights; then, if the content-return exemption allows it, act with elevated rights; otherwise perform an ordinary read check.

## 7. Deletion and the file store collection

### 7.1 Deferred deletion

Deleting an attachment, or replacing its content, never deletes a file immediately.

1. Collect the stored file names that are about to become unreferenced.
2. Delete the records, or write the new content, **first**.
3. For each collected name, add an empty marker file at the checklist directory of the file store, under the sanitized name.

Deleting the rows before touching the file system is deliberate. When two transactions delete the same attachment and the database aborts one of them, the aborted one must not have removed the file that the surviving one may still reference.

### 7.2 The collection

The collection runs inside the automatic cleanup job of [`scheduled-jobs.md`](scheduled-jobs.md), section 11. It does nothing unless the strategy is file storage.

1. Commit the current transaction. The lock of step 3 must be the first statement of its transaction: a snapshot taken before it would not contain the attachments that concurrent transactions are creating, and the collection would then treat their files as unreferenced.
2. Set a lock timeout of 10 seconds on this transaction.
3. Take a share-mode lock on the attachment table. If it cannot be obtained within the timeout, roll back and give up for this run; the next run will try again.
4. Walk the checklist directory and build the map from relative name to marker path. The relative name is rebuilt from the last directory component and the file name, which is what makes it match a stored file name.
5. For each chunk of names, sized by the maximum number of values one query may carry, read which of those names are still referenced by some attachment.
6. For each name in the chunk that is not referenced, delete the file and count it; log at debug level `_file_gc unlinked <absolute path>` on success, and at informational level `_file_gc could not unlink <absolute path>` with the traceback on failure. In both cases delete the marker.
7. Log `filestore gc <checked> checked, <removed> removed`, in which the placeholders are the number of names in the checklist and the number of files deleted.
8. Commit, which releases the lock.

The share-mode lock blocks concurrent writes to the attachment table for the duration of the collection but allows concurrent reads. Combined with the rule of section 5.1 that flushes before touching the file store, it guarantees that a file is deleted only when no attachment references it and none is about to.

## 8. Image variants

### 8.1 Deriving the requested size

When the caller supplies neither a width nor a height, the size is guessed from the field name.

| Field name | Guessed size |
|---|---|
| Exactly `image` | 1 024 by 1 024 |
| Beginning with the custom-field prefix `x_` | no resizing |
| Ending with an underscore and a number that is at least 16 | that number, squared |
| Ending with an underscore and a number below 16 | no resizing, because such a suffix is not a size |
| Anything else | no resizing |

### 8.2 The pipeline

1. Build a stream from the record's field, as specified in section 9.2. A refusal is swallowed, unless the caller marked the request as a download of attachments, in which case it propagates.
2. If no stream could be built, or its size is zero, use the placeholder: the path the entity declares for that field, or, when it declares none, the default placeholder image. The placeholder is read as a package resource restricted to the two raster image extensions.
3. If the source kind is remote, return the stream unchanged. A remote image is not resized.
4. If the media type does not begin with the image family, replace it with the generic binary type. This prevents a non-image from being served with an image type after the pipeline decided not to transform it.
5. If neither width nor height was given, guess them as in section 8.1.
6. Extend the validator: `<validator>-<width>x<height>-crop=<crop>-quality=<quality>`, in which the placeholders are the original validator, the two dimensions, the crop setting and the quality. The extension applies only when the validator is a text value.
7. Convert the last-modified value to an instant when it is a numeric timestamp, so that the conditional comparison has a date to work with.
8. Evaluate the conditional request against the extended validator and that instant.
9. If the resource is considered modified **and** at least one of width, height or crop is set: read the file into memory when the source was a path, transform the bytes, and set the size to the transformed length.
10. Produce the response as in section 9.3.

Extending the validator with the variant parameters is what lets one original produce many independently cacheable variants: the browser and any shared cache treat the 128-pixel variant and the 256-pixel variant as different resources with different validators, both derived from the same content digest, so a change of the original invalidates every variant at once.

### 8.3 The transformation

| Parameter | Effect |
|---|---|
| Size | The bounding box, as a width and a height. A zero dimension means unconstrained in that direction. |
| Crop | When false, the image is resized to fit the box, preserving the aspect ratio, and is never enlarged unless expansion was requested. When true, the image is cropped to the box's aspect ratio and then resized. The crop is centred; the values `top` and `bottom` move the vertical centre to 0 and to 1 respectively. |
| Quality | When non-zero, the re-encoding quality. When zero, the encoder's own default. |
| Output format | When given, the encoding to produce. |
| Padding | When given, transparent padding added around the image. |
| Colorize | When given, replaces the transparent parts with a colour: the one supplied, or the dominant colour of the image when none was. |

The order of operations is fixed: crop or resize first, then padding, then colorizing, then the re-encoding that applies the quality and the output format.

A request with no size, no quality, no crop, no colorizing, no output format, no padding and no resolution verification returns the original bytes untouched: the pipeline short-circuits before decoding anything, which is what makes an unparameterised request cheap.

## 9. Streaming

### 9.1 The stream object

A stream describes a response body that has not been produced yet. It has exactly one of three sources and a set of response attributes.

| Attribute | Meaning |
|---|---|
| Source kind | `data` for bytes in memory, `path` for a file on disk, `url` for a remote address. |
| Media type | The media type to declare. |
| Download name | The file name to propose. |
| As attachment | Whether the browser should save rather than display. Default false. |
| Conditional | Whether to honour conditional requests. Default true, and true in every path the platform builds. |
| Validator | The entity tag. |
| Last modified | The modification instant. |
| Maximum age | The cache lifetime in seconds. |
| Immutable | Whether to declare the content immutable, which also forces a one-year lifetime. Default false. |
| Size | The content length. |
| Public | Whether shared caches may store the response. Default false. |

Producing a response asserts that the source kind is one of the three and that the corresponding source attribute is set; a stream missing either fails as a programming error rather than serving an empty body.

### 9.2 Building a stream

| Built from | Source kind | Validator | Last modified | Public |
|---|---|---|---|---|
| A package resource | `path` | `<modification instant as a whole number>-<size>-<checksum of the path>` | the file's modification instant | as the caller requested |
| An attachment with a stored file | `path`, joined safely under the file store root | the attachment checksum | the file's modification instant | the attachment's public flag |
| An attachment with database content | `data` | the attachment checksum | the attachment's update instant | the attachment's public flag |
| An attachment of the type `url` whose address resolves to a package resource | as for a package resource | as for a package resource | as for a package resource | true |
| An attachment of the type `url` otherwise | `url` | the attachment checksum | none | the attachment's public flag |
| An attachment with no content at all | `data`, empty, size 0 | the attachment checksum | none | the attachment's public flag |
| A binary field of a record | `data`, after decoding the text-encoded form when it decodes strictly | the digest of the decoded bytes | the record's update instant when the entity keeps one | true when the acting user is the shared public user |
| A binary field declared as attachment-backed | the attachment holding it, found by the triple of entity, record and field name | as for that attachment | as for that attachment | as for that attachment |

A binary field whose stored value is text-encoded is decoded before streaming, after removing the line breaks that some encoders insert; a value that does not decode strictly is streamed as it stands. A binary field declared as attachment-backed whose attachment is missing raises `The related attachment does not exist.`

Building a stream from a field first refuses two programming errors: a record set that is not of length one, with `Expected singleton: <record set>`, and a field name that does not exist on the entity, with `Record has no field '<name>'.` A field that exists but is not binary is refused with `Field <field> is type '<type>' but it is only possible to stream Binary or Image fields.`

When the built stream carries no media type, it is guessed from the first bytes of the content, falling back to the generic binary type, or to the image type when the caller is the image path.

Reading a stream whose source kind is `url` is a programming error and is refused, because there are no local bytes to read.

An attachment counts as a remote source when it has an address, no file size, and the address begins with one of the three transport prefixes `http://`, `https://` or `ftp://`.

### 9.3 Producing the response

For the source kind `url`:

- If a maximum age is set, answer with a temporary redirect to the address and declare that maximum age.
- Otherwise answer with a permanent redirect to the address.

For the other two source kinds:

1. Build the response from the bytes or from the file, honouring conditional requests against the validator and the modification instant.
2. Declare the media type, the download name and the disposition, the disposition being the saving form when the stream asks for it.
3. Set the cache lifetime to one year when the response is immutable, and to the stream's own maximum age otherwise.

Then, for every source kind other than the redirect:

4. Declare that content-type sniffing is disabled.
5. Declare a content security policy. The default is the most restrictive one, which forbids the resource from issuing any request of its own; a caller may pass a different one, or none.
6. If the stream is public and the resulting maximum age is greater than zero, allow shared caches. Otherwise remove any shared-cache directive and mark the response private.
7. If the response is immutable, declare the content immutable.

A conditional request whose validator matches produces the not-modified status with no body, and no transformation and no file read are performed.

The disposition header is built with the encoded file name, which transports names containing characters outside the plain alphabet, quotation marks or separators correctly.

**Offloading the body to the front end.** When the deployment enables offloading and the file lies inside the file store, the response declares the file's path relative to the file store, prefixed by the fixed internal location `/web/filestore/`, and sets the content length to zero. The body is then produced by the component in front. Zeroing the content length explicitly is required: without it the front end waits for a body that never arrives.

### 9.4 Download names

1. Take the explicitly supplied name.
2. Otherwise take the value of the designated name field of the record, when that field name contains `name` or the acting user may read it.
3. Otherwise build the name from the physical table name of the entity, a hyphen, the record key, a hyphen and the field name.
4. Replace every carriage return and every line break in the name with an underscore.
5. Take the extension of the name, keep the first 100 characters of the name without its extension, and append the extension again.
6. If the name still has no extension and the media type is not the generic binary type, append the extension that corresponds to the media type.

Step 4 exists because a name containing a line break would let a caller inject a second header.

## 10. Uploading

An uploaded file becomes an attachment through one of three media-type policies.

| Policy | Media type used | File name used |
|---|---|---|
| Trust the client | The media type the client declared. | The name the client sent, unchanged. |
| Guess | The type detected from the first bytes of the content. When that yields the archive type or one of the compound-document types, the type is then re-derived from the corrected file name, because those two container types are shared by many office formats. | The client name with its extension corrected to match the detected type, unless it already had a valid one. |
| Force a given type | The given type, which must have both a family and a subtype. | The client name with its extension corrected to match, unless it already had a valid one. |

Any other value is a programming error and is refused as such.

The whole file is read into memory and passed as raw content. The write path of section 5 then applies in full, including the plain-text forcing and the automatic image reduction.

After creation, an extension point runs. The discussion capability package uses it to register the new attachment as the main attachment of the record it is attached to, without displacing an existing main attachment.

**Ownership tokens.** The discussion capability package defines a scoped token whose scope is attachment ownership. A caller holding it may act on the attachment as its owner even without write access. An operation that takes a list of attachments and a list of tokens requires one token position per attachment; a mismatch refuses with `An access token must be provided for each attachment.`

## 11. Attachments that serve a request path

An attachment whose type is `binary` and whose web address equals a request path is served by the request fallback when no endpoint matched that path, as specified in [`request-lifecycle.md`](request-lifecycle.md), section 8. The lookup condition is: the type is `binary` and the address equals the path exactly, optionally narrowed by an extra condition the caller supplies, ordered as the caller asks, and the first match wins.

The attachment must have content, either in the file store or in the database; otherwise the fallback declines and the request becomes a not-found.

Only members of the groups declared for served attachments, which is the settings administration group as shipped, may create or modify such an attachment. That restriction is the validation of section 1.

## 12. Attachments backing asset bundles

Compiled asset bundles are stored as attachments with all five of the following properties: the public flag set, an address beginning with `/web/assets/`, the referenced entity set to the view entity, the referenced record key set to 0, and the creator set to the superuser.

Regenerating the bundles deletes exactly the attachments matching all five conditions and then clears the asset cache container of [`caching.md`](caching.md), section 10. The five conditions together are what prevents the regeneration from deleting an attachment that merely happens to sit at a similar address.

## 13. Notification of changes

An attachment declares the acting user's own user record as its notification channel. A message about an attachment therefore reaches the user who is acting, not every follower of the record the attachment belongs to. This is what keeps an upload from notifying a whole thread.

## 14. Worked examples

### 14.1 Two records sharing one file

| Step | Effect |
|---|---|
| 1 | A user attaches a document of 1 200 000 bytes to a sales order. Its digest is `ab12…`; the name `ab/ab12…` is added to the checklist; the file is written at that path; the attachment stores that relative path, the size 1 200 000 and the digest. |
| 2 | Another user attaches the identical document to an invoice. The digest is the same. The existing file is found; its content is compared block by block and matches; nothing is written. A second attachment record stores the same relative path. |
| 3 | The first attachment is deleted. The row is deleted first, then a marker is placed in the checklist for `ab/ab12…`. |
| 4 | The collection runs, takes the table lock, reads that `ab/ab12…` is still referenced by the second attachment, therefore does not delete the file, and removes the marker. |
| 5 | The second attachment is deleted, a marker is placed again, the collection finds no reference, deletes the file, and removes the marker. The log line reads `filestore gc 1 checked, 1 removed`. |

### 14.2 A digest collision

Two different documents produce the same digest.

| Step | Effect |
|---|---|
| 1 | The first is stored at `cd/cd34…`. |
| 2 | The second is written. The path already exists, so the block comparison runs and finds a difference. |
| 3 | The write is refused with `The attachment collides with an existing file.` and the whole transaction aborts, therefore no attachment record is created either. |

### 14.3 A public logotype served through a shared cache

| Step | Effect |
|---|---|
| 1 | The logotype is requested at a path that carries the record key and a content fingerprint, with a requested size of 128 by 128. |
| 2 | The record is resolved. Its content-return exemption allows the read because the attachment is public, so the resolution acts with elevated rights. |
| 3 | The stream is built from the file store; its validator is the checksum `ef56…`; it is extended to `ef56…-128x128-crop=False-quality=0`. |
| 4 | The client sent no matching validator, therefore the image is read into memory, reduced to fit 128 by 128, and re-encoded. |
| 5 | The response declares that validator, a maximum age of 31 536 000 seconds with the immutable directive because the caller asked for the fingerprinted form, the shared-cache directive because the attachment is public, the sniffing-protection header and the restrictive content policy. |
| 6 | On the next request the client sends the validator. The comparison at step 8 of the pipeline matches, therefore the response is the not-modified status, and neither the file read nor the transformation is performed. |

### 14.4 A markup document uploaded by an ordinary user

| Step | Effect |
|---|---|
| 1 | The file name ends in a markup extension, so the detected media type is the markup type. |
| 2 | The acting user may not write view definitions, therefore the media type is replaced by `text/plain`. |
| 3 | The content is unchanged; the digest, the size and the indexed text are computed from it. Because the media type now begins with the text family, the indexed text is extracted, so the document becomes searchable. |
| 4 | Downloading it later declares the plain text type and the restrictive content policy, therefore the browser displays it as text and it cannot issue any request of its own. |

### 14.5 A batch creating ten identical attachments

Ten records each receive the same 500 000-byte document in one operation.

| Step | Effect |
|---|---|
| 1 | The content checks run ten times and produce the same media type. |
| 2 | The derived values are computed ten times and produce the same digest, so the map from digest to content holds exactly one entry. |
| 3 | Write access is checked once per distinct referenced record, that is ten times. |
| 4 | Ten rows are created, each with the same stored file name. |
| 5 | One file is written, because the map had one entry. The checklist gains one marker. |
| 6 | The next collection finds the name referenced ten times over and removes only the marker. |

## 15. Acceptance criteria

1. **Given** the file storage strategy, **when** an attachment with content is created, **then** its stored file name is the two-character prefix of its digest, a separator and its digest, its database content is empty, and the file exists at the sanitized absolute path.
2. **Given** the database storage strategy, **when** an attachment with content is created, **then** its stored file name is empty and its database content holds the bytes.
3. **Given** two attachments with identical content under the file storage strategy, **when** both exist, **then** exactly one file exists in the file store.
4. **Given** two attachments sharing a file, **when** one is deleted and the collection runs, **then** the file still exists and the checklist marker is gone.
5. **Given** an attachment whose file is no longer referenced, **when** the collection runs, **then** the file is deleted and the run is logged with the checked and removed counts.
6. **Given** a collection that cannot take the table lock within 10 seconds, **when** the timeout elapses, **then** the transaction is rolled back and nothing is deleted in that run.
7. **Given** content whose digest matches an existing file with different bytes, **when** it is written, **then** the operation is refused with `The attachment collides with an existing file.` and no record is created.
8. **Given** a transaction that writes a file and then aborts, **when** the next collection runs, **then** the orphaned file is removed, because it had been added to the checklist before it was written.
9. **Given** an attachment whose file has been removed from the file store, **when** its content is read, **then** empty content is returned and the failure is logged at informational level.
10. **Given** an attachment read under the size-only mode, **when** its text-encoded content is read, **then** a human-readable size is returned and the file store is not touched.
11. **Given** an image of 4 000 by 3 000 pixels in the lossy format with the default settings, **when** it is uploaded, **then** the stored content is 1 920 by 1 440 pixels re-encoded at quality 80.
12. **Given** an image of 1 600 by 1 200 pixels, **when** it is uploaded with the default settings, **then** the original bytes are stored unchanged.
13. **Given** the same 4 000 by 3 000 image and a bounding-box parameter that reads as false, **when** it is uploaded, **then** the original bytes are stored unchanged.
14. **Given** a vector image or one in the modern web format, **when** it is uploaded, **then** it is stored unchanged and the line "Post processing ignored : Empty source, SVG, or WEBP" is logged.
15. **Given** a palette image above the bounding box, **when** it is reduced, **then** the quality is zero, so the encoder's default is used and the palette is not altered.
16. **Given** a markup document uploaded by a user who may not write view definitions, **when** it is stored, **then** its media type is `text/plain`.
17. **Given** the same document uploaded by a user who may write view definitions, **when** it is stored, **then** its media type is the markup type.
18. **Given** an office document whose type begins with the office container prefix, **when** it is uploaded by a user who may not write view definitions, **then** its media type is unchanged.
19. **Given** an attachment of the type `binary` with a web address, **when** a user who is not in the served-attachment groups creates or writes it, **then** the operation is refused with `Sorry, you are not allowed to write on this document`.
20. **Given** an attachment whose referenced record the acting user may not read, **when** they read the attachment, **then** access is refused, unless the attachment is public.
21. **Given** an attachment with no referenced record created by another user, **when** a user who is not a settings user reads it, **then** access is refused.
22. **Given** an attachment that is the content of a field the acting user may not read, **when** they read the attachment, **then** access is refused.
23. **Given** an attachment referencing a record the acting user may read but not write, **when** they try to delete the attachment, **then** it is refused, because deleting is checked as writing.
24. **Given** an attachment on the acting user's own user record, **when** they write it, **then** it is allowed despite the rule that a user may not write on their own user record.
25. **Given** an attachment whose referenced entity is no longer in the registry, **when** it is read, **then** access is refused.
26. **Given** an ordinary search on attachments, **when** the condition mentions neither the key nor the field name, **then** attachments that are field content are excluded from the result.
27. **Given** a search whose condition names six distinct referenced entities, **when** it runs, **then** the filtered scan is used and the result is the same set of keys the rule-by-rule check would give.
28. **Given** a valid attachment access token, **when** the content is requested with it, **then** it is served regardless of the acting user's permissions; **given** an invalid one, **then** `Invalid access token` is raised.
29. **Given** a scoped field access token whose embedded expiry has passed, **when** it is verified, **then** it is rejected even though the digest matches.
30. **Given** two calls for a scoped token on the same record and field within one fourteen-day period, **when** both are generated, **then** they are identical.
31. **Given** an attachment request with a matching validator, **when** it is served, **then** the response has the not-modified status and no body.
32. **Given** a non-public attachment, **when** it is served, **then** the response is marked private and carries no shared-cache directive.
33. **Given** a public attachment served with a maximum age of zero, **when** it is served, **then** it is still marked private, because the shared-cache directive requires a positive lifetime.
34. **Given** a request for an image variant of 128 by 128 on a record whose field is empty, **when** it is served, **then** the entity's declared placeholder image, or the default placeholder, is served instead.
35. **Given** an image request with no width and no height on a field named with the numeric suffix 128, **when** it is served, **then** the variant is 128 by 128.
36. **Given** an image request with no width and no height on a field named with the numeric suffix 8, **when** it is served, **then** no resizing is performed.
37. **Given** an image request with no width and no height on a field named exactly `image`, **when** it is served, **then** the variant is 1 024 by 1 024.
38. **Given** an image request on a field whose name begins with the custom-field prefix, **when** it is served, **then** no resizing is performed.
39. **Given** a request for an image variant with no size, no crop, no quality and no format, **when** it is served, **then** the original bytes are returned untouched.
40. **Given** an attachment of the type `url` pointing at a remote address, **when** it is streamed with no maximum age, **then** the response is a permanent redirect to that address; **given** a maximum age, **then** it is a temporary redirect declaring that lifetime.
41. **Given** an attachment of the type `url` whose address resolves to a package resource, **when** it is streamed, **then** the file is served directly and the response is marked as cacheable by shared caches.
42. **Given** a download name of 150 characters plus an extension, **when** it is proposed, **then** the name is truncated to its first 100 characters and the extension is re-appended.
43. **Given** a download name containing a line break, **when** it is proposed, **then** the line break is replaced by an underscore.
44. **Given** a binary field declared as attachment-backed whose attachment is missing, **when** it is streamed, **then** `The related attachment does not exist.` is raised.
45. **Given** the migration operation invoked by a user who is not an administrator, **when** it is called, **then** it is refused with `Only administrators can execute this action.`
46. **Given** the migration operation under the file storage strategy, **when** it runs, **then** every attachment that had database content ends with a stored file name and empty database content, and its checksum is unchanged.
47. **Given** the migration operation, **when** it selects attachments, **then** attachments that are the content of a field are included.
48. **Given** two value sets with identical content, size and media type, **when** the create-only-when-absent operation runs on both, **then** one attachment exists and both calls return its key.
49. **Given** a value set whose text-encoded content is malformed, **when** the create-only-when-absent operation runs, **then** it is refused with `Attachment is not encoded in base64.`
50. **Given** an attachment being duplicated with no content supplied, **when** the copy is created, **then** it has the same digest, the same stored file name and therefore shares the same file.
51. **Given** an attachment whose referenced entity and record are changed to a record the acting user may not write, **when** the change is saved, **then** it is refused with `Sorry, you are not allowed to access this document.`
52. **Given** a caller that supplies a checksum, a file size or a stored file name, **when** the attachment is created or written, **then** those values are discarded and the derived ones are used.
53. **Given** the asset bundle regeneration, **when** it runs, **then** exactly the public attachments at an address beginning with `/web/assets/`, referencing the view entity at record key 0 and created by the superuser, are deleted, and the asset cache container is cleared.
54. **Given** an attachment of the type `url`, **when** the operation that migrates a remote attachment to local storage is called on it, **then** it is refused; **given** one of the type `binary`, **then** the call does nothing.

## 16. Reconciliation notes

1. The field identifiers were paraphrased in the source document. They are contractual and are reproduced exactly here: `res_model`, `res_id`, `res_field`, `res_name`, `url`, `raw`, `datas`, `db_datas`, `store_fname`, `file_size`, `checksum`, `mimetype`, `index_content`, `access_token`, `public`, `type`, `company_id`, `name` and `description`.
2. The storage-strategy parameter was paraphrased. Its reproduced name is `ir_attachment.location`, and the three image parameters are `base.image_autoresize_extensions`, `base.image_autoresize_max_px` and `base.image_autoresize_quality`.
3. The source document did not state that a file is added to the collection checklist **before** it is written. That ordering is what makes an aborted transaction leave a collectable file rather than a permanent orphan, and it is specified in sections 3 and 5.1 and asserted by acceptance criterion 8.
4. The source document treated creating and deleting an attachment as separate permissions. Both are evaluated as writing, on the attachment and on the referenced record. The rule set of section 6.1 states this once, at step 2.
5. The source document described the degraded search as a fixed batch scan of 1 000. The batch size is the prefetch maximum, and the unlimited form reads every candidate in one pass; both variants are specified in section 6.2.
6. The three constructor rows for an attachment of the type `url` disagreed on the validator and the public flag. The stream is built with the checksum as validator and the attachment's own public flag in every case, and only the source kind and the last-modified value differ; the corrected table is in section 9.2.
7. The endpoint catalogue the source document linked to does not exist in this repository. The request paths are described where they are used, and the request-side behaviour is in [`request-lifecycle.md`](request-lifecycle.md).
8. The cache-control behaviour of a stream was described in two places. It is specified once here, in section 9.3, and the catalogue of cache directives per resource kind in [`caching.md`](caching.md), section 14, points at it.
