# Glossary of the Contacts and Organizations domain

Every term this domain uses, defined in full. Terms are listed alphabetically. Where a term has a
reproduced storage name, that name is given in code font.

---

**Acting company.** The company on whose behalf the current reader or writer is operating. A user
may be allowed several companies and chooses one as the acting company; every company-dependent
value, every company-scoped record rule and every company-driven form rewrite is resolved against
it. Distinct from the *company of a record*, which is the company stored on the record itself.

**Active flag** (`active`). The true/false field that decides whether a record takes part in
ordinary searches. Clearing it *archives* the record: the row stays, every reference to it stays
valid, and it disappears from default lists. Setting it again *unarchives*. Every entity of this
domain that can be retired has one — the Party, the Party Tag, the Industry, the Bank, the Bank
Account, the Currency, the Language, the Company and the Blocked Number.

**Address field set.** The exact list of fields that travel through the Party hierarchy as one
unit: street (`street`), second street line (`street2`), postal code (`zip`), city (`city`),
state (`state_id`), country (`country_id`). When the extended-address behaviour is installed the
controlled city reference (`city_id`) joins the set. Nothing else is an address field for
synchronization purposes — not the latitude, not the longitude, not the decomposed street parts.

**Address resolution.** The procedure that answers "given this Party, which Party record should be
used as its invoicing address, and which as its shipping address?" It is a depth-first search
through the descendants stopping at organizations, followed by a walk up the ancestors, with a
fall-back to the `contact`-type result and finally to the Party itself. Specified in
[calculations.md](calculations.md) §5.

**Address type** (`type`). The selection on a Party that says what kind of postal location it is:
`contact` (an ordinary contact whose address is the organization's address), `invoice` (a billing
address), `delivery` (a shipping address), `other` (anything else). Only `contact`-type records
take part in address synchronization.

**Archive.** See *active flag*.

**Avatar.** The picture shown for a Party in lists, cards and message threads. Derived from the
stored image when one exists; otherwise generated from the Party's initial letter; otherwise a
placeholder chosen by the organization flag and the address type.

**Bank identifier code** (`bic`). The eight- or eleven-character code that identifies a financial
institution internationally, also called the business identifier code. Stored upper-cased on the
Bank.

**Blocked number.** A telephone number recorded in the blocked list so that automated messages are
never sent to it. Unblocking deactivates the record rather than deleting it, so the history
survives.

**Branch.** A Company whose parent company is set. A branch shares its root company's
root-delegated fields — in the foundation package, exactly the currency. The company hierarchy
cannot be changed after creation.

**Calling code** (`phone_code`). The international dialling prefix of a country, stored as a whole
number without the leading plus sign. Thirty-two for Belgium, one for the United States of
America, ninety-one for India.

**Commercial entity** (`commercial_partner_id`). The Party that carries the commercial
relationship — the legal person that signs, owes and is invoiced. Computed and stored: a Party
that is an organization, or that has no parent, is its own commercial entity; otherwise it is the
commercial entity of its parent. Every other domain that needs "the customer" as opposed to "the
address the goods go to" reads this field.

**Commercial field set.** The fields owned by the commercial entity rather than by the record
itself, and therefore pushed down from the commercial entity to every non-organization descendant:
the tax registration number (`vat`), the company registration number (`company_registry`) and the
industry (`industry_id`). Its *synchronised* subset is the set that also travels **up** from a
child to its parent: exactly the tax registration number.

**Company** (`res.company`). A legal entity operating inside the system. It delegates its name,
address, logo, telephone, electronic mail address, website, tax registration number and bank
accounts to a Party.

**Company-dependent field.** A field stored as a map from company to value. Reading it returns the
acting company's entry; writing it sets only that entry. A fall-back entry applies to companies
with no entry of their own. On the Party, the barcode is company-dependent.

**Company registration number** (`company_registry`). The Party's number in its national company
register, used when it differs from the tax registration number. Part of the commercial field set.
Must be unique across all parties of one country — enforced as a *warning*, not as a constraint.

**Complete name** (`complete_name`). The stored, indexed name used for ordering and for
record-name searching. For an organization it is the name. For a person or an address inside an
organization it is the commercial company name, a comma, a space and the record's own name; when
the record has no name of its own, the address type's label is substituted.

**Contact-type child.** A Party whose parent is set and whose address type is `contact`. These and
only these share the parent's address.

**Controlled city** (`res.city`). An entry in the per-country city list. A country may be flagged
so that every address in it must choose from the list rather than typing a city name.

**Country group** (`res.country.group`). A named set of countries identified by a code. The code
`EU_PREFIX` marks the countries whose tax registration numbers carry a two-letter prefix.

**Customs-union prefix.** The two-letter code placed in front of a tax registration number in the
countries that belong to the prefixing group. Two countries use a prefix that differs from their
country code: Greece, whose country code is `GR` but whose tax prefix is `EL`; and the United
Kingdom, whose country code is `GB` but which additionally uses the prefix `XI` for the part of
its territory that remains inside the union's goods regime.

**Decimal separator** (`decimal_point`). The character a language puts between the whole part and
the fractional part of a number.

**Delete cascade.** A reference whose deletion removes the referring record too. The Bank Account's
account holder link and the Party Tag's parent link are cascading.

**Delete restriction.** A reference whose target cannot be deleted while the reference exists. The
Party's country and state links are restricting.

