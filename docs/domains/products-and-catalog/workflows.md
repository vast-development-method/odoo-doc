# Products and Catalog — Workflows

End-to-end operational sequences, step by step, naming the role that performs each step, the
precondition of each step, and the records created or updated. Every algorithm invoked here is
specified in [calculations.md](calculations.md); every validation is specified in
[business-rules.md](business-rules.md).

Roles used throughout:

| Role | Held by | What it can do in this domain |
|---|---|---|
| **Internal user** | Every employee account | Read every catalog entity; run the label wizard |
| **Product manager** | Holders of the Products — Create privilege | Create, change and delete every catalog entity |
| **System administrator** | Holders of the system administration privilege | Everything a product manager can do, plus barcode nomenclatures and rules, system parameters and precisions |
| **Salesperson, buyer, warehouse operator** | Roles owned by other domains | Consume the catalog: configure a product, scan a barcode, add from the catalog grid |
| **Scheduled work runner** | The system itself | Runs the expiry reminder |

---

## 1. Creating a simple product

**Actor** product manager. **Precondition** none.

1. The product manager opens the product list and starts a new record.
2. The form is pre-filled: the type is `consu` ("Goods"), the sales price is 1.0, the sellable flag
   and the purchasable flag are true, the unit is the shipped unit "Units", the create-on-order
   field is `no`.
3. The manager types a name, optionally a category, a sales price, a cost, descriptions, an image
   and tags.
4. On save:
   1. The template row is written.
   2. Variant generation runs. The template has no attribute lines, so the cartesian product over
      an empty list of factors yields exactly one element — the empty combination — which passes the
      configuration filter (zero lines, zero members). One variant is created, with the template's
      activity flag and an empty combination.
   3. The mirrored fields are reconciled: for each of barcode, internal reference, cost, volume,
      weight and properties, if the manager supplied a value and the template's computed value came
      back empty — which it does, because the computation ran before the variant existed — the value
      is written again, and this time it reaches the variant.
5. The manager sees one product with one variant.

**Records created** one Product Template, one Product Variant.

---

## 2. Turning a simple product into a product with variants

**Actor** product manager. **Precondition** the template exists and its type is not `combo`.

1. The manager opens the product's attributes section and adds a line, choosing an attribute.
2. **If the chosen attribute never creates variants**, every value of that attribute is
   pre-selected automatically. Otherwise the current selection is filtered down to values of the
   chosen attribute.
3. The manager selects the values the product offers and saves.
4. On save, for each new line:
   1. The creation path searches for an **archived** line with the same template and the same
      attribute. If one exists it is revived and written with the new values, with materialisation
      suppressed; otherwise a new line is created.
   2. Value materialisation runs over every line of the batch (see
      [calculations.md](calculations.md), section 2), producing one Template Attribute Value per
      offered value, each with its extra price copied from the attribute value's default extra
      price.
   3. Variant generation runs (see [calculations.md](calculations.md), section 3).
5. The manager sees the generated variants in the variant list and may open each to set a barcode,
   an internal reference, a cost, a weight and a variant image.

**Records created** one Template Attribute Line per attribute, one Template Attribute Value per
offered value, one Product Variant per surviving combination. **Records deleted** the original
attribute-less variant, if nothing referenced it; otherwise it is archived.

### 2.1 Setting per-value extra prices

**Actor** product manager.

1. From the attribute line, the manager opens the value list.
2. For each Template Attribute Value, the manager types an extra price in the template's currency.
3. Saving writes the extra price. No variant is regenerated: the extra price does not affect which
   combinations exist, only their price.
4. Every variant's price extra and sales price are recomputed on the next read.

### 2.2 Pushing a default extra price to every product

**Actor** product manager. **Precondition** the attribute value has a default extra price and the
system shows that at least one materialised value differs from it.

1. From the attribute value, the manager runs the "Update product extra prices" action.
2. A dialog appears with the message "You are about to update the extra price of *the count*
   products.", where the count is the number of templates having an attribute line offering this
   value.
3. On confirmation, every Template Attribute Value of this attribute value, restricted to templates
   of the acting companies, has its extra price set to the attribute value's default extra price.

