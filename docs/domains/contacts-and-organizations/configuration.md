# Configuration of the Contacts and Organizations domain

Settings, system parameters, sequences, default records, security groups, the access-rights
matrix, record rules, scheduled jobs and the complete tables of shipped reference data.

---

# 1. Settings exposed on configuration screens

The domain contributes four settings. None of them is stored on a settings record of its own: each
is either a field of the Company or a system parameter written through the settings screen.

| Setting (storage name) | Where stored | Default | Meaning |
|---|---|---|---|
| Verify tax registration numbers (`vat_check_vies`) | a field of the Company; company-dependent on the settings screen | cleared | When set, writing a tax registration number calls the customs-union verification service and records the answer on the Party. Also switches on the per-Party "perform verification" computation. Shown in the accounting settings next to the customs-union setting, with the help text `Verify VAT numbers using the European VIES service` and the explanation `If this checkbox is ticked, the default fiscal position that applies will depend upon the output of the verification by the European VIES Service.` |
| Geocoding provider (`geoloc_provider_id`) | the system parameter `base_geolocalize.geo_provider` | the first provider in the table | Which external service resolves an address into coordinates. Labelled `API` on the screen. |
| Geocoding provider technical name (`geoloc_provider_techname`) | derived, read-only | — | Mirrors the chosen provider's technical name so that the screen can reveal the key input only for the second provider. |
| Geocoding service key (`geoloc_provider_googlemap_key`) | the system parameter `base_geolocalize.google_map_api_key` | empty | The key for the second provider. Labelled `Key`. Its help text points at the provider's own key documentation. |

The enrichment behaviour adds one derived, non-stored indicator to the settings screen:

| Indicator (storage name) | Meaning |
|---|---|
| Insufficient credit (`partner_autocomplete_insufficient_credit`) | Computed, true when the enrichment service account's credit balance is zero or below. A button beside it opens the service's top-up page in a new window. |

---

# 2. System parameters

| Key | Shipped value | Set by | Meaning |
|---|---|---|---|
| `base_geolocalize.geo_provider` | none (the first provider is used) | the settings screen | The internal identifier of the chosen geocoding provider. |
| `base_geolocalize.google_map_api_key` | none | the settings screen | The key for the second geocoding provider. |
| `iap.partner_autocomplete.endpoint` | none (a built-in default address is used) | manually | Overrides the address of the enrichment service. |
| `iap_vies.client_identifier` | none; generated on first use | automatically | A universally unique identifier that identifies this installation to the verification intermediary, so that only this installation can poll for updates on checks it requested. |
| `iap_vies.client_token` | none; generated on first use | automatically | A secret paired with the identifier above. |
| `iap_vies.endpoint` | none; the production address is used, or the test address when the tax-number package is running with demonstration data | manually | Overrides the address of the verification intermediary. Only the production and test addresses are accepted; any other value fails with `Invalid IAP VIES endpoint`. |
| `database.uuid` | generated at installation | automatically | Sent with every call to the enrichment and verification services so that the installation can be identified. |
| `base.template_portal_user_id` | the shipped portal user template | shipped | The template account copied when a portal account is created. Relevant here because creating a portal account creates or reuses a Party. |

**Generating the verification credentials.** When the identifier and the secret are absent, they
are created **in a separate transaction** so that a later failure in the caller's transaction does
not roll back credentials that the remote service has already recorded. The procedure:

1. If the scheduled polling job does not exist, or tests are running, return the fixed pair
   (`dummy_identifier`, `dummy_token`), which the service ignores.
2. Read the two parameters. If both exist, return them.
3. Open a new transaction, re-read them there (another request may have created them meanwhile),
   and if they are still absent generate a universally unique identifier and a
   uniform-resource-locator-safe random secret, and store both.
4. Return them.

---

# 3. Security groups and the access-rights matrix

## 3.1 Groups

| Group | Displayed name | Implied by | Implies | Notes |
|---|---|---|---|---|
| Internal user | `Role / User` | — | — | Carries the application-programming-interface key duration of ninety days. Access to the home menu. |
| Portal | `Role / Portal` | — | — | External counterparties with a sign-in. |
| Public | `Role / Public` | — | — | Visitors with no sign-in. |
| Contact creation | `Creation`, under the privilege `Contact` in the category `Master Data` | the administrator group | — | The domain's own maintainer group. Granted to the built-in super-user. |
| Administrator | `Role / Administrator` | — | the access-rights group and the sanitisation-bypass group | Granted to the built-in super-user and to the shipped administrator account. |
| Access rights | `Access Rights` | the administrator group | the internal-user group | Owns the Company entity. |
| Multi-company | `Multi Companies` | — | — | Reveals the company field and the company switcher. |
| Multi-currency | `Multi Currencies` | — | — | Granted to or withdrawn from the internal-user group automatically as currencies are activated and deactivated. |
| Technical features | `Technical Features` | the internal-user and administrator groups | — | Reveals technical inputs. |
| Export allowed | `Allowed`, under the privilege `Export` in the category `Master Data` | the administrator group | — | Controls whether a user may export records. |

## 3.2 Access-rights matrix

Read as: for the entity in the row and the group in the column, which of create, read, update and
delete are permitted. A dash means the group has no entry for the entity and therefore no access
at all through this mechanism.

| Entity | Public | Portal | Internal user | Contact creation | Administrator | Access rights |
|---|---|---|---|---|---|---|
| Party (`res.partner`) | read | read | read | create, read, update, delete | *(through contact creation)* | *(through contact creation)* |
| Party Tag (`res.partner.category`) | — | — | read | create, read, update, delete | *(through contact creation)* | — |
| Industry (`res.partner.industry`) | — | — | read | *(through the internal-user entry)* | create, read, update, delete | — |
| Bank (`res.bank`) | — | — | read | create, read, update, delete | create, read, update, delete | — |
| Bank Account (`res.partner.bank`) | — | — | read | create, read, update, delete | *(through contact creation)* | — |
| Country (`res.country`) | read | read | read | read | create, read, update, delete | — |
| Country State (`res.country.state`) | read | read | read | create, read, update, delete | *(through contact creation)* | — |
| Country Group (`res.country.group`) | read | read | read | create, read, update, delete | *(through contact creation)* | — |
| City (`res.city`) | — | — | read | create, read, update, delete | *(through contact creation)* | — |
| Currency (`res.currency`) | read | read | read | — | create, read, update, delete | — |
| Currency Rate (`res.currency.rate`) | read | read | read | — | create, read, update, delete | — |
| Language (`res.lang`) | read | read | read | — | create, read, update, delete | — |
| Company (`res.company`) | read | read | read | — | *(through access rights)* | create, read, update, delete |
| Geocoding Provider (`base.geo_provider`) | — | — | read | — | — | — |
| Blocked Number (`phone.blacklist`) | none (an explicit entry granting nothing) | — | — | — | create, read, update, delete | — |
| Unblock Number Wizard (`phone.blacklist.remove`) | — | — | — | — | create, read, update, delete | — |

Three entries deserve comment.

- **The blocked-number list has an explicit "grant nothing" entry** for every user with no group.
  This is how the platform refuses access to an entity without leaving it unreferenced.
- **The Industry** is readable by every internal user and writable only by the administrator,
  unlike the Party Tag, which the contact-creation group may edit.
- **The Company** is the only entity in the domain whose write access belongs to the access-rights
  group rather than the contact-creation group.

## 3.3 Record rules

| Entity | Rule name | Groups | Kind | Filter | Operations |
|---|---|---|---|---|---|
| Party | multi-company | everybody | global | the Party is a shared party, **or** its company is one of the reader's companies or an ancestor of one, **or** it has no company | all four |
| Party | portal and public subtree | portal, public | group-specific | the Party is the reader's commercial entity or a descendant of it | read only — create, update and delete are excluded |
| Bank Account | company | everybody | global | the account's company is one of the reader's companies or an ancestor of one, **or** it has no company | all four |
| Currency Rate | company | everybody | global | the rate's company is one of the reader's companies or an ancestor of one, **or** it has no company | all four |
| Company | portal | portal | group-specific | the company is in the reader's allowed set | all four |
| Company | internal user | internal user | group-specific | the company is in the reader's allowed set | all four |
| Company | public | public | group-specific | the company is in the reader's allowed set | all four |
| Company | access rights | access rights | group-specific | unrestricted | all four |

A **global** rule is intersected with every other rule; a **group-specific** rule is united with
the other group-specific rules that apply to the reader. So a portal visitor sees the intersection
of the multi-company rule and the subtree rule.

---

# 4. Sequences and numbering

**The domain defines no sequence and no numbering format.** A Party has no document number. Its
only stable identifiers are:

- the internal identifier, allocated by the storage engine;
- the free-text reference field, which the business fills itself and which has no uniqueness rule;
- the tax registration number and the company registration number, which come from outside the
  system and are validated, not generated;
- the barcode, which the business fills itself and which is unique per company.

A rebuild must **not** invent a customer-number sequence: every other domain that needs a stable
external reference uses the reference field or its own document numbering.

---

# 5. Scheduled jobs

| Job | Runs | Entity | What it does |
|---|---|---|---|
| Synchronise verification updates | every day, as the built-in super-user, active by default | Party | Calls the verification intermediary for updates on previously requested tax-number checks, groups the returned answers by tax registration number, and applies each answer to every Party carrying that number, writing a history entry on each. Installed only with the tax-number behaviour. Its very existence is what makes the system ask the intermediary for real credentials rather than the dummy pair. |

Two jobs from the platform's own maintenance set touch this domain indirectly and are named here
so a rebuild knows they exist:

| Job | Runs | Relevance |
|---|---|---|
| Automatic vacuum of internal data | every day | Removes expired transient records, including finished merge wizards and their groups. |
| Portal account deletion | every day | Deletes requested portal accounts; where the telephone behaviour is installed, the deletion path blocks the deleted account's telephone numbers when the visitor asked for it. |

**The domain has no scheduled job that touches parties, addresses, banks, countries, currencies or
languages.** Nothing expires; nothing is recomputed on a timer.

---

# 6. Shipped default records

| Record | Entity | Values |
|---|---|---|
| The main Party | Party | name `My Company`; organization flag set; company left empty; street, city, postal code and telephone all empty strings; the stored image set to the shipped default logo file. Created with the organization default in its context. Marked as not to be overwritten by later package updates. |
| The system Party | Party | name `System`; company set to the main company; electronic mail address `odoobot@example.com`; **archived**. This is the Party of the built-in super-user. |
| The administrator Party | Party | name `Administrator`; company set to the main company. |
| The public Party | Party | name `Public user`; **archived**. This is the Party of the built-in public user. |
| The main company | Company | name `My Company`; Party set to the main Party; currency set to the currency of the United States. |
| The twenty-one industries | Industry | see §8.6. |

Two further shipped parties exist only when demonstration data is installed, together with a set
of example organizations, contacts and addresses; a rebuild need not reproduce them.

**The main company's currency.** The shipped main company keeps the currency of the United States.
Because a Company's currency is activated on write, that currency is active in every installation
even though the shipped currency table marks almost every currency inactive.

---

# 7. Caches a rebuild must reproduce

The domain relies on four caches. Getting their invalidation wrong produces stale behaviour that is
extremely hard to diagnose, so each is listed with its exact invalidation trigger.

| Cache | Holds | Cleared by |
|---|---|---|
| Stable cache — calling code by country code | the calling code of each country | creating a country; writing a country's code or calling code; deleting a country |
| Stable cache — active language data | a read-only snapshot of every active language, keyed by each of the fourteen cached fields | creating, writing or deleting any language (every write also flushes first) |
| Stable cache — active currencies | the list of all currencies for the client | creating or deleting a currency; writing a currency's active flag, digits, code, symbol position or symbol |
| Interface template cache | compiled forms and printable templates | writing a country's input view or tax registration label |
| Whole cache | everything | creating or deleting a Company; writing a Company's active flag or sequence |
| Compiled style-sheet cache | the generated document style sheet | writing a Company's font, primary colour, secondary colour or document template |
| Accessible-branch cache, keyed by (allowed companies, company, user) | the branches of a company that the user may act for | the whole-cache clears above |
| Company-party cache | the identifiers of the parties of every Company | the whole-cache clears above |
| Telephone-index-exists cache, keyed by table name | whether the supporting index for the sanitised-number search exists | never, within a process |

---

# 8. Shipped reference data

Every table below is the complete shipped set. Values are reproduced exactly.

## 8.1 Countries

Two hundred and fifty-one countries ship. The columns are: the name; the two-letter code; the
international calling code; the currency; whether a state is required in an address; whether a
postal code is required; the country's own word for the tax registration number; where the
counterparty's name is placed relative to the address block; whether the controlled city list is
enforced; whether the country replaces the address input block with its own view; and the address
layout when it differs from the default.

The **default layout**, used wherever the layout column is blank, is:

```
%(street)s
%(street2)s
%(city)s %(state_code)s %(zip)s
%(country_name)s
```

In the table the line feeds inside a layout are written as the two characters `\n`.

