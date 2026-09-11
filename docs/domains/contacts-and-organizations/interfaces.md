# Interfaces of the Contacts and Organizations domain

Window actions and menus as user-visible navigation; every view and what it shows; named remote
operations with their inputs and outputs; routes; printable documents; templates and
notifications; external service integrations; import and export formats.

---

# 1. Menus

## 1.1 The contacts application

| Menu path | Opens | Visible to |
|---|---|---|
| **Contacts** *(application root)* | — | the internal-user group and the contact-creation group |
| Contacts → **Contacts** | the contacts window action | as above |
| Contacts → **Configuration** | — | the administrator group |
| Contacts → Configuration → **Contact Tags** | the tags window action | administrator |
| Contacts → Configuration → **Industries** | the industries window action | administrator |
| Contacts → Configuration → **Localization** | — | administrator |
| Contacts → Configuration → Localization → **Countries** | the countries window action | administrator |
| Contacts → Configuration → Localization → **Fed. States** | the states window action | administrator |
| Contacts → Configuration → Localization → **Country Group** | the country-groups window action | administrator |
| Contacts → Configuration → Localization → **Cities** | the cities window action (only with the extended-address behaviour) | administrator |
| Contacts → Configuration → **Bank Accounts** | — | administrator |
| Contacts → Configuration → Bank Accounts → **Banks** | the banks window action | administrator |
| Contacts → Configuration → Bank Accounts → **Bank Accounts** | the bank-accounts window action | administrator |

The application root carries the application icon and sequence twenty. The contacts entry has
sequence two; configuration has sequence thirty-five; inside configuration, tags are first,
industries fourth, localization fifth and bank accounts sixth; inside localization, countries are
first, states second, country group third and cities second (the cities entry is added by the
extended-address behaviour with the same sequence as states, so the two sort together).

## 1.2 Technical menus

| Menu path | Opens |
|---|---|
| Settings → Technical → **Telephone and Text Messaging** (shipped label `Phone / SMS`) → **Phone Blacklist** | the blocked-numbers window action (added by the telephone behaviour, sequence three under a parent of sequence three) |
| Settings → Translations → **Languages** | the languages window action |

## 1.3 Where else the directory appears

Other domains place their own entries pointing at the same Party entity — a customers entry in
the sales application, a vendors entry in the purchasing application. Those entries use the window
actions of §2.2 and §2.3, which differ from the contacts action only in their context.

---

# 2. Window actions

## 2.1 Contacts

| Property | Value |
|---|---|
| Name | `Contacts` |
| Entity | Party |
| Path | `contacts` |
| View modes, in order | list, cards, form, activity |
| Search view | the Party search view of §3.4 |
| Context | the organization flag defaulted to **set** |
| Empty-state text | `Create a Contact in your address book` followed by a sentence explaining that the system tracks all activities related to contacts |
| View bindings | cards at sequence one, list at sequence zero, form at sequence two |

Because the context defaults the organization flag, a record created straight from this action
starts as an **organization**.

## 2.2 Customers

Two separate actions carry the name `Customers`.

| Property | First action | Second action |
|---|---|---|
| Entity | Party | Party |
| Path | — | `ecommerce-customers` |
| View modes | list, cards, form | list, cards, form |
| Context | a search-mode marker set to the customer value | the same marker, **plus** the organization flag defaulted to set |
| Filter panel open on arrival | no | yes |
| Empty-state text | `Create a Contact in your address book` | `Create a new customer in your address book` |

The search-mode marker carries no behaviour in this domain; other domains read it to pre-select
filters.

## 2.3 Vendors

| Property | Value |
|---|---|
| Name | `Vendors` |
| Entity | Party |
| View modes, in order | cards, list, form |
| Context | the search-mode marker set to the supplier value, and the organization flag defaulted to set |
| Filter panel open on arrival | yes |
| Empty-state text | `Create a new vendor in your address book` |

## 2.4 Contact tags

| Property | Value |
|---|---|
| Name | `Contact Tags` |
| Entity | Party Tag |

## 2.5 Industries

| Property | Value |
|---|---|
| Name | `Industries` |
| Entity | Industry |
| View modes | list, form |

## 2.6 Countries

| Property | Value |
|---|---|
| Name | `Countries` |
| Entity | Country |
| Empty-state text | `No Country Found!` followed by `Manage the list of countries that can be set on your contacts.` |

Creation and deletion are disabled in both the list view and the form view.

## 2.7 States