### 2.3 Adding a value to every product that uses the attribute

**Actor** product manager.

1. From the attribute value, the manager runs the "Add to all products" action.
2. A dialog appears with the message "You are about to add the value \"*the value name*\" to *the
   count* products.", where the count is the number of templates having an attribute line for this
   value's attribute.
3. On confirmation, the value is linked into every such attribute line, restricted to templates of
   the acting companies. Each line's write triggers value materialisation and then variant
   generation, so variants appear across the catalogue in one operation.

**Caution** this operation can multiply the variant count of many templates at once and may trip the
generation ceiling on any of them, which aborts the whole operation.

---

## 3. Excluding a combination

**Actor** product manager. **Precondition** the template has at least two attribute lines with
materialised values.

1. The manager opens one Template Attribute Value and adds an exclusion.
2. The exclusion names this template (it is prefilled) and a list of the template's other active
   materialised values.
3. On save, the exclusion is created and variant generation runs for the template.
4. The combinations containing both the owning value and any excluded value disappear: their
   variants are deleted if unreferenced, archived otherwise.
5. If the exclusion set would leave no possible variant at all, the last variant's deletion deletes
   the template, and the operation is refused with "This configuration of product attributes,
   values, and exclusions would lead to no possible variant. Please archive or delete your product
   directly if intended."

**Removing an exclusion** deletes the record and re-runs generation, which recreates or reactivates
the combinations that are now possible again.

**Changing an exclusion's template** re-runs generation for **both** the old and the new template.

See [calculations.md](calculations.md), section 3.3 for the worked example.

---

## 4. Configuring a product with attributes on an order

**Actor** salesperson or storefront visitor. **Precondition** the template is active and
configurable.

1. The consuming domain asks the template for the single-variant shortcut. If the template has
   exactly one variant **and** is not configurable, the shortcut returns that variant's identifier
   and display name, and the configurator is skipped entirely.
2. Otherwise the configurator opens. It receives:
   - the template's valid attribute lines, each with its attribute, its display type and its active
     materialised values with their extra prices;
   - the attribute-exclusion payload (see [calculations.md](calculations.md), section 4.6);
   - the first possible combination, used as the initial selection.
3. As the user changes a value, the client re-evaluates which values are still selectable using the
   exclusion payload, the archived-combination list and the parent-exclusion map. A value that is
   unavailable is shown with the reason built from the mapped attribute names, for example
   "Not available with Color: Black" or "Not available with Customizable Desk (Legs: Steel)".
4. For each value flagged as free text, the user may type a string.
5. On confirmation the consuming domain calls the create-on-demand procedure with the **complete**
   combination, including the values of never-create-variant attributes.
   - If a variant exists, it is returned.
   - If none exists and the template has a dynamic attribute and the combination is possible, a
     variant is created.
   - Otherwise nothing is returned and the order line cannot be created.
6. The consuming domain creates the order line with:
   - the returned variant;
   - the subset of the chosen combination whose attributes never create variants, stored on the line
     so that the extra price can be applied and the choice can be printed;
   - one Attribute Custom Value per free-text value, holding the Template Attribute Value and the
     typed string.
7. The line's unit price is computed as in [calculations.md](calculations.md), section 7.

### 4.1 Repairing an impossible pre-filled combination

When a link or an import supplies a combination that is no longer possible, the consuming domain
asks for the **closest possible combination** (see [calculations.md](calculations.md), section 6.3).
The algorithm keeps as many of the requested values as it can, dropping them from the end — that is,
from the attribute that appears last in the configurator — until a completion exists.

---

## 5. Building a combo product

**Actor** product manager.

1. The manager creates the choice groups first. For each group:
   1. Create a Product Combo with a name and a sequence.
   2. Add one Product Combo Item per selectable product, each naming a variant whose type is not
      `combo`, with an optional extra price.
   3. Saving validates that the group has at least one item, that no product appears twice, and
      that no item names a combo product.
   4. The group's base price is computed as the minimum of its items' sales prices converted into
      the group's currency.