| Country name | Code | Calling code | Currency | State required | Postal code required | Tax label | Name position | Enforces cities | Own input view | Address layout (blank = the default layout) |
|---|---|---|---|---|---|---|---|---|---|---|
| Afghanistan | AF | 93 | AFN | no | yes |  | before | no | no |  |
| Albania | AL | 355 | ALL | no | yes |  | before | no | no |  |
| Algeria | DZ | 213 | DZD | no | yes |  | before | no | no |  |
| American Samoa | AS | 1684 | USD | no | yes |  | before | no | no |  |
| Andorra | AD | 376 | EUR | no | yes |  | before | no | no |  |
| Angola | AO | 244 | AOA | no | no |  | before | no | no |  |
| Anguilla | AI | 1264 | XCD | no | yes |  | before | no | no |  |
| Antarctica | AQ | 672 | XCD | no | yes |  | before | no | no |  |
| Antigua and Barbuda | AG | 1268 | XCD | no | yes |  | before | no | no |  |
| Argentina | AR | 54 | ARS | yes | yes | CUIT | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Armenia | AM | 374 | AMD | yes | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s\n%(country_name)s |
| Aruba | AW | 297 | AWG | no | yes |  | before | no | no |  |
| Australia | AU | 61 | AUD | yes | yes | ABN | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_code)s %(zip)s\n%(country_name)s |
| Austria | AT | 43 | EUR | no | yes | USt | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Azerbaijan | AZ | 994 | AZN | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Bahamas | BS | 1242 | BSD | no | yes |  | before | no | no |  |
| Bahrain | BH | 973 | BHD | no | yes |  | before | no | no |  |
| Bangladesh | BD | 880 | BDT | no | yes |  | before | no | no |  |
| Barbados | BB | 1246 | BBD | no | yes |  | before | no | no |  |
| Belarus | BY | 375 | BYN | no | yes |  | before | no | no |  |
| Belgium | BE | 32 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Belize | BZ | 501 | BZD | no | no |  | before | no | no |  |
| Benin | BJ | 229 | XOF | no | no |  | before | no | no |  |
| Bermuda | BM | 1441 | BMD | no | yes |  | before | no | no |  |
| Bhutan | BT | 975 | BTN | no | yes |  | before | no | no |  |
| Bolivia | BO | 591 | BOB | no | yes |  | before | no | no |  |
| Bonaire, Sint Eustatius and Saba | BQ | 599 | USD | no | yes |  | before | no | no |  |
| Bosnia and Herzegovina | BA | 387 | BAM | no | yes |  | before | no | no |  |
| Botswana | BW | 267 | BWP | no | yes |  | before | no | no |  |
| Bouvet Island | BV | 55 | NOK | no | yes |  | before | no | no |  |
| Brazil | BR | 55 | BRL | no | yes |  | before | no | yes | %(street)s\n%(street2)s\n%(city)s %(state_code)s\n%(zip)s\n%(country_name)s |
| British Indian Ocean Territory | IO | 246 | USD | no | yes |  | before | no | no |  |
| Brunei Darussalam | BN | 673 | BND | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(zip)s\n%(country_name)s |
| Bulgaria | BG | 359 | EUR | no | yes | VAT | before | no | no |  |
| Burkina Faso | BF | 226 | XOF | no | yes |  | before | no | no |  |
| Burundi | BI | 257 | BIF | no | yes |  | before | no | no |  |
| Cambodia | KH | 855 | KHR | no | yes |  | before | no | no |  |
| Cameroon | CM | 237 | XAF | no | yes |  | before | no | no |  |
| Canada | CA | 1 | CAD | yes | yes | GST/HST number | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_code)s %(zip)s\n%(country_name)s |
| Cape Verde | CV | 238 | CVE | no | yes |  | before | no | no |  |
| Cayman Islands | KY | 1345 | KYD | no | yes |  | before | no | no |  |
| Central African Republic | CF | 236 | XAF | no | yes |  | before | no | no |  |
| Chad | TD | 235 | XAF | no | yes |  | before | no | no |  |
| Chile | CL | 56 | CLP | no | no | RUT | before | no | no |  |
| China | CN | 86 | CNY | no | yes |  | before | yes | no | %(country_name)s, %(zip)s\n%(state_name)s %(city)s %(street)s %(street2)s |
| Christmas Island | CX | 61 | AUD | no | yes |  | before | no | no |  |
| Cocos (Keeling) Islands | CC | 61 | AUD | no | yes |  | before | no | no |  |
| Colombia | CO | 57 | COP | no | yes | NIT | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Comoros | KM | 269 | KMF | no | yes |  | before | no | no |  |
| Congo (DRC) | CD | 243 | CDF | no | yes |  | before | no | no |  |
| Congo (Republic) | CG | 242 | XAF | no | yes |  | before | no | no |  |
| Cook Islands | CK | 682 | NZD | no | yes |  | before | no | no |  |
| Costa Rica | CR | 506 | CRC | no | yes |  | before | no | no |  |
| Croatia | HR | 385 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s \n%(country_name)s |
| Cuba | CU | 53 | CUP | no | yes |  | before | no | no |  |
| Curaçao | CW | 599 | XCG | no | yes |  | before | no | no |  |
| Cyprus | CY | 357 | EUR | no | yes | VAT | before | no | no |  |
| Czech Republic | CZ | 420 | CZK | no | yes | VAT | before | no | no |  |
| Côte d'Ivoire | CI | 225 | XOF | no | yes |  | before | no | no |  |
| Denmark | DK | 45 | DKK | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Djibouti | DJ | 253 | DJF | no | yes |  | before | no | no |  |
| Dominica | DM | 1767 | XCD | no | yes |  | before | no | no |  |
| Dominican Republic | DO | 1849 | DOP | no | yes | RNC | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Ecuador | EC | 593 | USD | no | no | RUC | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(country_name)s |
| Egypt | EG | 20 | EGP | no | yes |  | before | no | yes | %(l10n_eg_building_no)s %(street)s\n%(city)s %(state_name)s\n%(zip)s\n%(country_name)s |
| El Salvador | SV | 503 | SVC | no | yes |  | before | no | no |  |
| Equatorial Guinea | GQ | 240 | XAF | no | yes |  | before | no | no |  |
| Eritrea | ER | 291 | ERN | no | yes |  | before | no | no |  |
| Estonia | EE | 372 | EUR | no | yes | VAT | before | no | no |  |
| Eswatini | SZ | 268 | SZL | no | yes |  | before | no | no |  |
| Ethiopia | ET | 251 | ETB | no | yes |  | before | no | no |  |
| Falkland Islands | FK | 500 | FKP | no | yes |  | before | no | no |  |
| Faroe Islands | FO | 298 | DKK | no | yes |  | before | no | no |  |
| Fiji | FJ | 679 | FJD | no | yes |  | before | no | no |  |
| Finland | FI | 358 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| France | FR | 33 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| French Guiana | GF | 594 | EUR | no | yes |  | before | no | no |  |
| French Polynesia | PF | 689 | XPF | no | yes | VAT | before | no | no |  |
| French Southern Territories | TF | 262 | EUR | no | yes |  | before | no | no |  |
| Gabon | GA | 241 | XAF | no | yes |  | before | no | no |  |
| Gambia | GM | 220 | GMD | no | yes |  | before | no | no |  |
| Georgia | GE | 995 | GEL | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Germany | DE | 49 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Ghana | GH | 233 | GHS | no | yes |  | before | no | no |  |
| Gibraltar | GI | 350 | GIP | no | yes |  | before | no | no |  |
| Greece | GR | 30 | EUR | no | yes | VAT | before | no | no |  |
| Greenland | GL | 299 | DKK | no | yes |  | before | no | no |  |
| Grenada | GD | 1473 | XCD | no | yes |  | before | no | no |  |
| Guadeloupe | GP | 590 | EUR | no | yes |  | before | no | no |  |
| Guam | GU | 1671 | USD | no | yes |  | before | no | no |  |
| Guatemala | GT | 502 | GTQ | no | yes | NIT | before | no | no |  |
| Guernsey | GG | 44 | GBP | no | yes |  | before | no | no |  |
| Guinea | GN | 224 | GNF | no | yes |  | before | no | no |  |
| Guinea-Bissau | GW | 245 | XOF | no | yes |  | before | no | no |  |
| Guyana | GY | 592 | GYD | no | yes |  | before | no | no |  |
| Haiti | HT | 509 | HTG | no | yes |  | before | no | no |  |
| Heard Island and McDonald Islands | HM | 672 | AUD | no | yes |  | before | no | no |  |
| Holy See (Vatican City State) | VA | 379 | EUR | no | yes |  | before | no | no |  |
| Honduras | HN | 504 | HNL | no | yes | RTN | before | no | no |  |
| Hong Kong | HK | 852 | HKD | no | no |  | before | no | no |  |
| Hungary | HU | 36 | HUF | no | yes | VAT | before | no | no |  |
| Iceland | IS | 354 | ISK | no | yes |  | before | no | no |  |
| India | IN | 91 | INR | yes | yes | GSTIN | before | no | no | %(street)s\n%(street2)s\n%(city)s %(zip)s\n%(state_name)s %(state_code)s\n%(country_name)s |
| Indonesia | ID | 62 | IDR | yes | yes | NPWP | before | no | no |  |
| Iran | IR | 98 | IRR | no | yes |  | before | no | no |  |
| Iraq | IQ | 964 | IQD | no | yes |  | before | no | no |  |
| Ireland | IE | 353 | EUR | no | no | VAT | before | no | no |  |
| Isle of Man | IM | 44 | GBP | no | yes |  | before | no | no |  |
| Israel | IL | 972 | ILS | no | yes |  | before | no | no |  |
| Italy | IT | 39 | EUR | yes | yes | VAT | before | no | no |  |
| Jamaica | JM | 1876 | JMD | no | yes |  | before | no | no |  |
| Japan | JP | 81 | JPY | yes | yes |  | after | no | no | %(zip)s\n%(state_name)s %(city)s\n%(street)s\n%(street2)s\n%(country_name)s |
| Jersey | JE | 44 | GBP | no | yes |  | before | no | no |  |
| Jordan | JO | 962 | JOD | no | yes |  | before | no | no |  |
| Kazakhstan | KZ | 7 | KZT | yes | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s\n%(country_name)s |
| Kenya | KE | 254 | KES | no | yes |  | before | no | no |  |
| Kiribati | KI | 686 | AUD | no | yes |  | before | no | no |  |
| Kosovo | XK | 383 | EUR | no | yes |  | before | no | no |  |
| Kuwait | KW | 965 | KWD | no | yes |  | before | no | no |  |
| Kyrgyzstan | KG | 996 | KGS | yes | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s\n%(country_name)s |
| Laos | LA | 856 | LAK | no | yes |  | before | no | no |  |
| Latvia | LV | 371 | EUR | no | yes | VAT | before | no | no |  |
| Lebanon | LB | 961 | LBP | no | yes |  | before | no | no |  |
| Lesotho | LS | 266 | LSL | no | yes |  | before | no | no |  |
| Liberia | LR | 231 | LRD | no | yes |  | before | no | no |  |
| Libya | LY | 218 | LYD | no | yes |  | before | no | no |  |
| Liechtenstein | LI | 423 | CHF | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Lithuania | LT | 370 | EUR | no | yes | VAT | before | no | no |  |
| Luxembourg | LU | 352 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s \n%(country_name)s |
| Macau | MO | 853 | MOP | no | no |  | before | no | no |  |
| Madagascar | MG | 261 | MGA | no | yes |  | before | no | no |  |
| Malawi | MW | 265 | MWK | no | yes |  | before | no | no |  |
| Malaysia | MY | 60 | MYR | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Maldives | MV | 960 | MVR | no | yes |  | before | no | no |  |
| Mali | ML | 223 | XOF | no | yes |  | before | no | no |  |
| Malta | MT | 356 | EUR | no | yes | VAT | before | no | no |  |
| Marshall Islands | MH | 692 | USD | no | yes |  | before | no | no |  |
| Martinique | MQ | 596 | EUR | no | yes |  | before | no | no |  |
| Mauritania | MR | 222 | MRU | no | yes |  | before | no | no |  |
| Mauritius | MU | 230 | MUR | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_code)s %(zip)s\n%(country_name)s |
| Mayotte | YT | 262 | EUR | no | yes |  | before | no | no |  |
| Mexico | MX | 52 | MXN | yes | yes | RFC | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s, %(state_code)s\n%(country_name)s |
| Micronesia | FM | 691 | USD | no | yes |  | before | no | no |  |
| Moldova | MD | 373 | MDL | no | yes |  | before | no | no |  |
| Monaco | MC | 377 | EUR | no | yes |  | before | no | no |  |
| Mongolia | MN | 976 | MNT | no | yes |  | before | no | no |  |
| Montenegro | ME | 382 | EUR | no | yes |  | before | no | no |  |
| Montserrat | MS | 1664 | XCD | no | yes |  | before | no | no |  |
| Morocco | MA | 212 | MAD | no | yes |  | before | no | no |  |
| Mozambique | MZ | 258 | MZN | no | yes | NUIT | before | no | no |  |
| Myanmar | MM | 95 | MMK | no | yes |  | before | no | no |  |
| Namibia | NA | 264 | NAD | no | yes |  | before | no | no |  |
| Nauru | NR | 674 | AUD | no | yes |  | before | no | no |  |
| Nepal | NP | 977 | NPR | no | yes |  | before | no | no |  |
| Netherlands | NL | 31 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| New Caledonia | NC | 687 | XPF | no | yes |  | before | no | no |  |
| New Zealand | NZ | 64 | NZD | no | yes | GST | before | no | no |  |
| Nicaragua | NI | 505 | NIO | no | yes |  | before | no | no |  |
| Niger | NE | 227 | XOF | no | yes |  | before | no | no |  |
| Nigeria | NG | 234 | NGN | no | yes |  | before | no | no |  |
| Niue | NU | 683 | NZD | no | yes |  | before | no | no |  |
| Norfolk Island | NF | 672 | AUD | no | yes |  | before | no | no |  |
| North Korea | KP | 850 | KPW | no | yes |  | before | no | no |  |
| North Macedonia | MK | 389 | MKD | no | yes |  | before | no | no |  |
| Northern Ireland | XI | 44 | GBP | no | yes |  | before | no | no |  |
| Northern Mariana Islands | MP | 1670 | USD | no | yes |  | before | no | no |  |
| Norway | NO | 47 | NOK | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Oman | OM | 968 | OMR | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Pakistan | PK | 92 | PKR | no | yes | NTN | before | no | no |  |
| Palau | PW | 680 | USD | no | yes |  | before | no | no |  |
| Panama | PA | 507 | PAB | no | yes | RUC | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| Papua New Guinea | PG | 675 | PGK | no | yes |  | before | no | no |  |
| Paraguay | PY | 595 | PYG | no | yes |  | before | no | no |  |
| Peru | PE | 51 | PEN | no | no | RUC | before | yes | yes | %(street)s\n%(l10n_pe_district_name)s\n%(zip)s%(city)s\n%(state_name)s\n%(country_name)s |
| Philippines | PH | 63 | PHP | no | yes |  | before | no | no |  |
| Pitcairn Islands | PN | 64 | NZD | no | yes |  | before | no | no |  |
| Poland | PL | 48 | PLN | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Portugal | PT | 351 | EUR | no | yes | VAT | before | no | no |  |
| Puerto Rico | PR | 1939 | USD | no | yes |  | before | no | no |  |
| Qatar | QA | 974 | QAR | no | yes |  | before | no | no |  |
| Romania | RO | 40 | RON | no | yes | VAT | before | no | no |  |
| Russian Federation | RU | 7 | RUB | no | yes |  | before | no | no |  |
| Rwanda | RW | 250 | RWF | no | yes |  | before | no | no |  |
| Réunion | RE | 262 | EUR | no | yes |  | before | no | no |  |
| Saint Barthélémy | BL | 590 | EUR | no | yes |  | before | no | no |  |
| Saint Helena, Ascension and Tristan da Cunha | SH | 290 | SHP | no | yes |  | before | no | no |  |
| Saint Kitts and Nevis | KN | 1869 | XCD | no | yes |  | before | no | no |  |
| Saint Lucia | LC | 1758 | XCD | no | yes |  | before | no | no |  |
| Saint Martin (French part) | MF | 590 | EUR | no | yes |  | before | no | no |  |
| Saint Pierre and Miquelon | PM | 508 | EUR | no | yes |  | before | no | no |  |
| Saint Vincent and the Grenadines | VC | 1784 | XCD | no | yes |  | before | no | no |  |
| Samoa | WS | 685 | WST | no | yes |  | before | no | no |  |
| San Marino | SM | 378 | EUR | no | yes |  | before | no | no |  |
| Saudi Arabia | SA | 966 | SAR | no | yes | VAT Number | before | no | yes | %(street)s\n%(street2)s\n%(city)s %(state_code)s %(zip)s\n%(l10n_sa_edi_building_number)s %(l10n_sa_edi_plot_identification)s\n%(country_name)s |
| Senegal | SN | 221 | XOF | no | yes |  | before | no | no |  |
| Serbia | RS | 381 | RSD | no | yes |  | before | no | no |  |
| Seychelles | SC | 248 | SCR | no | yes |  | before | no | no |  |
| Sierra Leone | SL | 232 | SLE | no | yes |  | before | no | no |  |
| Singapore | SG | 65 | SGD | no | yes | GST No. | before | no | no |  |
| Sint Maarten (Dutch part) | SX | 1721 | XCG | no | yes |  | before | no | no |  |
| Slovakia | SK | 421 | EUR | no | yes | VAT | before | no | no |  |
| Slovenia | SI | 386 | EUR | no | yes | VAT | before | no | yes | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Solomon Islands | SB | 677 | SBD | no | yes |  | before | no | no |  |
| Somalia | SO | 252 | SOS | no | yes |  | before | no | no |  |
| South Africa | ZA | 27 | ZAR | no | yes |  | before | no | no |  |
| South Georgia and the South Sandwich Islands | GS | 500 | GBP | no | yes |  | before | no | no |  |
| South Korea | KR | 82 | KRW | no | yes |  | before | no | yes | %(country_name)s %(state_name)s\n%(city)s %(street2)s %(street)s %(zip)s |
| South Sudan | SS | 211 | SSP | no | yes |  | before | no | no |  |
| Spain | ES | 34 | EUR | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(state_name)s\n%(country_name)s |
| Sri Lanka | LK | 94 | LKR | no | yes |  | before | no | no |  |
| State of Palestine | PS | 970 | ILS | no | yes |  | before | no | no |  |
| Sudan | SD | 249 | SDG | no | yes |  | before | no | no |  |
| Suriname | SR | 597 | SRD | no | yes |  | before | no | no |  |
| Svalbard and Jan Mayen | SJ | 47 | NOK | no | yes |  | before | no | no |  |
| Sweden | SE | 46 | SEK | no | yes | VAT | before | no | yes | %(street)s\n%(street2)s\n%(zip)s %(city)s %(state_code)s\n%(country_name)s |
| Switzerland | CH | 41 | CHF | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s |
| Syria | SY | 963 | SYP | no | yes |  | before | no | no |  |
| São Tomé and Príncipe | ST | 239 | STD | no | yes |  | before | no | no |  |
| Taiwan | TW | 886 | TWD | no | yes |  | before | yes | no |  |
| Tajikistan | TJ | 992 | TJS | yes | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s\n%(country_name)s |
| Tanzania | TZ | 255 | TZS | no | yes |  | before | no | no |  |
| Thailand | TH | 66 | THB | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s %(zip)s\n%(country_name)s |
| Timor-Leste | TL | 670 | USD | no | yes |  | before | no | no |  |
| Togo | TG | 228 | XOF | no | yes |  | before | no | no |  |
| Tokelau | TK | 690 | NZD | no | yes |  | before | no | no |  |
| Tonga | TO | 676 | TOP | no | yes |  | before | no | no |  |
| Trinidad and Tobago | TT | 1868 | TTD | no | yes |  | before | no | no |  |
| Tunisia | TN | 216 | TND | no | yes |  | before | no | no |  |
| Turkmenistan | TM | 993 | TMT | yes | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s\n%(country_name)s |
| Turks and Caicos Islands | TC | 1649 | USD | no | yes |  | before | no | no |  |
| Tuvalu | TV | 688 | AUD | no | yes |  | before | no | no |  |
| Türkiye | TR | 90 | TRY | no | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_name)s %(zip)s\n%(country_name)s |
| USA Minor Outlying Islands | UM | 699 | USD | no | yes |  | before | no | no |  |
| Uganda | UG | 256 | UGX | no | yes | TIN | before | no | no |  |
| Ukraine | UA | 380 | UAH | no | yes |  | before | no | no |  |
| United Arab Emirates | AE | 971 | AED | yes | yes | TRN | before | no | no |  |
| United Kingdom | GB | 44 | GBP | no | yes | VAT | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s\n%(zip)s\n%(country_name)s |
| United States | US | 1 | USD | yes | yes |  | before | no | no | %(street)s\n%(street2)s\n%(city)s %(state_code)s %(zip)s\n%(country_name)s |
| Uruguay | UY | 598 | UYU | no | yes | RUT | before | no | no |  |
| Uzbekistan | UZ | 998 | UZS | no | yes | TIN | before | no | no |  |
| Vanuatu | VU | 678 | VUV | no | yes |  | before | no | no |  |
| Venezuela | VE | 58 | VEF | no | yes |  | before | no | no |  |
| Vietnam | VN | 84 | VND | no | no |  | before | no | no | %(street)s\n%(street2)s\n%(city)s\n%(state_name)s %(country_name)s |
| Virgin Islands (British) | VG | 1284 | USD | no | yes |  | before | no | no |  |
| Virgin Islands (USA) | VI | 1340 | USD | no | yes |  | before | no | no |  |
| Wallis and Futuna | WF | 681 | XPF | no | yes |  | before | no | no |  |
| Western Sahara | EH | 212 | MAD | no | yes |  | before | no | no |  |
| Yemen | YE | 967 | YER | no | yes |  | before | no | no |  |
| Zambia | ZM | 260 | ZMW | no | yes | TPIN | before | no | no |  |
| Zimbabwe | ZW | 263 | ZIG | no | yes |  | before | no | no |  |
| Åland Islands | AX | 358 | EUR | no | yes |  | before | no | no |  |

**Notes on the table.**

- The calling code column is blank for the few territories that share another country's plan or
  have none recorded.
- The postal-code-required column is `yes` by default; the countries showing `no` are those that
  have explicitly cleared the flag.
- A country showing `yes` under "own input view" replaces the whole address input block on the
  Party form with a view of its own, which may add country-specific inputs such as a building
  number or a district.
- The special code `XI` is a customs territory, not a sovereign state; it exists so that the
  tax-number arithmetic can treat that territory's registrations correctly.

## 8.2 Country states

Two thousand one hundred and thirty-four states ship, listed by country code and then by state
code. The pair (country, code) is unique; the same code may recur under different countries.