**Display name** (`display_name`). The computed, unstored label used wherever a record is referred
to. For a Party it is built from the complete name plus decorations chosen by the reading context.

**Extended address.** The optional behaviour that decomposes the street into a street name, a house
number and a door number, and that adds the controlled city list.

**Formatted display name.** A reading context that asks for the two-part display used by the
reference-field dropdown: the organization's name, a tab character, and the record's own name (or
its address-type label) between double hyphens.

**Free-text company name** (`company_name`). The name of the organization a Party belongs to when
that organization has no record of its own. Cleared the moment a parent is set. The "create the
parent organization" operation turns it into a real record.

**Generated initials avatar.** The vector image produced for a Party that has a name but no stored
picture: a square filled with a colour derived from the name and the creation timestamp, carrying
the upper-cased first character of the name in white.

**Geocoding.** Turning a postal address into a latitude and a longitude by calling an external
service. The system ships two providers.

**Grouping specification** (`grouping`). The list that tells a language how to break the whole part
of a number into groups. `[3,0]` means "three digits, then repeat three forever". `[3,2,0]` means
"three digits, then two, then repeat two forever".

**Hierarchy path** (`parent_path`). A materialised path of ancestor identifiers maintained by the
platform on tree-shaped entities, so that "is a descendant of" is answered by one prefix
comparison. Present on the Party Tag and on the Company.

**Industry** (`res.partner.industry`). The economic sector of an organization, from a shipped list
with single-letter section codes. Part of the commercial field set.

**International form.** The presentation of a telephone number with the plus sign, the calling code
and the national number separated by spaces, for example `+32 2 290 34 90`.

**International bank account number.** The standardised account identifier that begins with a
two-letter country code and two check digits. The foundation package stores it as an ordinary
account number; the behaviour that understands it adds a second account-type value and the check
arithmetic, which belongs to
[Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md).

**Layout** (`address_format`). The substitution template that renders a postal address for a
country. Keys are written as a percent sign, a parenthesised key and the letter `s`. Missing keys
substitute to the empty string.

**Locale code** (`code`). The identifier of a Language, of the shape "language part, underscore,
territory part" — for example `pt_BR`. Cannot be changed once the record exists.

**Merge.** The procedure that absorbs two or three parties into one, rewriting every foreign key,
every polymorphic reference, every company-dependent value and every default value, merging the
bank accounts, merging the field values, and deleting the sources.

**National form.** The presentation of a telephone number as it would be dialled inside its own
country, for example `02 290 34 90`.

**Organization flag** (`is_company`). The true/false field that says a Party is an organization
rather than a person. An organization is always its own commercial entity.

**Party** (`res.partner`). The universal counterparty record: person, organization or address.

**Party tag** (`res.partner.category`). A free, hierarchical label applied to parties. The display
name joins the whole ancestor chain with the separator ` / `.

**Placeholder avatar.** The shipped picture used when a Party has no image and no name to generate
initials from, or when its shape calls for a symbolic icon: a generic company picture for an
organization, a truck for a delivery address, a bill for an invoice address, a puzzle piece for
an "other" address, a grey silhouette otherwise.

**Public page.** The published web page of a Party, served at a path built from the Party's
identifier and name. Controlled by the published flag.

**Record rule.** A filter automatically added to every search and every read for a given entity and
a given group. This domain ships a multi-company rule on the Party, a subtree rule for portal and
public users, a multi-company rule on the Bank Account, and four rules on the Company.

**Root company.** The topmost company of a company hierarchy. Root-delegated fields are copied from
it to every branch.

**Root-delegated field.** A Company field that must be identical on a root and on all its branches.
In the foundation package the set contains exactly the currency.

**Sanitised account number** (`sanitized_acc_number`). The account number with every
non-alphanumeric character removed and the remainder upper-cased. Uniqueness, searching and the
merge comparison all use this value, never the raw one.

**Sanitised telephone number** (`phone_sanitized`). The first of a record's telephone fields that
can be normalised into the strict international form — a plus sign, the calling code and the
national number with no separators. Comparison with the blocked list uses this value.

**Shared party** (`partner_share`). True when a Party either has no user account at all, or has
only accounts that are themselves shared (portal or public). The multi-company record rule uses it
so that the Party of an internal user of another company stays selectable.

**State** (`res.country.state`). An administrative subdivision of a country. Its code is unique
within its country, not globally.

**Strict international form.** A telephone number written as a plus sign immediately followed by
the calling code and the national number with no spaces or punctuation, for example
`+3222903490`. This is the form stored in the sanitised number field and in the blocked list.

**Synchronised commercial field set.** The subset of the commercial field set that also travels
**up** the hierarchy: exactly the tax registration number.

**Tax registration number** (`vat`). The Party's value-added tax identifier or its national
equivalent. The single character `/` means "this counterparty deliberately has none".

**Thousands separator** (`thousands_sep`). The character a language puts between digit groups. May
be empty.

**Time zone** (`tz`). The named zone used when printing dates and times for a Party and when
importing or exporting time values. When empty, Coordinated Universal Time is used.

**Tracked field.** A field whose changes are recorded as entries in a record's message history.
On the Blocked Number the telephone number and the active flag are tracked; on the Party the
intra-community validity flag and the published flag are tracked when their behaviours are
installed.

**Unarchive.** See *active flag*.

**Verification service.** The external service that answers whether a customs-union tax
registration number is currently assigned. The system calls it through an intermediary, records a
pending state, and receives the final answer either by a callback or by a daily poll.