| Property | Value |
|---|---|
| Name | `Fed. States` |
| Entity | Country State |
| Default view | the editable list |
| Empty-state text | `Create a State` followed by `Federal States belong to countries and are part of your contacts' addresses.` |

## 2.8 Country groups

| Property | Value |
|---|---|
| Name | `Country Group` |
| Entity | Country Group |
| Empty-state text | `Create a Country Group` followed by a sentence suggesting groups for countries frequently selected together |

## 2.9 Cities

| Property | Value |
|---|---|
| Name | `Cities` |
| Entity | City |
| View modes | list |
| Empty-state text | a paragraph explaining that the list holds the cities assignable to parties and that a per-country option can force addresses in that country to use it |

## 2.10 Banks

| Property | Value |
|---|---|
| Name | `Banks` |
| Entity | Bank |
| View modes | list, form |
| Search view | the bank search view |
| Empty-state text | `Create a Bank` followed by `Banks are the financial institutions at which you and your contacts have their accounts.` |

## 2.11 Bank accounts

| Property | Value |
|---|---|
| Name | `Bank Accounts` |
| Entity | Bank Account |
| View modes | list, form |
| Empty-state text | `Create a Bank Account` followed by `From here you can manage all bank accounts linked to you and your contacts.` |

## 2.12 Languages

| Property | Value |
|---|---|
| Name | `Languages` |
| Entity | Language |
| Context | the archived-record filter switched **off**, so inactive languages are listed |
| Search view | the language search view |

## 2.13 Blocked numbers

| Property | Value |
|---|---|
| Name | `Blacklisted Phone Numbers` |
| Entity | Blocked Number |
| Default view | the blocked-number list |
| Search view | the blocked-number search view |
| Empty-state text | `Add a phone number in the blacklist` followed by a sentence saying that blocked numbers no longer receive text-message campaigns |

## 2.14 Deduplicate contacts

| Property | Value |
|---|---|
| Name | `Deduplicate Contacts` |
| Entity | Merge Wizard |
| View mode | form |
| Target | a modal dialogue |
| Context | the archived-record filter switched off |

## 2.15 Merge (bound to a selection)

| Property | Value |
|---|---|
| Name | `Merge` |
| Entity | Merge Wizard |
| View mode | form |
| Target | a modal dialogue |
| Bound to | the Party entity |
| Offered in | the list view and the card view |

Being *bound* means the action appears in the action menu whenever records of the Party entity are
selected in one of those two views.

## 2.16 Branches

Returned by an operation rather than shipped as a record.

| Property | Value |
|---|---|
| Name | `Branches` |
| Entity | Company |
| Filter | the parent is the company the operation was called on |
| Context | the archived-record filter switched off; the parent defaulted to that company |
| View modes | list, cards, form |

---

# 3. Views of the Party

## 3.1 The list view

Priority eight. Sample data enabled. Multiple-record editing enabled — with the explicit caution,
recorded in the source, that the fields which carry form-level change handlers are **not** made
editable in that mode: the parent, the country, the state, the organization/person selector and
the company.

| Column | Shown by default | Notes |
|---|---|---|
| the smallest avatar | yes | rendered as a rounded twenty-four-pixel image with no label |
| complete name | never (column hidden) | present only so that customised views can refer to it |
| display name, labelled `Name` | yes | |
| tax registration number | hidden | read-only |
| electronic mail address | yes | |
| telephone | yes | forced to left-to-right rendering |
| salesperson | hidden | rendered as an avatar; restricted to non-shared users |
| street | hidden | |
| city | hidden | |
| state | hidden | read-only |
| country | yes | read-only |
| application statistics | yes | a two-hundred-and-seventy-pixel column rendered by the statistics widget |
| tags | hidden | rendered as coloured chips |
| company | hidden | only for the multi-company group; read-only |
| custom properties | hidden | |

## 3.2 The form view

Priority one. Its structure, top to bottom:

1. **The duplicate warning block**, visible only while editing and only when a tax-registration
   twin or a company-registration twin exists. Four mutually exclusive wordings; see
   [business-rules.md](business-rules.md) §7.3. Each twin is displayed with the address and the
   tax number suppressed from its display name.
2. **The button box**, empty in the foundation package and filled by other behaviours — the
   publish control, the geolocation stat button, per-application counters.
3. **The archived ribbon**, a red banner shown when the active flag is cleared.
4. **The identity block**: the stored image rendered by the contact-image widget at one hundred
   and thirty pixels with the smallest avatar as its preview; the organization/person selector as
   a horizontal radio pair; the name as a large heading, with two alternative inputs carrying
   different placeholders depending on the organization flag and both required when the address
   type is `contact`; the electronic mail address, required when a user account is attached; the
   telephone.