| Country code | State code | State name |
|---|---|---|
| AE | AJ | Ajman |
| AE | AZ | Abu Dhabi |
| AE | DU | Dubai |
| AE | FU | Fujairah |
| AE | RK | Ras al-Khaimah |
| AE | SH | Sharjah |
| AE | UQ | Umm al-Quwain |
| AM | AM-AG | Aragaçotn |
| AM | AM-AR | Ararat |
| AM | AM-AV | Armavir |
| AM | AM-ER | Erevan |
| AM | AM-GR | Geġarkunik |
| AM | AM-KT | Kotayk |
| AM | AM-LO | Loṙi |
| AM | AM-SH | Širak |
| AM | AM-SU | Syunik |
| AM | AM-TV | Tavuš |
| AM | AM-VD | Vayoć Jor |
| AR | A | Salta |
| AR | B | Buenos Aires |
| AR | C | Ciudad Autónoma de Buenos Aires |
| AR | D | San Luis |
| AR | E | Entre Ríos |
| AR | F | La Rioja |
| AR | G | Santiago Del Estero |
| AR | H | Chaco |
| AR | J | San Juan |
| AR | K | Catamarca |
| AR | L | La Pampa |
| AR | M | Mendoza |
| AR | N | Misiones |
| AR | P | Formosa |
| AR | Q | Neuquén |
| AR | R | Río Negro |
| AR | S | Santa Fe |
| AR | T | Tucumán |
| AR | U | Chubut |
| AR | V | Tierra del Fuego |
| AR | W | Corrientes |
| AR | X | Córdoba |
| AR | Y | Jujuy |
| AR | Z | Santa Cruz |
| AT | 1 | Burgenland |
| AT | 2 | Kärnten |
| AT | 3 | Niederösterreich |
| AT | 4 | Oberösterreich |
| AT | 5 | Salzburg |
| AT | 6 | Steiermark |
| AT | 7 | Tirol |
| AT | 8 | Vorarlberg |
| AT | 9 | Wien |
| AU | ACT | Australian Capital Territory |
| AU | NSW | New South Wales |
| AU | NT | Northern Territory |
| AU | QLD | Queensland |
| AU | SA | South Australia |
| AU | TAS | Tasmania |
| AU | VIC | Victoria |
| AU | WA | Western Australia |
| AZ | AZ-ABS | Abşeron |
| AZ | AZ-AGA | Ağstafa |
| AZ | AZ-AGC | Ağcabədi |
| AZ | AZ-AGM | Ağdam |
| AZ | AZ-AGS | Ağdaş |
| AZ | AZ-AGU | Ağsu |
| AZ | AZ-AST | Astara |
| AZ | AZ-BA | Bakı |
| AZ | AZ-BAB | Babək |
| AZ | AZ-BAL | Balakən |
| AZ | AZ-BAR | Bərdə |
| AZ | AZ-BEY | Beyləqan |
| AZ | AZ-BIL | Biləsuvar |
| AZ | AZ-CAB | Cəbrayıl |
| AZ | AZ-CAL | Cəlilabad |
| AZ | AZ-CUL | Culfa |
| AZ | AZ-DAS | Daşkəsən |
| AZ | AZ-FUZ | Füzuli |
| AZ | AZ-GA | Gəncə |
| AZ | AZ-GAD | Gədəbəy |
| AZ | AZ-GOR | Goranboy |
| AZ | AZ-GOY | Göyçay |
| AZ | AZ-GYG | Göygöl |
| AZ | AZ-HAC | Hacıqabul |
| AZ | AZ-IMI | İmişli |
| AZ | AZ-ISM | İsmayıllı |
| AZ | AZ-KAL | Kəlbəcər |
| AZ | AZ-KAN | Kǝngǝrli |
| AZ | AZ-KUR | Kürdəmir |
| AZ | AZ-LA | Lənkəran |
| AZ | AZ-LAC | Laçın |
| AZ | AZ-LAN | Lənkəran rayon |
| AZ | AZ-LER | Lerik |
| AZ | AZ-MAS | Masallı |
| AZ | AZ-MI | Mingəçevir |
| AZ | AZ-NA | Naftalan |
| AZ | AZ-NEF | Neftçala |
| AZ | AZ-NV | Naxçıvan |
| AZ | AZ-OGU | Oğuz |
| AZ | AZ-ORD | Ordubad |
| AZ | AZ-QAB | Qəbələ |
| AZ | AZ-QAX | Qax |
| AZ | AZ-QAZ | Qazax |
| AZ | AZ-QBA | Quba |
| AZ | AZ-QBI | Qubadlı |
| AZ | AZ-QOB | Qobustan |
| AZ | AZ-QUS | Qusar |
| AZ | AZ-SA | Şəki |
| AZ | AZ-SAB | Sabirabad |
| AZ | AZ-SAD | Sədərək |
| AZ | AZ-SAH | Şahbuz |
| AZ | AZ-SAK | Şəki rayon |
| AZ | AZ-SAL | Salyan |
| AZ | AZ-SAR | Şərur |
| AZ | AZ-SAT | Saatlı |
| AZ | AZ-SBN | Şabran |
| AZ | AZ-SIY | Siyəzən |
| AZ | AZ-SKR | Şəmkir |
| AZ | AZ-SM | Sumqayıt |
| AZ | AZ-SMI | Şamaxı |
| AZ | AZ-SMX | Samux |
| AZ | AZ-SR | Şirvan |
| AZ | AZ-SUS | Şuşa |
| AZ | AZ-TAR | Tərtər |
| AZ | AZ-TOV | Tovuz |
| AZ | AZ-UCA | Ucar |
| AZ | AZ-XA | Xankəndi |
| AZ | AZ-XAC | Xaçmaz |
| AZ | AZ-XCI | Xocalı |
| AZ | AZ-XIZ | Xızı |
| AZ | AZ-XVD | Xocavənd |
| AZ | AZ-YAR | Yardımlı |
| AZ | AZ-YE | Yevlax |
| AZ | AZ-YEV | Yevlax rayon |
| AZ | AZ-ZAN | Zəngilan |
| AZ | AZ-ZAQ | Zaqatala |
| AZ | AZ-ZAR | Zərdab |
| BD | BD-A | Barishal |
| BD | BD-B | Chattogram |
| BD | BD-C | Dhaka |
| BD | BD-D | Khulna |
| BD | BD-E | Rajshahi |
| BD | BD-F | Rangpur |
| BD | BD-G | Sylhet |
| BD | BD-H | Mymensingh |
| BE | VAN | Antwerpen |
| BE | VBR | Vlaams-Brabant |
| BE | VLI | Limburg |
| BE | VOV | Oost-Vlaanderen |
| BE | VWV | West-Vlaanderen |
| BE | WBR | Brabant wallon |
| BE | WHT | Hainaut |
| BE | WLG | Liège |
| BE | WLX | Luxembourg |
| BE | WNA | Namur |
| BN | B | Brunei-Muara |
| BN | K | Belait |
| BN | P | Temburong |
| BN | T | Tutong |
| BR | AC | Acre |
| BR | AL | Alagoas |
| BR | AM | Amazonas |
| BR | AP | Amapá |
| BR | BA | Bahia |
| BR | CE | Ceará |
| BR | DF | Distrito Federal |
| BR | ES | Espírito Santo |
| BR | GO | Goiás |
| BR | MA | Maranhão |
| BR | MG | Minas Gerais |
| BR | MS | Mato Grosso do Sul |
| BR | MT | Mato Grosso |
| BR | PA | Pará |
| BR | PB | Paraíba |
| BR | PE | Pernambuco |
| BR | PI | Piauí |
| BR | PR | Paraná |
| BR | RJ | Rio de Janeiro |
| BR | RN | Rio Grande do Norte |
| BR | RO | Rondônia |
| BR | RR | Roraima |
| BR | RS | Rio Grande do Sul |
| BR | SC | Santa Catarina |
| BR | SE | Sergipe |
| BR | SP | São Paulo |
| BR | TO | Tocantins |
| CA | AB | Alberta |
| CA | BC | British Columbia |
| CA | MB | Manitoba |
| CA | NB | New Brunswick |
| CA | NL | Newfoundland and Labrador |
| CA | NS | Nova Scotia |
| CA | NT | Northwest Territories |
| CA | NU | Nunavut |
| CA | ON | Ontario |
| CA | PE | Prince Edward Island |
| CA | QC | Quebec |
| CA | SK | Saskatchewan |
| CA | YT | Yukon |
| CH | AG | Aargau |
| CH | AG-FR | Argovie |
| CH | AG-IT | Argovia |
| CH | AI | Appenzell Innerrhoden |
| CH | AI-FR | Appenzell Rhodes-Intérieures |
| CH | AI-IT | Appenzello Interno |
| CH | AR | Appenzell Ausserrhoden |
| CH | AR-FR | Appenzell Rhodes-Extérieures |
| CH | AR-IT | Appenzello Esterno |
| CH | BE | Bern |
| CH | BE-FR | Berne |
| CH | BE-IT | Berna |
| CH | BL | Basel-Landschaft |
| CH | BL-FR | Bâle-Campagne |
| CH | BL-IT | Basilea-Campagna |
| CH | BS | Basel-Stadt |
| CH | BS-FR | Bâle-Ville |
| CH | BS-IT | Basilea-Città |
| CH | FR | Freiburg |
| CH | FR-FR | Fribourg |
| CH | FR-IT | Friburgo |
| CH | GE | Genf |
| CH | GE-FR | Genève |
| CH | GE-IT | Ginevra |
| CH | GL | Glarus |
| CH | GL-FR | Glaris |
| CH | GL-IT | Glanora |
| CH | GR | Graubünden |
| CH | GR-FR | Grisons |
| CH | GR-IT | Grigioni |
| CH | JU | Jura |
| CH | JU-IT | Giura |
| CH | LU | Luzern |
| CH | LU-FR | Lucerne |
| CH | LU-IT | Lucerna |
| CH | NE | Neuenburg |
| CH | NE-FR | Neuchâtel |
| CH | NW | Nidwalden |
| CH | NW-FR | Nidwald |
| CH | NW-IT | Nidvaldo |
| CH | OW | Obwalden |
| CH | OW-FR | Obwald |
| CH | OW-IT | Obvaldo |
| CH | SG | St. Gallen |
| CH | SG-FR | Saint-Gall |
| CH | SG-IT | San Gallo |
| CH | SH | Schaffhausen |
| CH | SH-FR | Schaffhouse |
| CH | SH-IT | Sciaffusa |
| CH | SO | Solothurn |
| CH | SO-FR | Soleure |
| CH | SO-IT | Soletta |
| CH | SZ | Schwyz |
| CH | SZ-IT | Svitto |
| CH | TG | Thurgau |
| CH | TG-FR | Thurgovie |
| CH | TG-IT | Turgovia |
| CH | TI | Tessin |
| CH | TI-IT | Ticino |
| CH | UR | Uri |
| CH | VD | Waadt |
| CH | VD-FR | Vaud |
| CH | VS | Wallis |
| CH | VS-FR | Valais |
| CH | VS-IT | Vallese |
| CH | ZG | Zug |
| CH | ZG-FR | Zoug |
| CH | ZG-IT | Zugo |
| CH | ZH | Zürich |
| CH | ZH-FR | Zurich |
| CH | ZH-IT | Zurigo |
| CL | CL-AI | Aysén del Gral. Carlos Ibáñez del Campo |
| CL | CL-AN | Antofagasta |
| CL | CL-AP | Arica y Parinacota |
| CL | CL-AR | de la Araucania |
| CL | CL-AT | Atacama |
| CL | CL-BI | del BíoBio |
| CL | CL-CO | Coquimbo |
| CL | CL-LI | del Libertador Gral. Bernardo O'Higgins |
| CL | CL-LL | de los Lagos |
| CL | CL-LR | Los Ríos |
| CL | CL-MA | Magallanes |
| CL | CL-ML | del Maule |
| CL | CL-NB | del Ñuble |
| CL | CL-RM | Metropolitana |
| CL | CL-TA | Tarapacá |
| CL | CL-VS | Valparaíso |
| CN | 京 | 北京市 |
| CN | 冀 | 河北省 |
| CN | 台 | 台湾省 |
| CN | 吉 | 吉林省 |
| CN | 宁 | 宁夏回族自治区 |
| CN | 新 | 新疆维吾尔自治区 |
| CN | 晋 | 山西省 |
| CN | 桂 | 广西壮族自治区 |
| CN | 沪 | 上海市 |
| CN | 津 | 天津市 |
| CN | 浙 | 浙江省 |
| CN | 渝 | 重庆市 |
| CN | 港 | 香港特别行政区 |
| CN | 湘 | 湖南省 |
| CN | 滇 | 云南省 |
| CN | 澳 | 澳门特别行政区 |
| CN | 琼 | 海南省 |
| CN | 甘 | 甘肃省 |
| CN | 皖 | 安徽省 |
| CN | 粤 | 广东省 |
| CN | 苏 | 江苏省 |
| CN | 蒙 | 内蒙古自治区 |
| CN | 藏 | 西藏自治区 |
| CN | 蜀 | 四川省 |
| CN | 豫 | 河南省 |
| CN | 赣 | 江西省 |
| CN | 辽 | 辽宁省 |
| CN | 鄂 | 湖北省 |
| CN | 闽 | 福建省 |
| CN | 陕 | 陕西省 |
| CN | 青 | 青海省 |
| CN | 鲁 | 山东省 |
| CN | 黑 | 黑龙江省 |
| CN | 黔 | 贵州省 |
| CO | AMA | Amazonas |
| CO | ANT | Antioquia |
| CO | ARA | Arauca |
| CO | ATL | Atlántico |
| CO | BOL | Bolívar |
| CO | BOY | Boyacá |
| CO | CAL | Caldas |
| CO | CAQ | Caquetá |
| CO | CAS | Casanare |
| CO | CAU | Cauca |
| CO | CES | Cesar |
| CO | CHO | Chocó |
| CO | COR | Córdoba |
| CO | CUN | Cundinamarca |
| CO | DC | Bogotá |
| CO | GUA | Guainía |
| CO | GUV | Guaviare |
| CO | HUI | Huila |
| CO | LAG | La Guajira |
| CO | MAG | Magdalena |
| CO | MET | Meta |
| CO | NAR | Nariño |
| CO | NSA | Norte de Santander |
| CO | PUT | Putumayo |
| CO | QUI | Quindío |
| CO | RIS | Risaralda |
| CO | SAN | Santander |
| CO | SAP | San Andrés y Providencia |
| CO | SUC | Sucre |
| CO | TOL | Tolima |
| CO | VAC | Valle del Cauca |
| CO | VAU | Vaupés |
| CO | VID | Vichada |
| CR | 1 | San José |
| CR | 2 | Alajuela |
| CR | 3 | Cartago |
| CR | 4 | Heredia |
| CR | 5 | Guanacaste |
| CR | 6 | Puntarenas |
| CR | 7 | Limón |
| DE | DE-BB | Brandenburg |
| DE | DE-BE | Berlin |
| DE | DE-BW | Baden-Württemberg |
| DE | DE-BY | Bayern |
| DE | DE-HB | Bremen |
| DE | DE-HE | Hessen |
| DE | DE-HH | Hamburg |
| DE | DE-MV | Mecklenburg-Vorpommern |
| DE | DE-NI | Niedersachsen |
| DE | DE-NW | Nordrhein-Westfalen |
| DE | DE-RP | Rheinland-Pfalz |
| DE | DE-SH | Schleswig-Holstein |
| DE | DE-SL | Saarland |
| DE | DE-SN | Sachsen |
| DE | DE-ST | Sachsen-Anhalt |
| DE | DE-TH | Thüringen |
| DO | AZU | Azua |
| DO | BAH | Bahoruco |
| DO | BAR | Barahona |
| DO | DAJ | Dajabón |
| DO | DN | Distrito Nacional |
| DO | DUA | Duarte |
| DO | ELP | Elías Piña |
| DO | ELS | El Seibo |
| DO | ESP | Espaillat |
| DO | HAM | Hato Mayor |
| DO | HEM | Hermanas Mirabal |
| DO | IND | Independencia |
| DO | LA | La Altagracia |
| DO | LR | La Romana |
| DO | LV | La Vega |
| DO | MC | Monte Cristi |
| DO | MON | Monseñor Nouel |
| DO | MP | Monte Plata |
| DO | MTS | María Trinidad Sánchez |
| DO | PED | Pedernales |
| DO | PER | Peravia |
| DO | PP | Puerto Plata |
| DO | SAM | Samaná |
| DO | SC | San Cristóbal |
| DO | SD | Santo Domingo |
| DO | SJ | San Juan |
| DO | SJO | San José de Ocoa |
| DO | SPM | San Pedro de Macorís |
| DO | SRA | Sánchez Ramírez |
| DO | SRO | Santiago Rodríguez |
| DO | STGO | Santiago |
| DO | VAL | Valverde |
| EC | 01 | Azuay |
| EC | 02 | Bolivar |
| EC | 03 | Canar |
| EC | 04 | Carchi |
| EC | 05 | Cotopaxi |
| EC | 06 | Chimborazo |
| EC | 07 | El Oro |
| EC | 08 | Esmeraldas |
| EC | 09 | Guayas |
| EC | 10 | Imbabura |
| EC | 11 | Loja |
| EC | 12 | Los Rios |
| EC | 13 | Manabi |
| EC | 14 | Morona Santiago |
| EC | 15 | Napo |
| EC | 16 | Pastaza |
| EC | 17 | Pichincha |
| EC | 18 | Tungurahua |
| EC | 19 | Zamora Chinchipe |
| EC | 20 | Galapagos |
| EC | 21 | Sucumbios |
| EC | 22 | Orellana |
| EC | 23 | Santo Domingo de los Tsachilas |
| EC | 24 | Santa Elena |
| EE | EE-37 | Harjumaa |
| EE | EE-39 | Hiiumaa |
| EE | EE-44 | Ida-Virumaa |
| EE | EE-49 | Jõgevamaa |
| EE | EE-51 | Järvamaa |
| EE | EE-57 | Läänemaa |
| EE | EE-59 | Lääne-Virumaa |
| EE | EE-65 | Põlvamaa |
| EE | EE-67 | Pärnumaa |
| EE | EE-70 | Raplamaa |
| EE | EE-74 | Saaremaa |
| EE | EE-78 | Tartumaa |
| EE | EE-82 | Valgamaa |
| EE | EE-84 | Viljandimaa |
| EE | EE-86 | Võrumaa |
| EG | ALX | Alexandria |
| EG | ASN | Aswan |
| EG | AST | Asyut |
| EG | BA | Red Sea |
| EG | BH | Beheira |
| EG | BNS | Beni Suef |
| EG | C | Cairo |
| EG | DK | Dakahlia |
| EG | DT | Damietta |
| EG | FYM | Faiyum |
| EG | GH | Gharbia |
| EG | GZ | Giza |
| EG | HU | Helwan |
| EG | IS | Ismailia |
| EG | JS | South Sinai |
| EG | KB | Qalyubia |
| EG | KFS | Kafr el-Sheikh |
| EG | KN | Qena |
| EG | LX | Luxor |
| EG | MN | Minya |
| EG | MNF | Monufia |
| EG | MT | Matrouh |
| EG | PTS | Port Said |
| EG | SHG | Sohag |
| EG | SHR | Al Sharqia |
| EG | SIN | North Sinai |
| EG | SU | 6th of October |
| EG | SUZ | Suez |
| EG | WAD | New Valley |
| ES | A | Alacant (Alicante) |
| ES | AB | Albacete |
| ES | AL | Almería |
| ES | AV | Ávila |
| ES | B | Barcelona |
| ES | BA | Badajoz |
| ES | BI | Bizkaia (Vizcaya) |
| ES | BU | Burgos |
| ES | C | A Coruña (La Coruña) |
| ES | CA | Cádiz |
| ES | CC | Cáceres |
| ES | CE | Ceuta |
| ES | CO | Córdoba |
| ES | CR | Ciudad Real |
| ES | CS | Castelló (Castellón) |
| ES | CU | Cuenca |
| ES | GC | Las Palmas |
| ES | GI | Girona (Gerona) |
| ES | GR | Granada |
| ES | GU | Guadalajara |
| ES | H | Huelva |
| ES | HU | Huesca |
| ES | J | Jaén |
| ES | L | Lleida (Lérida) |
| ES | LE | León |
| ES | LO | La Rioja |
| ES | LU | Lugo |
| ES | M | Madrid |
| ES | MA | Málaga |
| ES | ME | Melilla |
| ES | MU | Murcia |
| ES | NA | Navarra (Nafarroa) |
| ES | O | Asturias |
| ES | OR | Ourense (Orense) |
| ES | P | Palencia |
| ES | PM | Illes Balears (Islas Baleares) |
| ES | PO | Pontevedra |
| ES | S | Cantabria |
| ES | SA | Salamanca |
| ES | SE | Sevilla |
| ES | SG | Segovia |
| ES | SO | Soria |
| ES | SS | Gipuzkoa (Guipúzcoa) |
| ES | T | Tarragona |
| ES | TE | Teruel |
| ES | TF | Santa Cruz de Tenerife |
| ES | TO | Toledo |
| ES | V | València (Valencia) |
| ES | VA | Valladolid |
| ES | VI | Araba/Álava |
| ES | Z | Zaragoza |
| ES | ZA | Zamora |
| ET | AA | Addis Ababa |
| ET | AF | Afar |
| ET | AM | Amhara |
| ET | BN | Benishangul-Gumuz |
| ET | DR | Dire Dawa |
| ET | GM | Gambella Peoples |
| ET | HR | Harrari Peoples |
| ET | OR | Oromia |
| ET | SM | Somali |
| ET | SP | Southern Peoples, Nations, and Nationalities |
| ET | TG | Tigray |
| FI | FI-01 | Ahvenanmaa |
| FI | FI-02 | Etelä-Karjala |
| FI | FI-03 | Etelä-Pohjanmaa |
| FI | FI-04 | Etelä-Savo |
| FI | FI-05 | Kainuu |
| FI | FI-06 | Kanta-Häme |
| FI | FI-07 | Keski-Pohjanmaa |
| FI | FI-08 | Keski-Suomi |
| FI | FI-09 | Kymenlaakso |
| FI | FI-10 | Lappi |
| FI | FI-11 | Pirkanmaa |
| FI | FI-12 | Pohjanmaa |
| FI | FI-13 | Pohjois-Karjala |
| FI | FI-14 | Pohjois-Pohjanmaa |
| FI | FI-15 | Pohjois-Savo |
| FI | FI-16 | Päijät-Häme |
| FI | FI-17 | Satakunta |
| FI | FI-18 | Uusimaa |
| FI | FI-19 | Varsinais-Suomi |
| GB | A1 | Aberdeenshire |
| GB | A5 | Angus |
| GB | A7 | Argyll |
| GB | A9 | Avon |
| GB | B1 | Ayrshire |
| GB | B3 | Banffshire |
| GB | B5 | Bedfordshire |
| GB | B7 | Berkshire |
| GB | B9 | Berwickshire |
| GB | C1 | Buckinghamshire |
| GB | C3 | Caithness |
| GB | C5 | Cambridgeshire |
| GB | C6 | Channel Islands |
| GB | C7 | Cheshire |
| GB | C9 | Clackmannanshire |
| GB | D1 | Cleveland |
| GB | D3 | Clwyd |
| GB | D5 | County Antrim |
| GB | D7 | County Armagh |
| GB | D9 | County Down |
| GB | E1 | County Durham |
| GB | E3 | County Fermanagh |
| GB | E5 | County Londonderry |
| GB | E7 | County Tyrone |
| GB | E9 | Cornwall |
| GB | F1 | Cumbria |
| GB | F3 | Derbyshire |
| GB | F5 | Devon |
| GB | F7 | Dorset |
| GB | F9 | Dumfriesshire |
| GB | G1 | Dunbartonshire |
| GB | G3 | Dyfed |
| GB | G5 | East Lothian |
| GB | G7 | East Sussex |
| GB | G9 | Essex |
| GB | H1 | Fife |
| GB | H3 | Gloucestershire |
| GB | H7 | Gwent |
| GB | H9 | Gwynedd |
| GB | I1 | Hampshire |
| GB | I3 | Herefordshire |
| GB | I5 | Hertfordshire |
| GB | I7 | Inverness-Shire |
| GB | I9 | Isle of Arran |
| GB | J1 | Isle of Barra |
| GB | J3 | Isle of Benbecula |
| GB | J5 | Isle of Bute |
| GB | J7 | Isle of Canna |
| GB | J9 | Isle of Coll |
| GB | K1 | Isle of Colonsay |
| GB | K3 | Isle of Cumbrae |
| GB | K5 | Isle of Eigg |
| GB | K7 | Isle of Gigha |
| GB | K9 | Isle of Harris |
| GB | L1 | Isle of Iona |
| GB | L2 | Isle of Islay |
| GB | L5 | Isle of Jura |
| GB | L7 | Isle of Lewis |
| GB | L9 | Isle of Man |
| GB | M1 | Isle of Mull |
| GB | M3 | Isle of North Uist |
| GB | M5 | Orkney |
| GB | M7 | Isle of Rhum |
| GB | M9 | Isle of Scalpay |
| GB | N1 | Shetland Islands |
| GB | N3 | Isle of Skye |
| GB | N5 | Isle of South Uist |
| GB | N7 | Isle of Tiree |
| GB | N9 | Isle of Wight |
| GB | O1 | Isles of Scilly |
| GB | O5 | Kent |
| GB | O7 | Kincardineshire |
| GB | O9 | Kinross-Shire |
| GB | P1 | Kirkcudbrightshire |
| GB | P3 | Lanarkshire |
| GB | P5 | Lancashire |
| GB | P7 | Leicestershire |
| GB | P9 | Lincolnshire |
| GB | Q1 | London |
| GB | Q3 | Merseyside |
| GB | Q5 | Mid Glamorgan |
| GB | Q7 | Midlothian |
| GB | Q9 | Middlesex |
| GB | R1 | Morayshire |
| GB | R3 | Nairnshire |
| GB | R5 | Norfolk |
| GB | R7 | North Humberside |
| GB | R9 | North Yorkshire |
| GB | S1 | Northamptonshire |
| GB | S3 | Northumberland |
| GB | S5 | Nottinghamshire |
| GB | S7 | Oxfordshire |
| GB | S9 | Peeblesshire |
| GB | T1 | Perthshire |
| GB | T3 | Powys |
| GB | T5 | Renfrewshire |
| GB | T7 | Ross-Shire |
| GB | T9 | Roxburghshire |
| GB | U3 | Selkirkshire |
| GB | U5 | Shropshire |
| GB | U7 | Somerset |
| GB | U9 | South Glamorgan |
| GB | V1 | South Humberside |
| GB | V3 | South Yorkshire |
| GB | V5 | Staffordshire |
| GB | V7 | Stirlingshire |
| GB | V9 | Suffolk |
| GB | W1 | Surrey |
| GB | W3 | Sutherland |
| GB | W5 | Tyne and Wear |
| GB | W7 | Warwickshire |
| GB | W9 | West Glamorgan |
| GB | X1 | West Lothian |
| GB | X3 | West Midlands |
| GB | X5 | West Sussex |
| GB | X7 | West Yorkshire |
| GB | X9 | Wigtownshire |
| GB | Y1 | Wiltshire |
| GB | Y3 | Worcestershire |
| GE | GE-AB | Abkhazia |
| GE | GE-AJ | Ajaria |
| GE | GE-GU | Guria |
| GE | GE-IM | Imereti |
| GE | GE-KA | Kakheti |
| GE | GE-KK | Kvemo Kartli |
| GE | GE-MM | Mtskheta-Mtianeti |
| GE | GE-RL | Racha-Lechkhumi and Kvemo Svaneti |
| GE | GE-SJ | Samtskhe-Javakheti |
| GE | GE-SK | Shida Kartli |
| GE | GE-SZ | Samegrelo-Zemo Svaneti |
| GE | GE-TB | Tbilisi |
| GT | AVE | Alta Verapaz |
| GT | BVE | Baja Verapaz |
| GT | CMT | Chimaltenango |
| GT | CQM | Chiquimula |
| GT | EPR | El Progreso |
| GT | ESC | Escuintla |
| GT | GUA | Guatemala |
| GT | HUE | Huehuetenango |
| GT | IZA | Izabal |
| GT | JAL | Jalapa |
| GT | JUT | Jutiapa |
| GT | PET | Petén |
| GT | QUE | Quetzaltenango |
| GT | QUI | Quiché |
| GT | RET | Retalhuleu |
| GT | SAC | Sacatepéquez |
| GT | SMA | San Marcos |
| GT | SOL | Sololá |
| GT | SRO | Santa Rosa |
| GT | SUC | Suchitepéquez |
| GT | TOT | Totonicapán |
| GT | ZAC | Zacapa |
| HK | HK | Hong Kong Island |
| HK | KLN | Kowloon |
| HK | NT | New Territories |
| ID | AC | Aceh |
| ID | BA | Bali |
| ID | BB | Bangka Belitung |
| ID | BE | Bengkulu |
| ID | BT | Banten |
| ID | GO | Gorontalo |
| ID | JA | Jambi |
| ID | JB | Jawa Barat |
| ID | JI | Jawa Timur |
| ID | JK | Jakarta |
| ID | JT | Jawa Tengah |
| ID | KB | Kalimantan Barat |
| ID | KI | Kalimantan Timur |
| ID | KR | Kepulauan Riau |
| ID | KS | Kalimantan Selatan |
| ID | KT | Kalimantan Tengah |
| ID | KU | Kalimantan Utara |
| ID | LA | Lampung |
| ID | MA | Maluku |
| ID | MU | Maluku Utara |
| ID | NB | Nusa Tenggara Barat |
| ID | NT | Nusa Tenggara Timur |
| ID | PA | Papua |
| ID | PB | Papua Barat |
| ID | PD | Papua Barat Daya |
| ID | PE | Papua Pegunungan |
| ID | PS | Papua Selatan |
| ID | PT | Papua Tengah |
| ID | RI | Riau |
| ID | SA | Sulawesi Utara |
| ID | SB | Sumatra Barat |
| ID | SG | Sulawesi Tenggara |
| ID | SN | Sulawesi Selatan |
| ID | SR | Sulawesi Barat |
| ID | SS | Sumatra Selatan |
| ID | ST | Sulawesi Tengah |
| ID | SU | Sumatra Utara |
| ID | YO | Yogyakarta |
| IE | AH | Armagh |
| IE | AM | Antrim |
| IE | C | Cork |
| IE | CE | Clare |
| IE | CN | Cavan |
| IE | CW | Carlow |
| IE | D | Dublin |
| IE | DL | Donegal |
| IE | DN | Down |
| IE | FH | Fermanagh |
| IE | G | Galway |
| IE | KE | Kildare |
| IE | KK | Kilkenny |
| IE | KY | Kerry |
| IE | LD | Longford |
| IE | LH | Louth |
| IE | LK | Limerick |
| IE | LM | Leitrim |
| IE | LS | Laois |
| IE | LY | Londonderry |
| IE | MH | Meath |
| IE | MN | Monaghan |
| IE | MO | Mayo |
| IE | OY | Offaly |
| IE | RN | Roscommon |
| IE | SO | Sligo |
| IE | TE | Tyrone |
| IE | TR | Tipperary |
| IE | WD | Waterford |
| IE | WH | Westmeath |
| IE | WW | Wicklow |
| IE | WX | Wexford |
| IN | AN | Andaman and Nicobar |
| IN | AP | Andhra Pradesh |
| IN | AR | Arunachal Pradesh |
| IN | AS | Assam |
| IN | BR | Bihar |
| IN | CG | Chattisgarh |
| IN | CH | Chandigarh |
| IN | DD | Daman and Diu |
| IN | DL | Delhi |
| IN | DN | Dadra and Nagar Haveli |
| IN | GA | Goa |
| IN | GJ | Gujarat |
| IN | HP | Himachal Pradesh |
| IN | HR | Haryana |
| IN | IN_OC | Foreign Country |
| IN | IN_OT | Other Territory |
| IN | JH | Jharkhand |
| IN | JK | Jammu and Kashmir |
| IN | KA | Karnataka |
| IN | KL | Kerala |
| IN | LA | Ladakh |
| IN | LD | Lakshadweep |
| IN | MH | Maharashtra |
| IN | ML | Meghalaya |
| IN | MN | Manipur |
| IN | MP | Madhya Pradesh |
| IN | MZ | Mizoram |
| IN | NL | Nagaland |
| IN | OD | Odisha |
| IN | PB | Punjab |
| IN | PY | Puducherry |
| IN | RJ | Rajasthan |
| IN | SK | Sikkim |
| IN | TN | Tamil Nadu |
| IN | TR | Tripura |
| IN | TS | Telangana |
| IN | UK | Uttarakhand |
| IN | UP | Uttar Pradesh |
| IN | WB | West Bengal |
| IQ | IQ-AN | Al Anbar |
| IQ | IQ-AN-AR | الأنبار |
| IQ | IQ-AR | Erbil |
| IQ | IQ-AR-AR | أربيل |
| IQ | IQ-BA | Al Basrah |
| IQ | IQ-BA-AR | البصرة |
| IQ | IQ-BB | Babil |
| IQ | IQ-BB-AR | بابل |
| IQ | IQ-BG | Baghdad |
| IQ | IQ-BG-AR | بغداد |
| IQ | IQ-DA | Duhok |
| IQ | IQ-DA-AR | دهوك |
| IQ | IQ-DI | Diyala |
| IQ | IQ-DI-AR | ديالى |
| IQ | IQ-DQ | Dhi Qar |
| IQ | IQ-DQ-AR | ذي قار |
| IQ | IQ-KA | Karbala' |
| IQ | IQ-KA-AR | كربلاء |
| IQ | IQ-KI | Kirkuk |
| IQ | IQ-KI-AR | كركوك |
| IQ | IQ-MA | Maysan |
| IQ | IQ-MA-AR | ميسان |
| IQ | IQ-MU | Al Muthanna |
| IQ | IQ-MU-AR | المثنى |
| IQ | IQ-NA | Najaf |
| IQ | IQ-NA-AR | النجف |
| IQ | IQ-NI | Ninawa |
| IQ | IQ-NI-AR | نينوى |
| IQ | IQ-QA | Al Qādisiyyah |
| IQ | IQ-QA-AR | القادسية |
| IQ | IQ-SD | Salah Al Din |
| IQ | IQ-SD-AR | صلاح الدين |
| IQ | IQ-SU | Sulaymaniyah |
| IQ | IQ-SU-AR | السليمانية |
| IQ | IQ-WA | Wasit |
| IQ | IQ-WA-AR | واسط |
| IT | AG | Agrigento |
| IT | AL | Alessandria |
| IT | AN | Ancona |
| IT | AO | Aosta |
| IT | AP | Ascoli Piceno |
| IT | AQ | L'Aquila |
| IT | AR | Arezzo |
| IT | AT | Asti |
| IT | AV | Avellino |
| IT | BA | Bari |
| IT | BG | Bergamo |
| IT | BI | Biella |
| IT | BL | Belluno |
| IT | BN | Benevento |
| IT | BO | Bologna |
| IT | BR | Brindisi |
| IT | BS | Brescia |
| IT | BT | Barletta-Andria-Trani |
| IT | BZ | Bolzano |
| IT | CA | Cagliari |
| IT | CB | Campobasso |
| IT | CE | Caserta |
| IT | CH | Chieti |
| IT | CI | Carbonia-Iglesias |
| IT | CL | Caltanissetta |
| IT | CN | Cuneo |
| IT | CO | Como |
| IT | CR | Cremona |
| IT | CS | Cosenza |
| IT | CT | Catania |
| IT | CZ | Catanzaro |
| IT | EN | Enna |
| IT | FC | Forlì-Cesena |
| IT | FE | Ferrara |
| IT | FG | Foggia |
| IT | FI | Firenze |
| IT | FM | Fermo |
| IT | FR | Frosinone |
| IT | GE | Genova |
| IT | GO | Gorizia |
| IT | GR | Grosseto |
| IT | IM | Imperia |
| IT | IS | Isernia |
| IT | KR | Crotone |
| IT | LC | Lecco |
| IT | LE | Lecce |
| IT | LI | Livorno |
| IT | LO | Lodi |
| IT | LT | Latina |
| IT | LU | Lucca |
| IT | MB | Monza e Brianza |
| IT | MC | Macerata |
| IT | ME | Messina |
| IT | MI | Milano |
| IT | MN | Mantova |
| IT | MO | Modena |
| IT | MS | Massa-Carrara |
| IT | MT | Matera |
| IT | NA | Napoli |
| IT | NO | Novara |
| IT | NU | Nuoro |
| IT | OG | Ogliastra |
| IT | OR | Oristano |
| IT | OT | Olbia-Tempio |
| IT | PA | Palermo |
| IT | PC | Piacenza |
| IT | PD | Padova |
| IT | PE | Pescara |
| IT | PG | Perugia |
| IT | PI | Pisa |
| IT | PN | Pordenone |
| IT | PO | Prato |
| IT | PR | Parma |
| IT | PT | Pistoia |
| IT | PU | Pesaro e Urbino |
| IT | PV | Pavia |
| IT | PZ | Potenza |
| IT | RA | Ravenna |
| IT | RC | Reggio Calabria |
| IT | RE | Reggio Emilia |
| IT | RG | Ragusa |
| IT | RI | Rieti |
| IT | RM | Roma |
| IT | RN | Rimini |
| IT | RO | Rovigo |
| IT | SA | Salerno |
| IT | SI | Siena |
| IT | SO | Sondrio |
| IT | SP | La Spezia |
| IT | SR | Siracusa |
| IT | SS | Sassari |
| IT | SU | Sud Sardegna |
| IT | SV | Savona |
| IT | TA | Taranto |
| IT | TE | Teramo |
| IT | TN | Trento |
| IT | TO | Torino |
| IT | TP | Trapani |
| IT | TR | Terni |
| IT | TS | Trieste |
| IT | TV | Treviso |
| IT | UD | Udine |
| IT | VA | Varese |
| IT | VB | Verbano-Cusio-Ossola |
| IT | VC | Vercelli |
| IT | VE | Venezia |
| IT | VI | Vicenza |
| IT | VR | Verona |
| IT | VS | Medio Campidano |
| IT | VT | Viterbo |
| IT | VV | Vibo Valentia |
| JO | JO-AJ | Ajloun |
| JO | JO-AM | Amman |
| JO | JO-AQ | Aqaba |
| JO | JO-AT | Tafileh |
| JO | JO-AZ | Zarqa |
| JO | JO-BA | Balqa |
| JO | JO-IR | Irbid |
| JO | JO-JA | Jerash |
| JO | JO-KA | Karak |
| JO | JO-MA | Mafraq |
| JO | JO-MD | Madaba |
| JO | JO-MN | Maan |
| JP | JP-01 | 北海道 |
| JP | JP-02 | 青森県 |
| JP | JP-03 | 岩手県 |
| JP | JP-04 | 宮城県 |
| JP | JP-05 | 秋田県 |
| JP | JP-06 | 山形県 |
| JP | JP-07 | 福島県 |
| JP | JP-08 | 茨城県 |
| JP | JP-09 | 栃木県 |
| JP | JP-10 | 群馬県 |
| JP | JP-11 | 埼玉県 |
| JP | JP-12 | 千葉県 |
| JP | JP-13 | 東京都 |
| JP | JP-14 | 神奈川県 |
| JP | JP-15 | 新潟県 |
| JP | JP-16 | 富山県 |
| JP | JP-17 | 石川県 |
| JP | JP-18 | 福井県 |
| JP | JP-19 | 山梨県 |
| JP | JP-20 | 長野県 |
| JP | JP-21 | 岐阜県 |
| JP | JP-22 | 静岡県 |
| JP | JP-23 | 愛知県 |
| JP | JP-24 | 三重県 |
| JP | JP-25 | 滋賀県 |
| JP | JP-26 | 京都府 |
| JP | JP-27 | 大阪府 |
| JP | JP-28 | 兵庫県 |
| JP | JP-29 | 奈良県 |
| JP | JP-30 | 和歌山県 |
| JP | JP-31 | 鳥取県 |
| JP | JP-32 | 島根県 |
| JP | JP-33 | 岡山県 |
| JP | JP-34 | 広島県 |
| JP | JP-35 | 山口県 |
| JP | JP-36 | 徳島県 |
| JP | JP-37 | 香川県 |
| JP | JP-38 | 愛媛県 |
| JP | JP-39 | 高知県 |
| JP | JP-40 | 福岡県 |
| JP | JP-41 | 佐賀県 |
| JP | JP-42 | 長崎県 |
| JP | JP-43 | 熊本県 |
| JP | JP-44 | 大分県 |
| JP | JP-45 | 宮崎県 |
| JP | JP-46 | 鹿児島県 |
| JP | JP-47 | 沖縄県 |
| KE | KE-01 | Baringo |
| KE | KE-02 | Bomet |
| KE | KE-03 | Bungoma |
| KE | KE-04 | Busia |
| KE | KE-05 | Elgeyo/Marakwet |
| KE | KE-06 | Embu |
| KE | KE-07 | Garissa |
| KE | KE-08 | Homa Bay |
| KE | KE-09 | Isiolo |
| KE | KE-10 | Kajiado |
| KE | KE-11 | Kakamega |
| KE | KE-12 | Kericho |
| KE | KE-13 | Kiambu |
| KE | KE-14 | Kilifi |
| KE | KE-15 | Kirinyaga |
| KE | KE-16 | Kisii |
| KE | KE-17 | Kisumu |
| KE | KE-18 | Kitui |
| KE | KE-19 | Kwale |
| KE | KE-20 | Laikipia |
| KE | KE-21 | Lamu |
| KE | KE-22 | Machakos |
| KE | KE-23 | Makueni |
| KE | KE-24 | Mandera |
| KE | KE-25 | Marsabit |
| KE | KE-26 | Meru |
| KE | KE-27 | Migori |
| KE | KE-28 | Mombasa |
| KE | KE-29 | Murang'a |
| KE | KE-30 | Nairobi City |
| KE | KE-31 | Nakuru |
| KE | KE-32 | Nandi |
| KE | KE-33 | Narok |
| KE | KE-34 | Nyamira |
| KE | KE-35 | Nyandarua |
| KE | KE-36 | Nyeri |
| KE | KE-37 | Samburu |
| KE | KE-38 | Siaya |
| KE | KE-39 | Taita/Taveta |
| KE | KE-40 | Tana River |
| KE | KE-41 | Tharaka-Nithi |
| KE | KE-42 | Trans Nzoia |
| KE | KE-43 | Turkana |
| KE | KE-44 | Uasin Gishu |
| KE | KE-45 | Vihiga |
| KE | KE-46 | Wajir |
| KE | KE-47 | West Pokot |
| KG | KG-B | Баткенская область |
| KG | KG-C | Чуйская область |
| KG | KG-GB | Бишкек |
| KG | KG-GO | Ош |
| KG | KG-J | Джалал-Абадская область |
| KG | KG-N | Нарынская область |
| KG | KG-O | Ошская область |
| KG | KG-T | Таласская область |
| KG | KG-Y | Иссык-Кульская область |
| KR | KR-11 | 서울특별시 |
| KR | KR-26 | 부산광역시 |
| KR | KR-27 | 대구광역시 |
| KR | KR-28 | 인천광역시 |
| KR | KR-29 | 광주광역시 |
| KR | KR-30 | 대전광역시 |
| KR | KR-31 | 울산광역시 |
| KR | KR-41 | 경기도 |
| KR | KR-42 | 강원도 |
| KR | KR-43 | 충청북도 |
| KR | KR-44 | 충청남도 |
| KR | KR-45 | 전라북도 |
| KR | KR-46 | 전라남도 |
| KR | KR-47 | 경상북도 |
| KR | KR-48 | 경상남도 |
| KR | KR-49 | 제주특별자치도 |
| KR | KR-50 | 세종특별자치시 |
| KZ | KZ-10 | Абайская область |
| KZ | KZ-11 | Акмолинская область |
| KZ | KZ-15 | Актюбинская область |
| KZ | KZ-19 | Алматинская область |
| KZ | KZ-23 | Атырауская область |
| KZ | KZ-27 | Западно-Казахстанская область |
| KZ | KZ-31 | Жамбылская область |
| KZ | KZ-33 | Жетысуская область |
| KZ | KZ-35 | Карагандинская область |
| KZ | KZ-39 | Костанайская область |
| KZ | KZ-43 | Кызылординская область |
| KZ | KZ-47 | Мангистауская область |
| KZ | KZ-55 | Павлодарская область |
| KZ | KZ-59 | Северо-Казахстанская область |
| KZ | KZ-61 | Туркестанская область |
| KZ | KZ-62 | Улытауская область |
| KZ | KZ-63 | Восточно-Казахстанская область |
| KZ | KZ-71 | Город Астана |
| KZ | KZ-75 | Город Алма-Ата |
| KZ | KZ-79 | Город Шымкент |
| LT | LT-AL | Alytaus apskritis |
| LT | LT-KL | Klaipėdos apskritis |
| LT | LT-KU | Kauno apskritis |
| LT | LT-MR | Marijampolės apskritis |
| LT | LT-PN | Panevėžio apskritis |
| LT | LT-SA | Šiaulių apskritis |
| LT | LT-TA | Tauragės apskritis |
| LT | LT-TE | Telšių apskritis |
| LT | LT-UT | Utenos apskritis |
| LT | LT-VL | Vilniaus apskritis |
| LV | LV-001 | Aglonas novads |
| LV | LV-002 | Aizkraukles novads |
| LV | LV-003 | Aizputes novads |
| LV | LV-004 | Aknīstes novads |
| LV | LV-005 | Alojas novads |
| LV | LV-006 | Alsungas novads |
| LV | LV-007 | Alūksnes novads |
| LV | LV-008 | Amatas novads |
| LV | LV-009 | Apes novads |
| LV | LV-010 | Auces novads |
| LV | LV-011 | Ādažu novads |
| LV | LV-012 | Babītes novads |
| LV | LV-013 | Baldones novads |
| LV | LV-014 | Baltinavas novads |
| LV | LV-015 | Balvu novads |
| LV | LV-016 | Bauskas novads |
| LV | LV-017 | Beverīnas novads |
| LV | LV-018 | Brocēnu novads |
| LV | LV-019 | Burtnieku novads |
| LV | LV-020 | Carnikavas novads |
| LV | LV-021 | Cesvaines novads |
| LV | LV-022 | Cēsu novads |
| LV | LV-023 | Ciblas novads |
| LV | LV-024 | Dagdas novads |
| LV | LV-025 | Daugavpils novads |
| LV | LV-026 | Dobeles novads |
| LV | LV-027 | Dundagas novads |
| LV | LV-028 | Durbes novads |
| LV | LV-029 | Engures novads |
| LV | LV-030 | Ērgļu novads |
| LV | LV-031 | Garkalnes novads |
| LV | LV-032 | Grobiņas novads |
| LV | LV-033 | Gulbenes novads |
| LV | LV-034 | Iecavas novads |
| LV | LV-035 | Ikšķiles novads |
| LV | LV-036 | Ilūkstes novads |
| LV | LV-037 | Inčukalna novads |
| LV | LV-038 | Jaunjelgavas novads |
| LV | LV-039 | Jaunpiebalgas novads |
| LV | LV-040 | Jaunpils novads |
| LV | LV-041 | Jelgavas novads |
| LV | LV-042 | Jēkabpils novads |
| LV | LV-043 | Kandavas novads |
| LV | LV-044 | Kārsavas novads |
| LV | LV-045 | Kocēnu novads |
| LV | LV-046 | Kokneses novads |
| LV | LV-047 | Krāslavas novads |
| LV | LV-048 | Krimuldas novads |
| LV | LV-049 | Krustpils novads |
| LV | LV-050 | Kuldīgas novads |
| LV | LV-051 | Ķeguma novads |
| LV | LV-052 | Ķekavas novads |
| LV | LV-053 | Lielvārdes novads |
| LV | LV-054 | Limbažu novads |
| LV | LV-055 | Līgatnes novads |
| LV | LV-056 | Līvānu novads |
| LV | LV-057 | Lubānas novads |
| LV | LV-058 | Ludzas novads |
| LV | LV-059 | Madonas novads |
| LV | LV-060 | Mazsalacas novads |
| LV | LV-061 | Mālpils novads |
| LV | LV-062 | Mārupes novads |
| LV | LV-063 | Mērsraga novads |
| LV | LV-064 | Naukšēnu novads |
| LV | LV-065 | Neretas novads |
| LV | LV-066 | Nīcas novads |
| LV | LV-067 | Ogres novads |
| LV | LV-068 | Olaines novads |
| LV | LV-069 | Ozolnieku novads |
| LV | LV-070 | Pārgaujas novads |
| LV | LV-071 | Pāvilostas novads |
| LV | LV-072 | Pļaviņu novads |
| LV | LV-073 | Preiļu novads |
| LV | LV-074 | Priekules novads |
| LV | LV-075 | Priekuļu novads |
| LV | LV-076 | Raunas novads |
| LV | LV-077 | Rēzeknes novads |
| LV | LV-078 | Riebiņu novads |
| LV | LV-079 | Rojas novads |
| LV | LV-080 | Ropažu novads |
| LV | LV-081 | Rucavas novads |
| LV | LV-082 | Rugāju novads |
| LV | LV-083 | Rundāles novads |
| LV | LV-084 | Rūjienas novads |
| LV | LV-085 | Salas novads |
| LV | LV-086 | Salacgrīvas novads |
| LV | LV-087 | Salaspils novads |
| LV | LV-088 | Saldus novads |
| LV | LV-089 | Saulkrastu novads |
| LV | LV-090 | Sējas novads |
| LV | LV-091 | Siguldas novads |
| LV | LV-092 | Skrīveru novads |
| LV | LV-093 | Skrundas novads |
| LV | LV-094 | Smiltenes novads |
| LV | LV-095 | Stopiņu novads |
| LV | LV-096 | Strenču novads |
| LV | LV-097 | Talsu novads |
| LV | LV-098 | Tērvetes novads |
| LV | LV-099 | Tukuma novads |
| LV | LV-100 | Vaiņodes novads |
| LV | LV-101 | Valkas novads |
| LV | LV-102 | Varakļānu novads |
| LV | LV-103 | Vārkavas novads |
| LV | LV-104 | Vecpiebalgas novads |
| LV | LV-105 | Vecumnieku novads |
| LV | LV-106 | Ventspils novads |
| LV | LV-107 | Viesītes novads |
| LV | LV-108 | Viļakas novads |
| LV | LV-109 | Viļānu novads |
| LV | LV-110 | Zilupes novads |
| LV | LV-DGV | Daugavpils |
| LV | LV-JEL | Jelgava |
| LV | LV-JKB | Jēkabpils |
| LV | LV-JUR | Jūrmala |
| LV | LV-LPX | Liepāja |
| LV | LV-REZ | Rēzekne |
| LV | LV-RIX | Rīga |
| LV | LV-VEN | Ventspils |
| LV | LV-VMR | Valmiera |
| MN | 01 | Архангай |
| MN | 02 | Баян-Өлгий |
| MN | 03 | Баянхонгор |
| MN | 04 | Булган |
| MN | 05 | Говь-Алтай |
| MN | 06 | Дорноговь |
| MN | 07 | Дорнод |
| MN | 08 | Дундговь |
| MN | 09 | Завхан |
| MN | 10 | Өвөрхангай |
| MN | 11 | Өмнөговь |
| MN | 12 | Сүхбаатар |
| MN | 13 | Сэлэнгэ |
| MN | 14 | Төв |
| MN | 15 | Увс |
| MN | 16 | Ховд |
| MN | 17 | Хөвсгөл |
| MN | 18 | Хэнтий |
| MN | 19 | Дархан-Уул |
| MN | 20 | Орхон |
| MN | 23 | УБ - Хан Уул |
| MN | 24 | УБ - Баянзүрх |
| MN | 25 | УБ - Сүхбаатар |
| MN | 26 | УБ - Баянгол |
| MN | 27 | УБ - Багануур |
| MN | 28 | УБ - Багахангай |
| MN | 29 | УБ - Налайх |
| MN | 32 | Говьсүмбэр |
| MN | 34 | УБ - Сонгино Хайрхан |
| MN | 35 | УБ - Чингэлтэй |
| MX | AGU | Aguascalientes |
| MX | BCN | Baja California |
| MX | BCS | Baja California Sur |
| MX | CAM | Campeche |
| MX | CHH | Chihuahua |
| MX | CHP | Chiapas |
| MX | CMX | Ciudad de México |
| MX | COA | Coahuila |
| MX | COL | Colima |
| MX | DUR | Durango |
| MX | GRO | Guerrero |
| MX | GUA | Guanajuato |
| MX | HID | Hidalgo |
| MX | JAL | Jalisco |
| MX | MEX | México |
| MX | MIC | Michoacán |
| MX | MOR | Morelos |
| MX | NAY | Nayarit |
| MX | NLE | Nuevo León |
| MX | OAX | Oaxaca |
| MX | PUE | Puebla |
| MX | QUE | Querétaro |
| MX | ROO | Quintana Roo |
| MX | SIN | Sinaloa |
| MX | SLP | San Luis Potosí |
| MX | SON | Sonora |
| MX | TAB | Tabasco |
| MX | TAM | Tamaulipas |
| MX | TLA | Tlaxcala |
| MX | VER | Veracruz |
| MX | YUC | Yucatán |
| MX | ZAC | Zacatecas |
| MY | MY-01 | Johor |
| MY | MY-02 | Kedah |
| MY | MY-03 | Kelantan |
| MY | MY-04 | Melaka |
| MY | MY-05 | Negeri Sembilan |
| MY | MY-06 | Pahang |
| MY | MY-07 | Pulau Pinang |
| MY | MY-08 | Perak |
| MY | MY-09 | Perlis |
| MY | MY-10 | Selangor |
| MY | MY-11 | Terengganu |
| MY | MY-12 | Sabah |
| MY | MY-13 | Sarawak |
| MY | MY-14 | Kuala Lumpur |
| MY | MY-15 | Labuan |
| MY | MY-16 | Putrajaya |
| NG | NG-AB | Abia |
| NG | NG-AD | Adamawa |
| NG | NG-AK | Akwa Ibom |
| NG | NG-AN | Anambra |
| NG | NG-BA | Bauchi |
| NG | NG-BE | Benue |
| NG | NG-BO | Borno |
| NG | NG-BY | Bayelsa |
| NG | NG-CR | Cross River |
| NG | NG-DE | Delta |
| NG | NG-EB | Ebonyi |
| NG | NG-ED | Edo |
| NG | NG-EK | Ekiti |
| NG | NG-EN | Enugu |
| NG | NG-FC | FCT |
| NG | NG-GO | Gombe |
| NG | NG-IM | Imo |
| NG | NG-JI | Jigawa |
| NG | NG-KD | Kaduna |
| NG | NG-KE | Kebbi |
| NG | NG-KN | Kano |
| NG | NG-KO | Kogi |
| NG | NG-KT | Katsina |
| NG | NG-KW | Kwara |
| NG | NG-LA | Lagos |
| NG | NG-NA | Nasarawa |
| NG | NG-NI | Niger |
| NG | NG-OG | Ogun |
| NG | NG-ON | Ondo |
| NG | NG-OS | Osun |
| NG | NG-OY | Oyo |
| NG | NG-PL | Plateau |
| NG | NG-RI | Rivers |
| NG | NG-SO | Sokoto |
| NG | NG-TA | Taraba |
| NG | NG-YO | Yobe |
| NG | NG-ZA | Zamfara |
| NL | BQ1 | Bonaire |
| NL | BQ2 | Saba |
| NL | BQ3 | Sint Eustatius |
| NL | DR | Drenthe |
| NL | FL | Flevoland |
| NL | FR | Friesland |
| NL | GE | Gelderland |
| NL | GR | Groningen |
| NL | LI | Limburg |
| NL | NB | Noord-Brabant |
| NL | NH | Noord-Holland |
| NL | OV | Overijssel |
| NL | UT | Utrecht |
| NL | ZE | Zeeland |
| NL | ZH | Zuid-Holland |
| NO | NO-03 | Oslo |
| NO | NO-11 | Rogaland |
| NO | NO-15 | Møre og Romsdal |
| NO | NO-18 | Nordland |
| NO | NO-21 | Svalbard |
| NO | NO-22 | Jan Mayen |
| NO | NO-30 | Viken |
| NO | NO-34 | Innlandet |
| NO | NO-38 | Vestfold og Telemark |
| NO | NO-42 | Agder |
| NO | NO-46 | Vestland |
| NO | NO-50 | Trøndelag |
| NO | NO-54 | Troms og Finnmark / Romsa ja Finnmárku |
| NZ | AUK | Auckland |
| NZ | BOP | Bay of Plenty |
| NZ | CAN | Canterbury |
| NZ | GIS | Gisborne |
| NZ | HKB | Hawke's Bay |
| NZ | MBH | Marlborough |
| NZ | MWT | Manawatu-Wanganui |
| NZ | NSN | Nelson |
| NZ | NTL | Northland |
| NZ | OTA | Otago |
| NZ | STL | Southland |
| NZ | TAS | Tasman |
| NZ | TKI | Taranaki |
| NZ | WGN | Wellington |
| NZ | WKO | Waikato |
| NZ | WTC | West Coast |
| PE | 01 | Amazonas |
| PE | 02 | Áncash |
| PE | 03 | Apurímac |
| PE | 04 | Arequipa |
| PE | 05 | Ayacucho |
| PE | 06 | Cajamarca |
| PE | 07 | Callao |
| PE | 08 | Cusco |
| PE | 09 | Huancavelica |
| PE | 10 | Huánuco |
| PE | 11 | Ica |
| PE | 12 | Junin |
| PE | 13 | La Libertad |
| PE | 14 | Lambayeque |
| PE | 15 | Lima |
| PE | 16 | Loreto |
| PE | 17 | Madre de Dios |
| PE | 18 | Moquegua |
| PE | 19 | Pasco |
| PE | 20 | Piura |
| PE | 21 | Puno |
| PE | 22 | San Martín |
| PE | 23 | Tacna |
| PE | 24 | Tumbes |
| PE | 25 | Ucayali |
| PH | PH-00 | National Capital Region |
| PH | PH-ABR | Abra |
| PH | PH-AGN | Agusan del Norte |
| PH | PH-AGS | Agusan del Sur |
| PH | PH-AKL | Aklan |
| PH | PH-ALB | Albay |
| PH | PH-ANT | Antique |
| PH | PH-APA | Apayao |
| PH | PH-AUR | Aurora |
| PH | PH-BAN | Bataan |
| PH | PH-BAS | Basilan |
| PH | PH-BEN | Benguet |
| PH | PH-BIL | Biliran |
| PH | PH-BOH | Bohol |
| PH | PH-BTG | Batangas |
| PH | PH-BTN | Batanes |
| PH | PH-BUK | Bukidnon |
| PH | PH-BUL | Bulacan |
| PH | PH-CAG | Cagayan |
| PH | PH-CAM | Camiguin |
| PH | PH-CAN | Camarines Norte |
| PH | PH-CAP | Capiz |
| PH | PH-CAS | Camarines Sur |
| PH | PH-CAT | Catanduanes |
| PH | PH-CAV | Cavite |
| PH | PH-CEB | Cebu |
| PH | PH-COM | Davao de Oro |
| PH | PH-DAO | Davao Oriental |
| PH | PH-DAS | Davao del Sur |
| PH | PH-DAV | Davao del Norte |
| PH | PH-DIN | Dinagat Islands |
| PH | PH-DVO | Davao Occidental |
| PH | PH-EAS | Eastern Samar |
| PH | PH-GUI | Guimaras |
| PH | PH-IFU | Ifugao |
| PH | PH-ILI | Iloilo |
| PH | PH-ILN | Ilocos Norte |
| PH | PH-ILS | Ilocos Sur |
| PH | PH-ISA | Isabela |
| PH | PH-KAL | Kalinga |
| PH | PH-LAG | Laguna |
| PH | PH-LAN | Lanao del Norte |
| PH | PH-LAS | Lanao del Sur |
| PH | PH-LEY | Leyte |
| PH | PH-LUN | La Union |
| PH | PH-MAD | Marinduque |
| PH | PH-MAS | Masbate |
| PH | PH-MDC | Mindoro Occidental |
| PH | PH-MDR | Mindoro Oriental |
| PH | PH-MGN | Maguindanao del Norte |
| PH | PH-MGS | Maguindanao del Sur |
| PH | PH-MOU | Mountain Province |
| PH | PH-MSC | Misamis Occidental |
| PH | PH-MSR | Misamis Oriental |
| PH | PH-NCO | Cotabato |
| PH | PH-NEC | Negros Occidental |
| PH | PH-NER | Negros Oriental |
| PH | PH-NSA | Northern Samar |
| PH | PH-NUE | Nueva Ecija |
| PH | PH-NUV | Nueva Vizcaya |
| PH | PH-PAM | Pampanga |
| PH | PH-PAN | Pangasinan |
| PH | PH-PLW | Palawan |
| PH | PH-QUE | Quezon |
| PH | PH-QUI | Quirino |
| PH | PH-RIZ | Rizal |
| PH | PH-ROM | Romblon |
| PH | PH-SAR | Sarangani |
| PH | PH-SCO | South Cotabato |
| PH | PH-SIG | Siquijor |
| PH | PH-SLE | Southern Leyte |
| PH | PH-SLU | Sulu |
| PH | PH-SOR | Sorsogon |
| PH | PH-SUK | Sultan Kudarat |
| PH | PH-SUN | Surigao del Norte |
| PH | PH-SUR | Surigao del Sur |
| PH | PH-TAR | Tarlac |
| PH | PH-TAW | Tawi-Tawi |
| PH | PH-WSA | Samar |
| PH | PH-ZAN | Zamboanga del Norte |
| PH | PH-ZAS | Zamboanga del Sur |
| PH | PH-ZMB | Zambales |
| PH | PH-ZSI | Zamboanga Sibugay |
| PK | AJK | Azad Jammu and Kashmir |
| PK | BA | Balochistan |
| PK | GB | Gilgit-Baltistan |
| PK | IS/ICT | Islamabad Capital Territory |
| PK | KP/KPK | Khyber Pakhtunkhwa |
| PK | PB | Punjab |
| PK | SD | Sindh |
| PL | DŚ | dolnośląskie |
| PL | KP | kujawsko-pomorskie |
| PL | LB | lubelskie |
| PL | LS | lubuskie |
| PL | MP | małopolskie |
| PL | MZ | mazowieckie |
| PL | OP | opolskie |
| PL | PK | podkarpackie |
| PL | PL | podlaskie |
| PL | PM | pomorskie |
| PL | WM | warmińsko-mazurskie |
| PL | WP | wielkopolskie |
| PL | ZP | zachodniopomorskie |
| PL | ŁD | łódzkie |
| PL | ŚK | świętokrzyskie |
| PL | ŚL | śląskie |
| PT | 01 | Aveiro |
| PT | 02 | Beja |
| PT | 03 | Braga |
| PT | 04 | Bragança |
| PT | 05 | Castelo Branco |
| PT | 06 | Coimbra |
| PT | 07 | Évora |
| PT | 08 | Faro |
| PT | 09 | Guarda |
| PT | 10 | Leiria |
| PT | 11 | Lisboa |
| PT | 12 | Portalegre |
| PT | 13 | Porto |
| PT | 14 | Santarém |
| PT | 15 | Setúbal |
| PT | 16 | Viana do Castelo |
| PT | 17 | Vila Real |
| PT | 18 | Viseu |
| PT | 20 | Açores |
| PT | 30 | Madeira |
| RO | AB | Alba |
| RO | AG | Argeș |
| RO | AR | Arad |
| RO | B | București |
| RO | BC | Bacău |
| RO | BH | Bihor |
| RO | BN | Bistrița-Năsăud |
| RO | BR | Brăila |
| RO | BT | Botoșani |
| RO | BV | Brașov |
| RO | BZ | Buzău |
| RO | CJ | Cluj |
| RO | CL | Călărași |
| RO | CS | Caraș Severin |
| RO | CT | Constanța |
| RO | CV | Covasna |
| RO | DB | Dâmbovița |
| RO | DJ | Dolj |
| RO | GJ | Gorj |
| RO | GL | Galați |
| RO | GR | Giurgiu |
| RO | HD | Hunedoara |
| RO | HR | Harghita |
| RO | IF | Ilfov |
| RO | IL | Ialomița |
| RO | IS | Iași |
| RO | MH | Mehedinți |
| RO | MM | Maramureș |
| RO | MS | Mureș |
| RO | NT | Neamț |
| RO | OT | Olt |
| RO | PH | Prahova |
| RO | SB | Sibiu |
| RO | SJ | Sălaj |
| RO | SM | Satu Mare |
| RO | SV | Suceava |
| RO | TL | Tulcea |
| RO | TM | Timiș |
| RO | TR | Teleorman |
| RO | VL | Vâlcea |
| RO | VN | Vrancea |
| RO | VS | Vaslui |
| RU | AD | Republic of Adygeya |
| RU | AL | Altai Republic |
| RU | ALT | Altai Krai |
| RU | AMU | Amur Oblast |
| RU | ARK | Arkhangelsk Oblast |
| RU | AST | Astrakhan Oblast |
| RU | BA | Republic of Bashkortostan |
| RU | BEL | Belgorod Oblast |
| RU | BRY | Bryansk Oblast |
| RU | BU | Republic of Buryatia |
| RU | CE | Chechen Republic |
| RU | CHE | Chelyabinsk Oblast |
| RU | CHU | Chukotka Autonomous Okrug |
| RU | CU | Chuvash Republic |
| RU | DA | Republic of Dagestan |
| RU | IN | Republic of Ingushetia |
| RU | IRK | Irkutsk Oblast |
| RU | IVA | Ivanovo Oblast |
| RU | KAM | Kamchatka Krai |
| RU | KB | Kabardino-Balkarian Republic |
| RU | KC | Karachay–Cherkess Republic |
| RU | KDA | Krasnodar Krai |
| RU | KEM | Kemerovo Oblast |
| RU | KGD | Kaliningrad Oblast |
| RU | KGN | Kurgan Oblast |
| RU | KHA | Khabarovsk Krai |
| RU | KHM | Khanty-Mansi Autonomous Okrug |
| RU | KIR | Kirov Oblast |
| RU | KK | Republic of Khakassia |
| RU | KL | Republic of Kalmykia |
| RU | KLU | Kaluga Oblast |
| RU | KO | Komi Republic |
| RU | KOS | Kostroma Oblast |
| RU | KR | Republic of Karelia |
| RU | KRS | Kursk Oblast |
| RU | KYA | Krasnoyarsk Krai |
| RU | LEN | Leningrad Oblast |
| RU | LIP | Lipetsk Oblast |
| RU | MAG | Magadan Oblast |
| RU | ME | Mari El Republic |
| RU | MO | Republic of Mordovia |
| RU | MOS | Moscow Oblast |
| RU | MOW | Moscow |
| RU | MUR | Murmansk Oblast |
| RU | NGR | Novgorod Oblast |
| RU | NIZ | Nizhny Novgorod Oblast |
| RU | NVS | Novosibirsk Oblast |
| RU | OMS | Omsk Oblast |
| RU | ORE | Orenburg Oblast |
| RU | ORL | Oryol Oblast |
| RU | PER | Perm Krai |
| RU | PNZ | Penza Oblast |
| RU | PRI | Primorsky Krai |
| RU | PSK | Pskov Oblast |
| RU | ROS | Rostov Oblast |
| RU | RYA | Ryazan Oblast |
| RU | SA | Sakha Republic (Yakutia) |
| RU | SAK | Sakhalin Oblast |
| RU | SAM | Samara Oblast |
| RU | SAR | Saratov Oblast |
| RU | SE | Republic of North Ossetia–Alania |
| RU | SMO | Smolensk Oblast |
| RU | SPE | Saint Petersburg |
| RU | STA | Stavropol Krai |
| RU | SVE | Sverdlovsk Oblast |
| RU | TA | Republic of Tatarstan |
| RU | TAM | Tambov Oblast |
| RU | TOM | Tomsk Oblast |
| RU | TUL | Tula Oblast |
| RU | TVE | Tver Oblast |
| RU | TY | Tyva Republic |
| RU | TYU | Tyumen Oblast |
| RU | UD | Udmurtia |
| RU | ULY | Ulyanovsk Oblast |
| RU | VGG | Volgograd Oblast |
| RU | VLA | Vladimir Oblast |
| RU | VLG | Vologda Oblast |
| RU | VOR | Voronezh Oblast |
| RU | YAN | Yamalo-Nenets Autonomous Okrug |
| RU | YAR | Yaroslavl Oblast |
| RU | YEV | Jewish Autonomous Oblast |
| SA | ABQ | Abqaiq |
| SA | ABT | Al Baha |
| SA | AHA | Al Hada |
| SA | AHB | Abha |
| SA | AJF | Jouf |
| SA | AKH | Al Kharj |
| SA | ALH | Al Hasa |
| SA | ALK | Al Khobar |
| SA | AMU | Al Muajjiz |
| SA | AQI | Qaisumah |
| SA | AQK | Al Khobar |
| SA | ARR | Ar Rass |
| SA | ASF | Asfan |
| SA | ASQ | Al Shuqaiq |
| SA | AUT | Al 'Uthmaniyah |
| SA | AWI | Waisumah |
| SA | BAH | Al Bahah |
| SA | BDN | Badanah |
| SA | BHH | Bisha |
| SA | BRU | Buraydah |
| SA | BUR | Buraydah |
| SA | DAW | Ad Dawadami |
| SA | DHA | Dhahran |
| SA | DHU | Dhuba |
| SA | DMM | Ad Dammam |
| SA | EAM | Nejran |
| SA | EJH | Wedjh |
| SA | ELQ | Gassim |
| SA | FJJ | Fiji |
| SA | GIZ | Jizan |
| SA | HAD | Al Hadithah |
| SA | HAS | Hail |
| SA | HBT | Hafar al Batin |
| SA | HOF | Hofuf |
| SA | HZM | Hazm Al Jalamid |
| SA | JBI | Al Jubayl Industrial City |
| SA | JEC | Jazan Economic City |
| SA | JED | Jeddah |
| SA | JIC | Jeddah Industrial City 2 & 3 |
| SA | JUB | Jubail |
| SA | JUT | Juaymah Terminal |
| SA | JYC | Jeddah Yachts Club Port |
| SA | KAC | King Abdullah City |
| SA | KFH | King Fhad |
| SA | KHU | Al Khuraibah |
| SA | KKH | King Khalid |
| SA | KMX | Khamis Mushayt |
| SA | LIT | Lith |
| SA | LJW | Al Jawf |
| SA | MAK | Makkah |
| SA | MAN | Manailih |
| SA | MED | Madinah |
| SA | MHY | Muhayil |
| SA | MJH | Majma |
| SA | MUF | Manfouha |
| SA | NJN | Najran |
| SA | QAH | Al Qahmah |
| SA | QAL | Qalsn |
| SA | QRN | Al Qurainah |
| SA | QTF | Qatif |
| SA | QUN | Al Qunfudah |
| SA | QUR | Qurayyah |
| SA | RAB | Rabigh |
| SA | RAD | Harad |
| SA | RAE | Arar |
| SA | RAH | Rafha |
| SA | RAM | Ras al Mishab |
| SA | RAR | Ras al Khafji |
| SA | RAZ | Ras Al-Khair |
| SA | RTA | Ras Tanura |
| SA | RUH | Riyadh |
| SA | RYP | Riyadh Dry Port |
| SA | SAF | Safaniya |
| SA | SAL | Salwá |
| SA | SAY | Sayhat |
| SA | SHD | Shadqam |
| SA | SHU | Shuaibah |
| SA | SHW | Sharurah |
| SA | SLF | Sulayel |
| SA | SUH | Salboukh |
| SA | TFZ | Tusdeer Free Zone |
| SA | TIF | Taif |
| SA | TUI | Turaif |
| SA | TUU | Tabuk |
| SA | UDH | Udhailiyah |
| SA | URY | Gurayat |
| SA | UZH | Unayzah |
| SA | VLA | Umm Lajj |
| SA | WAE | Wadi ad Dawasir |
| SA | YBI | Yanbu Industrial City |
| SA | YNB | Yanbu commercial city |
| SA | ZUL | Zilfi |
| SA | ZUY | Zulayfayn |
| SE | SE-AB | Stockholms län |
| SE | SE-AC | Västerbottens län |
| SE | SE-BD | Norrbottens län |
| SE | SE-C | Uppsala län |
| SE | SE-D | Södermanlands län |
| SE | SE-E | Östergötlands län |
| SE | SE-F | Jönköpings län |
| SE | SE-G | Kronobergs län |
| SE | SE-H | Kalmar län |
| SE | SE-I | Gotlands län |
| SE | SE-K | Blekinge län |
| SE | SE-M | Skåne län |
| SE | SE-N | Hallands län |
| SE | SE-O | Västra Götalands län |
| SE | SE-S | Värmlands län |
| SE | SE-T | Örebro län |
| SE | SE-U | Västmanlands län |
| SE | SE-W | Dalarnas län |
| SE | SE-X | Gävleborgs län |
| SE | SE-Y | Västernorrlands län |
| SE | SE-Z | Jämtlands län |
| SO | BN | Banaadir |
| SO | GM | Galmudug |
| SO | HS | Hirshabelle |
| SO | JL | Jubaland |
| SO | KG | Koonfur Galbeed |
| SO | PL | Puntland |
| SO | SL | Somaliland |
| SO | SSC | Khatumo |
| TH | TH-10 | กรุงเทพมหานคร |
| TH | TH-11 | สมุทรปราการ |
| TH | TH-12 | นนทบุรี |
| TH | TH-13 | ปทุมธานี |
| TH | TH-14 | พระนครศรีอยุธยา |
| TH | TH-15 | อ่างทอง |
| TH | TH-16 | ลพบุรี |
| TH | TH-17 | สิงห์บุรี |
| TH | TH-18 | ชัยนาท |
| TH | TH-19 | สระบุรี |
| TH | TH-20 | ชลบุรี |
| TH | TH-21 | ระยอง |
| TH | TH-22 | จันทบุรี |
| TH | TH-23 | ตราด |
| TH | TH-24 | ฉะเชิงเทรา |
| TH | TH-25 | ปราจีนบุรี |
| TH | TH-26 | นครนายก |
| TH | TH-27 | สระแก้ว |
| TH | TH-30 | นครราชสีมา |
| TH | TH-31 | บุรีรัมย์ |
| TH | TH-32 | สุรินทร์ |
| TH | TH-33 | ศรีสะเกษ |
| TH | TH-34 | อุบลราชธานี |
| TH | TH-35 | ยโสธร |
| TH | TH-36 | ชัยภูมิ |
| TH | TH-37 | อำนาจเจริญ |
| TH | TH-38 | บึงกาฬ |
| TH | TH-39 | หนองบัวลำภู |
| TH | TH-40 | ขอนแก่น |
| TH | TH-41 | อุดรธานี |
| TH | TH-42 | เลย |
| TH | TH-43 | หนองคาย |
| TH | TH-44 | มหาสารคาม |
| TH | TH-45 | ร้อยเอ็ด |
| TH | TH-46 | กาฬสินธุ์ |
| TH | TH-47 | สกลนคร |
| TH | TH-48 | นครพนม |
| TH | TH-49 | มุกดาหาร |
| TH | TH-50 | เชียงใหม่ |
| TH | TH-51 | ลำพูน |
| TH | TH-52 | ลำปาง |
| TH | TH-53 | อุตรดิตถ์ |
| TH | TH-54 | แพร่ |
| TH | TH-55 | น่าน |
| TH | TH-56 | พะเยา |
| TH | TH-57 | เชียงราย |
| TH | TH-58 | แม่ฮ่องสอน |
| TH | TH-60 | นครสวรรค์ |
| TH | TH-61 | อุทัยธานี |
| TH | TH-62 | กำแพงเพชร |
| TH | TH-63 | ตาก |
| TH | TH-64 | สุโขทัย |
| TH | TH-65 | พิษณุโลก |
| TH | TH-66 | พิจิตร |
| TH | TH-67 | เพชรบูรณ์ |
| TH | TH-70 | ราชบุรี |
| TH | TH-71 | กาญจนบุรี |
| TH | TH-72 | สุพรรณบุรี |
| TH | TH-73 | นครปฐม |
| TH | TH-74 | สมุทรสาคร |
| TH | TH-75 | สมุทรสงคราม |
| TH | TH-76 | เพชรบุรี |
| TH | TH-77 | ประจวบคีรีขันธ์ |
| TH | TH-80 | นครศรีธรรมราช |
| TH | TH-81 | กระบี่ |
| TH | TH-82 | พังงา |
| TH | TH-83 | ภูเก็ต |
| TH | TH-84 | สุราษฎร์ธานี |
| TH | TH-85 | ระนอง |
| TH | TH-86 | ชุมพร |
| TH | TH-90 | สงขลา |
| TH | TH-91 | สตูล |
| TH | TH-92 | ตรัง |
| TH | TH-93 | พัทลุง |
| TH | TH-94 | ปัตตานี |
| TH | TH-95 | ยะลา |
| TH | TH-96 | นราธิวาส |
| TJ | TJ-DU | Душанбе |
| TJ | TJ-GB | Вилояти Мухтори Кӯҳистони Бадахшон |
| TJ | TJ-KT | Вилояти Хатлон |
| TJ | TJ-RA | Ноҳияҳои тобеи ҷумҳурӣ |
| TJ | TJ-SU | Вилояти Суғд |
| TM | TM-A | Ahal |
| TM | TM-B | Balkan |
| TM | TM-D | Daşoguz |
| TM | TM-L | Lebap |
| TM | TM-M | Mary |
| TM | TM-S | Aşgabat |
| TR | 01 | Adana |
| TR | 02 | Adıyaman |
| TR | 03 | Afyonkarahisar |
| TR | 04 | Ağrı |
| TR | 05 | Amasya |
| TR | 06 | Ankara |
| TR | 07 | Antalya |
| TR | 08 | Artvin |
| TR | 09 | Aydın |
| TR | 10 | Balıkesir |
| TR | 11 | Bilecik |
| TR | 12 | Bingöl |
| TR | 13 | Bitlis |
| TR | 14 | Bolu |
| TR | 15 | Burdur |
| TR | 16 | Bursa |
| TR | 17 | Çanakkale |
| TR | 18 | Çankırı |
| TR | 19 | Çorum |
| TR | 20 | Denizli |
| TR | 21 | Diyarbakır |
| TR | 22 | Edirne |
| TR | 23 | Elazığ |
| TR | 24 | Erzincan |
| TR | 25 | Erzurum |
| TR | 26 | Eskişehir |
| TR | 27 | Gaziantep |
| TR | 28 | Giresun |
| TR | 29 | Gümüşhane |
| TR | 30 | Hakkari |
| TR | 31 | Hatay |
| TR | 32 | Isparta |
| TR | 33 | Mersin |
| TR | 34 | İstanbul |
| TR | 35 | İzmir |
| TR | 36 | Kars |
| TR | 37 | Kastamonu |
| TR | 38 | Kayseri |
| TR | 39 | Kırklareli |
| TR | 40 | Kırşehir |
| TR | 41 | Kocaeli |
| TR | 42 | Konya |
| TR | 43 | Kütahya |
| TR | 44 | Malatya |
| TR | 45 | Manisa |
| TR | 46 | Kahramanmaraş |
| TR | 47 | Mardin |
| TR | 48 | Muğla |
| TR | 49 | Muş |
| TR | 50 | Nevşehir |
| TR | 51 | Niğde |
| TR | 52 | Ordu |
| TR | 53 | Rize |
| TR | 54 | Sakarya |
| TR | 55 | Samsun |
| TR | 56 | Siirt |
| TR | 57 | Sinop |
| TR | 58 | Sivas |
| TR | 59 | Tekirdağ |
| TR | 60 | Tokat |
| TR | 61 | Trabzon |
| TR | 62 | Tunceli |
| TR | 63 | Şanlıurfa |
| TR | 64 | Uşak |
| TR | 65 | Van |
| TR | 66 | Yozgat |
| TR | 67 | Zonguldak |
| TR | 68 | Aksaray |
| TR | 69 | Bayburt |
| TR | 70 | Karaman |
| TR | 71 | Kırıkkale |
| TR | 72 | Batman |
| TR | 73 | Şırnak |
| TR | 74 | Bartın |
| TR | 75 | Ardahan |
| TR | 76 | Iğdır |
| TR | 77 | Yalova |
| TR | 78 | Karabük |
| TR | 79 | Kilis |
| TR | 80 | Osmaniye |
| TR | 81 | Düzce |
| TW | CHH | 彰化縣 |
| TW | CIC | 嘉義市 |
| TW | CIH | 嘉義縣 |
| TW | HCH | 新竹縣 |
| TW | HCT | 新竹市 |
| TW | HLH | 花蓮縣 |
| TW | ILH | 宜蘭縣 |
| TW | KHC | 高雄市 |
| TW | KLC | 基隆市 |
| TW | KMC | 金門縣 |
| TW | LCC | 連江縣 |
| TW | MLH | 苗栗縣 |
| TW | NTC | 南投縣 |
| TW | NTPC | 新北市 |
| TW | PHC | 澎湖縣 |
| TW | PTH | 屏東縣 |
| TW | TCC | 台中市 |
| TW | TNH | 台南市 |
| TW | TPC | 台北市 |
| TW | TTH | 台東縣 |
| TW | TYC | 桃園市 |
| TW | YLH | 雲林縣 |
| US | AA | Armed Forces Americas |
| US | AE | Armed Forces Europe |
| US | AK | Alaska |
| US | AL | Alabama |
| US | AP | Armed Forces Pacific |
| US | AR | Arkansas |
| US | AS | American Samoa |
| US | AZ | Arizona |
| US | CA | California |
| US | CO | Colorado |
| US | CT | Connecticut |
| US | DC | District of Columbia |
| US | DE | Delaware |
| US | FL | Florida |
| US | FM | Federated States of Micronesia |
| US | GA | Georgia |
| US | GU | Guam |
| US | HI | Hawaii |
| US | IA | Iowa |
| US | ID | Idaho |
| US | IL | Illinois |
| US | IN | Indiana |
| US | KS | Kansas |
| US | KY | Kentucky |
| US | LA | Louisiana |
| US | MA | Massachusetts |
| US | MD | Maryland |
| US | ME | Maine |
| US | MH | Marshall Islands |
| US | MI | Michigan |
| US | MN | Minnesota |
| US | MO | Missouri |
| US | MP | Northern Mariana Islands |
| US | MS | Mississippi |
| US | MT | Montana |
| US | NC | North Carolina |
| US | ND | North Dakota |
| US | NE | Nebraska |
| US | NH | New Hampshire |
| US | NJ | New Jersey |
| US | NM | New Mexico |
| US | NV | Nevada |
| US | NY | New York |
| US | OH | Ohio |
| US | OK | Oklahoma |
| US | OR | Oregon |
| US | PA | Pennsylvania |
| US | PR | Puerto Rico |
| US | PW | Palau |
| US | RI | Rhode Island |
| US | SC | South Carolina |
| US | SD | South Dakota |
| US | TN | Tennessee |
| US | TX | Texas |
| US | UT | Utah |
| US | VA | Virginia |
| US | VI | Virgin Islands |
| US | VT | Vermont |
| US | WA | Washington |
| US | WI | Wisconsin |
| US | WV | West Virginia |
| US | WY | Wyoming |
| UY | AR | Artigas |
| UY | CA | Canelones |
| UY | CL | Cerro Largo |
| UY | CO | Colonia |
| UY | DU | Durazno |
| UY | FD | Florida |
| UY | FS | Flores |
| UY | LA | Lavalleja |
| UY | MA | Maldonado |
| UY | MO | Montevideo |
| UY | PA | Paysandú |
| UY | RN | Río Negro |
| UY | RO | Rocha |
| UY | RV | Rivera |
| UY | SA | Salto |
| UY | SJ | San José |
| UY | SO | Soriano |
| UY | TA | Tacuarembó |
| UY | TT | Treinta y Tres |
| UZ | UZ-AN | Andijon |
| UZ | UZ-BU | Buxoro |
| UZ | UZ-FA | Farg‘ona |
| UZ | UZ-JI | Jizzax |
| UZ | UZ-NG | Namangan |
| UZ | UZ-NW | Navoiy |
| UZ | UZ-QA | Qashqadaryo |
| UZ | UZ-QR | Qoraqalpog‘iston Respublikasi |
| UZ | UZ-SA | Samarqand |
| UZ | UZ-SI | Sirdaryo |
| UZ | UZ-SU | Surxondaryo |
| UZ | UZ-TK | Toshkent (shahar) |
| UZ | UZ-TO | Toshkent (viloyat) |
| UZ | UZ-XO | Xorazm |
| VN | VN-01 | Lai Châu |
| VN | VN-02 | Lào Cai |
| VN | VN-04 | Cao Bằng |
| VN | VN-05 | Sơn La |
| VN | VN-07 | Tuyên Quang |
| VN | VN-09 | Lạng Sơn |
| VN | VN-13 | Quảng Ninh |
| VN | VN-18 | Ninh Bình |
| VN | VN-21 | Thanh Hóa |
| VN | VN-22 | Nghệ An |
| VN | VN-23 | Hà Tĩnh |
| VN | VN-25 | Quảng Trị |
| VN | VN-26 | Thừa Thiên - Huế |
| VN | VN-29 | Quảng Ngãi |
| VN | VN-30 | Gia Lai |
| VN | VN-33 | Đắk Lắk |
| VN | VN-34 | Khánh Hòa |
| VN | VN-35 | Lâm Đồng |
| VN | VN-37 | Tây Ninh |
| VN | VN-39 | Đồng Nai |
| VN | VN-44 | An Giang |
| VN | VN-45 | Đồng Tháp |
| VN | VN-49 | Vĩnh Long |
| VN | VN-56 | Bắc Ninh |
| VN | VN-59 | Cà Mau |
| VN | VN-66 | Hưng Yên |
| VN | VN-68 | Phú Thọ |
| VN | VN-69 | Thái Nguyên |
| VN | VN-71 | Điện Biên |
| VN | VN-CT | TP Cần Thơ |
| VN | VN-DN | TP Đà Nẵng |
| VN | VN-HN | Hà Nội |
| VN | VN-HP | TP Hải Phòng |
| VN | VN-SG | TP Hồ Chí Minh |
| ZA | EC | Eastern Cape |
| ZA | FS | Free State |
| ZA | GP | Gauteng |
| ZA | KZN | KwaZulu-Natal |
| ZA | LP | Limpopo |
| ZA | MP | Mpumalanga |
| ZA | NC | Northern Cape |
| ZA | NW | North West |
| ZA | WC | Western Cape |

