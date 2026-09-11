# Contacts and Organizations

## Scope

This domain specifies the single, shared directory of **parties** that every other domain of the
system points at. A customer on a sales order, a vendor on a purchase order, an account holder on
a bank account, the employer on an employment record, the recipient of an electronic mail message,
the owner of a company record, the billing address on an invoice, the shipping address on a
delivery note, the author of a chat message — all of them are the same entity, the Party
(`res.partner`, table `res_partner`). The system does not have separate customer, supplier,
address and person tables; it has one table with a self-referencing hierarchy and a set of flags,
and the whole business meaning of "customer", "shipping address", "the legal entity that owes us
money" is derived from that hierarchy and those flags by the rules documented here.

Because every domain depends on it, this is a foundation domain. Nothing here produces accounting
entries, stock movements or fulfilment obligations. What it produces is **identity**: who a
document is for, under what legal name, at what postal address, in what language, in what
currency, under what tax registration, with what bank account, and which of several related
records is the one that carries the commercial relationship.

Concretely the domain covers:

- **The Party entity in full** — every stored field, every computed field with the exact
  expression that computes it, the self-referencing parent link, the address-type selection, the
  person/organization flag, the archival rules, the uniqueness constraints, the create and write
  side effects, and the deletion guards.
- **The three-way hierarchy semantics** — a Party may be an organization, a person, or a bare
  address; a Party may have a parent; and from those two facts the system derives a *commercial
  entity* (the legal counterparty), a *complete name*, a *display name*, an *address*, and the set
  of fields that are owned by the commercial entity rather than by the record itself.
- **Field synchronization across the hierarchy** — the exact algorithm that copies address fields
  down from a parent to its contact-type children, copies address fields *up* from a child to a
  childless parent, copies commercial fields down from the commercial entity to all descendants,
  and copies the tax registration number up from a child to its parent. This is the single most
  behaviour-defining algorithm of the domain and it is given as numbered steps with worked
  examples in both directions.
- **Address resolution** — the depth-first search that answers the question "given this Party,
  which Party record is its invoicing address, and which is its shipping address?", including the
  boundary at organization records, the fall-back chain, and the ancestor walk.
- **Naming and display** — the complete name computation (which prefixes a person's name with the
  name of its commercial entity, and which substitutes the address-type label when a name is
  missing), the display name computation with its six context switches (formatted display, show
  address, show electronic mail address, show tax registration number, show internal identifier,
  language), the record-name search fields, and the ordering rule.
- **Postal addresses** — the six address fields, the country-driven layout string with its
  substitution keys, the rendering algorithm, the default layout, the layout validation
  constraint, the state/country consistency rules, the input-form reordering rule driven by the
  layout, and the optional extended address decomposition into street name, house number and door
  number with the exact two-pass parsing grammar.
- **Countries, states, country groups** — every field, the uniqueness constraints, the code
  upper-casing, the flag image path derivation with its ten mapping exceptions and two no-flag
  countries, the calling code, the tax registration label override, the state and postal-code
  requirement flags, and the complete shipped table of two hundred and fifty countries, their
  states, and the country groups.
- **Cities** — the optional controlled city list, the per-country enforcement flag, and the
  cascade from a chosen city to the free-text city, postal code and state.
- **Tax registration numbers** — the storage field, the country prefix convention, the per-country
  validation algorithms written out as arithmetic for every country the system validates, the
  duplicate detection across the hierarchy and across the customs union, the label override per
  country, the check-on-save setting, the remote verification service contract, and the company
  registration number with its own duplicate rule.
- **Telephone numbers** — the parse/format contract, the four output formats, the country
  resolution order used to interpret a national number, the sanitized-number field, the blocked
  number list with its create/activate/deactivate semantics, the search behaviour on telephone
  fields including the plus-prefix and double-zero-prefix equivalence, and the eight shipped
  numbering-plan corrections.
- **Bank accounts and banks** — the account entity with its sanitized number, its uniqueness rule
  per holder, its holder-name derivation, its outgoing-payment permission flag, its archive-
  instead-of-delete rule, the find-or-create algorithm with the own-company guard, and the bank
  entity with its bank identifier code and display rule. The complete shipped bank table is
  enumerated.
- **Currencies and languages as they bear on parties** — the currency reference data table, the
  language entity with its date, time, first-day-of-week, digit-grouping, decimal-mark and
  thousands-mark settings, the number formatting algorithm with the grouping interspersion
  procedure written out and worked examples, the activation and deactivation rules, and the
  complete shipped language table.
- **Companies as parties** — the company record's delegation to a Party record, the fields the
  company adds (report layout, logo, colours, parent company chain, e-mail and telephone), the
  consistency constraint that binds a company's Party to that company, and the multi-company
  visibility rule on parties.
- **Merging two or three parties into one** — the grouping query that finds candidate duplicates,
  the five grouping criteria, the two exclusion filters, the destination-selection rule, the five
  safety checks with their exact messages, the foreign-key rewriting pass, the reference-field
  rewriting pass, the company-dependent value merge, the bank-account merge, the field-value merge
  with its precedence rule, and the deletion of the sources.