5. **The left column**:
   - the parent, labelled `Company`, restricted to organizations, opening with the organization
     default and the current salesperson as defaults and with the address suppressed in its
     dropdown; hidden when the record is a parentless organization or when a free-text company
     name is set;
   - the free-text company name with a `Create` button beside it, shown only when a free-text name
     is set and the record is not an organization;
   - the address label, which reads the computed address-type description for a person and the
     fixed word `Address` for an organization;
   - the address block, carrying the marker class that the layout reordering targets: street,
     second street line, city, state, postal code, country. The state input passes the current
     country and postal code in its context and forbids opening and quick creation; the country
     input forbids opening and creation.
6. **The right column**: the job position (hidden for an organization); the tax registration
   number with the example placeholder `e.g. BE0477472701`; the website rendered as a link; the
   language, hidden when only one language is active; the tags as coloured chips with the
   placeholder `e.g. "B2B", "VIP", "Consulting", ...`.
7. **The custom properties block.**
8. **The notebook**, with three pages:

   | Page | Name | Contents |
   |---|---|---|
   | `Contacts` | `contact_addresses` | the children, as cards with an embedded form; autofocused |
   | `Sales & Purchase` | `sales_purchases` | three groups — `Sales` holding the salesperson (restricted to non-shared users, rendered as an avatar); `Purchase`, empty in the foundation package; `Misc` holding the company registration number (visible only for a parentless organization, with the computed placeholder), the reference, the company (for the multi-company group, read-only when a parent is set, with the placeholder `Visible to all`), and the industry (visible only for an organization, creation forbidden) |
   | `Notes` | `internal_notes` | the notes field and an empty warnings group that other domains fill |

### 3.2.1 The children card

Each child card shows: the smallest avatar — rendered to cover the frame for a `contact` and to
fit inside it with padding for the other three types; the name, or, when the name is empty, the
address type; the electronic mail address with an envelope mark; the telephone with a handset
mark; the job position with a case mark, hidden for an organization; and, **only for the three
address types**, the city and the country with a pin mark, separated by a comma when both are
present.

The list of children is opened with these defaults: the parent, all six address fields, the
language and the salesperson copied from the record being edited, and the address type set to
`contact` when the record is an organization and to `other` otherwise.

### 3.2.2 The embedded child form

Top to bottom: the address type as a horizontal radio group, required; the stored image at one
hundred pixels; the name as a heading, required for the `contact` type; the electronic mail
address (required when a user account is attached, and passing a marker that enables the
avatar-by-address service); the telephone; the job position, shown only for a non-organization of
type `contact`; then, **hidden entirely for the `contact` type**, a group holding the address-type
label and the address block; then the company (hidden, present so that the value inherited from
the parent survives the save) and the notes; then the language (hidden, present for the same
reason).

## 3.3 The card view

Priority one. Sample data enabled. Each card shows:

- the archived ribbon when the active flag is cleared;
- for a **person**, the smallest avatar filling the frame with the **parent's** image superimposed
  in the lower-right quarter at a quarter of the width, falling back to the parent's smallest
  avatar when the parent has no stored image;
- for an **organization**, the smallest avatar alone;
- the complete name in a hidden element (present for customised views) and the display name as the
  visible heading;
- the electronic mail address, the telephone, and the city and country separated by a comma, each
  with its own mark and each hidden when empty;
- the custom properties;
- the application statistics in the card footer.

## 3.4 The search view

| Search input | Behaviour |
|---|---|
| `name` | matches the **display name** with a containment comparison, so an organization's contacts are found by typing the organization's name |
| parent | restricted to organizations; matched with a descendant comparison, so selecting an organization finds its whole subtree |
| electronic mail address | containment comparison |
| telephone | plain matching in the foundation package; **replaced** by the telephone-number search field when the telephone behaviour is installed, which brings the plus-prefix equivalence of [calculations.md](calculations.md) §14 |
| tags, labelled `Tag` | descendant comparison, so a parent tag finds its children's parties |
| salesperson | plain |
| custom properties | plain |

| Filter | Condition |
|---|---|
| `Persons` | the organization flag is cleared |
| `Companies` | the organization flag is set |
| `Archived` | the active flag is cleared |

| Grouping | By |
|---|---|
| `Salesperson` | the salesperson |
| `Company` | the parent |
| `Country` | the country |
| `Properties` | the custom properties |