## 8.3 Country groups

Nineteen country groups ship. The member column lists the two-letter codes of the members. The
excluded-states column exists on the groups that a localisation refines by removing a territory
that is politically inside a country but outside its customs regime.

| Group name | Code | Members | Member country codes | Excluded states |
|---|---|---|---|---|
| DOM-TOM | DOM-TOM | 10 | BL, GF, GP, MF, MQ, NC, PF, PM, RE, YT |  |
| Eurasian Economic Union | EEU | 5 | AM, BY, KG, KZ, RU |  |
| European Union | EU | 27 | AT, BE, BG, CY, CZ, DE, DK, EE, ES, FI, FR, GR, HR, HU, IE, IT, LT, LU, LV, MT, NL, PL, PT, RO, SE, SI, SK |  |
| European Union Prefixed Countries | EU_PREFIX | 31 | AT, BE, BG, CH, CY, CZ, DE, DK, EE, ES, FI, FR, GB, GR, HR, HU, IE, IT, LT, LU, LV, MT, NL, NO, PL, PT, RO, SE, SI, SK, SM |  |
| European Union VAT | EU-VAT | 28 | AT, BE, BG, CY, CZ, DE, DK, EE, ES, FI, FR, GR, HR, HU, IE, IT, LT, LU, LV, MC, MT, NL, PL, PT, RO, SE, SI, SK | ES_CE, ES_ML, ES_TF, ES_GC, NL_BQ1, NL_BQ2, NL_BQ3 |
| European Union VAT (Without Monaco) | EU-VAT-no-mc | 27 | AT, BE, BG, CY, CZ, DE, DK, EE, ES, FI, FR, GR, HR, HU, IE, IT, LT, LU, LV, MT, NL, PL, PT, RO, SE, SI, SK | ES_CE, ES_ML, ES_TF, ES_GC, NL_BQ1, NL_BQ2, NL_BQ3 |
| France and Monaco | FR-MC | 2 | FR, MC |  |
| France and Monaco and Drom | FR-MC-DROM | 7 | FR, GF, GP, MC, MQ, RE, YT |  |
| GCC VAT implementing States | GCC-VAT | 3 | AE, BH, SA |  |
| Gulf Cooperation Council (GCC) | GCC | 6 | AE, BH, KW, OM, QA, SA |  |
| India Inter-State Group | IN-INTER | 1 | IN | IN_OC |
| Intrastat | INTRASTAT | 29 | AT, BE, BG, CY, CZ, DE, DK, EE, ES, FI, FR, GB, GR, HR, HU, IE, IT, LT, LU, LV, MT, NL, PL, PT, RO, SE, SI, SK, XI |  |
| Latin America Identification | LATAMID | 8 | AR, BR, CL, CO, EC, GT, PE, UY |  |
| Mainland Spain VAT | ES-VAT | 1 | ES | ES_CE, ES_ML, ES_TF, ES_GC |
| Netherlands VAT | NL-VAT | 1 | NL | NL_BQ1, NL_BQ2, NL_BQ3 |
| SEPA Countries | SEPA | 49 | AD, AT, AX, BE, BG, BL, CH, CY, CZ, DE, DK, EE, ES, FI, FR, GB, GF, GG, GI, GP, GR, HR, HU, IE, IM, IS, IT, JE, LI, LT, LU, LV, MC, MF, MQ, MT, NL, NO, PL, PM, PT, RE, RO, SE, SI, SK, SM, VA, YT |  |
| South America | SA | 15 | AR, BO, BR, CL, CO, EC, FK, GF, GS, GY, PE, PY, SR, UY, VE |  |
| Switzerland and Liechtenstein | CH-LI | 2 | CH, LI |  |
| United Kingdom and Northern Ireland | UKXI | 2 | GB, XI |  |