- **Enrichment, geolocation and the public page** — the external company-data lookup contract and
  the fields it fills, the geocoding contract with its two providers and the coordinate reset on
  address change, and the published partner page with its route and its publication state.
- **Avatars and images** — the five image sizes, the derived avatar chain, the placeholder
  selection by organization flag and address type, and the generated initials avatar with its
  exact colour derivation and its exact vector document.
- **Tags and industries** — the recursive tag tree with its cycle guard and its slash-joined
  display name, the random colour default, and the industry list with the complete shipped table.
- **Configuration and security** — the two security groups of the domain, the access-rights matrix
  for every entity, the record rules, the system parameters, the shipped default records, the
  window actions and menus, the views and what each shows, the remote operations, the routes, and
  the import template.

## Capabilities covered

| Capability | Where specified |
|---|---|
| Create an organization, a person, a bare address | [workflows.md](workflows.md) §1–§4 |
| Attach a contact or an address to an organization | [workflows.md](workflows.md) §5 |
| Keep a child address in step with its parent | [calculations.md](calculations.md) §3, [workflows.md](workflows.md) §6 |
| Resolve the legal counterparty of any Party | [calculations.md](calculations.md) §2 |
| Resolve the invoicing and shipping address of a Party | [calculations.md](calculations.md) §5 |
| Render a Party's postal address for a country | [calculations.md](calculations.md) §6 |
| Build a Party's complete name and display name | [calculations.md](calculations.md) §1 |
| Validate a tax registration number | [business-rules.md](business-rules.md) §6, [calculations.md](calculations.md) §8 |
| Detect a second Party with the same tax registration number | [business-rules.md](business-rules.md) §7 |
| Normalise and validate a telephone number | [calculations.md](calculations.md) §9 |
| Block and unblock a telephone number | [workflows.md](workflows.md) §12 |
| Register a bank account for a Party | [workflows.md](workflows.md) §9 |
| Merge duplicate parties | [workflows.md](workflows.md) §10, [calculations.md](calculations.md) §11 |
| Archive and delete a Party | [workflows.md](workflows.md) §8, [business-rules.md](business-rules.md) §9 |
| Enrich a Party from an external company directory | [workflows.md](workflows.md) §13 |
| Geocode a Party's address | [workflows.md](workflows.md) §14 |
| Publish a Party as a public page | [workflows.md](workflows.md) §15 |
| Format a number for a language | [calculations.md](calculations.md) §10 |
| Generate an initials avatar | [calculations.md](calculations.md) §12 |
| Import parties from a tabular file | [interfaces.md](interfaces.md) §9 |

## Entity list

### Core

| Entity | Transport name | Table | Purpose |
|---|---|---|---|
| Party | `res.partner` | `res_partner` | Every person, organization and address the system knows; the universal counterparty. |
| Party Tag | `res.partner.category` | `res_partner_category` | Free classification of parties, arranged as a tree. |
| Industry | `res.partner.industry` | `res_partner_industry` | The line of business of an organization. |
| Party Title | `res.partner.title` | `res_partner_title` | Courtesy title of a person, with a short form. |

### Geography

| Entity | Transport name | Table | Purpose |
|---|---|---|---|
| Country | `res.country` | `res_country` | Sovereign or dependent territory with its two-letter code, address layout, calling code and requirement flags. |
| Country State | `res.country.state` | `res_country_state` | Administrative subdivision of a country. |
| Country Group | `res.country.group` | `res_country_group` | Named set of countries used for customs unions and shipping zones. |
| City | `res.city` | `res_city` | Controlled city list, optionally enforced per country. |

### Money, language and time

| Entity | Transport name | Table | Purpose |
|---|---|---|---|
| Currency | `res.currency` | `res_currency` | Monetary unit with its symbol, decimal places, rounding step and symbol position. |
| Language | `res.lang` | `res_lang` | Locale with date, time, week-start, digit-grouping and separator settings. |

### Banking

| Entity | Transport name | Table | Purpose |
|---|---|---|---|
| Bank | `res.bank` | `res_bank` | A financial institution with its bank identifier code and address. |
| Bank Account | `res.partner.bank` | `res_partner_bank` | An account number held by a Party at a Bank. |

### Organization

| Entity | Transport name | Table | Purpose |
|---|---|---|---|
| Company | `res.company` | `res_company` | A legal entity operating inside the system; delegates its identity fields to a Party. |

### Telephone

| Entity | Transport name | Table | Purpose |
|---|---|---|---|
| Blocked Number | `phone.blacklist` | `phone_blacklist` | A telephone number that must never receive automated messages. |
| Telephone Thread Mixin | `mail.thread.phone` | *(abstract)* | Behaviour that gives any record a sanitized number and a blocked-state flag. |
| Unblock Number Wizard | `phone.blacklist.remove` | *(transient)* | Two-field dialogue that removes a number from the blocked list with a reason. |