## 3.5 The simplified form and the address form

Two smaller forms ship:

- a **simplified form**, used where a Party must be created inside another record's dialogue;
- an **address form**, priority-ordered so that it can be substituted, holding the address block
  and the website. It is the shape a country's own input view is expected to take.

The extended-address behaviour ships a **third** at priority nine hundred: the same address block
with the street split into three inputs — a wide street-name input, a narrow house-number input
with the placeholder `House #`, and a narrow door-number input with the placeholder `Door #` —
laid out on one row, plus the combined street shown read-only, plus the controlled-city input
shown only for countries that enforce cities. Every input in it is read-only when the address type
is `contact` and a parent is set.

---

# 4. Views of the other entities

## 4.1 Party Tag

| View | Contents |
|---|---|
| form | the name and the parent tag |
| list | editable at the bottom, multiple-record editing, sample data; the name and the parent tag, the parent restricted so that a tag cannot be its own parent |
| search | the name; the display name labelled `Category`; an `Archived` filter; groupings by parent tag and by colour |

## 4.2 Industry

| View | Contents |
|---|---|
| form | the short name and the full name |
| list | editable at the bottom; the short name and the full name |
| search | the name; an `Archived` filter |

## 4.3 Country

| View | Contents |
|---|---|
| list | creation and deletion disabled; the name and the code |
| form | creation and deletion disabled; a button box; the flag image rendered from its computed path at one hundred and twenty-eight pixels; a left group with the name, the currency and the code; a right group with the calling code (numeric formatting disabled so the digits are not grouped), the tax registration label, the postal-code requirement and the state requirement; an **advanced address formatting** group visible only to the technical-features group, holding the input view with the hint `Choose a subview of partners that includes only address fields, to change the way users can input addresses.`, the layout with the placeholder `Address format...` and the hint `Change the way addresses are displayed in reports`, and the name position; and finally the states as an editable list of name and code |
| search | the name, matched against **either** the name or the code with a containment comparison; the calling code |

The extended-address behaviour adds to the form: a stat button labelled `Cities` opening the city
list filtered and defaulted to this country, and the enforcement flag beside the calling code.

## 4.4 Country State

| View | Contents |
|---|---|
| list | editable at the bottom; the name, the code and the country (creation and opening forbidden) |
| form | the name, the code and the country |
| search | the name; the country; a grouping by country |

## 4.5 Country Group

| View | Contents |
|---|---|
| list | the name and the code |
| form | the name as a heading with the placeholder `e.g. Europe`; the code; the countries as chips with opening and creation forbidden |

## 4.6 City

| View | Contents |
|---|---|
| list | editable at the top; the name, the postal code, the country (creation and opening forbidden) and the state with the country defaulted |
| search | the name, matched against **either** the name or the postal code with a containment comparison; the country |

## 4.7 Bank

| View | Contents |
|---|---|
| form | the archived ribbon; a four-column group with the name and the identifier code; an address group labelled `Bank Address` using the same marker class as the Party's address block; a communications group with the telephone (forced left-to-right) and the electronic mail address |
| list | the name, the identifier code and the country |
| search | the name; an `Archived` filter |

## 4.8 Bank Account

| View | Contents |
|---|---|
| form | priority fifteen; the archived ribbon; a left group with the account number, the clearing number, the holder, the holder name and the institution; a right group with the "send money" permission as a toggle, the company (multi-company group only, creation forbidden) and the currency (multi-currency group only, creation forbidden); a notes page; and a footer button labelled `Archive` calling the reload-forcing archive operation |
| list | multiple-record editing; archived rows greyed out; a drag handle bound to the sequence; the account number; the holder (hidden by default); the institution's name labelled `Bank`; the company (multi-company group, hidden); the "send money" permission as a toggle; the active flag as a toggle (hidden) |
| search | an input labelled `Bank Name` matching **either** the institution's name **or** the account number with a containment comparison; the company (hidden unless the context says otherwise); the holder; an `Archived` filter with the hint `Show inactive bank account` |

## 4.9 Language

| View | Contents |
|---|---|
| list | limited to two hundred rows; a header button `Activate` calling the bulk-activation operation; the name; the locale code, the language code and the direction, all three only for the technical-features group; the active flag; a row button `Activate` (when inactive) and `Update` (when active) opening the language-installation dialogue; a row button `Disable` calling the archive operation when active |
| form | a stat button `Activate and Translate` opening the language-installation dialogue; the flag image; the name as a heading with the placeholder `e.g. French`; a left group with the locale code, the language code and the active flag as a toggle; a right group with the direction, the grouping, the decimal separator, the thousands separator, the date format, the time format and the first day of week |
| search | the name, matched against the name, the locale code **or** the language code with a containment comparison; the direction; an `Active` filter |