**The two group codes this domain itself reads.**

- `EU_PREFIX` — the thirty countries whose tax registration numbers carry a two-letter prefix.
  Membership switches on the prefix handling in the tax-number normalise-and-validate routine and
  in the duplicate detection. Note that it contains four countries that are **not** in the customs
  union proper: Switzerland, Norway, the United Kingdom and San Marino.
- `EU` — the customs union proper, used by the tax-number routine to decide whether a number
  prefixed `EU` should be left alone.

A blank entry in the member list means that the group's definition referenced an identifier that
resolves to no shipped country in this catalogue; it is reproduced so the count is honest.

## 8.4 Currencies

One hundred and seventy-three currencies ship. Almost all of them ship **inactive**: a currency
becomes active when it is chosen as a Company's currency, or when an administrator activates it.
The rounding column is the rounding factor — the multiple to which amounts in that currency are
rounded — and the decimal-places field is derived from it.

| Code | Name | Symbol | Numeric code | Rounding | Symbol position | Unit label | Subunit label | Active on install |
|---|---|---|---|---|---|---|---|---|
| AED | United Arab Emirates dirham | AED | 784 | 0.01 | after | Dirham | Fils | no |
| AFN | Afghan afghani | Afs | 971 | 0.01 | after | Afghani | Puls | no |
| ALL | Albanian lek | L | 008 | 0.01 | after | Lek | Qindarke | no |
| AMD | Armenian dram | դր. | 051 | 0.01 | after | Dram | Luma | no |
| ANG | Netherlands Antillean guilder | ƒ | 532 | 0.01 | before | Guilder | Cents | no |
| AOA | Angolan kwanza | Kz | 973 | 0.01 | after | Kwanza | Centimos | no |
| ARS | Argentine peso | $ | 032 | 0.01 | before | Peso | Centavos | no |
| AUD | Australian dollar | $ | 036 | 0.01 | before | Dollars | Cents | yes |
| AWG | Aruban florin | Afl. | 533 | 0.01 | after | Guilder | Cents | no |
| AZN | Azerbaijani manat | ₼ | 944 | 0.01 | after | Manat | Qapik | no |
| BAM | Bosnia and Herzegovina convertible mark | KM | 977 | 0.01 | after | Mark | Fening | no |
| BBD | Barbados dollar | Bds$ | 052 | 0.01 | after | Dollars | Cents | no |
| BDT | Bangladeshi taka | ৳ | 050 | 0.01 | after | Taka | Paisa | no |
| BGN | Bulgarian lev | лв | 975 | 0.01 | after | Lev | Stotinki | no |
| BHD | Bahraini dinar | BD | 048 | 0.001 | after | Dinar | Fils | no |
| BIF | Burundian franc | FBu | 108 | 1 | after | Franc | Centime | no |
| BMD | Bermudian dollar | BD$ | 060 | 0.01 | after | Dollars | Cents | no |
| BND | Brunei dollar | B$ | 096 | 0.01 | before | Dollars | Cents | no |
| BOB | Boliviano | Bs. | 068 | 0.01 | after | Boliviano | Centavos | no |
| BRL | Brazilian real | R$ | 986 | 0.01 | before | Real | Centavos | no |
| BSD | Bahamian dollar | B$ | 044 | 0.01 | after | Dollars | Cents | no |
| BTN | Bhutanese ngultrum | Nu. | 064 | 0.01 | after | Ngultrum | Chhertum | no |
| BWP | Botswana pula | P | 072 | 0.01 | after | Pula | Thebe | no |
| BYN | Belarusian ruble | Br | 974 | 0.01 | after | Rubles | Kopeks | no |
| BYR | Belarusian ruble | BR | 974 | 1 | after | Ruble BYR | Kapeyka | no |
| BZD | Belize dollar | BZ$ | 084 | 0.01 | after | Dollars | Cents | no |
| CAD | Canadian dollar | $ | 124 | 0.01 | after | Dollars | Cents | no |
| CDF | Congolese franc | Fr | 976 | 0.01 | after | Franc | Centime | no |
| CHF | Swiss franc | CHF | 756 | 0.01 | before | Franc | Centimes | no |
| CLF | Unidad de Fomento | $ | 990 | 0.0001 | before | Peso | Centavos | no |
| CLP | Chilean peso | $ | 152 | 1 | before | Peso | Centavos | no |
| CNH | Chinese yuan - Offshore | ¥ |  | 0.01 | before | Yuan | Fen | no |
| CNY | Chinese yuan | ¥ | 156 | 0.01 | before | Yuan | Fen | yes |
| COP | Colombian peso | $ | 170 | 0.01 | before | Peso | Centavos | no |
| COU | Unidad de Valor Real | $ | 970 | 0.01 | before | Peso | centavo | no |
| CRC | Costa Rican colón | ₡ | 188 | 0.01 | before | Colon | Centimos | no |
| CUC | Cuban convertible peso | $ | 931 | 0.01 | after | Cuban convertible peso |  | no |
| CUP | Cuban peso | $ | 192 | 0.01 | after | Peso | Centavos | no |
| CVE | Cape Verdean escudo | $ | 132 | 0.01 | after | Escudo | Centavo | no |
| CZK | Czech koruna | Kč | 203 | 0.01 | after | Koruna | Halers | no |
| DJF | Djiboutian franc | Fdj | 262 | 1 | after | Franc | Centime | no |
| DKK | Danish krone | kr | 208 | 0.01 | before | Krone | Ore | no |
| DOP | Dominican peso | RD$ | 214 | 0.01 | before | Pesos | Centavos | no |
| DZD | Algerian dinar | DA | 012 | 0.01 | after | Dinar | Centimes | no |
| EGP | Egyptian pound | LE | 818 | 0.01 | after | Pound | Piastres | no |
| ERN | Eritrean nakfa | Nfk | 232 | 0.01 | after | Nakfa | Cents | no |
| ETB | Ethiopian birr | Br | 230 | 0.01 | after | Birr | Cents | no |
| EUR | Euro | € | 978 | 0.01 | after | Euros | Cents | no |
| FJD | Fiji dollar | FJ$ | 242 | 0.01 | after | Dollars | Cents | no |
| FKP | Falkland Islands pound | £ | 238 | 0.01 | after | Pound | Penny | no |
| GBP | Pound sterling | £ | 826 | 0.01 | before | Sterling | Penny | yes |
| GEL | Georgian lari | ლ | 981 | 0.01 | after | Lari | Tetri | no |
| GHS | Ghanaian cedi | GH¢ | 936 | 0.01 | after | Cedi | Pesewas | no |
| GIP | Gibraltar pound | £ | 292 | 0.01 | after | Pound | Penny | no |
| GMD | Gambian dalasi | D | 270 | 0.01 | after | Dalasi | Butut | no |
| GNF | Guinean franc | FG | 324 | 1 | after | Franc | Centime | no |
| GTQ | Guatemalan Quetzal | Q | 320 | 0.01 | after | Quetzales | Centavo | no |
| GYD | Guyanese dollar | $ | 328 | 0.01 | after | Dollars | Cents | no |
| HKD | Hong Kong dollar | $ | 344 | 0.01 | before | Dollars | Cents | no |
| HNL | Honduran lempira | L | 340 | 0.01 | before | Lempiras | Centavos | no |
| HRK | Croatian kuna | kn | 191 | 0.01 | after | Kuna | Lipa | no |
| HTG | Haitian gourde | G | 332 | 0.01 | after | Gourde | Centime | no |
| HUF | Hungarian forint | Ft | 348 | 0.01 | after | Forint | Filler | no |
| IDR | Indonesian rupiah | Rp | 360 | 0.01 | before | Rupiah | Sen | no |
| ILS | Israeli new shekel | ₪ | 376 | 0.01 | before | Shekel | Agorot | no |
| INR | Indian rupee | ₹ | 356 | 0.01 | before | Rupees | Paise | yes |
| IQD | Iraqi dinar | ع.د | 368 | 0.001 | after | Dinar | Fils | no |
| IRR | Iranian rial | ﷼ | 364 | 0.01 | after | Dinar | Fils | no |
| ISK | Icelandic króna | kr | 352 | 1 | after | Krona | Aurar | no |
| JMD | Jamaican dollar | $ | 388 | 0.01 | after | Dollars | Cents | no |
| JOD | Jordanian dinar | د.ا | 400 | 0.001 | after | Dinar | Fils | no |
| JPY | Japanese yen | ¥ | 392 | 1.00 | before | Yen | Cen | yes |
| KES | Kenyan shilling | KSh | 404 | 0.01 | after | Shilling | Cents | no |
| KGS | Kyrgyzstani som | лв | 417 | 0.01 | after | Som | Tyiyn | no |
| KHR | Cambodian riel | ៛ | 116 | 0.01 | after | Riel | Sen | no |
| KMF | Comorian franc | CF | 174 | 1 | after | Franc | Centime | no |
| KPW | North Korean won | ₩ | 408 | 0.01 | after | Won | Chon | no |
| KRW | South Korean won | ₩ | 410 | 1 | before | Won | Chon | no |
| KWD | Kuwaiti dinar | د.ك | 414 | 0.001 | after | Dinar | Fils | no |
| KYD | Cayman Islands dollar | $ | 136 | 0.01 | after | Dollars | Cents | no |
| KZT | Kazakhstani tenge | ₸ | 398 | 0.01 | after | Tenge | Tiin | no |
| LAK | Lao kip | ₭ | 418 | 0.01 | after | Kip | Att | no |
| LBP | Lebanese pound | ل.ل | 422 | 0.01 | after | Pound | Piastres | no |
| LKR | Sri Lankan rupee | Rs | 144 | 0.01 | after | Rupee | Cents | no |
| LRD | Liberian dollar | L$ | 430 | 0.01 | after | Dollars | Cents | no |
| LSL | Lesotho loti | M | 426 | 0.01 | after | Loti | Sente | no |
| LTL | Lithuanian litas | Lt | 840 | 0.01 | after | Litas | Centas | no |
| LVL | Latvian lats | Ls | 840 | 0.01 | after | Lats | Santims | no |
| LYD | Libyan dinar | ل.د | 434 | 0.001 | after | Dinar | Dirham | no |
| MAD | Moroccan dirham | DH | 504 | 0.01 | after | Dirham | Centimes | no |
| MDL | Moldovan leu | L | 498 | 0.01 | after | Leu | Ban | no |
| MGA | Malagasy ariary | Ar | 969 | 0.01 | after | Ariary | Iraimbilanja | no |
| MKD | Macedonian denar | ден | 807 | 0.01 | after | Denar | Deni | no |
| MMK | Myanmar kyat | K | 104 | 0.01 | after | Kyat | Pya | no |
| MNT | Mongolian tögrög | ₮ | 496 | 0.01 | after | Tugrik | Mongo | no |
| MOP | Macanese pataca | MOP$ | 446 | 0.01 | after | Pataca | Sin | no |
| MRO | Mauritanian ouguiya (old) | UM | 478 | 0.01 | after | Ouguiya | Khoums | no |
| MRU | Mauritanian ouguiya | UM | 478 | 0.01 | after | Ouguiya | Khoums | no |
| MUR | Mauritian rupee | Rs | 480 | 0.01 | after | Rupee | Cents | no |
| MVR | Maldivian rufiyaa | Rf | 462 | 0.01 | before | Rufiyaa | Laari | no |
| MWK | Malawian kwacha | MK | 454 | 0.01 | after | Kwacha | Tambala | no |
| MXN | Mexican peso | $ | 484 | 0.01 | before | Pesos | Centavos | no |
| MYR | Malaysian ringgit | RM | 458 | 0.01 | before | Ringgit | Sen | no |
| MZN | Mozambican metical | MT | 943 | 0.01 | after | Metical | Centavo | no |
| NAD | Namibian dollar | $ | 516 | 0.01 | after | Dollars | Cents | no |
| NGN | Nigerian naira | ₦ | 566 | 0.01 | after | Naira | Kobo | no |
| NIO | Nicaraguan córdoba | C$ | 558 | 0.01 | after | Cordoba | Centavos | no |
| NOK | Norwegian krone | kr | 578 | 0.01 | before | Krone | Ore | no |
| NPR | Nepalese rupee | ₨ | 524 | 0.01 | after | Rupee | Paisa | no |
| NZD | New Zealand dollar | $ | 554 | 0.01 | before | Dollars | Cents | yes |
| OMR | Omani rial | ر.ع. | 512 | 0.001 | after | Rial | Baisa | no |
| OTR |  | OTR |  | 1 | after |  |  | yes |
| PAB | Panamanian balboa | B/. | 590 | 0.01 | after | Balboa | Centesimo | no |
| PEN | Peruvian sol | S/ | 604 | 0.01 | before | Soles | Centimos | no |
| PGK | Papua New Guinean kina | K | 598 | 0.01 | after | Kina | Toea | no |
| PHP | Philippine peso | ₱ | 608 | 0.01 | after | Peso | Centavos | no |
| PKR | Pakistani rupee | Rs. | 586 | 0.01 | before | Rupee | Paisa | no |
| PLN | Polish złoty | zł | 985 | 0.01 | after | Zloty | Groszy | no |
| PYG | Paraguayan guaraní | ₲ | 600 | 1 | after | Guarani | Centimos | no |
| QAR | Qatari riyal | QR | 634 | 0.01 | after | Riyal | Dirham | no |
| RON | Romanian leu | lei | 946 | 0.01 | after | Leu | Bani | no |
| RSD | Serbian dinar | din. | 941 | 0.01 | after | Dinar | Para | no |
| RUB | Russian ruble | руб | 643 | 0.01 | after | Ruble | Kopek | no |
| RWF | Rwandan franc | RF | 646 | 1 | after | Franc | Santime | no |
| SAR | Saudi riyal | SR | 682 | 0.01 | after | Riyal | Halala | no |
| SBD | Solomon Islands dollar | SI$ | 090 | 0.01 | after | Dollars | Cents | no |
| SCR | Seychellois rupee | SR | 690 | 0.01 | after | Rupee | Cents | no |
| SDG | Sudanese pound | ج.س. | 938 | 0.01 | after |  |  | no |
| SEK | Swedish krona | kr | 752 | 0.01 | after | Krona | Ore | no |
| SGD | Singapore dollar | S$ | 702 | 0.01 | before | Dollars | Cents | no |
| SHP | Saint Helena pound | £ | 654 | 0.01 | after | Pound | Penny | no |
| SLE | Sierra Leonean leone | Le | 694 | 0.01 | after | Leone | Cents | no |
| SLL | Sierra Leonean leone | Le | 694 | 0.01 | after | Leone | Cents | no |
| SOS | Somali shilling | Sh. | 706 | 0.01 | after | Shillings | Senti | no |
| SRD | Surinamese dollar | $ | 968 | 0.01 | after | Dollars | Cents | no |
| SSP | South Sudanese pound | £ | 728 | 0.01 | after | Pounds | Piasters | no |
| STD | São Tomé and Príncipe dobra | Db | 678 | 0.01 | after | Dobra | Centimo | no |
| STN | São Tomé and Príncipe dobra | Db | 678 | 0.01 | after | Dobra | cêntimo | no |
| SVC | Salvadoran Colon | ¢ | 222 | 0.01 | after | Colones | Centavo | no |
| SYP | Syrian pound | £ | 760 | 0.01 | after | Pound | Piastrp | no |
| SZL | Swazi lilangeni | E | 748 | 0.01 | after | Lilangeni | Cents | no |
| THB | Thai baht | ฿ | 764 | 0.01 | after | Baht | Satang | no |
| TJS | Tajikistani somoni | TJS | 972 | 0.01 | after | Somoni | Diram | no |
| TMT | Turkmenistan manat | T | 934 | 0.01 | after | Manat | Tenge | no |
| TND | Tunisian dinar | DT | 788 | 0.001 | after | Dinar | Millimes | no |
| TOP | Tongan paʻanga | T$ | 776 | 0.01 | after | Paanga | Seniti | no |
| TRY | Turkish lira | ₺ | 949 | 0.01 | after | Lira | Kurus | no |
| TTD | Trinidad and Tobago dollar | $ | 780 | 0.01 | after | Dollars | Cents | no |
| TWD | New Taiwan dollar | NT$ | 901 | 1.00 | before | Dollars | Cents | yes |
| TZS | Tanzanian shilling | TSh | 834 | 0.01 | after | Shilling | Senti | no |
| UAH | Ukraine Hryvnia | ₴ | 980 | 0.01 | after | Hryvnia | Kopiyka | no |
| UF |  | UF |  | 0.01 | after | Development Unit |  | yes |
| UGX | Ugandan shilling | USh | 800 | 1 | after | Shilling | Cents | no |
| USD | United States dollar | $ | 840 | 0.01 | before | Dollars | Cents | no |
| UTM |  | UTM |  | 0.01 | after | Monthly Tax Unit |  | yes |
| UYI | Uruguay Peso en Unidades Indexadas | $ | 940 | 0.0001 | after | Peso | centésimo | yes |
| UYU | Uruguayan peso | $ | 858 | 0.01 | after | Peso | Centesimos | no |
| UYW | Unidad previsional | $ | 858 | 0.0001 | after | peso | centésimo | no |
| UZS | Uzbekistan som | лв | 860 | 0.01 | after | Som | Tiyin | no |
| VEF | Venezuelan bolívar fuerte | Bs.F | 937 | 0.01 | after | Bolivar | Centimos | no |
| VES | Venezuelan bolívar soberano | Bs | 937 | 0.01 | after |  |  | no |
| VND | Vietnamese đồng | ₫ | 704 | 1.00 | after | Dong | Xu | no |
| VUV | Vanuatu vatu | VT | 548 | 1 | after | Vatu |  | no |
| WST | Samoan tālā | WS$ | 882 | 0.01 | after | Tala | Sene | no |
| XAF | CFA franc BEAC | FCFA | 950 | 1 | after | Franc | Centimes | no |
| XCD | East Caribbean dollar | $ | 951 | 0.01 | after | Dollars | Cents | no |
| XCG | Caribbean Guilder | Cg |  | 0.01 | after | Guilder | Cents | no |
| XOF | CFA franc BCEAO | CFA | 952 | 1 | after | Franc | Centimes | no |
| XPF | CFP franc | XPF | 953 | 1.00 | after | Franc | Centimes | no |
| YER | Yemeni rial | ﷼ | 886 | 0.01 | after | Rial | Fils | no |
| ZAR | South African rand | R | 710 | 0.01 | before | Rand | Cents | no |
| ZIG | Zimbabwe Gold | ZiG |  | 0.01 | after | ZiGs |  | no |
| ZMW | Zambian kwacha | ZK | 967 | 0.01 | after | Kwacha | Ngwee | no |