2. The manager creates or opens the product that will be the combo.
3. The manager sets its type to `combo`. This is refused if the product already has attribute lines
   ("Combo products can't have attributes.") or if the product is itself an item of some combo
   ("This product is part of a combo, so its type can't be changed to \"combo\"."). On success the
   purchasable flag is set to false automatically.
4. The manager links the choice groups and sets the combo product's sales price.
5. Saving validates that at least one choice group is linked ("A combo product must contain at least
   1 combo choice.") and, if the combo is sellable, that every product reachable through the groups
   is itself sellable ("A sellable combo product can only contain sellable products.").

**Selling it.** The consuming domain creates one line for the combo product — always at a unit price
of zero — plus one child line per chosen item, each priced by the proration of
[calculations.md](calculations.md), section 8.2.

**Changing the type away from combo** empties the choice group list silently.

**A variant disappearing** — because variant generation unlinked it — deletes every combo item
naming it, without warning. The group may then fall below one item, which the next validation of
that group will refuse.

---

## 6. Barcodes

### 6.1 Assigning a product barcode

**Actor** product manager.

1. The manager opens a variant (or a template with exactly one variant, which writes through) and
   types a barcode.
2. On save the uniqueness validation runs, within the record's company:
   - against other products, producing "Barcode(s) already assigned:" followed by one line per
     colliding barcode;
   - against packaging barcodes, producing "A packaging already uses the barcode".
3. **If the nomenclature has a value-carrying rule that matches this barcode**, the stored barcode
   must have zeros in the value positions, because the parser compares the scan's zeroed base code
   against the stored value. The manager is responsible for this; nothing enforces it.

### 6.2 Assigning a packaging barcode

**Actor** product manager.

1. From a variant or from a unit of measure, the manager opens the packaging barcode list.
2. The manager creates a row naming the unit, the product and the barcode, with the company
   defaulting to the acting company.
3. On save, the global uniqueness constraint refuses a barcode already used by another packaging
   ("A barcode can only be assigned to one packaging."), and the validation refuses a barcode
   already used by a product ("A product already uses the barcode").

### 6.3 Configuring a nomenclature

**Actor** system administrator.

1. The administrator opens the barcode nomenclature list and creates or edits a nomenclature.
2. For a **classic** nomenclature: set the conversion policy, then add rules. Each rule has a
   sequence, a name, a result type, an encoding, a pattern and — for alias rules — an alias string.
   The rules are tried in ascending sequence.
3. For a **Global Standards One** nomenclature: set the mode flag, optionally override the separator
   expression, then add rules. The conversion policy is hidden, and the encoding of each rule is
   forced to the Global Standards One symbology. Each rule additionally has a content type, a
   decimal-usage flag for measure rules and an associated unit for measure rules.
4. Pattern validation runs on save (see [business-rules.md](business-rules.md), sections 13.3 and
   13.4).
5. The administrator points a company at the nomenclature. Every scan made for that company is
   thereafter parsed by it.

### 6.4 Scanning

**Actor** warehouse operator, cashier or any user on a barcode-enabled form.

1. The client collects the keystrokes a scanner produces. Two keystrokes are considered part of the
   same scan when they are at most *N* milliseconds apart, where *N* is the system parameter
   `barcode.max_time_between_keys_in_ms` ("maximum time between keys, in milliseconds"), defaulting
   to one hundred and fifty. The value is delivered to the client in the session payload, but only
   for internal users.
2. The assembled string is written into the form's scan field.
3. The scan handler blanks the field again and passes the string to the form's implementation. A
   form that includes the barcode contract but does not implement the handler raises "In order to
   use barcodes.barcode_events_mixin, method on_barcode_scanned must be implemented".
4. The implementation parses the string against the company's nomenclature:
   - a string beginning with `urn:` is decoded as a uniform resource identifier;
   - a Global Standards One nomenclature decomposes it into an ordered list of typed records;
   - otherwise the first matching rule produces one typed record.
5. The implementation acts on the result: resolve a product, set a quantity, select a lot, set an
   expiry date, open a location, and so on. Those actions belong to the consuming domain.

### 6.5 Searching for a product by a scanned code

**Actor** any user.

1. The user pastes or scans a code into a product search box.
2. If the acting company's nomenclature is a Global Standards One nomenclature and the search is on
   the barcode field, the condition is rewritten by the unpadding preprocessing of
   [calculations.md](calculations.md), section 11.9: the code is parsed, the first record whose type
   the caller cares about supplies its value, leading zeros are stripped, and the comparison becomes
   a contains comparison.
3. Otherwise the staged name search of [calculations.md](calculations.md), section 12.4 runs:
   exact internal reference, then exact barcode, then partial internal reference, then partial name,
   then bracketed code, then vendor code and vendor name.

---

## 7. Printing labels

**Actor** internal user.

1. The user selects one or more products — templates or variants — and runs the "Print Labels"
   action.
2. The action refuses immediately if any selected product's type is `service`, with "Labels cannot
   be printed for products of service type".
3. The label layout dialog opens, pre-filled with the selected templates or the selected variants.
4. The user chooses a format among Dymo, two-by-seven-with-price, four-by-seven-with-price,
   four-by-twelve and four-by-twelve-with-price; a number of copies; optional extra markup; and
   optionally a pricelist whose price is printed.
5. On confirmation:
   1. A copy count of zero or less raises "You need to set a positive quantity."
   2. The printable document is chosen from the format: the Dymo format uses the single-label
      document; any format containing the letter `x` uses the document named for its column and row
      counts, with the suffix for the no-price variant appended when the format does not contain the
      word `price`.
   3. If neither a template list nor a variant list is filled, the operation raises "No product to
      print, if the product is archived please unarchive it before printing its label."
   4. The report data is assembled: the owner model, a map from product identifier to copy count,
      the dialog's identifier, and whether prices are printed.
   5. The report builder re-reads the products in the chosen model with internal-reference display
      switched off, ordered by name descending — because the template consumes the map from the
      end, which yields ascending name order on the page — and produces, per product, a list of
      (barcode, copy count) pairs.
   6. The page count is computed as in [calculations.md](calculations.md), section 14.2.
   7. The document is rendered and offered for download; the dialog closes.

**Additional printable document.** From a packaging barcode, the "Packaging Barcodes" document
prints the barcodes of one unit of measure; its file name is "Products packaging - *the unit's
name*".

---

## 8. Managing product documents

### 8.1 Uploading through the product form

**Actor** product manager.

1. From the product form, the manager opens the documents view.
2. The manager drops one or more files. The client posts them to the upload route with the owner
   model and the owner identifier.
3. The route validates the owner model — only a product template or a product variant is accepted —
   checks that the record exists and that the caller has write access, then creates one document per
   file with the file name, the owner, the owner's company and the browser-declared content type.
4. A per-file failure is logged and reported as an error text; the other files are still attempted.
   A fully successful upload reports "All files uploaded".

### 8.2 Attaching a file through the message thread

**Actor** any user who may post on the product.

1. The user attaches a file to a message on a product template or a product variant.
2. The attachment is created with the product as its owner and with no field name.
3. The automatic hook creates a Product Document for it as a privileged writer, so the file appears
   in the product's document list without any further action.

### 8.3 Attaching a link

**Actor** product manager.

1. The manager creates a document whose kind is a web address and types the address.
2. An address not starting with `https://`, `http://` or `ftp://` is refused with "Please enter a
   valid uniform resource locator…", reproduced in full in
   [business-rules.md](business-rules.md), section 12.1.

### 8.4 Deleting

Deleting a document deletes its underlying attachment. Deleting the attachment deletes the document
through the cascading link. Archiving a document leaves the attachment untouched.

---

## 9. Importing products with their attribute values from one file

**Actor** product manager. **Precondition** the import file has a column matched to the
product-values field, whose cells hold comma-separated `attribute:value` pairs.

This is the only workflow in the domain that spans two entities in one operation, and its ordering
is load-bearing.

### 9.1 Splitting the file

1. The import is directed at the template entity. If the product-values column is absent, the
   ordinary import runs and nothing below applies.
2. The rows are split in two by whether their product-values cell, once trimmed, is non-empty:
   - rows **without** product values go to the template import, with the product-values column
     removed;
   - rows **with** product values go to the variant import, with every column kept.
3. The template rows are imported first. If any error is reported, the whole import stops and
   returns that result.
4. The variant rows are then imported through the variant entity, with a flag telling it that the
   call comes from the template import so that it does not recurse.
5. If the variant import reports an error, the whole import stops and returns that result.
6. Otherwise the identifiers of the templates of the imported variants are appended to the result,
   the messages are merged, and the next-row cursor is the sum of the two cursors.

A direct import into the **variant** entity carrying a product-values column, without the
from-template flag, is redirected to the template entity and the resulting template identifiers are
translated back into variant identifiers.

### 9.2 Creating the variant rows

The variant creation path, for the rows carrying product values, runs as a privileged writer with
variant creation suppressed and value materialisation suppressed:

1. Rows **without** product values in this batch are created the ordinary way first.
2. Every remaining row must have a trimmed name or a template reference, otherwise "Unable to import
   products with attribute values but without name of product set".
3. **Parse the product values.** For each row, split the cell on commas; split each piece on the
   first colon into an attribute name and a value name; trim both. Failures:
   - no attribute part → "Unable to import products with attribute value without attribute name
     (defined as: attribute:value): *the cell*";
   - the same attribute twice in one row → "It is not possible to import different values for the
     same attribute: *the cell*";
   - the same value twice for one attribute in one row → "Duplicate values in attribute values are
     not allowed: *the cell*".
   The parse accumulates, per attribute name, the ordered set of value names seen across the whole
   batch.
4. **Create missing attributes.** Search the attributes by name. Every attribute name not found is
   created with variant creation **dynamic** and display type **radio**.
5. **Find or create the attribute values.** Search, in one query, for values whose name is in the
   collected set and whose attribute has the matching name. Create the missing ones.
6. **Find or create the templates.** Search templates whose name is one of the row names, or whose
   identifier is one of the row template references. For every row name that matched nothing and
   has not already been queued, queue a template creation holding only the row's **required**
   fields. Create the queued templates **with** variant creation enabled, so that each gets a first
   variant carrying the defaults.
7. **Capture the defaults.** Read, from the first variant of every involved template, the values of
   every column of the import. These are the fallback values for empty cells.
8. **Remove the placeholder variants.** Delete or archive every variant of the involved templates
   whose combination is empty: they exist only because the templates were created with variant
   creation enabled, and they are about to be replaced by the real combinations.
9. **Create or extend the attribute lines.** For each (template, attribute) pair seen in the rows,
   find the existing line and add the new values to it, or queue a line creation with the template,
   the attribute and the values. Create the queued lines.
10. **Resolve the materialised values.** For each (template, attribute value) pair, find the
    Template Attribute Value on the corresponding line.
11. **Rewrite the rows.** For each row: drop the name and substitute the template identifier if the
    row had none; replace the product-values cell by the list of resolved Template Attribute Value
    identifiers.
12. **Fill the blanks.** For each row, every empty cell is replaced by the captured default for that
    template, and the product-values and identifier columns are dropped.
13. Create the variants from the rewritten rows.

### 9.3 Updating an existing variant through the import

When the import matches an existing variant and the row carries product values, the supplied set of
pairs must equal the variant's own set. Otherwise "The exitings product has different attribute
value. …". When they match, every empty cell is dropped together with the product-values column,
so the import asserts the combination without blanking anything.

---

## 10. Filling a document from the catalog grid

**Actor** salesperson, buyer or anyone editing an order-like document that implements the catalog
contract.

1. The user opens the document and runs its "add from catalog" action.
2. The action opens a card grid over the variant entity with:
   - the catalog card view and the catalog search view;
   - a filter built by the document: by default, products whose company is empty or is a parent of
     the document's company, and whose type is not `combo`;
   - a context carrying whether the unit column should be shown (that is, whether the acting user
     holds the units group), the document's identifier and the document's transport name.
3. The grid asks the server, through the order-lines-information route, for the per-product payload
   of the products currently on screen. The document:
   1. Groups its own lines by product, over the products on screen.
   2. For each such product, builds the per-line payload — quantity, price, unit name, and whatever
      the document adds — and stamps the product's type and reference onto it, filling the unit name
      from the product when the line did not supply one.
   3. For the products with no line, builds the default payload: quantity zero, a read-only flag
      from the document's own read-only test, plus the product's type, unit name and reference.
4. The user types a quantity on a card. The grid calls the update-order-line-information route,
   which asks the document to create, change or remove the corresponding line and returns the
   resulting unit price.
5. Both routes run with the document's own company as the acting company.

The payload keys are fixed by the contract: the product identifier, the quantity, the product type,
the price, the unit display name, the reference, and the read-only flag. Additional keys may be
added by the document but the listed ones must be present with those names.

---

## 11. Entering quantities through a product matrix

**Actor** salesperson or buyer. **Precondition** the template has at least one valid attribute line.

1. The user chooses a template on a document that supports the matrix and opens the grid.
2. The document asks the template for its matrix (see [calculations.md](calculations.md),
   section 9), passing the document's company, the document's currency and whether extra prices
   should be shown — they are shown on sales documents and hidden on purchase documents.
3. The grid renders:
   - a header row whose first cell is the template's display name and whose remaining cells are the
     first attribute line's values, each with its converted extra price when non-zero;
   - one row per combination of the remaining attribute lines, whose header cell joins those values
     with a bullet separator and carries their summed converted extra price when non-zero;
   - one input cell per combination, carrying the sorted value identifiers, an initial quantity of
     zero and a possibility flag.
4. Cells whose possibility flag is false are rendered as unavailable and accept no input.
5. The user types quantities and confirms. The document creates one line per non-zero cell, calling
   the create-on-demand procedure for each so that dynamic templates materialise the variants that
   are actually ordered.

---

## 12. Expiry

### 12.1 Enabling expiry on a product

**Actor** product manager. **Precondition** the product is tracked by lot or by serial number.

1. The manager ticks "Use Expiration Date" and fills the four day counts: the expiration day count
   (forwards from receipt) and the best-before, removal and alert day counts (backwards from the
   expiration date).
2. Writing the tracking mode back to "no tracking" silently clears the use-expiration-date flag.

### 12.2 Receiving goods

**Actor** warehouse operator.

1. The operator processes an incoming transfer for a product that uses expiration dates.
2. Each move line computes a default expiration date from the transfer's scheduled moment — or
   today when there is none — plus the product's expiration day count, unless the line already has
   one. It then computes a removal date by subtracting the product's removal day count.
3. When the operator asks for a run of serial numbers, each generated line gets the same default
   expiration date.
4. On completion, each line that creates a lot copies its expiration date onto the new lot.
5. The lot's derived-date computation then fills the best-before, removal and alert dates from the
   expiration date and the product's day counts (initialisation branch).

### 12.3 Changing a lot's expiration date

**Actor** warehouse operator or quality manager.

1. The user writes a new expiration date on the lot.
2. The derived-date computation takes the **shift branch**: the three derived dates move by the same
   signed amount as the expiration date, preserving any manual adjustment. Derived dates that were
   empty stay empty.

### 12.4 The alert reminder

**Actor** the scheduled work runner, as part of the replenishment run.

1. Select every lot whose alert date is at or before today's date and whose reminded flag is false.
2. Keep only the lots that have a strictly positive quantity in an internal location.
3. For each, schedule a to-do activity with the summary "Alert Date Reached" and the note "The alert
   date has been reached for this lot/serial number", assigned to the product's responsible user as
   seen from the lot's company, or the product's responsible user without a company restriction, or
   the superuser.
4. Set the reminded flag on **every** lot selected in step 1, including the ones dropped in step 2.

### 12.5 Delivering expired goods

**Actor** warehouse operator. See [state-machines.md](state-machines.md), section 9 for the state
table.

1. The operator asks to complete an outgoing transfer.
2. The pre-completion check looks for move lines whose lot has the expiry alert flag, or whose
   removal date is at or before the current moment. If none is found, or the caller already
   confirmed, completion proceeds.
3. Otherwise the confirmation dialog opens, listing the offending lots. Its message names the
   product and the lot when exactly one lot is involved, and is generic when several are.
4. **Confirm and proceed** retries the completion with the expiry check suppressed.
5. **Discard expired products** deletes every move line of the listed transfers that uses expiration
   dates and whose removal date is **strictly before** the current moment, then retries the
   completion with a cleaned context.

### 12.6 Consuming the oldest first

When the removal strategy in force is first expiry first out, the candidate stock quantity records
are ordered by removal date, then entry date, then identifier. The strategy record itself is shipped
by this domain under the method name `fefo` and the shipped label `First Expiry First Out (FEFO)`,
whose parenthesised four letters abbreviate the four words before them.

### 12.7 Forecasting with expiry

The forecast report gains three behaviours when any product in scope uses expiration dates:

1. A header figure "to remove", computed as

   ```formula
   to_remove = quantity_on_hand + incoming − outgoing − forecast
   ```

   which is the amount the forecast has already written off as no longer fresh.
2. The stock quantity filter excludes records whose removal date is at or before today; a separate
   filter selects exactly those records for the "expired" figures.
3. Before the free-stock line, the report inserts:
   - a "to remove now" line carrying the unreserved expired quantity, when that quantity is not zero
     at the product's rounding;
   - one "to remove on *the removal date*" line per removal-date day, carrying the free quantity at
     that date after subtracting the quantities already taken from stock by the report's own moves,
     oldest first;
   and it compensates the free stock by adding back the **reserved** expired quantity, because a
   reservation already made survives the removal date.

---

## 13. Sending a product-specific e-mail when an invoice is posted

**Actor** the accounting user posting the invoice; the send itself is performed by the system.

1. A product manager sets a message template on a product template.
2. An accounting user posts a customer invoice containing a line for that product.
3. After posting succeeds, for each posted record whose kind is a customer invoice, and for each of
   its lines whose product has a message template, a message is posted on the invoice from that
   template, using the light notification layout and the comment subtype — which means the
   customer's followers are notified.
4. One message is posted **per line**, so a product appearing on two lines sends two messages, and
   an invoice with three such products sends three.
5. When the posting is performed by the system itself rather than by a user, the send is re-attributed
   to the superuser so that the message has a consistent author.

---

## 14. Maintaining the catalogue

### 14.1 Archiving a product

**Actor** product manager.

1. The manager archives a template. Every variant, archived ones included, is archived.
2. The template disappears from ordinary searches. Documents referencing its variants keep working.
3. No combination of the template is possible while it is archived: the possible-combination
   generator returns nothing for an archived template.

### 14.2 Archiving one variant

1. The manager archives a variant.
2. If that was the template's last active variant and the template is active, the template is
   archived too.
3. The variant's combination stops being possible.

### 14.3 Deleting

1. Deleting a template deletes its variants through the cascading link.
2. Deleting a variant deletes its template when it was the last variant (counting archived ones)
   **and** the template has no dynamic attribute. Before deleting, a variant image is moved up to
   the template if the template had none.
3. Deleting an attribute, an attribute value or a template attribute line is guarded or falls back
   to archiving, as specified in [business-rules.md](business-rules.md).

### 14.4 Duplicating a product

1. The manager duplicates a template.
2. The copy's name is the original's followed by " (copy)".
3. The attribute lines are copied; the materialised values are regenerated from them; the variants
   are regenerated from those.
4. The extra prices are then re-applied: walking the original's lines and the copy's lines in
   parallel, and within each line the original's values and the copy's values in parallel, every
   non-zero extra price is copied across, guarded by a check that the attribute and the underlying
   attribute value match.
5. Barcodes are **not** copied (the field is excluded from duplication), so the copy has no barcode
   and no uniqueness conflict arises.

### 14.5 Reorganising categories

1. The manager creates or moves a category. A move that would make the category its own ancestor is
   refused with "You cannot create recursive categories."
2. Moving a category rewrites the complete name of the category and of every descendant.
3. Deleting a category deletes its whole subtree; templates in the deleted categories are left with
   no category.

### 14.6 Defining per-category product properties

1. The manager opens a category and defines the property schema in its product-properties
   definition.
2. Every template in that category gains those custom fields in its properties bag.
3. Changing a template's category changes which properties it has; the properties bag is copied when
   a template is duplicated.