## 4.10 Blocked Number

| View | Contents |
|---|---|
| list | sample data; the creation date labelled `Blacklist Date`; the number |
| form | duplication disabled; a header with two buttons — `Unblacklist`, shown when the record is active and has a number, which opens the unblock dialogue with the number defaulted; and `Blacklist`, shown when the record is archived and has a number, which re-adds it; an `Archived` badge; the number; and the message thread |
| search | the number; an `Archived` filter |

## 4.11 Unblock Number Wizard

A dialogue holding the number, read-only and labelled `Phone Number`, and a reason labelled
`Reason` with the placeholder `e.g "Asked to receive our next newsletters"`. Two footer buttons:
`Remove phone from blacklist` and `Discard`.

## 4.12 Merge Wizard

One form, shown as a modal, whose sections appear and disappear with the state.

| Section | Visible when | Contents |
|---|---|---|
| finished banner | the state is `finished` | the heading `There are no more contacts to merge for this request` and a button `Deduplicate the other Contacts` reopening the deduplication action |
| explanatory paragraph | the state is `option` | a sentence explaining that selecting several fields proposes only records having **all** of them in common, not one of them |
| group count | the state is `selection` or `finished`, and the count is non-zero | the number of candidate groups |
| grouping criteria, headed `Search duplicates based on duplicated data in` | the state is `option` | five checkboxes: electronic mail address, name, organization flag, tax registration number, parent |
| exclusions, headed `Exclude contacts having` | the state is `option` | two checkboxes: a user account; accounting entries |
| options | the state is `option` or `finished` | the maximum number of groups, read-only once finished |
| the group, headed `Merge the following contacts` | the state is `selection` | a paragraph explaining that the selected contacts will be merged, that every document linked to one of them will be redirected to the destination, and that contacts can be removed from the list; the destination, restricted to the listed parties, required, and displayed with the internal identifier appended; and the list itself showing the internal identifier, the display name, the electronic mail address, the organization flag, the tax registration number and the country |

Footer buttons: `Merge Contacts` (hidden in `option` and `finished`); `Skip these contacts` (only
in `selection`); `Merge with Manual Check`, `Merge Automatically` and
`Merge Automatically all process` (only in `option`, the last two with confirmation questions);
`Cancel` (hidden when finished) and `Close` (only when finished).

## 4.13 Geocoding on the Party form

The geolocation behaviour inserts a page named `Partner Assignment` before the notes page, holding
a group titled `Geolocation` with: the latitude, labelled `Lat :`; the longitude, labelled
`Long:`; the date of the last resolution, prefixed `Updated on:` and shown only when set; and one
of two buttons — `Compute based on address` when both coordinates are zero, `Refresh` otherwise —
both calling the resolution operation.

## 4.14 The tax-number additions to the Party form

Priority fifteen. The tax registration input is moved into a container beside a label reading
`Tax ID`, and the intra-community validity flag is shown next to it, both hidden unless the
"perform verification" computation says the check applies to this Party.

---

# 5. Named operations callable from a client

Every operation below is callable by name over the platform's remote-operation transport. Inputs
and outputs are given in words.

## 5.1 On the Party