**Reading the rounding column.** A rounding factor of `0.01` means two decimal places; `0.001`
means three; `1.0` means whole units only, which is how currencies with no subunit are expressed;
`0.05` means amounts round to the nearest five hundredths, which is how a currency whose smallest
coin is five subunits is expressed.

## 8.5 Languages

Ninety-three languages ship. All of them ship inactive except the base language, which the
installation activates. The grouping column is the digit-grouping specification described in
[calculations.md](calculations.md) §10.3.

| Name | Locale code | Language code | Direction | Date format | Time format | First day of week | Grouping | Decimal separator | Thousands separator |
|---|---|---|---|---|---|---|---|---|---|
| Amharic / አምሃርኛ | `am_ET` | `am_ET` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| Arabic / الْعَرَبيّة | `ar_001` | `ar` | Right-to-Left | `%d/%m/%Y` | `%I:%M:%S %p` | Saturday | `[3,0]` | `.` | `,` |
| Arabic (Syria) / الْعَرَبيّة | `ar_SY` | `ar_SY` | Right-to-Left | `%d/%m/%Y` | `%I:%M:%S %p` | Saturday | `[3,0]` | `.` | `,` |
| Azerbaijani / Azərbaycanca | `az_AZ` | `az` | Left-to-Right | `%Y-%m-%d` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Belarusian / Беларуская мова | `be_BY` | `be` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | space |
| Bulgarian / български език | `bg_BG` | `bg` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | (empty) |
| Bengali / বাংলা | `bn_IN` | `bn_IN` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Monday | `[3,0]` | `,` | (empty) |
| Bosnian / bosanski jezik | `bs_BA` | `bs` | Left-to-Right | `%Y-%m-%d` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Catalan / Català | `ca_ES` | `ca_ES` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Czech / Čeština | `cs_CZ` | `cs_CZ` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Danish / Dansk | `da_DK` | `da_DK` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| German (CH) / Deutsch (CH) | `de_CH` | `de_CH` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | apostrophe |
| German / Deutsch | `de_DE` | `de` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Greek / Ελληνικά | `el_GR` | `el_GR` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Monday | `[3,0]` | `,` | `.` |
| English (AU) | `en_AU` | `en_AU` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| English (CA) | `en_CA` | `en_CA` | Left-to-Right | `%Y-%m-%d` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| English (UK) | `en_GB` | `en_GB` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | `,` |
| English (IN) | `en_IN` | `en_IN` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Sunday | `[3,2,0]` | `.` | `,` |
| English (NZ) | `en_NZ` | `en_NZ` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| English (US) | `en_US` | `en` | Left-to-Right | `%m/%d/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| Spanish (Latin America) / Español (América Latina) | `es_419` | `es_419` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Spanish (AR) / Español (AR) | `es_AR` | `es_AR` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | `.` |
| Spanish (BO) / Español (BO) | `es_BO` | `es_BO` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Spanish (CL) / Español (CL) | `es_CL` | `es_CL` | Left-to-Right | `%d-%m-%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Spanish (CO) / Español (CO) | `es_CO` | `es_CO` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | `.` |
| Spanish (CR) / Español (CR) | `es_CR` | `es_CR` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | `,` |
| Spanish (DO) / Español (DO) | `es_DO` | `es_DO` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Monday | `[3,0]` | `.` | `,` |
| Spanish (EC) / Español (EC) | `es_EC` | `es_EC` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Spanish / Español | `es_ES` | `es` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Spanish (GT) / Español (GT) | `es_GT` | `es_GT` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Spanish (MX) / Español (MX) | `es_MX` | `es_MX` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Spanish (PA) / Español (PA) | `es_PA` | `es_PA` | Left-to-Right | `%m/%d/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Spanish (PE) / Español (PE) | `es_PE` | `es_PE` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Spanish (PY) / Español (PY) | `es_PY` | `es_PY` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | `.` |
| Spanish (UY) / Español (UY) | `es_UY` | `es_UY` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Spanish (VE) / Español (VE) | `es_VE` | `es_VE` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | `.` |
| Estonian / Eesti keel | `et_EE` | `et` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Basque / Euskara | `eu_ES` | `eu_ES` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | (empty) |
| Persian / فارسی | `fa_IR` | `fa` | Right-to-Left | `%Y/%m/%d` | `%H:%M:%S` | Saturday | `[3,0]` | `.` | `,` |
| Finnish / Suomi | `fi_FI` | `fi` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| French (BE) / Français (BE) | `fr_BE` | `fr_BE` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| French (CA) / Français (CA) | `fr_CA` | `fr_CA` | Left-to-Right | `%Y-%m-%d` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | non-breaking space |
| French (CH) / Français (CH) | `fr_CH` | `fr_CH` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | apostrophe |
| French / Français | `fr_FR` | `fr` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Galician / Galego | `gl_ES` | `gl` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | (empty) |
| Gujarati / ગુજરાતી | `gu_IN` | `gu` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| Hebrew / עברית | `he_IL` | `he` | Right-to-Left | `%d.%m.%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Hindi / हिंदी | `hi_IN` | `hi` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Sunday | `[3,2,0]` | `.` | `,` |
| Croatian / hrvatski jezik | `hr_HR` | `hr` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Hungarian / Magyar | `hu_HU` | `hu` | Left-to-Right | `%Y.%m.%d` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Indonesian / Bahasa Indonesia | `id_ID` | `id` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | `.` |
| Italian / Italiano | `it_IT` | `it` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Japanese / 日本語 | `ja_JP` | `ja` | Left-to-Right | `%Y/%m/%d` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Georgian / ქართული ენა | `ka_GE` | `ka` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Kabyle / Taqbaylit | `kab_DZ` | `kab` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Saturday | `[3,0]` | `.` | `,` |
| Khmer / ភាសាខ្មែរ | `km_KH` | `km` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Korean (KP) / 한국어 (KP) | `ko_KP` | `ko_KP` | Left-to-Right | `%Y.%m.%d` | `%I:%M:%S %p` | Monday | `[3,0]` | `.` | `,` |
| Korean (KR) / 한국어 (KR) | `ko_KR` | `ko_KR` | Left-to-Right | `%Y.%m.%d` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Luxembourgish | `lb_LU` | `lb` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Lao / ພາສາລາວ | `lo_LA` | `lo` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Lithuanian / Lietuvių kalba | `lt_LT` | `lt` | Left-to-Right | `%Y-%m-%d` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Latvian / latviešu valoda | `lv_LV` | `lv` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Macedonian / македонски јазик | `mk_MK` | `mk` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Malayalam / മലയാളം | `ml_IN` | `ml` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Mongolian / монгол | `mn_MN` | `mn` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | apostrophe |
| Malay / Bahasa Melayu | `ms_MY` | `ms` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | `,` |
| Burmese / ဗမာစာ | `my_MM` | `my` | Left-to-Right | `%m/%d/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| Norwegian Bokmål / Norsk bokmål | `nb_NO` | `nb_NO` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Dutch (BE) / Nederlands (BE) | `nl_BE` | `nl_BE` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Dutch / Nederlands | `nl_NL` | `nl` | Left-to-Right | `%d-%m-%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Polish / Język polski | `pl_PL` | `pl` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | (empty) |
| Portuguese (AO) / Português (AO) | `pt_AO` | `pt_AO` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | (empty) |
| Portuguese (BR) / Português (BR) | `pt_BR` | `pt_BR` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | `.` |
| Portuguese / Português | `pt_PT` | `pt` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | (empty) |
| Romanian / română | `ro_RO` | `ro` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Russian / русский язык | `ru_RU` | `ru` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Slovak / Slovenský jazyk | `sk_SK` | `sk` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Slovenian / slovenščina | `sl_SI` | `sl` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Albanian / Shqip | `sq_AL` | `sq` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Serbian (Cyrillic) / српски | `sr@Cyrl` | `sr@Cyrl` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `,` | (empty) |
| Serbian (Latin) / srpski | `sr@latin` | `sr@latin` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Swedish / Svenska | `sv_SE` | `sv` | Left-to-Right | `%Y-%m-%d` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Swahili / Kiswahili | `sw` | `sw` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | `,` |
| Telugu / తెలుగు | `te_IN` | `te` | Left-to-Right | `%d/%m/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| Thai / ภาษาไทย | `th_TH` | `th` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Tagalog / Filipino | `tl_PH` | `tl` | Left-to-Right | `%m/%d/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | `,` |
| Turkish / Türkçe | `tr_TR` | `tr` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Ukrainian / українська | `uk_UA` | `uk` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | non-breaking space |
| Uzbek / Oʻzbekcha | `uz_UZ` | `uz` | Left-to-Right | `%d.%m.%Y` | `%H:%M:%S` | Monday | `[3,0]` | `.` | non-breaking space |
| Vietnamese / Tiếng Việt | `vi_VN` | `vi` | Left-to-Right | `%d/%m/%Y` | `%H:%M:%S` | Monday | `[3,0]` | `,` | `.` |
| Chinese, Simplified / 简体中文 | `zh_CN` | `zh_CN` | Left-to-Right | `%Y/%m/%d` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |
| Chinese, Traditional (HK) / 繁體中文 (香港) | `zh_HK` | `zh_HK` | Left-to-Right | `%m/%d/%Y` | `%I:%M:%S %p` | Sunday | `[3,0]` | `.` | `,` |
| Chinese, Traditional (TW) / 繁體中文 (台灣) | `zh_TW` | `zh_TW` | Left-to-Right | `%Y/%m/%d` | `%H:%M:%S` | Sunday | `[3,0]` | `.` | `,` |