### Merging

| Entity | Transport name | Table | Purpose |
|---|---|---|---|
| Merge Wizard | `base.partner.merge.automatic.wizard` | *(transient)* | Finds duplicate groups and merges two or three parties into one. |
| Merge Group | `base.partner.merge.line` | *(transient)* | One candidate group of duplicate parties inside a merge run. |

### Supporting behaviour (abstract)

| Entity | Transport name | Purpose |
|---|---|---|
| Image Holder | `image.mixin` | Five derived image sizes from one stored image. |
| Avatar Holder | `avatar.mixin` | Avatar chain over the image sizes, with placeholder and generated-initials fall-back. |
| Address Layout Holder | `format.address.mixin` | Rewrites an input form so the address fields follow the company country's layout. |
| Tax Label Holder | `format.vat.label.mixin` | Relabels the tax registration field with the company country's own word for it. |
| Geocoder | `base.geocoder` | Turns an address string into a latitude and longitude through an external provider. |
| Geocoding Provider | `base.geo_provider` | Named external geocoding service. |

## Reading order

1. **[README.md](README.md)** — this file. Scope, entity inventory, vocabulary orientation.
2. **[glossary.md](glossary.md)** — read second. The domain uses a handful of terms (commercial
   entity, complete name, address type, contact-type child, sanitized number, company-dependent
   value) whose precise meaning the rest of the specification assumes.
3. **[entities.md](entities.md)** — the complete field-by-field definition of every entity.
4. **[state-machines.md](state-machines.md)** — the state fields of the domain. There are few, and
   they are small, but the archival lifecycle and the merge wizard lifecycle are genuine state
   machines with guards.
5. **[calculations.md](calculations.md)** — every algorithm and formula: naming, commercial-entity
   resolution, hierarchy synchronization, address resolution, address rendering, street parsing,
   tax-number check digits, telephone formatting, number formatting, merging, avatar generation.
6. **[business-rules.md](business-rules.md)** — every validation, every constraint, every error
   message, every permission check, every edge case.
7. **[workflows.md](workflows.md)** — the end-to-end operational procedures that combine the above.
8. **[configuration.md](configuration.md)** — settings, parameters, security groups, access rights,
   record rules, shipped reference data as complete tables, scheduled jobs.
9. **[interfaces.md](interfaces.md)** — menus, actions, views, named remote operations, routes,
   reports, templates, external services, import and export.
10. **[accounting-effects.md](accounting-effects.md)** — short: this domain produces no journal
    entries; it explains precisely how it nonetheless determines them elsewhere.
11. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given/When/Then scenarios with
    concrete numbers, to be used as the conformance suite for a rebuild.

## Dependencies on other domains

This domain depends on **nothing**. It is the base of the dependency graph: a rebuild must
implement it before any other domain, because every other domain stores a reference to the Party
entity.

The following domains depend on it, and the dependency is named here so a rebuild knows what
contract it must honour.

| Domain | What it takes from here |
|---|---|
| [General Ledger](../general-ledger/README.md) | The Party on a Journal Item; the Company entity and its currency; the commercial entity used to group receivable and payable balances. |
| [Accounts Receivable](../accounts-receivable/README.md) | The customer Party and its invoicing address; the commercial entity that owns the credit limit; the language used to render the invoice; the tax registration number printed on it; the country that selects the fiscal position. |
| [Accounts Payable](../accounts-payable/README.md) | The vendor Party; the vendor bank account and the constraint that ties it to the vendor; duplicate-bill detection keyed on the vendor. |
| [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) | The Bank Account entity in full, including the sanitized number, the outgoing-payment permission flag and the find-or-create algorithm. |
| [Multi-Currency](../multi-currency/README.md) | The Currency entity and the language-driven amount formatting. |
| [Taxes](../taxes/README.md) | The country and state of the Party, which select the fiscal position; the tax registration number, which drives the reverse-charge and customs-union decisions. |
| [Sales](../sales/README.md), [Purchasing](../purchasing/README.md) | The counterparty, its addresses, its language, its salesperson, its tags, its payment terms holder. |
| [Inventory Operations](../inventory-operations/README.md) | The destination address of a delivery, resolved by the address-resolution algorithm. |
| [Products and Catalog](../products-and-catalog/README.md) | The vendor Party on a supplier price line. |

## Conventions used in this specification

- **Identifiers in code font** are reproduced exactly because external contracts depend on them.
  Every such identifier is given its full name in words on first use in each file.
- **Formulas** are written in fenced blocks labelled `formula`, using named quantities in words.
- **Algorithms** are numbered prose steps with stated preconditions, postconditions and failure
  conditions.
- **Error messages** are quoted exactly as the system produces them, with placeholders replaced by
  a description of what they hold.
- Where the source leaves a behaviour implicit, the specification states the behaviour explicitly
  and marks it with the phrase **industry-standard default**.