| Operation | Inputs | Output | Notes |
|---|---|---|---|
| `name_create` | one text label | a pair: the new internal identifier and the new display name | Parses a combined name-and-address string; see [calculations.md](calculations.md) §4.1. Fails with `Couldn't create contact without email address!` when the context demands an address and none is found. |
| `find_or_create` | an address-like string; optionally a flag demanding a valid address | the found or created Party | See [calculations.md](calculations.md) §4.2. Fails with `An email is required for find_or_create to work` on an empty input and with `A valid email is required for find_or_create to work properly.` when the flag is set and no address is found. |
| `address_get` | a list of wanted address types | a map from address type to internal identifier, always containing a `contact` entry | See [calculations.md](calculations.md) §5. |
| `create_company` | none; called on one record | the boolean true | See [workflows.md](workflows.md) §4. |
| `open_commercial_entity` | none; called on one record | a window action opening the commercial entity's form | |
| `get_import_templates` | none | a one-element list holding the label `Import Template for Contacts` and the path of the shipped spreadsheet | |
| `view_header_get` | a view identifier and a view type | a heading, or nothing | Returns `Partners: <the tag's name>` when the reading context names a tag; otherwise defers. |
| `geo_localize` | none | the boolean true, or false when suppressed | Installed by the geolocation behaviour. See [workflows.md](workflows.md) §14.2. |
| `autocomplete_by_name` | a query string, a country identifier, optionally a time budget in seconds (default fifteen) | a list of normalised suggestions, or an empty list | Installed by the enrichment behaviour. A country identifier of the boolean false means "use the acting company's country"; a country identifier of zero means "do not filter by country". |
| `autocomplete_by_vat` | a tax registration number, a country identifier, optionally a time budget | a list of normalised suggestions, or an empty list | Falls back to the customs-union verification service when the enrichment service fails; see [workflows.md](workflows.md) §13.1. |
| `enrich_by_duns` | a global business identifier, optionally a time budget | one normalised suggestion, possibly carrying an error indication | |
| `enrich_by_gst` | a national goods-and-services-tax number, optionally a time budget | as above | |
| `enrich_by_domain` | an internet domain, optionally a time budget | as above | |
| `iap_partner_autocomplete_get_tag_ids` | a list of pairs (classification code, classification name) | the identifiers of the matching tags, creating any that do not exist | Looks the names up in the product classification table when that behaviour is installed, and otherwise uses the names supplied. |
| `enrich_company_message_post` | the service's payload | nothing | Posts an internal note rendered from a shipped template; see [workflows.md](workflows.md) §13.5. |

## 5.2 On the Bank Account

| Operation | Inputs | Output |
|---|---|---|
| `get_supported_account_types` | none | the list of account-type values with their labels; the foundation package returns one pair, `bank` labelled `Normal` |
| `retrieve_acc_type` | an account number | the account type; the foundation package always answers `bank` |
| `action_archive_bank` | none; called on one record | archives the record and returns a client action asking the interface to reload |

## 5.3 On the Language

| Operation | Inputs | Output |
|---|---|---|
| `get_installed` | none | the list of pairs (locale code, name) of every **active** language, ordered by name. Marked as a read-only operation. |
| `format` | a format specification, a value, and a grouping flag | the formatted text; see [calculations.md](calculations.md) §10 |
| `install_lang` | none | the boolean true; activates or creates the configured language and makes it the default language of new parties |
| `action_activate_langs` | none; called on a set | unarchives them and returns a success notification carrying the message `The languages that you selected have been successfully installed. Users can choose their favorite language in their preferences.` |

## 5.4 On the Blocked Number

| Operation | Inputs | Output |
|---|---|---|
| `add` | a number, optionally a reason | the blocked-number records |
| `remove` | a number, optionally a reason | the records, now archived |
| `action_add` | none; called on one record | re-adds the record's number |
| `phone_action_blacklist_remove` | none | a window action opening the unblock dialogue as a medium modal, with the question `Are you sure you want to unblacklist this phone number?` as its title |

On any record carrying the telephone behaviour, the same-named operation checks write access on the
blocked list first, titles the dialogue
`Are you sure you want to unblacklist this Phone Number?`, and fails with
`You do not have the access right to unblacklist phone numbers. Please contact your administrator.`
when access is missing.

## 5.5 On the Company

| Operation | Inputs | Output |
|---|---|---|
| `action_all_company_branches` | none; called on one record | a window action listing the direct branches |
| `install_l10n_modules` | none | installs the country-specific packages, or returns a readiness flag when suppressed |
| `iap_enrich_auto` | none | the boolean true; runs the automatic enrichment once per company |

## 5.6 On the Merge Wizard

| Operation | Inputs | Output |
|---|---|---|
| `action_start_manual_process` | none | runs the grouping query and returns the next-screen action |
| `action_start_automatic_process` | none | runs the query and merges every group, then returns the wizard's own window action |
| `action_update_all_process` | none | runs the parent-migration pass, then a fresh automatic merge, then returns the next-screen action |
| `action_merge` | none | merges the current group and returns the next-screen action |
| `action_skip` | none | discards the current group and returns the next-screen action |

---