**Observations.**

- Only two grouping specifications occur: `[3,0]` — group by three, repeating — and `[3,2,0]` —
  three, then two, repeating. The second is used by the languages of the Indian subcontinent.
- Three thousands separators occur: the comma, the full stop and the non-breaking space. A rebuild
  must store the non-breaking space as the actual character, not as an ordinary space: the
  separator field is explicitly **not** trimmed.
- The decimal separator is either the full stop or the comma. Never empty — it is required.
- The first day of the week is Sunday, Monday or Saturday depending on the language.

## 8.6 Industries

Twenty-one industries ship, following the standard economic-activity classification with
single-letter section codes. The two records with codes beyond the standard sections are
reproduced as they ship.

| Short name | Full name |
|---|---|
| Agriculture | A - AGRICULTURE, FORESTRY AND FISHING |
| Mining | B - MINING AND QUARRYING |
| Manufacturing | C - MANUFACTURING |
| Energy supply | D - ELECTRICITY, GAS, STEAM AND AIR CONDITIONING SUPPLY |
| Water supply | E - WATER SUPPLY; SEWERAGE, WASTE MANAGEMENT AND REMEDIATION ACTIVITIES |
| ECO liable to deduct TCS u/s 52 | E-Commerce operator liable to deduct TCS under section 52 |
| ECO liable to pay GST u/s 9(5) | E-Commerce operator liable to pay tax under section 9(5) |
| Construction | F - CONSTRUCTION |
| Wholesale/Retail | G - WHOLESALE AND RETAIL TRADE; REPAIR OF MOTOR VEHICLES AND MOTORCYCLES |
| Transportation/Logistics | H - TRANSPORTATION AND STORAGE |
| Food/Hospitality | I - ACCOMMODATION AND FOOD SERVICE ACTIVITIES |
| IT/Communication | J - INFORMATION AND COMMUNICATION |
| Finance/Insurance | K - FINANCIAL AND INSURANCE ACTIVITIES |
| Real Estate | L - REAL ESTATE ACTIVITIES |
| Scientific | M - PROFESSIONAL, SCIENTIFIC AND TECHNICAL ACTIVITIES |
| Administrative/Utilities | N - ADMINISTRATIVE AND SUPPORT SERVICE ACTIVITIES |
| Public Administration | O - PUBLIC ADMINISTRATION AND DEFENCE; COMPULSORY SOCIAL SECURITY |
| Education | P - EDUCATION |
| Health/Social | Q - HUMAN HEALTH AND SOCIAL WORK ACTIVITIES |
| Entertainment | R - ARTS, ENTERTAINMENT AND RECREATION |
| Other Services | S - OTHER SERVICE ACTIVITIES |
| Households | T - ACTIVITIES OF HOUSEHOLDS AS EMPLOYERS; UNDIFFERENTIATED GOODS- AND SERVICES-PRODUCING ACTIVITIES OF HOUSEHOLDS FOR OWN USE |
| Extraterritorial | U - ACTIVITIES OF EXTRATERRITORIAL ORGANISATIONS AND BODIES |