# 6. Routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/partners/<the slug>` | hypertext transfer protocol, rendered inside the site layout | public | Serves a Party's public page. The slug is decomposed into an internal identifier; the Party is read with elevated rights; the page is served when the Party exists and is published, or when the visitor holds the restricted-editor right. When the slug differs from the canonical one, the visitor is redirected to the canonical address. Otherwise the not-found response is returned. |
| `/base_vat/1/webhook_update_vies` | hypertext transfer protocol, cross-site-request protection **disabled**, the session deliberately not saved | public | Receives the verification service's answer for a check that was previously pending. Takes two values: a signed token and a status. The token is verified against the recorded signature; on failure a warning is logged and nothing happens. On success, every Party whose tax registration number equals the one carried in the token has its validity flag updated and a history entry written. The whole operation runs as the built-in super-user. |

The signed token is produced when the check is first requested. It signs the scope name, the tax
registration number and an expiry of seven days, so that only the service that received it can
call back, and only for a week.

---

# 7. Printable documents, templates and notifications

## 7.1 Printable documents

**The domain ships no printable document of its own.** There is no contact sheet, no address label
and no directory listing. What it supplies to other domains' printable documents is specified in
[workflows.md](workflows.md) §21: the commercial company name, the rendered address, the name
position, the tax registration number with its country-specific label, and the language.

## 7.2 Message subtypes

| Subtype | Entity | Description | Subscribed by default |
|---|---|---|---|
| `Partner published` | Party | `Partner Published` | no |
| `Partner unpublished` | Party | `Partner Unpublished` | no |

Both are shipped by the public-page behaviour and are selected by a subtype resolver that inspects
the change to the published flag.

## 7.3 Templates

| Template | Used by | Contents |
|---|---|---|
| the enrichment result note | the enrichment behaviour | renders the telephone, the name, the electronic mail address, the entity type, the tax or company registration number, the website, the stored image, the street, the second street line, the postal code, the city, the country name, the state code, and the classification codes; posted as an internal note |

## 7.4 Notifications

| Notification | Trigger | Level | Text |
|---|---|---|---|
| geocoding failure | one or more parties could not be resolved | danger | title `Warning`, message `No match found for <the display names, separated by a comma and a space> address(es).` |
| language activation | the bulk-activation button | success | `The languages that you selected have been successfully installed. Users can choose their favorite language in their preferences.` |
| twelve-hour clock warning | a date or time format mixing a twenty-four-hour directive with a morning/afternoon directive | notification | title `Using 24-hour clock format with AM/PM can cause issues.`, message `Changing to 12-hour clock format instead.`; the format is silently rewritten to the twelve-hour directive |

## 7.5 History entries written by this domain

| Entry | Written on | Text |
|---|---|---|
| verification pending | the Party | `The VIES check is pending. The status will be updated soon.` |
| verification failed | the Party | `The VIES check failed. Please check the Tax ID manually.` |
| verification answered | the Party | `The Intra-Community validity has been updated to: <the status>.` |
| unblock reason | the Blocked Number | `Unblock Reason: <the reason>` as a rich-text paragraph |
| portal deletion | the Blocked Number | `Blocked by deletion of portal account <the deleted user's name> by <the acting user's name> (#<the acting user's identifier>)` |

Plus the automatic tracking entries for the Blocked Number's number and active flag, for the
Party's intra-community validity flag, and for the Party's published flag.

---

# 8. External service integrations

Three external services are contacted. In each case the contract is given as the request shape and
the answer shape, so that a rebuild can substitute its own provider.

## 8.1 The company-enrichment service

| Aspect | Contract |
|---|---|
| Address | a default host, overridable by the system parameter `iap.partner_autocomplete.endpoint`; the path is a fixed version prefix followed by the action name |
| Actions | `search_by_name`, `search_by_vat`, `enrich_by_duns`, `enrich_by_gst`, `enrich_by_domain` |
| Transport | a remote procedure call carrying a structured document, with a caller-supplied time budget (default fifteen seconds; five seconds for the automatic company enrichment) |
| Always sent | the installation's unique identifier; the platform version; the caller's language; the service account token; the acting company's country code; the acting company's postal code |
| Sent per action | `search_by_name`: the query text and the query country code. `search_by_vat`: the number and the query country code. The three enrichment actions: the corresponding identifier. |
| Answer | a structured document holding a `data` entry — a list of suggestions for the two search actions, one record for the three enrichment actions — and optionally an `error` entry or a `credit_error` entry |
| Failure mapping | no account token → the text `No account token`; insufficient credit, or the "test mode" refusal → `Insufficient Credit`; a transport, access or usage failure → the failure's own text |
| Suppression | the call raises immediately with the text `Test mode` while a test is running, so no test ever contacts the service |

The suggestion payload is normalised before use; see [workflows.md](workflows.md) §13.3.

## 8.2 The tax-number verification intermediary