## 8.7 Banks

One hundred and fifteen bank records ship, almost all of them from country-specific packages and
active; one shipped by the foundation package is inactive and serves only as an example.

| Bank name | Bank identifier code | Country | City | Street | Postal code | Telephone | Electronic mail address | Active |
|---|---|---|---|---|---|---|---|---|
| Reserve |  |  |  |  |  |  |  | no |
| "Swedbank", AB | HABALT22 | LT | Vilnius | Konstitucijos pr. 20A | LT-03502 | +370 5 2684444 | info@swedbak.lt | yes |
| AB "Citadele" bankas | INDULT2X | LT | Vilnius | K. Kalinausko g. 13 | LT-03107 | +370 5 2664600 | info@citadele.lt | yes |
| AB SEB bankas | CBVILT2X | LT | Vilnius | Gedimino pr. 12, | LT-01103 | +370 5 2682800 | info@seb.lt | yes |
| AB Šiaulių bankas | CBSBLT26 | LT | Šiauliai | Tilžės g.149 | LT-76348 | +370 4 1595607 | info@sb.lt | yes |
| Danske bank A/S Lietuvos filialas | SMPOLT22 | LT | Vilnius | Saltoniškių g. 2 | LT-08500 | +370 5 215666 | info@danskebank.lt | yes |
| Luminor Bank AB | AGBLLT2X | LT | Vilnius | Konstitucijos pr. 21A | LT-08130 | +370 5 2393444 | info@luminor.lt | yes |
| Nordea Bank AB | NDEALT2X | LT | Vilnius | Didžioji g. 18 | LT-01128 | +370 5 2361361 | info@nordea.lt | yes |
| UAB "Paysera LT" | EVIULT21 | LT | Vilnius | Mėnulio g. 7 | LT-04326 | +370 5 2071558 | info@paysera.lt | yes |
| UAB Medicinos bankas | MDBALT22 | LT | Vilnius | Pamėnkalnio g. 40 | LT-01114 | +370 800 60700 | info@medbank.lt | yes |
| ABC CAPITAL |  | MX |  |  |  |  |  | yes |
| ACCIVAL |  | MX |  |  |  |  |  | yes |
| ACTINVER |  | MX |  |  |  |  |  | yes |
| AFIRME |  | MX |  |  |  |  |  | yes |
| AMERICAN EXPRESS |  | MX |  |  |  |  |  | yes |
| ASEA |  | MX |  |  |  |  |  | yes |
| AUTOFIN |  | MX |  |  |  |  |  | yes |
| AZTECA |  | MX |  |  |  |  |  | yes |
| B&B |  | MX |  |  |  |  |  | yes |
| BAJIO |  | MX |  |  |  |  |  | yes |
| BAMSA |  | MX |  |  |  |  |  | yes |
| BANAMEX |  | MX |  |  |  |  |  | yes |
| BANCO FAMSA |  | MX |  |  |  |  |  | yes |
| BANCO FINTERRA |  | MX |  |  |  |  |  | yes |
| BANCO S3 |  | MX |  |  |  |  |  | yes |
| BANCOMEXT |  | MX |  |  |  |  |  | yes |
| BANCOPPEL |  | MX |  |  |  |  |  | yes |
| BANCREA |  | MX |  |  |  |  |  | yes |
| BANJERCITO |  | MX |  |  |  |  |  | yes |
| BANK OF CHINA |  | MX |  |  |  |  |  | yes |
| BANKAOOL |  | MX |  |  |  |  |  | yes |
| BANOBRAS |  | MX |  |  |  |  |  | yes |
| BANORTE/IXE |  | MX |  |  |  |  |  | yes |
| BANREGIO |  | MX |  |  |  |  |  | yes |
| BANSEFI |  | MX |  |  |  |  |  | yes |
| BANSI |  | MX |  |  |  |  |  | yes |
| BARCLAYS |  | MX |  |  |  |  |  | yes |
| BBASE |  | MX |  |  |  |  |  | yes |
| BBVA BANCOMER |  | MX |  |  |  |  |  | yes |
| BMONEX |  | MX |  |  |  |  |  | yes |
| BMULTIVA |  | MX |  |  |  |  |  | yes |
| BULLTICK CB |  | MX |  |  |  |  |  | yes |
| CB ACTINVER |  | MX |  |  |  |  |  | yes |
| CB INTERCAM |  | MX |  |  |  |  |  | yes |
| CB JPMORGAN |  | MX |  |  |  |  |  | yes |
| CBDEUTSCHE |  | MX |  |  |  |  |  | yes |
| CI BOLSA |  | MX |  |  |  |  |  | yes |
| CIBANCO |  | MX |  |  |  |  |  | yes |
| CLS |  | MX |  |  |  |  |  | yes |
| COMPARTAMOS |  | MX |  |  |  |  |  | yes |
| CONSUBANCO |  | MX |  |  |  |  |  | yes |
| CREDIT SUISSE |  | MX |  |  |  |  |  | yes |
| DEUTSCHE |  | MX |  |  |  |  |  | yes |
| DONDÉ |  | MX |  |  |  |  |  | yes |
| ESTRUCTURADORES |  | MX |  |  |  |  |  | yes |
| EVERCORE |  | MX |  |  |  |  |  | yes |
| FINAMEX |  | MX |  |  |  |  |  | yes |
| FINCOMUN |  | MX |  |  |  |  |  | yes |
| FORJADORES |  | MX |  |  |  |  |  | yes |
| GBM |  | MX |  |  |  |  |  | yes |
| HDI SEGUROS |  | MX |  |  |  |  |  | yes |
| HIPOTECARIA FEDERAL |  | MX |  |  |  |  |  | yes |
| HSBC |  | MX |  |  |  |  |  | yes |
| ICBC |  | MX |  |  |  |  |  | yes |
| INBURSA |  | MX |  |  |  |  |  | yes |
| INDEVAL |  | MX |  |  |  |  |  | yes |
| INMOBILIARIO |  | MX |  |  |  |  |  | yes |
| INTERACCIONES |  | MX |  |  |  |  |  | yes |
| INTERCAM BANCO |  | MX |  |  |  |  |  | yes |
| INTERCAM BANCO |  | MX |  |  |  |  |  | yes |
| INVEX |  | MX |  |  |  |  |  | yes |
| JP MORGAN |  | MX |  |  |  |  |  | yes |
| KUSPIT |  | MX |  |  |  |  |  | yes |
| LIBERTAD |  | MX |  |  |  |  |  | yes |
| MAPFRE |  | MX |  |  |  |  |  | yes |
| MASARI |  | MX |  |  |  |  |  | yes |
| MERRILL LYNCH |  | MX |  |  |  |  |  | yes |
| MIFEL |  | MX |  |  |  |  |  | yes |
| MIZUHO BANK |  | MX |  |  |  |  |  | yes |
| MONEXCB |  | MX |  |  |  |  |  | yes |
| N/A |  | MX |  |  |  |  |  | yes |
| NAFIN |  | MX |  |  |  |  |  | yes |
| NU |  | MX |  |  |  |  |  | yes |
| OACTIN |  | MX |  |  |  |  |  | yes |
| OPCIONES EMPRESARIALES DEL NOROESTE |  | MX |  |  |  |  |  | yes |
| ORDER |  | MX |  |  |  |  |  | yes |
| PAGATODO |  | MX |  |  |  |  |  | yes |
| PROFUTURO |  | MX |  |  |  |  |  | yes |
| PROGRESO |  | MX |  |  |  |  |  | yes |
| REFORMA |  | MX |  |  |  |  |  | yes |
| SABADELL |  | MX |  |  |  |  |  | yes |
| SANTANDER |  | MX |  |  |  |  |  | yes |
| SCOTIABANK |  | MX |  |  |  |  |  | yes |
| SEGMTY |  | MX |  |  |  |  |  | yes |
| SHINHAN |  | MX |  |  |  |  |  | yes |
| SKANDIA |  | MX |  |  |  |  |  | yes |
| SKANDIA |  | MX |  |  |  |  |  | yes |
| SOFIEXPRESS |  | MX |  |  |  |  |  | yes |
| STERLING |  | MX |  |  |  |  |  | yes |
| STP |  | MX |  |  |  |  |  | yes |
| SU CASITA |  | MX |  |  |  |  |  | yes |
| TELECOMM |  | MX |  |  |  |  |  | yes |
| THE ROYAL BANK |  | MX |  |  |  |  |  | yes |
| TIBER |  | MX |  |  |  |  |  | yes |
| TOKYO |  | MX |  |  |  |  |  | yes |
| UBS BANK |  | MX |  |  |  |  |  | yes |
| UNAGRA |  | MX |  |  |  |  |  | yes |
| UNICA |  | MX |  |  |  |  |  | yes |
| VALMEX |  | MX |  |  |  |  |  | yes |
| VALUE |  | MX |  |  |  |  |  | yes |
| VE POR MAS |  | MX |  |  |  |  |  | yes |
| VECTOR |  | MX |  |  |  |  |  | yes |
| VOLKSWAGEN |  | MX |  |  |  |  |  | yes |
| ZURICH |  | MX |  |  |  |  |  | yes |
| ZURICHVI |  | MX |  |  |  |  |  | yes |

## 8.8 Party tags

Twenty-one party tags ship, all of them from one country-specific electronic-invoicing package
where each tag names a kind of party identifier that the national format can carry. Nineteen of
them ship archived; only the two that the format requires ship active.

| Tag name | Active |
|---|---|
| ABONENO | no |
| ARACIKURUMETIKET | no |
| ARACIKURUMVKN | no |
| BAYINO | no |
| CIFTCINO | no |
| DISTRIBUTORNO | no |
| DOSYANO | no |
| EPDKNO | no |
| HASTANO | no |
| HIZMETNO | no |
| IMALATCINO | no |
| MERSISNO | yes |
| MUSTERINO | no |
| PASAPORTNO | no |
| SAYACNO | no |
| SUBENO | no |
| TAPDKNO | no |
| TELEFONNO | no |
| TESISATNO | no |
| TICARETSICILNO | yes |
| URETICINO | no |

## 8.9 Controlled cities

Two thousand eight hundred and fifty controlled-city records ship, from the country-specific
packages of the countries that enforce the city list. They are not reproduced here: they are
per-country data belonging to [Fiscal Localizations](../fiscal-localizations/README.md), and the
city entity itself carries no behaviour beyond the cascade documented in
[entities.md](entities.md) §7.3. What a rebuild must reproduce from this domain is the entity, the
enforcement flag, the cascade and the lookup — not the list.

## 8.10 Geocoding providers

| Technical name | Displayed name |
|---|---|
| `openstreetmap` | `Open Street Map` |
| `googlemap` | `Google Place Map` |

## 8.11 Shipped parties and companies

See §6.

---

# 9. What the domain does **not** configure

- **No sequence.** See §4.
- **No approval workflow, no validation rule set, no locking configuration.**
- **No per-company defaults** beyond the company-dependent barcode. In particular the language,
  the country and the currency of a new Party are not company-defaults: the language comes from a
  user-defined default or the session, and the country and currency come from nowhere at all.
- **No numbering format for bank accounts.** The account number is whatever the counterparty says
  it is; only its sanitised form is derived.
- **No retention policy.** Nothing in this domain is ever deleted on a schedule.