| Aspect | Contract |
|---|---|
| Address | one of exactly two hosts — a production one and a test one — chosen by whether the tax-number package carries demonstration data, and overridable by the system parameter `iap_vies.endpoint`. Any other value fails with `Invalid IAP VIES endpoint`. |
| Request a check | a form-encoded request to the path `/api/vies/1/check_validity` carrying: the tax registration number; the installation's unique identifier; the client identifier and the client secret; the address at which the service should call back; and a signed token valid for seven days. Time budget twenty seconds. |
| Answer | a structured document carrying a `status` entry whose value is one of `valid`, `unassigned`, `pending`, `fault`. A transport failure, or an answer with no status, is treated as `fault`. |
| Poll for updates | a form-encoded request to the path `/api/vies/1/check_update` carrying the installation's unique identifier, the client identifier and the client secret. Time budget ten seconds. |
| Answer | a map from tax registration number to status. Each entry is applied to every Party carrying that number. |
| Call back | the service posts to `/base_vat/1/webhook_update_vies` with the signed token and a status. |
| Credentials | generated on first use, in a separate transaction; see [configuration.md](configuration.md) §2. When the polling job does not exist, or a test is running, the fixed pair `dummy_identifier` and `dummy_token` is used and the service ignores it. |

The enrichment behaviour additionally calls the union's own verification service **directly**, as
a fall-back when the enrichment service fails a tax-number lookup; the answer carries a validity
flag, a name and a multi-line address, which is decomposed as described in
[workflows.md](workflows.md) §13.1.

## 8.3 The geocoding services

| Provider | Forward lookup | Reverse lookup |
|---|---|---|
| the open mapping service | a request to the service's search path carrying a format marker asking for a structured document and the query string; the first result's latitude and longitude are taken; a non-success status is logged as a warning but the body is still parsed | a request to the service's reverse path carrying a format marker, the latitude and the longitude, with a ten-second budget; the address block of the answer supplies the country code, the city — preferring in order the city district, the town, the village and the city — and the postal code |
| the commercial mapping service | a request to the service's geocoding path carrying a sensor marker set to false, the address and the key; when the caller forces a country, a components filter naming that country code is added. A status of "zero results" yields nothing; any other non-success status raises; otherwise the first result's geometry supplies the latitude and the longitude | not implemented |

Both forward lookups identify the caller with a fixed user-agent string naming the platform and its
contact page. The reverse lookup refuses to run while tests are running, with the message
`OpenStreetMap calls disabled in testing environment.`

---

# 9. Import and export

## 9.1 The import template

The Party entity advertises exactly one import template: the label
`Import Template for Contacts` and a shipped spreadsheet at a fixed path inside the foundation
package's static files.

## 9.2 Import behaviour specific to this domain

| Behaviour | Effect |
|---|---|
| state/country repair | every row naming a state whose country differs from the row's country has the state replaced by a same-coded state in the right country, or emptied |
| verification suppressed | the intra-community validity computation is removed from the queue on both create and write |
| geocoding suppressed | the resolution operation returns immediately |
| country-package installation suppressed | creating a Company during an import does not install packages |

Everything else behaves as an ordinary create: every validation runs, the field synchronization
runs, and the tax-number check runs in raising mode.

## 9.3 Exportable fields

Every stored field is exportable. One field is marked as **default export compatible** — the name
— which means it is offered first in the export field picker and is the field a bare export
produces.

Three fields deserve a note for a rebuild's export contract:

- the **complete name** and the **display name** are computed; the complete name is stored and
  therefore exportable, the display name is not stored and is exported by computation;
- the **barcode** is company-dependent: an export produces the acting company's value;
- the **avatar** fields are computed and can be exported, but they will contain either the stored
  image, a generated image or a placeholder, so an export followed by an import round-trips a
  placeholder into a real stored image. A rebuild should export the stored image field instead.

## 9.4 What other domains import into this one

| Source | What it creates |
|---|---|
| an inbound electronic mail message | a Party through the find-or-create procedure |
| a received electronic invoice | a Party, and possibly a Bank Account through the find-or-create procedure of [calculations.md](calculations.md) §7.3 |
| a storefront checkout | a Party for the buyer and, when the buyer gives a different shipping address, a `delivery`-type child |
| a portal sign-up | a Party for the new account |
| creating a Company | a Party carrying the company's identity |
| creating a user account | a Party for the user |

Each of those paths must honour every rule of this domain: the check constraint on the name, the
synchronization, the tax-number validation and the duplicate warnings.
