# Reference data

The records a fresh installation must contain before anyone uses it. Nothing here is transactional: these are the countries, currencies, languages, units, tax structures, message subtypes, numbering rules and access groups that the rest of the specification assumes already exist. A rebuild that does not ship them produces a system in which a partner cannot be given a country, an amount cannot be given a currency, a document cannot be numbered and a user cannot be given a role.

This document enumerates the sets, states the rules that govern each of them, and points at the machine-readable catalogue that holds every record. The catalogue is [`../../schemas/data/reference-data/`](../../schemas/data/reference-data/): one document per record set, keyed by external identifier; a `data-files` folder holding the tabular sets; a `chart-templates` folder holding the country accounting templates; and an index. The operational catalogues [`../../schemas/operational/`](../../schemas/operational/) hold the decimal precisions, activity types, message subtypes, numbering sequences, groups, privileges, access rights, record rules, scheduled jobs and system parameters. A summary table of every set is also generated as [`../references/reference-data.md`](../references/reference-data.md).

How these records are loaded, how a re-installation or an update rewrites them, and what protects a record a user has edited, are specified in [`data-loading-and-exchange.md`](data-loading-and-exchange.md). The fields of each entity are specified in the domain folder that owns it and on its reference page under [`../references/entities/`](../references/entities/).

## 1. What a fresh installation contains

Reference data reaches the tables through three routes, and the counts of each are different because they count different things.

| Route | Where it lives | Sets | Records |
|---|---|---|---|
| Declaration documents | One document per entity in the catalogue, keyed by external identifier | 155 | 19,008 declarations producing 18,737 distinct records; the difference is the 271 declarations by which one package adds fields to a record another package created |
| Tabular data files | The `data-files` folder, one document per package and entity | 69 | 41,336 rows |
| Country accounting templates | The `chart-templates` folder, one document per package, entity and chart | 127 packages covering 151 charts | 66,368 rows |

Three rules apply to all three routes and are what make an installation reproducible:

1. **Every shipped record carries an external identifier.** It is the key by which a later update finds the record again, and the key by which the record is removed when its package is removed.
2. **A record whose external identifier is marked not updatable is written once and never rewritten.** This is how a shipped starting value becomes the tenant's own value: a chart of accounts, a payment term, a message template.
3. **A record removed from a package's data files is deleted from the tenant on the next update of that package**, unless it is marked not updatable.

Throughout this document a value written in quotation marks is a label or a message reproduced exactly as the system shows it, and a value written in code font is a stored identifier reproduced exactly.

Not every set below is installed in every tenant: a set belongs to a capability package, and the package must be installed for the set to exist. The counts are those of an installation carrying every package. The country accounting templates are the extreme case: a tenant installs one country package, not one hundred and twenty-seven.

## 2. The founding records

These exist before anything else and every other record points at them, directly or indirectly.

| Record | External identifier | Entity | What it is |
|---|---|---|---|
| The first company | `base.main_company` | Company (`res.company`) | The only company a fresh installation has. Its name is "My Company", its currency is the United States dollar (`base.USD`) and its party record is the main party. A localization package may attach a paper format to it and the working-time package attaches the standard calendar. |
| The party of the first company | `base.main_partner` | Party (`res.partner`) | An organization record with no company of its own, so that it is visible to every company of a group. |
| The system party | `base.partner_root` | Party (`res.partner`) | The counterparty of the automated user. Archived, so that it never appears in a list. |
| The automated user | `base.user_root` | User (`res.users`) | The identity under which scheduled jobs, automated rules and system messages act. It has no login. Its notification preference is the internal inbox. |
| The administrator party | `base.partner_admin` | Party (`res.partner`) | The counterparty of the administrator. |
| The administrator | `base.user_admin` | User (`res.users`) | The first human account. Its login and its initial password are both `admin`, and a rebuild must force a change at first sign-in. It carries the administration groups of every installed package. |
| The administrator's settings | `base.user_admin_settings` | User Settings (`res.users.settings`) | The per-user preferences record of the administrator. |
| The anonymous user | `base.public_user` | User (`res.users`) | The identity under which an unauthenticated visitor reads public pages. Archived, with the login `public`, an empty password and the public group as its only group. |
| The anonymous party | `base.public_partner` | Party (`res.partner`) | The counterparty of the anonymous user. Archived. |
| The portal user template | `base.template_portal_user_id` | User (`res.users`) | An archived account whose groups are copied when an external user is granted portal access. Its login is `portaltemplate`. |
| The standard working schedule | `resource.resource_calendar_std` | Working Schedule (`resource.calendar`) | Forty hours a week, attached to the first company. |

## 3. Countries, subdivisions, cities and country groups

### 3.1 Countries

The entity Country (`res.country`, table `res_country`) is declared 261 times by 11 packages and produces **251 distinct countries**. The ten extra declarations are localization packages adding a country-specific code to a country the foundation package already created.

| Rule | Statement |
|---|---|
| Identity | The external identifier of a country is the foundation package, a dot and the two-letter code in lower case, for example `base.us`, `base.fr`, `base.be`. |
| Name uniqueness | The constraint `_name_uniq` over `name`, with the message "The name of the country must be unique!" |
| Code uniqueness | The constraint `_code_uniq` over `code`, with the message "The code of the country must be unique!" |
| Ordering | By name, then by identifier. |
| Address layout | The field `address_format` holds the template used to render a postal address for that country. Its default places the street on the first line, the second street line on the second, the city, the subdivision code and the postal code on the third, and the country name on the fourth. |
| Address input | The field `address_view_id` may name a view that replaces the standard address block for that country. |
| Currency | The field `currency_id` names the country's currency, and is what a company created for that country defaults to. |
| Calling code | The field `phone_code` holds the international dialling prefix as a whole number. |
| Name position | The field `name_position` decides whether the recipient's name is rendered before or after the address; the default is before. |
| Tax registration label | The field `vat_label` holds the local name of the tax registration number, so that a form asks for it by the name the reader expects. |
| Subdivision required | The field `state_required` decides whether an address in that country must carry a subdivision; the default is false. |
| Postal code required | The field `zip_required` decides whether an address in that country must carry a postal code; the default is **true**. |
| City list enforced | The field `enforce_cities` decides whether the city must be chosen from the shipped city list rather than typed. |

Nine localization packages additionally rewrite existing countries through tabular data files, adding codes used by their tax administrations: 228 rows for one South American country package and 167 for another. Those rows update countries; they do not create new ones.

### 3.2 Country subdivisions

The entity Country state (`res.country.state`, table `res_country_state`) holds the first-level subdivisions — states, provinces, regions, departments, cantons, governorates.

| Source | Records |
|---|---|
| The foundation package's tabular data file | 2,131 |
| Declaration documents of one localization package | 39 |
| Tabular data files of three further localization packages | 31 |

| Rule | Statement |
|---|---|
| Required fields | The country (`country_id`), the name (`name`) and the code (`code`) are all required. |
| Uniqueness | The constraint `_name_code_uniq` over the pair (`country_id`, `code`), with the message "The code of the state must be unique by country!" |
| Ordering | By code, then by identifier. |
| Identity | The external identifier is the foundation package, a dot, the word `state`, an underscore, the country code and an ordinal, for example `base.state_au_1`. |

### 3.3 Cities

The entity City (`res.city`, table `res_city`) is shipped only where a country enforces a city list: 2,850 cities are declared by one country package and 6,137 further rows arrive through the tabular data files of that and other country packages. A city belongs to a country and, where the country has them, to a subdivision.

### 3.4 Country groups

The entity Country Group (`res.country.group`, table `res_country_group`) holds 19 named sets of countries used by tax rules, delivery rules and pricelists. Each carries a name, a code and the list of member countries; two of them also carry a list of excluded subdivisions, because a tax territory can be smaller than a country.

| External identifier | Code | Meaning |
|---|---|---|
| `base.europe` | `EU` | The member states of the European Union. |
| `base.europe_prefix` | `EU_PREFIX` | The member states whose tax registration number carries the country prefix. |
| `account.europe_vat` | `EU-VAT` | The member states for value-added tax purposes. |
| `account.intrastat` | `INTRASTAT` | The member states that report intra-community trade statistics. |
| `l10n_fr_account.europe_vat_without_mc` | `EU-VAT-no-mc` | The same set without the principality that shares its neighbour's tax system. |
| `base.sepa_zone` | `SEPA` | The countries of the single euro payments area. |
| `base.south_america` | `SA` | The South American countries. |
| `base.gulf_cooperation_council` | `GCC` | The six Gulf Cooperation Council states. |
| `l10n_ae.gcc_countries_group` | `GCC-VAT` | The Gulf states that have implemented value-added tax. |
| `base.eurasian_economic_union` | `EEU` | The five Eurasian Economic Union states. |
| `base.ch_and_li` | `CH-LI` | The two countries of the Swiss customs union. |
| `base.dom-tom` | `DOM-TOM` | The French overseas departments and territories. |
| `l10n_fr.fr_and_mc` | `FR-MC` | France and the neighbouring principality. |
| `l10n_fr.fr_and_mc_and_drom` | `FR-MC-DROM` | The same, plus the overseas departments. |
| `l10n_es.mainland_es` | `ES-VAT` | Spain excluding the island and enclave territories, which are listed as excluded subdivisions. |
| `l10n_nl.mainland_nl` | `NL-VAT` | The European part of the Netherlands, with the overseas subdivisions excluded. |
| `l10n_in.inter_state_group` | `IN-INTER` | India, with one union territory excluded, used to decide whether a supply is inter-state. |
| `l10n_latam_base.latam` | `LATAMID` | The Latin American countries that share an identification-type scheme. |
| `l10n_uk.country_group_ukxi` | `UKXI` | The United Kingdom together with the province that stays inside the European goods regime. |

### 3.5 Banks

The entity Bank (`res.bank`, table `res_bank`) is shipped with 115 declared records from three packages, and 3,539 further rows through the tabular data files of country packages, so that a bank account can be attached to a known institution by its bank identifier code.

## 4. Currencies

The entity Currency (`res.currency`, table `res_currency`) is declared 186 times by 7 packages and produces **173 distinct currencies**. Seventy further rows arrive through tabular data files of country packages, adding local codes to currencies that already exist.

| Rule | Statement |
|---|---|
| Identity | The external identifier is the foundation package, a dot and the three-letter alphabetic code, for example `base.EUR`, `base.USD`. |
| Code uniqueness | The constraint `_unique_name` over `name`, with the message "The currency code must be unique!" |
| Rounding factor | The field `rounding` holds the smallest amount the currency can express, default `0.01`. The constraint `_rounding_gt_zero` refuses a value that is not strictly positive, with the message "The rounding factor must be greater than 0!" |
| Decimal places | Derived from the rounding factor, not stored independently. The derivation and the rounding algorithm are in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), sections 5.3 and 5.4. |
| Symbol | Required. The field `position` decides whether it is rendered before or after the amount; the default is after. |
| Numeric code | The field `iso_numeric` holds the three-digit numeric code. |
| Unit and subunit names | The fields `currency_unit_label` and `currency_subunit_label` hold the words used when an amount is written out in letters, for example on a cheque. |
| Active by default | Every currency is shipped archived except the ones a company uses; the field `active` defaults to true on the record but the loading data marks all but a few as inactive, so that the currency list a user sees is short. |
| Ordering | Active first, then by name. |

No exchange rate is shipped. A fresh installation has no Currency Rate record at all, and every amount in a second currency therefore needs a rate to be entered or fetched before it can be converted. The rate model is specified in [`../domains/multi-currency/`](../domains/multi-currency/).

## 5. Languages

The entity Languages (`res.lang`, table `res_lang`) is populated from the foundation package's tabular data file, which carries **93 languages**, and six declaration records that correct the address code of three of them and attach a flag image to three others.

| Rule | Statement |
|---|---|
| Identity | The external identifier is the foundation package, a dot, the word `lang`, an underscore and the locale code, for example `base.lang_en`, `base.lang_fr`, `base.lang_sr@latin`. |
| Locale code | The field `code`, required, holds the language and territory, for example `en_US`, `fr_BE`, `pt_BR`. |
| Address code | The field `url_code`, required, holds the short form used in an address path, for example `en`, `fr`, `es`. The constraint `_url_code_uniq` refuses a duplicate with the message "The URL code of the language must be unique!" |
| Name uniqueness | The constraint `_name_uniq` over `name`, with the message "The name of the language must be unique!" |
| Code uniqueness | The constraint `_code_uniq` over `code`, with the message "The code of the language must be unique!" |
| Direction | The field `direction`, required, is left-to-right or right-to-left; the default is left-to-right. |
| Date and time format | The fields `date_format` and `time_format`, both required, hold the rendering patterns; the defaults are the month, day and four-digit year separated by slashes, and the twenty-four-hour clock with seconds. |
| First day of the week | The field `week_start`, required, default the seventh day, which is Sunday. This is a display setting only; every period computation anchors weeks on Monday, as stated in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), section 6.5. |
| Digit grouping | The field `grouping`, required, holds the grouping pattern, default three digits then no further grouping. |
| Separators | The fields `decimal_point`, required, and `thousands_sep` hold the decimal and grouping characters; the defaults are the full stop and the comma. |
| Ordering | Active first, then by name. |
| Active by default | Only the base language is active in a fresh installation. Activating another language triggers the loading of its translation catalogues. |

The base language, whose code is `en_US`, is the language every translatable value is stored under and the language every fallback ends at. It can never be removed.

## 6. Units of measure

The entity Product Unit of Measure (`uom.uom`, table `uom_uom`) is declared 121 times by 10 packages and produces **56 distinct units**. There is no separate unit category entity: a unit belongs to a family by pointing at a reference unit through `relative_uom_id` and stating how many of that reference unit it contains through `relative_factor`, and the family is the tree that this link forms. The tree is materialized in the `parent_path` column, as stated in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), section 14.

| Rule | Statement |
|---|---|
| Conversion | An amount is converted between two units of the same tree by multiplying by the factors up to the common root and dividing by the factors down to the target. |
| Factor | The field `relative_factor` is required and defaults to 1. The constraint `_factor_gt_zero` refuses zero with the message "The conversion ratio for a unit of measure cannot be 0!" |
| Root of a family | A unit whose `relative_uom_id` is empty is the root of its family and its factor is 1. |
| Ordering | By sequence, then by reference unit, then by identifier. |
| Rounding | The number of decimal places used for a quantity comes from the named decimal precision "Product Unit" of section 7, not from the unit. |

The thirty units the unit package ships:

| External identifier | Name | Contains | Of the reference unit |
|---|---|---|---|
| `uom.product_uom_unit` | Units | 1 | root of the countable family |
| `uom.product_uom_pack_6` | Pack of 6 | 6 | `uom.product_uom_unit` |
| `uom.product_uom_dozen` | Dozens | 12 | `uom.product_uom_unit` |
| `uom.product_uom_hour` | Hours | 1 | root of the duration family |
| `uom.product_uom_day` | Days | 8 | `uom.product_uom_hour` |
| `uom.product_uom_minute` | Minutes | 0.0166667 | `uom.product_uom_hour` |
| `uom.product_uom_millimeter` | Millimetre | 1 | root of the length family |
| `uom.product_uom_cm` | Centimetre | 10 | `uom.product_uom_millimeter` |
| `uom.product_uom_meter` | Metre | 100 | `uom.product_uom_cm` |
| `uom.product_uom_km` | Kilometre | 1000 | `uom.product_uom_meter` |
| `uom.product_uom_inch` | Inch | 2.54 | `uom.product_uom_cm` |
| `uom.product_uom_foot` | Foot | 12 | `uom.product_uom_inch` |
| `uom.product_uom_yard` | Yard | 3 | `uom.product_uom_foot` |
| `uom.product_uom_mile` | Mile | 1760 | `uom.product_uom_yard` |
| `uom.product_uom_square_meter` | Square metre | 1 | root of the area family |
| `uom.product_uom_square_foot` | Square foot | 0.092903 | `uom.product_uom_square_meter` |
| `uom.product_uom_milliliter` | Millilitre | 1 | root of the volume family |
| `uom.product_uom_litre` | Litre | 1000 | `uom.product_uom_milliliter` |
| `uom.product_uom_cubic_meter` | Cubic metre | 1000 | `uom.product_uom_litre` |
| `uom.product_uom_floz` | Fluid ounce, United States | 0.0295735 | `uom.product_uom_litre` |
| `uom.product_uom_qt` | Quart, United States | 32 | `uom.product_uom_floz` |
| `uom.product_uom_gal` | Gallon, United States | 4 | `uom.product_uom_qt` |
| `uom.product_uom_cubic_inch` | Cubic inch | 0.0163871 | `uom.product_uom_litre` |
| `uom.product_uom_cubic_foot` | Cubic foot | 1728 | `uom.product_uom_cubic_inch` |
| `uom.product_uom_gram` | Gram | 1 | root of the mass family |
| `uom.product_uom_kgm` | Kilogram | 1000 | `uom.product_uom_gram` |
| `uom.product_uom_ton` | Tonne | 1000 | `uom.product_uom_kgm` |
| `uom.product_uom_oz` | Ounce | 28.3495 | `uom.product_uom_gram` |
| `uom.product_uom_lb` | Pound | 16 | `uom.product_uom_oz` |
| `uom.product_uom_kwh` | Kilowatt hour | 1 | root of the energy family |

**A worked conversion.** Three dozens expressed in units: three multiplied by the factor twelve of the dozen gives thirty-six units. The same three dozens expressed in packs of six: thirty-six units divided by the factor six of the pack gives six packs. A quantity of 2.5 miles in millimetres: 2.5 × 1760 yards = 4 400 yards; × 3 = 13 200 feet; × 12 = 158 400 inches; × 2.54 = 402 336 centimetres; × 10 = 4 023 360 millimetres. Rounded to the two decimal places of the named precision "Product Unit", the result is 4 023 360.00 millimetres.

Twenty-six further units are added by other packages: a kilometre and hour reused as expense and timesheet units, three service units and a set of packaging and energy units used by country packages, each pointing at one of the roots above or at a unit of its own package.

## 7. Decimal precisions

The entity Decimal Precision (`decimal.precision`, table `decimal_precision`) holds a usage name and a number of decimal places. A decimal field may name a usage instead of declaring a fixed scale, and the scale is then read at write time. Resolving a usage that does not exist yields two decimal places. A usage name is unique; a duplicate is refused with "Only one value can be defined for each given usage!" Reducing a precision does not rewrite stored values, and the interface warns about that.

7 precisions are shipped.

| External identifier | Usage name | Package | Decimal places |
|---|---|---|---|
| `product.decimal_discount` | "Discount" | `product` | 2 |
| `account.decimal_payment` | "Payment Terms" | `account` | 6 |
| `analytic.decimal_percentage_analytic` | "Percentage Analytic" | `analytic` | 2 |
| `product.decimal_price` | "Product Price" | `product` | 2 |
| `uom.decimal_product_uom` | "Product Unit" | `uom` | 2 |
| `product.decimal_stock_weight` | "Stock Weight" | `product` | 2 |
| `product.decimal_volume` | "Volume" | `product` | 2 |

## 8. Contact classification

### 8.1 Industries

The entity Industry (`res.partner.industry`, table `res_partner_industry`) holds 23 records: the twenty-one sections of the standard statistical classification of economic activities, each with a short name and the full section text, plus two records added by one country package for electronic-commerce operators with a withholding obligation. A party names at most one industry.

| External identifier | Short name | Full name |
|---|---|---|
| `base.res_partner_industry_A` | Agriculture | A - AGRICULTURE, FORESTRY AND FISHING |
| `base.res_partner_industry_B` | Mining | B - MINING AND QUARRYING |
| `base.res_partner_industry_C` | Manufacturing | C - MANUFACTURING |
| `base.res_partner_industry_D` | Energy supply | D - ELECTRICITY, GAS, STEAM AND AIR CONDITIONING SUPPLY |
| `base.res_partner_industry_E` | Water supply | E - WATER SUPPLY; SEWERAGE, WASTE MANAGEMENT AND REMEDIATION ACTIVITIES |
| `base.res_partner_industry_F` | Construction | F - CONSTRUCTION |
| `base.res_partner_industry_G` | Wholesale/Retail | G - WHOLESALE AND RETAIL TRADE; REPAIR OF MOTOR VEHICLES AND MOTORCYCLES |
| `base.res_partner_industry_H` | Transportation/Logistics | H - TRANSPORTATION AND STORAGE |
| `base.res_partner_industry_I` | Food/Hospitality | I - ACCOMMODATION AND FOOD SERVICE ACTIVITIES |
| `base.res_partner_industry_J` | Information and communication | J - INFORMATION AND COMMUNICATION |
| `base.res_partner_industry_K` | Finance/Insurance | K - FINANCIAL AND INSURANCE ACTIVITIES |
| `base.res_partner_industry_L` | Real Estate | L - REAL ESTATE ACTIVITIES |
| `base.res_partner_industry_M` | Scientific | M - PROFESSIONAL, SCIENTIFIC AND TECHNICAL ACTIVITIES |
| `base.res_partner_industry_N` | Administrative/Utilities | N - ADMINISTRATIVE AND SUPPORT SERVICE ACTIVITIES |
| `base.res_partner_industry_O` | Public Administration | O - PUBLIC ADMINISTRATION AND DEFENCE; COMPULSORY SOCIAL SECURITY |
| `base.res_partner_industry_P` | Education | P - EDUCATION |
| `base.res_partner_industry_Q` | Health/Social | Q - HUMAN HEALTH AND SOCIAL WORK ACTIVITIES |
| `base.res_partner_industry_R` | Entertainment | R - ARTS, ENTERTAINMENT AND RECREATION |
| `base.res_partner_industry_S` | Other Services | S - OTHER SERVICE ACTIVITIES |
| `base.res_partner_industry_T` | Households | T - ACTIVITIES OF HOUSEHOLDS AS EMPLOYERS; UNDIFFERENTIATED GOODS- AND SERVICES-PRODUCING ACTIVITIES OF HOUSEHOLDS FOR OWN USE |
| `base.res_partner_industry_U` | Extraterritorial | U - ACTIVITIES OF EXTRATERRITORIAL ORGANISATIONS AND BODIES |
| `l10n_in.eco_under_section_52` | Electronic-commerce operator liable to collect tax at source | E-Commerce operator liable to deduct TCS under section 52 |
| `l10n_in.eco_under_section_9_5` | Electronic-commerce operator liable to pay tax | E-Commerce operator liable to pay tax under section 9(5) |

The short name is what a form shows; the full name is the classification text and is searchable, so that a user who types the classification wording finds the record.

### 8.2 Partner titles

There is no separate title entity in this system, and therefore no shipped set of titles. A courtesy title, where a document needs one, is part of the party's name or of the address rendering of its country. A rebuild must not create a title table expecting to find shipped records for it.

### 8.3 Partner tags

The entity Partner Tags (`res.partner.category`, table `res_partner_category`) ships 21 records, all from one country package, which uses them to mark parties by their electronic-invoicing profile. The foundation package ships none: the tag tree is the tenant's own.

## 9. Payment terms, payment methods and delivery terms

### 9.1 Payment terms

The entity Payment Terms (`account.payment.term`, table `account_payment_term`) ships 10 records, all from the accounting package. Each carries a name, a note rendered on the document, and one or more instalment lines.

| External identifier | Name | Instalments |
|---|---|---|
| `account.account_payment_term_immediate` | Immediate Payment | One line of 100 per cent, zero days. |
| `account.account_payment_term_15days` | 15 Days | One line of 100 per cent, fifteen days. |
| `account.account_payment_term_21days` | 21 Days | One line of 100 per cent, twenty-one days. |
| `account.account_payment_term_30days` | 30 Days | One line of 100 per cent, thirty days. |
| `account.account_payment_term_45days` | 45 Days | One line of 100 per cent, forty-five days. |
| `account.account_payment_term_end_following_month` | End of Following Month | One line of 100 per cent, due at the end of the month following the document date. |
| `account.account_payment_term_30_days_end_month_the_10` | 10 Days after End of Next Month | One line of 100 per cent, due ten days after the end of the following month. |
| `account.account_payment_term_advance_60days` | 30% Now, Balance 60 Days | Two lines: thirty per cent immediately and the balance at sixty days. |
| `account.account_payment_term_30days_early_discount` | 2/7 Net 30 | One line of 100 per cent at thirty days, with a two per cent discount if paid within seven days, shown on the invoice. |
| `account.account_payment_term_90days_on_the_10th` | 90 days, on the 10th | One line of 100 per cent, due ninety days later on the tenth of the month. |

### 9.2 Payment methods of the ledger

The entity Payment Methods (`account.payment.method`, table `account_payment_method`) ships 8 records. The constraint `_name_code_unique` makes the pair (`code`, `payment_type`) unique, with the message "The combination code/payment type already exists!"

| External identifier | Name | Code | Direction | Package |
|---|---|---|---|---|
| `account.account_payment_method_manual_in` | Manual Payment | `manual` | inbound | Accounting |
| `account.account_payment_method_manual_out` | Manual Payment | `manual` | outbound | Accounting |
| `account_check_printing.account_payment_method_check` | Checks | `check_printing` | outbound | Cheque Printing |
| `l10n_latam_check.account_payment_method_own_checks` | Own Checks | `own_checks` | outbound | Latin American cheques |
| `l10n_latam_check.account_payment_method_new_third_party_checks` | New Third Party Checks | `new_third_party_checks` | inbound | Latin American cheques |
| `l10n_latam_check.account_payment_method_in_third_party_checks` | Existing Third Party Checks | `in_third_party_checks` | inbound | Latin American cheques |
| `l10n_latam_check.account_payment_method_out_third_party_checks` | Existing Third Party Checks | `out_third_party_checks` | outbound | Latin American cheques |
| `l10n_latam_check.account_payment_method_return_third_party_checks` | Return Third Party Checks | `return_third_party_checks` | outbound | Latin American cheques |

### 9.3 Payment methods and providers of the online checkout

Two further sets serve online payment: the entity Payment Method (`payment.method`, table `payment_method`) with **237 records** from six packages, one per card scheme, wallet or local scheme; and the entity Payment Provider (`payment.provider`, table `payment_provider`) with **49 records** from forty-nine packages, one per acquirer, each shipped disabled and carrying the methods it supports. The state machine of a transaction and the enabling of a provider are specified in the payment providers folder `../domains/payment-providers/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

### 9.4 Delivery terms

The entity Incoterms (`account.incoterms`, table `account_incoterms`) ships 15 records: the eleven current international commercial terms and four superseded ones that one country package still requires for electronic invoicing.

| External identifier | Code | Name |
|---|---|---|
| `account.incoterm_EXW` | `EXW` | EX WORKS |
| `account.incoterm_FCA` | `FCA` | FREE CARRIER |
| `account.incoterm_FAS` | `FAS` | FREE ALONGSIDE SHIP |
| `account.incoterm_FOB` | `FOB` | FREE ON BOARD |
| `account.incoterm_CFR` | `CFR` | COST AND FREIGHT |
| `account.incoterm_CIF` | `CIF` | COST, INSURANCE AND FREIGHT |
| `account.incoterm_CPT` | `CPT` | CARRIAGE PAID TO |
| `account.incoterm_CIP` | `CIP` | CARRIAGE AND INSURANCE PAID TO |
| `account.incoterm_DPU` | `DPU` | DELIVERED AT PLACE UNLOADED |
| `account.incoterm_DAP` | `DAP` | DELIVERED AT PLACE |
| `account.incoterm_DDP` | `DDP` | DELIVERED DUTY PAID |
| `l10n_tr_nilvera_einvoice_extended.incoterm_DAF` | `DAF` | DELIVERED AT FRONTIER |
| `l10n_tr_nilvera_einvoice_extended.incoterm_DDU` | `DDU` | DELIVERED DUTY UNPAID |
| `l10n_tr_nilvera_einvoice_extended.incoterm_DEQ` | `DEQ` | DELIVERED EX QUAY (DUTY PAID) |
| `l10n_tr_nilvera_einvoice_extended.incoterm_DES` | `DES` | DELIVERED EX SHIP |

The name is reproduced in the capital letters the standard uses, because it is printed on shipping documents.

## 10. Tax tags and financial report structures

### 10.1 Account and tax tags

The entity Account Tag (`account.account.tag`, table `account_account_tag`) is declared 519 times by 15 packages and produces **517 distinct tags**; 1,838 further rows arrive through the tabular data files of country packages, which is where the bulk of the national tax grids live.

| Rule | Statement |
|---|---|
| Applicability | The field `applicability`, required, default `accounts`, says whether the tag classifies accounts or tax distribution lines. A tag with the applicability of taxes is a **tax grid**: it names a box of a tax return, and every tax distribution line that carries it feeds that box. |
| Country | The field `country_id` scopes a tag to a country, because two countries may use the same box name for different things. |
| Uniqueness | The constraint `_name_uniq` over the triple (`name`, `applicability`, `country_id`), with the message "A tag with the same name and applicability already exists in this country." |
| Sign | A tax grid is referenced twice by a distribution line, once with a positive and once with a negative sign, so that a refund reverses the box it fed. |

### 10.2 Financial report structures

Four sets together define every shipped statement — balance sheet, profit and loss, tax return, ageing, cash flow and the national statements of each country package.

| Set | Transport name | Records | What one record is |
|---|---|---|---|
| Accounting Report | `account.report` | 165 | One statement: its name, its country, its root report, its columns and its lines. |
| Report column | `account.report.column` | 275 | One column of a statement, with its figure type and its expression label. |
| Report line | `account.report.line` | 6,076 | One line of a statement, with its code, its level in the hierarchy, its parent line and its ordering. |
| Report expression | `account.report.expression` | 6,555 | One expression attached to a line and a column: how the figure is computed — from a tax grid, from an account code range, from a filter, from another expression or from an externally supplied value. |

The evaluation of these structures is specified in [`../domains/financial-reporting/`](../domains/financial-reporting/); what matters here is that they are data, not behaviour, and a rebuild that ships them differently produces different statements from the same ledger.

A fifth set, Report external value (`account.report.external.value`), ships no records: it holds figures a user types into a statement.

## 11. Country accounting templates

A country package ships its chart of accounts, its taxes and its fiscal positions as **template** rows rather than as records. The rows are not loaded into the working tables at installation of the package; they are loaded into a company's own chart when a user selects that chart for the company. This is why a tenant with one country package still gets one chart, not one hundred and twenty-seven.

127 packages ship templates covering 151 distinct charts, 66,368 rows in all. A chart code is what a company selects; several charts may live in one package, because one country may offer a company chart, an association chart and a sole-trader chart.

Rows by kind:

| Record set | Transport name | Rows |
|---|---|---|
| Account | `account.account` | 34,196 |
| Tax | `account.tax` | 21,667 |
| Account Group | `account.group` | 8,581 |
| Tax Group | `account.tax.group` | 1,090 |
| Fiscal Position | `account.fiscal.position` | 572 |
| account.asset | `account.asset` | 133 |
| account.account.group | `account.account.group` | 91 |
| Turkish Tax Codes (GIB Codes) | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | 38 |

Per package:

| Package | Chart codes | Accounts | Account groups | Taxes | Tax groups | Fiscal positions | Rows |
|---|---|---|---|---|---|---|---|
| `account` | `generic_coa` | 46 |  | 16 | 2 | 2 | 66 |
| `l10n_ae` | `ae` | 170 |  | 74 | 5 | 9 | 258 |
| `l10n_ar` | `ar_base`, `ar_ex`, `ar_ri` | 299 | 58 | 292 | 120 | 4 | 773 |
| `l10n_ar_withholding` | `ar_base`, `ar_ex`, `ar_ri` | 1 |  | 407 | 27 |  | 435 |
| `l10n_at` | `at` | 237 |  | 260 | 7 | 13 | 517 |
| `l10n_au` | `au` | 114 |  | 130 | 4 | 4 | 252 |
| `l10n_bd` | `bd` | 135 | 31 | 64 | 7 | 2 | 239 |
| `l10n_be` | `be`, `be_asso`, `be_comp` | 537 | 340 | 432 | 4 | 12 | 1,346 |
| `l10n_be_pos_restaurant` | `be` | 0 |  | 4 |  |  | 4 |
| `l10n_bf` | `bf`, `bf_syscebnl` | 0 |  | 76 | 6 | 4 | 86 |
| `l10n_bg` | `bg` | 332 | 67 | 112 | 3 | 4 | 518 |
| `l10n_bh` | `bh` | 137 | 31 | 56 | 4 | 3 | 231 |
| `l10n_bj` | `bj`, `bj_syscebnl` | 0 |  | 84 | 4 | 4 | 92 |
| `l10n_bo` | `bo` | 127 | 43 | 99 | 9 | 3 | 281 |
| `l10n_br` | `br` | 1,085 |  | 1,292 | 160 | 6 | 2,543 |
| `l10n_ca` | `ca_2023` | 341 | 168 | 101 | 8 | 14 | 632 |
| `l10n_cd` | `cd`, `cd_syscebnl` | 0 |  | 112 | 4 | 4 | 120 |
| `l10n_cf` | `cf`, `cf_syscebnl` | 0 |  | 64 | 6 | 4 | 74 |
| `l10n_cg` | `cg`, `cg_syscebnl` | 0 |  | 68 | 8 | 4 | 80 |
| `l10n_ch` | `ch` | 209 |  | 110 | 8 | 2 | 329 |
| `l10n_ci` | `ci`, `ci_syscebnl` | 0 |  | 120 | 6 | 4 | 130 |
| `l10n_cl` | `cl` | 195 |  | 108 | 5 | 25 | 333 |
| `l10n_cm` | `cm`, `cm_syscebnl` | 0 |  | 56 | 4 | 4 | 64 |
| `l10n_cn` | `cn`, `cn_common`, `cn_large_bis` | 361 |  | 48 | 3 |  | 412 |
| `l10n_co` | `co` | 378 | 401 | 248 | 43 |  | 1,070 |
| `l10n_cr` | `cr` | 98 |  | 8 | 1 |  | 107 |
| `l10n_cy` | `cy` | 169 |  | 124 | 5 | 5 | 303 |
| `l10n_cz` | `cz` | 277 | 313 | 148 | 4 | 8 | 750 |
| `l10n_de` | `de_skr03`, `de_skr04` | 2,466 |  | 484 | 12 | 60 | 3,022 |
| `l10n_dk` | `dk` | 488 | 41 | 351 | 1 | 8 | 889 |
| `l10n_do` | `do` | 284 | 95 | 139 | 14 | 15 | 547 |
| `l10n_dz` | `dz` | 294 | 68 | 168 | 3 | 4 | 537 |
| `l10n_ec` | `ec` | 501 | 112 | 728 | 17 | 2 | 1,360 |
| `l10n_ee` | `ee` | 211 | 103 | 290 | 7 | 10 | 621 |
| `l10n_eg` | `eg` | 205 |  | 120 | 14 | 2 | 349 |
| `l10n_es` | `es_assec`, `es_canary_common`, `es_common`, `es_common_mainland`, `es_coop_full`, `es_coop_pymes`, `es_full`, `es_pymes` | 1,048 | 946 | 1,246 | 49 | 39 | 3,346 |
| `l10n_es_edi_facturae` | `es_canary_common`, `es_common_mainland` | 0 |  | 300 |  |  | 300 |
| `l10n_es_edi_verifactu` | `es_canary_common`, `es_common_mainland` | 0 |  | 273 |  |  | 273 |
| `l10n_et` | `et` | 100 |  | 52 | 5 |  | 157 |
| `l10n_fi` | `fi` | 971 |  | 239 | 6 | 6 | 1,222 |
| `l10n_fr_account` | `fr` | 656 | 182 | 304 | 6 | 10 | 1,158 |
| `l10n_ga` | `ga`, `ga_syscebnl` | 0 |  | 154 | 16 | 4 | 174 |
| `l10n_ge` | `ge` | 256 | 50 | 150 | 5 | 4 | 465 |
| `l10n_gn` | `gn`, `gn_syscebnl` | 0 |  | 56 | 4 | 4 | 64 |
| `l10n_gq` | `gq`, `gq_syscebnl` | 0 |  | 80 | 8 | 4 | 92 |
| `l10n_gr` | `gr` | 454 | 187 | 300 | 7 | 5 | 953 |
| `l10n_gt` | `gt` | 86 |  | 16 | 3 |  | 113 |
| `l10n_gw` | `gw`, `gw_syscebnl` | 0 |  | 96 | 8 | 4 | 108 |
| `l10n_hk` | `hk` | 75 |  |  |  |  | 75 |
| `l10n_hn` | `hn` | 34 |  | 8 | 1 |  | 43 |
| `l10n_hr` | `hr` | 554 | 89 | 142 | 4 | 5 | 794 |
| `l10n_hr_edi` | `hr` | 0 |  | 15 |  |  | 15 |
| `l10n_hr_kuna` | `hr_kuna` | 1,674 |  | 140 | 5 | 7 | 1,826 |
| `l10n_hu` | `hu` | 389 | 74 | 174 | 5 | 5 | 647 |
| `l10n_hu_edi` | `hu` | 0 |  | 39 |  |  | 39 |
| `l10n_id` | `id` | 114 |  | 78 | 6 |  | 198 |
| `l10n_ie` | `ie` | 145 |  | 186 | 5 | 4 | 340 |
| `l10n_il` | `il` | 86 | 10 | 58 | 4 | 6 | 164 |
| `l10n_in` | `in` | 102 |  | 1,741 | 12 | 2 | 1,857 |
| `l10n_iq` | `iq` | 137 | 32 | 20 | 6 |  | 195 |
| `l10n_it` | `it` | 184 |  | 522 | 14 | 6 | 726 |
| `l10n_it_edi` | `it` | 0 |  | 14 |  |  | 14 |
| `l10n_it_edi_doi` | `it` | 0 |  | 4 |  | 1 | 5 |
| `l10n_jo` | `jo_standard` | 140 | 31 | 116 | 5 | 2 | 294 |
| `l10n_jp` | `jp` | 128 |  | 32 | 3 | 3 | 166 |
| `l10n_ke` | `ke` | 127 |  | 116 | 5 | 2 | 250 |
| `l10n_kh` | `kh` | 106 | 85 | 197 | 13 | 3 | 404 |
| `l10n_km` | `km`, `km_syscebnl` | 0 |  | 64 | 12 | 4 | 80 |
| `l10n_kr` | `kr` | 181 | 69 | 256 | 1 |  | 507 |
| `l10n_kw` | `kw` | 136 | 31 |  |  |  | 167 |
| `l10n_kz` | `kz` | 280 |  | 96 | 4 | 3 | 383 |
| `l10n_lb_account` | `lb` | 373 | 234 | 32 | 3 | 2 | 644 |
| `l10n_lk` | `lk` | 100 |  | 236 | 5 | 2 | 343 |
| `l10n_lt` | `lt` | 189 |  | 260 | 5 | 4 | 458 |
| `l10n_lu` | `lu` | 746 | 973 | 1,006 | 12 | 5 | 2,742 |
| `l10n_lv` | `lv` | 242 | 55 | 142 | 5 | 4 | 448 |
| `l10n_ma` | `ma` | 634 | 147 | 318 | 5 | 8 | 1,112 |
| `l10n_ml` | `ml`, `ml_syscebnl` | 0 |  | 104 | 6 | 4 | 114 |
| `l10n_mn` | `mn` | 304 |  | 164 | 6 | 4 | 478 |
| `l10n_mr` | `mr` | 611 | 129 | 60 | 4 | 2 | 821 |
| `l10n_mt` | `mt` | 481 |  | 88 | 4 | 5 | 578 |
| `l10n_mu_account` | `mu` | 42 |  | 48 | 2 | 2 | 94 |
| `l10n_mx` | `mx` | 140 | 1,079 | 138 | 14 | 5 | 1,385 |
| `l10n_my` | `my` | 77 |  | 140 | 10 | 5 | 232 |
| `l10n_mz` | `mz` | 311 | 101 | 44 | 3 | 2 | 461 |
| `l10n_ne` | `ne`, `ne_syscebnl` | 0 |  | 92 | 4 | 4 | 100 |
| `l10n_ng` | `ng` | 0 |  | 48 | 4 | 2 | 54 |
| `l10n_nl` | `nl` | 349 |  | 144 | 3 | 21 | 517 |
| `l10n_no` | `no` | 745 |  | 148 | 5 |  | 898 |
| `l10n_nz` | `nz` | 100 |  | 24 | 3 | 2 | 129 |
| `l10n_om` | `om` | 139 | 31 | 56 | 4 | 2 | 232 |
| `l10n_pa` | `pa` | 105 |  | 8 | 1 |  | 114 |
| `l10n_pe` | `pe` | 1,228 | 83 | 69 | 15 | 2 | 1,397 |
| `l10n_ph` | `ph` | 104 |  | 254 | 4 | 2 | 364 |
| `l10n_pk` | `pk` | 123 | 44 | 244 | 13 |  | 432 |
| `l10n_pl` | `pl` | 238 | 136 | 128 | 4 | 4 | 510 |
| `l10n_pt` | `pt` | 642 | 211 | 204 | 10 | 4 | 1,071 |
| `l10n_qa` | `qa` | 136 | 31 |  |  |  | 167 |
| `l10n_ro` | `ro` | 579 | 139 | 340 | 18 | 9 | 1,085 |
| `l10n_rs` | `rs` | 420 | 86 | 52 | 4 | 3 | 565 |
| `l10n_rw` | `rw` | 164 |  | 36 | 2 | 2 | 204 |
| `l10n_sa` | `sa` | 169 |  | 174 | 5 | 2 | 350 |
| `l10n_sa_edi` | `sa` | 0 |  | 15 |  |  | 15 |
| `l10n_se` | `se`, `se_K2`, `se_K3` | 1,214 | 537 | 176 | 12 | 20 | 1,959 |
| `l10n_sg` | `sg` | 135 |  | 149 | 5 | 2 | 291 |
| `l10n_si` | `si` | 607 | 78 | 256 | 8 | 17 | 966 |
| `l10n_sk` | `sk` | 345 | 296 | 178 | 4 | 4 | 837 |
| `l10n_sn` | `sn`, `sn_syscebnl` | 0 |  | 80 | 6 | 4 | 90 |
| `l10n_syscohada` | `syscebnl`, `syscohada` | 1,586 |  |  |  |  | 1,677 |
| `l10n_td` | `td`, `td_syscebnl` | 0 |  | 64 | 6 | 4 | 74 |
| `l10n_tg` | `tg`, `tg_syscebnl` | 0 |  | 88 | 4 | 4 | 96 |
| `l10n_th` | `th` | 144 |  | 72 | 5 |  | 233 |
| `l10n_tn` | `tn` | 450 | 162 | 116 | 9 | 3 | 740 |
| `l10n_tr` | `tr` | 267 | 64 | 110 | 13 |  | 454 |
| `l10n_tr_nilvera_einvoice_extended` | `tr` | 0 |  | 270 |  |  | 308 |
| `l10n_tw` | `tw` | 135 |  | 290 | 19 | 8 | 452 |
| `l10n_tz_account` | `tz` | 164 |  | 40 | 2 | 4 | 210 |
| `l10n_ua` | `ua_psbo` | 332 | 51 | 49 | 7 |  | 439 |
| `l10n_ug` | `ug` | 124 |  | 80 | 1 | 2 | 207 |
| `l10n_uk` | `uk` | 118 |  | 76 | 4 | 3 | 210 |
| `l10n_us_account` | `us` | 101 | 14 | 184 | 23 | 2 | 331 |
| `l10n_uy` | `uy` | 166 |  | 48 | 4 | 2 | 228 |
| `l10n_uz` | `uz` | 269 | 90 | 16 | 3 |  | 378 |
| `l10n_ve` | `ve` | 266 |  | 32 | 4 |  | 302 |
| `l10n_vn` | `vn` | 216 | 106 | 60 | 6 |  | 388 |
| `l10n_za` | `za` | 117 |  | 64 | 2 |  | 183 |
| `l10n_zm_account` | `zm` | 89 | 47 | 76 | 3 | 2 | 217 |

Each template row carries an identifier that is unique inside its chart, not inside the installation, because the same chart may be instantiated for several companies; the loading of a chart into a company prefixes every identifier with the company. The per-company instantiation is specified in the fiscal localizations folder `../domains/fiscal-localizations/`, listed in the domain index [`../domains/README.md`](../domains/README.md), and the accounts, taxes and fiscal positions themselves in [`../domains/general-ledger/`](../domains/general-ledger/) and [`../domains/taxes/`](../domains/taxes/).

## 12. Activity types

The entity Activity Type (`mail.activity.type`, table `mail_activity_type`) defines the kinds of thing a user can be asked to do on a record. 16 are shipped, one of which is a second declaration that attaches the meeting behaviour to the meeting type once the calendar package is installed.

| Field | Rule |
|---|---|
| `res_model` | When empty the type is offered on every entity; when set it is offered only on that entity. |
| `category` | The behaviour the type triggers: the default is a plain reminder; `meeting` opens the calendar; `phonecall` opens the telephone panel; `upload_file` asks for a document and closes itself when one is attached. |
| `delay_count`, `delay_unit`, `delay_from` | The default due date: a count of days, weeks or months, measured from the previous activity or from the moment the activity is created. |
| `chaining_type` | Whether completing the activity suggests or forces the next one; the default is to suggest. |
| `sequence` | The order in the menu; the default is ten. |
| `decoration_type` | The colour a client uses for the deadline. |
| `icon` | The pictogram a client shows. |

| External identifier | Name | Package | Entity | Action kind | Delay in days | Sequence | Default summary |
|---|---|---|---|---|---|---|---|
| `mail.mail_activity_data_meeting` | "extended by this package" | `calendar` | every entity | meeting | 0 | 10 |  |
| `fleet.mail_act_fleet_contract_to_renew` | "Contract to Renew" | `fleet` | `fleet.vehicle.log.contract` | default | 0 | 10 | Contract to Renew |
| `hr_expense.mail_act_expense_approval` | "Expense Approval" | `hr_expense` | `hr.expense` | default | 0 | 10 | Expense Approval |
| `hr_holidays.mail_act_leave_allocation_approval` | "Allocation Approval" | `hr_holidays` | `hr.leave.allocation` | default | 0 | 10 | Allocation Approval |
| `hr_holidays.mail_act_leave_allocation_second_approval` | "Allocation Second Approval" | `hr_holidays` | `hr.leave.allocation` | default | 0 | 10 | Allocation Second Approval |
| `hr_holidays.mail_act_leave_approval` | "Time Off Approval" | `hr_holidays` | `hr.leave` | default | 15 | 10 | Time Off Approval |
| `hr_holidays.mail_act_leave_second_approval` | "Time Off Second Approve" | `hr_holidays` | `hr.leave` | default | 0 | 10 | Time Off Second Approve |
| `hr_skills.mail_activity_data_upload_certification` | "Certifications" | `hr_skills` | `hr.employee` | upload_file | 5 | 25 | Upload a certification |
| `mail.mail_activity_data_call` | "Call" | `mail` | every entity | phonecall | 2 | 6 | Call |
| `mail.mail_activity_data_email` | "Email" | `mail` | every entity | default | 0 | 3 | Email |
| `mail.mail_activity_data_meeting` | "Meeting" | `mail` | every entity | default | 0 | 9 | Meeting |
| `mail.mail_activity_data_todo` | "To-Do" | `mail` | every entity | default | 5 | 2 | To-Do |
| `mail.mail_activity_data_upload_document` | "Document" | `mail` | every entity | upload_file | 5 | 25 | Document |
| `mail.mail_activity_data_warning` | "Exception" | `mail` | every entity | default | 0 | 99 |  |
| `maintenance.mail_act_maintenance_request` | "Maintenance Request" | `maintenance` | `maintenance.request` | default | 0 | 10 | Maintenance Request |
| `point_of_sale.mail_activity_old_session` | "Session open over 7 days" | `point_of_sale` | `pos.session` | default | 0 | 10 | note |

## 13. Message subtypes

The entity Message subtypes (`mail.message.subtype`, table `mail_message_subtype`) classifies the messages posted on a record and decides who is notified. 103 are shipped.

| Field | Rule |
|---|---|
| `res_model` | When empty the subtype applies to every entity; when set it applies to that entity only. |
| `default` | Whether a new follower is subscribed to the subtype automatically; the default is true. |
| `internal` | When true, only internal users see messages of this subtype; external and portal followers do not. |
| `hidden` | When true, the subtype is not offered in the subscription dialogue. |
| `parent_id` and `relation_field` | Together they propagate a subscription from a parent record to a child: a follower of the parent, subscribed to the parent subtype, is notified of the child subtype through the named link field. |
| `sequence` | The order in the subscription dialogue; the default is one. |
| `track_recipients` | Whether the recipients of a tracked change are recorded on the message. |
| `description` | The sentence shown under the subtype in the subscription dialogue. It may be left empty, and 41 of the 103 shipped subtypes leave it empty; an empty cell in the table below means the subtype ships without a description. |

Three subtypes of the messaging package are the ones every entity with a discussion thread relies on: the discussion subtype, under which a user's own message is posted; the note subtype, which is internal only; and the activity subtype, under which the completion of an activity is logged.

| External identifier | Name | Package | Entity | Default | Internal only | Hidden | Description |
|---|---|---|---|---|---|---|---|
| `account.mt_invoice_created` | "Invoice Created" | `account` | `account.move` | no |  | yes | Invoice Created |
| `account.mt_invoice_paid` | "Paid" | `account` | `account.move` | no |  |  | Invoice paid |
| `account.mt_invoice_validated` | "Validated" | `account` | `account.move` | no |  |  | Invoice validated |
| `calendar.subtype_invitation` | "Invitation" | `calendar` | `calendar.event` | no |  |  |  |
| `crm.mt_lead_create` | "Opportunity Created" | `crm` | `crm.lead` | no |  | yes | Lead/Opportunity created |
| `crm.mt_lead_lost` | "Opportunity Lost" | `crm` | `crm.lead` | no |  |  | Opportunity lost |
| `crm.mt_lead_restored` | "Opportunity Restored" | `crm` | `crm.lead` | no |  |  | Opportunity restored |
| `crm.mt_lead_stage` | "Stage Changed" | `crm` | `crm.lead` | no |  |  | Stage changed |
| `crm.mt_lead_won` | "Opportunity Won" | `crm` | `crm.lead` | no |  |  | Opportunity won |
| `crm.mt_salesteam_lead` | "Opportunity Created" | `crm` | `crm.team` | yes |  |  |  |
| `crm.mt_salesteam_lead_lost` | "Opportunity Lost" | `crm` | `crm.team` | no |  |  |  |
| `crm.mt_salesteam_lead_restored` | "Opportunity Restored" | `crm` | `crm.team` | no |  |  |  |
| `crm.mt_salesteam_lead_stage` | "Opportunity Stage Changed" | `crm` | `crm.team` | yes |  |  |  |
| `crm.mt_salesteam_lead_won` | "Opportunity Won" | `crm` | `crm.team` | yes |  |  |  |
| `event_booth.mt_event_booth_booked` | "Booth Booked" | `event_booth` | `event.event` | no |  |  |  |
| `fleet.mt_fleet_driver_updated` | "Changed Driver" | `fleet` | `fleet.vehicle` | yes |  |  | Changed Driver |
| `hr.mt_contract_close` | "Expired" | `hr` | `hr.version` | no |  |  | Contract expired |
| `hr.mt_contract_pending` | "To Renew" | `hr` | `hr.version` | yes |  |  | Contract about to expire |
| `hr.mt_department_contract_pending` | "Contract to Renew" | `hr` | `hr.department` | no |  |  | Contract about to expire |
| `hr_expense.mt_expense_approved` | "Approved" | `hr_expense` | `hr.expense` | no |  |  | Expense approved |
| `hr_expense.mt_expense_entry_delete` | "Journal Entry Deleted" | `hr_expense` | `hr.expense` | no |  |  | Journal entry deleted |
| `hr_expense.mt_expense_entry_draft` | "Journal Entry Reset to Draft" | `hr_expense` | `hr.expense` | no |  |  | Journal entry reset to draft |
| `hr_expense.mt_expense_paid` | "Paid" | `hr_expense` | `hr.expense` | no |  |  | Expense paid |
| `hr_expense.mt_expense_refused` | "Refused" | `hr_expense` | `hr.expense` | no |  |  | Expense refused |
| `hr_expense.mt_expense_reset` | "Draft" | `hr_expense` | `hr.expense` | no |  |  | Expense reset to Draft |
| `hr_holidays.mt_leave` | "Time Off" | `hr_holidays` | `hr.leave` | yes |  |  | Time Off Request |
| `hr_holidays.mt_leave_allocation` | "Allocation Request" | `hr_holidays` | `hr.leave.allocation` | yes |  |  | Allocation Request |
| `hr_holidays.mt_leave_sick` | "Sick Time Off" | `hr_holidays` | `hr.leave` | yes |  |  | Sick Time Off |
| `hr_holidays.mt_leave_unpaid` | "Unpaid Time Off" | `hr_holidays` | `hr.leave` | yes |  |  | Unpaid Time Off |
| `hr_recruitment.mt_applicant_hired` | "Applicant Hired" | `hr_recruitment` | `hr.applicant` | yes |  |  |  |
| `hr_recruitment.mt_applicant_new` | "New Applicant" | `hr_recruitment` | `hr.applicant` | no |  | yes | Applicant created |
| `hr_recruitment.mt_applicant_stage_changed` | "Stage Changed" | `hr_recruitment` | `hr.applicant` | no |  |  | Stage changed |
| `hr_recruitment.mt_department_new` | "Job Position Created" | `hr_recruitment` | `hr.department` | no |  |  |  |
| `hr_recruitment.mt_job_applicant_hired` | "Applicant Hired" | `hr_recruitment` | `hr.job` | yes |  |  |  |
| `hr_recruitment.mt_job_applicant_new` | "New Applicant" | `hr_recruitment` | `hr.job` | no |  |  |  |
| `hr_recruitment.mt_job_applicant_stage_changed` | "Applicant Stage Changed" | `hr_recruitment` | `hr.job` | no |  |  |  |
| `hr_recruitment.mt_job_new` | "Job Position created" | `hr_recruitment` | `hr.job` | no |  | yes | Job Position created |
| `hr_recruitment.mt_talent_new` | "New Talent" | `hr_recruitment` | `hr.applicant` | no |  |  | Talent created |
| `mail.mt_activities` | "Activities" | `mail` | every entity | no | yes |  |  |
| `mail.mt_comment` | "Discussions" | `mail` | every entity | yes |  |  |  |
| `mail.mt_note` | "Note" | `mail` | every entity | no | yes |  |  |
| `maintenance.mt_cat_mat_assign` | "Equipment Assigned" | `maintenance` | `maintenance.equipment.category` | yes |  |  |  |
| `maintenance.mt_cat_req_created` | "Maintenance Request Created" | `maintenance` | `maintenance.equipment.category` | yes |  |  |  |
| `maintenance.mt_mat_assign` | "Equipment Assigned" | `maintenance` | `maintenance.equipment` | yes |  |  | Equipment Assigned |
| `maintenance.mt_req_created` | "Request Created" | `maintenance` | `maintenance.request` | no |  | yes | Maintenance Request created |
| `maintenance.mt_req_status` | "Status Changed" | `maintenance` | `maintenance.request` | yes |  |  | Status changed |
| `mrp.mrp_mo_in_cancelled` | "Manufacturing Order Cancelled" | `mrp` | `mrp.production` | no |  |  | Manufacturing Order Cancelled |
| `mrp.mrp_mo_in_confirmed` | "Manufacturing Order Confirmed" | `mrp` | `mrp.production` | no |  |  | Manufacturing Order Confirmed |
| `mrp.mrp_mo_in_done` | "Manufacturing Order Done" | `mrp` | `mrp.production` | no |  |  | Manufacturing Order Done |
| `mrp.mrp_mo_in_progress` | "Manufacturing Order Progress" | `mrp` | `mrp.production` | no |  |  | Manufacturing Order Progress |
| `mrp.mrp_mo_in_to_close` | "Manufacturing Order To Close" | `mrp` | `mrp.production` | no |  |  | Manufacturing Order To Close |
| `project.mt_project_stage_change` | "Project Stage Changed" | `project` | `project.project` | no |  | yes |  |
| `project.mt_project_task_approved` | "Task Approved" | `project` | `project.project` | no |  |  |  |
| `project.mt_project_task_canceled` | "Task Canceled" | `project` | `project.project` | no |  |  |  |
| `project.mt_project_task_changes_requested` | "Changes Requested" | `project` | `project.project` | no |  |  |  |
| `project.mt_project_task_done` | "Task Done" | `project` | `project.project` | no |  |  |  |
| `project.mt_project_task_in_progress` | "Task In Progress" | `project` | `project.project` | no |  |  |  |
| `project.mt_project_task_new` | "Task Created" | `project` | `project.project` | no |  |  |  |
| `project.mt_project_task_rating` | "Task Rating" | `project` | `project.project` | no |  | yes |  |
| `project.mt_project_task_stage` | "Task Stage Changed" | `project` | `project.project` | no |  |  |  |
| `project.mt_project_task_waiting` | "Task Waiting" | `project` | `project.project` | no |  | yes |  |
| `project.mt_project_update_create` | "Update Created" | `project` | `project.project` | no |  | yes |  |
| `project.mt_task_approved` | "Task Approved" | `project` | `project.task` | no |  |  | Task approved |
| `project.mt_task_canceled` | "Task Cancelled" | `project` | `project.task` | no |  |  | Task cancelled |
| `project.mt_task_changes_requested` | "Changes Requested" | `project` | `project.task` | no |  |  | Changes Requested |
| `project.mt_task_done` | "Task Done" | `project` | `project.task` | no |  |  | Task done |
| `project.mt_task_in_progress` | "Task In Progress" | `project` | `project.task` | no |  |  | Task In Progress |
| `project.mt_task_new` | "Task Created" | `project` | `project.task` | no |  | yes | Task Created |
| `project.mt_task_rating` | "Task Rating" | `project` | `project.task` | no |  | yes |  |
| `project.mt_task_stage` | "Stage Changed" | `project` | `project.task` | no |  |  | Stage changed |
| `project.mt_task_waiting` | "Task Waiting" | `project` | `project.task` | no |  | yes | Task Waiting |
| `project.mt_update_create` | "Update Created" | `project` | `project.update` | no |  | yes | Update Created |
| `purchase.mt_rfq_approved` | "Request for Quotation Approved" | `purchase` | `purchase.order` | no |  |  |  |
| `purchase.mt_rfq_confirmed` | "Request for Quotation Confirmed" | `purchase` | `purchase.order` | no |  |  |  |
| `purchase.mt_rfq_sent` | "Request for Quotation Sent" | `purchase` | `purchase.order` | no |  |  |  |
| `sale.mt_order_confirmed` | "Sales Order Confirmed" | `sale` | `sale.order` | no |  |  | Quotation confirmed |
| `sale.mt_order_sent` | "Quotation sent" | `sale` | `sale.order` | no |  |  | Quotation sent |
| `sale.mt_order_viewed` | "Quotation Viewed" | `sale` | `sale.order` | yes | yes |  |  |
| `sale.mt_salesteam_invoice_paid` | "Invoice Paid" | `sale` | `crm.team` | yes |  |  |  |
| `sale.mt_salesteam_invoice_posted` | "Invoice Posted" | `sale` | `crm.team` | yes |  |  |  |
| `sale.mt_salesteam_order_confirmed` | "Sales Order Confirmed" | `sale` | `crm.team` | yes |  |  |  |
| `sale.mt_salesteam_order_sent` | "Quotation sent" | `sale` | `crm.team` | yes |  |  |  |
| `sale.mt_salesteam_order_viewed` | "Quotation Viewed" | `sale` | `crm.team` | yes |  |  |  |
| `stock_landed_costs.mt_stock_landed_cost_open` | "Done" | `stock_landed_costs` | `stock.landed.cost` | yes |  |  | Landed cost validated |
| `stock_picking_batch.mt_batch_state` | "Stage Changed" | `stock_picking_batch` | `stock.picking.batch` | no |  |  | Stage Changed |
| `survey.mt_survey_survey_user_input_completed` | "Participation completed" | `survey` | `survey.survey` | no |  |  | New participation completed. |
| `survey.mt_survey_user_input_completed` | "Participation completed" | `survey` | `survey.user_input` | no |  | yes | Participation completed. |
| `website_blog.mt_blog_blog_published` | "Published Post" | `website_blog` | `blog.blog` | yes |  |  | Published Post |
| `website_blog.mt_blog_blog_published` | "Published Post" | `website_blog` | `blog.blog` | yes |  |  | Published Post |
| `website_event.mt_event_published` | "Event published" | `website_event` | `event.event` | no |  |  | Event published |
| `website_event.mt_event_unpublished` | "Event unpublished" | `website_event` | `event.event` | no |  |  | Event unpublished |
| `website_event_track.mt_event_track` | "New Track" | `website_event_track` | `event.event` | no |  |  |  |
| `website_event_track.mt_track_blocked` | "Track Blocked" | `website_event_track` | `event.track` | no | yes |  | Track blocked |
| `website_event_track.mt_track_ready` | "Track Ready" | `website_event_track` | `event.track` | yes | yes |  | Track Ready for Next Stage |
| `website_forum.mt_answer_edit` | "Answer Edited" | `website_forum` | `forum.post` | no |  |  | Answer Edited |
| `website_forum.mt_answer_new` | "New Answer" | `website_forum` | `forum.post` | yes |  |  | New Answer |
| `website_forum.mt_forum_answer_new` | "New Answer" | `website_forum` | `forum.forum` | yes |  |  |  |
| `website_forum.mt_forum_question_new` | "New Question" | `website_forum` | `forum.forum` | yes |  |  |  |
| `website_forum.mt_question_edit` | "Question Edited" | `website_forum` | `forum.post` | no |  |  | Question Edited |
| `website_forum.mt_question_new` | "New Question" | `website_forum` | `forum.post` | yes |  |  | New Question |
| `website_partner.mt_partner_published` | "Partner published" | `website_partner` | `res.partner` | no |  |  | Partner Published |
| `website_partner.mt_partner_unpublished` | "Partner unpublished" | `website_partner` | `res.partner` | no |  |  | Partner Unpublished |
| `website_slides.mt_channel_slide_published` | "Presentation Published" | `website_slides` | `slide.channel` | yes |  |  | Presentation Published |

## 14. Numbering sequences

The entity Sequence (`ir.sequence`, table `ir_sequence`) ships 16 counter records. Every other document number in the system is derived from the highest number already used, not from a counter; that mechanism is specified in [`persistence-identity-and-values.md`](persistence-identity-and-values.md), section 4.4.

A shipped sequence whose company is empty serves every company. A sequence is found by its code, and when several sequences share a code the one belonging to the active company wins over the shared one.

| External identifier | Name | Package | Code | Prefix | Padding | Company |
|---|---|---|---|---|---|---|
| `account.seq_account_payment` | "Payment" | `account` | `account.payment` | `PAY` | 5 | every company |
| `crm_iap_mine.ir_sequence_crm_iap_mine` | "Lead Mining Request" | `crm_iap_mine` | `crm.iap.lead.mining.request` | `LMR` | 3 | every company |
| `hr_expense.seq_hr_expense_invoice` | "Expense invoice" | `hr_expense` | `hr.expense.invoice` | `EXP/` | 3 | the active company |
| `l10n_be.seq_declarantnum` | "Declarantnum" | `l10n_be` | `declarantnum` |  | 5 | every company |
| `point_of_sale.seq_pos_session` | "Point of Sale Session" | `point_of_sale` | `pos.session` | `/` | 5 | every company |
| `purchase.seq_purchase_order` | "Purchase Order" | `purchase` | `purchase.order` | `P` | 5 | every company |
| `purchase_requisition.seq_blanket_order` | "Blanket Order" | `purchase_requisition` | `purchase.requisition.blanket.order` | `BO` | 5 | every company |
| `purchase_requisition.seq_purchase_template` | "Purchase Template" | `purchase_requisition` | `purchase.requisition.purchase.template` | `PT` | 5 | every company |
| `sale.seq_sale_order` | "Sales Order" | `sale` | `sale.order` | `S` | 5 | every company |
| `stock.seq_package` | "Packages" | `stock` | `stock.package` | `PACK` | 7 | every company |
| `stock.seq_picking_internal` | "Picking INT" | `stock` | `stock.picking` | `INT/` | 5 | every company |
| `stock.sequence_mrp_op` | "Stock orderpoint" | `stock` | `stock.orderpoint` | `OP/` | 5 | every company |
| `stock.sequence_production_lots` | "Serial Numbers" | `stock` | `stock.lot.serial` |  | 7 | every company |
| `stock_landed_costs.seq_stock_landed_costs` | "Stock Landed Costs" | `stock_landed_costs` | `stock.landed.cost` | `LC/%(year)s/` | 4 | every company |
| `stock_picking_batch.seq_picking_batch` | "Batch Transfer" | `stock_picking_batch` | `picking.batch` | `BATCH/` | 5 | every company |
| `stock_picking_batch.seq_picking_wave` | "Wave Transfer" | `stock_picking_batch` | `picking.wave` | `WAVE/` | 5 | every company |

Each of these is shipped with the standard implementation, which draws from a database generator and therefore leaves a gap when a transaction rolls back. A tenant that needs consecutive numbers changes the implementation on the record.

## 15. Access groups and privileges

### 15.1 Privileges

The entity Privileges (`res.groups.privilege`, table `res_groups_privilege`) groups the access groups of one functional area, so that the user form shows one selector per area instead of a flat list of groups. 29 are shipped.

| External identifier | Name | Package category | Sequence |
|---|---|---|---|
| `sales_team.res_groups_privilege_sales` | "Sales" | `base.module_category_sales` | 1 |
| `project.res_groups_privilege_project` | "Project" | `base.module_category_services` | 3 |
| `mrp.res_groups_privilege_manufacturing` | "Manufacturing" | `base.module_category_supply_chain` | 5 |
| `account.res_groups_privilege_accounting` | "Accounting" | `base.module_category_accounting` | 7 |
| `purchase.res_groups_privilege_purchase` | "Purchase" | `base.module_category_supply_chain` | 8 |
| `hr.res_groups_privilege_employees` | "Employees" | `base.module_category_human_resources` | 9 |
| `product.res_groups_privilege_product` | "Products" | `base.module_category_master_data` | 9 |
| `hr_holidays.res_groups_privilege_time_off` | "Time Off" | `base.module_category_human_resources` | 10 |
| `hr_recruitment.res_groups_privilege_recruitment` | "Recruitment" | `base.module_category_human_resources` | 11 |
| `hr_expense.res_groups_privilege_expenses` | "Expenses" | `base.module_category_human_resources` | 12 |
| `hr_timesheet.res_groups_privilege_timesheets` | "Timesheets" | `base.module_category_services` | 13 |
| `hr_attendance.res_groups_privilege_attendances` | "Attendances" | `base.module_category_human_resources` | 14 |
| `lunch.res_groups_privilege_lunch` | "Lunch" | `base.module_category_human_resources` | 16 |
| `fleet.res_groups_privilege_fleet` | "Fleet" | `base.module_category_human_resources` | 17 |
| `event.res_groups_privilege_events` | "Events" | `base.module_category_marketing` | 18 |
| `mass_mailing.res_groups_privilege_email_marketing` | "Email Marketing" | `base.module_category_marketing` | 19 |
| `im_livechat.res_groups_privilege_live_chat` | "Live Chat" | `base.module_category_website` | 20 |
| `survey.res_groups_privilege_surveys` | "Surveys" | `base.module_category_marketing` | 20 |
| `point_of_sale.res_groups_privilege_point_of_sale` | "Point of Sale" | `base.module_category_sales` | 21 |
| `website_slides.res_groups_privilege_elearning` | "Electronic Learning" | `base.module_category_website` | 21 |
| `website.res_groups_privilege_website` | "Website" | `base.module_category_website` | 23 |
| `spreadsheet_dashboard.res_groups_privilege_dashboard` | "Dashboard" | `base.module_category_productivity` | 30 |
| `account.res_group_privilege_accounting_bank` | "Bank" | `base.module_category_accounting` | 50 |
| `mail.res_groups_privilege_canned_response` | "Canned Responses" | `base.module_category_marketing` | 100 |
| `marketing_card.res_groups_privilege_marketing_card` | "Marketing Card" | `base.module_category_marketing` | 100 |
| `base.res_groups_privilege_contact` | "Contact" | `module_category_master_data` |  |
| `base.res_groups_privilege_export` | "Export" | `module_category_master_data` |  |
| `stock.res_groups_privilege_inventory` | "Inventory" | `base.module_category_supply_chain` |  |
| `maintenance.res_groups_privilege_maintenance` | "Maintenance" | `base.module_category_supply_chain` |  |

### 15.2 Access groups

The entity Access Groups (`res.groups`, table `res_groups`) ships 140 groups. A group may imply other groups, so that granting the manager group grants the user group with it; the implication graph is transitive and is what a rebuild must reproduce, because an access right granted to an implied group is granted to every group that implies it. A group marked as a portal group is one whose members are external users and are therefore subject to the record rules that restrict a portal reader to their own documents.

The complete access matrix — which group may read, write, create and delete which entity, and under which record rule — is generated as [`../references/access-matrix-by-group.md`](../references/access-matrix-by-group.md) and [`../references/groups-and-access.md`](../references/groups-and-access.md), from the 1,933 shipped access rights and 576 shipped record rules of [`../../schemas/operational/`](../../schemas/operational/). The rules themselves are specified in [`../overview/security-model.md`](../overview/security-model.md).

| External identifier | Name | Package | Privilege | Portal group | Comment |
|---|---|---|---|---|---|
| `account.group_account_manager` | "Administrator" | `account` | `res_groups_privilege_accounting` |  | Full access, including configuration rights. |
| `account.group_cash_rounding` | "Allow the cash rounding management" | `account` |  |  |  |
| `account.group_account_basic` | "Basic" | `account` |  |  |  |
| `account.group_delivery_invoice_address` | "Delivery Address" | `account` |  |  |  |
| `account.group_account_invoice` | "Invoicing" | `account` | `res_groups_privilege_accounting` |  | Invoices, payments and basic invoice reporting. |
| `account.group_partial_purchase_deductibility` | "Partial Purchase Deductibility" | `account` |  |  |  |
| `account.group_account_readonly` | "Show Accounting Features - Readonly" | `account` |  |  |  |
| `account.group_account_user` | "Show Full Accounting Features" | `account` |  |  |  |
| `account.group_account_secured` | "Show Inalterability Features" | `account` |  |  |  |
| `account.group_validate_bank_account` | "Validate bank account" | `account` | `res_group_privilege_accounting_bank` |  |  |
| `analytic.group_analytic_accounting` | "Analytic Accounting" | `analytic` |  |  |  |
| `api_doc.group_allow_doc` | "Technical Documentation" | `api_doc` |  |  |  |
| `base.group_erp_manager` | "Access Rights" | `base` |  |  |  |
| `base.group_allow_export` | "Allowed" | `base` | `res_groups_privilege_export` |  |  |
| `base.group_sanitize_override` | "Bypass Rich Text Field Sanitize" | `base` |  |  |  |
| `base.group_partner_manager` | "Creation" | `base` | `res_groups_privilege_contact` |  |  |
| `base.default_user_group` | "Default access for new users" | `base` |  |  |  |
| `base.group_multi_company` | "Multi Companies" | `base` |  |  |  |
| `base.group_multi_currency` | "Multi Currencies" | `base` |  |  |  |
| `base.group_system` | "Role / Administrator" | `base` |  |  | Access to the settings to configure the apps |
| `base.group_portal` | "Role / Portal" | `base` |  |  | Portal members have specific access rights (such as record rules and restricted menus).                 They usually do not belong to the usual the system groups. |
| `base.group_public` | "Role / Public" | `base` |  |  | Public users have specific access rights (such as record rules and restricted menus).                 They usually do not belong to the usual the system groups. |
| `base.group_user` | "Role / User" | `base` |  |  | Access to the home menu |
| `base.group_no_one` | "Technical Features" | `base` |  |  |  |
| `crm.group_use_lead` | "Show Lead Menu" | `crm` |  |  |  |
| `crm.group_use_recurring_revenues` | "Show Recurring Revenues Menu" | `crm` |  |  |  |
| `event.group_event_manager` | "Administrator" | `event` | `res_groups_privilege_events` |  |  |
| `event.group_event_registration_desk` | "Registration Desk" | `event` | `res_groups_privilege_events` |  |  |
| `event.group_event_user` | "User" | `event` | `res_groups_privilege_events` |  |  |
| `sales_team.group_sale_salesman` | "" | `event_sale` |  |  |  |
| `fleet.fleet_group_manager` | "Administrator" | `fleet` | `res_groups_privilege_fleet` |  |  |
| `fleet.fleet_group_user` | "Officer: Manage all vehicles" | `fleet` | `res_groups_privilege_fleet` |  |  |
| `hr.group_hr_manager` | "Administrator" | `hr` | `res_groups_privilege_employees` |  | The user will have access to the human resources configuration as well as statistic reports. |
| `hr.group_hr_user` | "Officer: Manage all employees" | `hr` | `res_groups_privilege_employees` |  | The user will be able to create and edit employees. |
| `base.group_user` | "" | `hr_attendance` |  |  |  |
| `hr_attendance.group_hr_attendance_manager` | "Administrator" | `hr_attendance` | `res_groups_privilege_attendances` |  |  |
| `hr_attendance.group_hr_attendance_user` | "Officer: Manage all attendances" | `hr_attendance` | `res_groups_privilege_attendances` |  | The user will have access to all attendance records and reports for all employees. |
| `hr_attendance.group_hr_attendance_officer` | "Officer: Manage attendances" | `hr_attendance` |  |  | The user will have access to the attendance records and reporting of employees where he's set as an attendance manager |
| `hr_attendance.group_hr_attendance_own_reader` | "User: Read his own attendances" | `hr_attendance` |  |  | The user will have access to his own attendances on his user / employee profile |
| `hr_expense.group_hr_expense_manager` | "Administrator" | `hr_expense` | `res_groups_privilege_expenses` |  |  |
| `hr_expense.group_hr_expense_user` | "All Approver" | `hr_expense` | `res_groups_privilege_expenses` |  |  |
| `hr_expense.group_hr_expense_team_approver` | "Team Approver" | `hr_expense` | `res_groups_privilege_expenses` |  |  |
| `hr_holidays.group_hr_holidays_manager` | "Administrator" | `hr_holidays` | `res_groups_privilege_time_off` |  | Can manage and configure all holidays and leave requests.  A user without any rights on Time Off will be able to see the application, create his own holidays and manage the requests of the users he's manager of. |
| `hr_holidays.group_hr_holidays_user` | "Officer: Manage all requests" | `hr_holidays` | `res_groups_privilege_time_off` |  |  |
| `hr_holidays.group_hr_holidays_responsible` | "Time Off Responsible" | `hr_holidays` |  |  |  |
| `hr.group_hr_user` | "" | `hr_maintenance` |  |  |  |
| `base.group_user` | "" | `hr_recruitment` |  |  |  |
| `hr_recruitment.group_hr_recruitment_manager` | "Administrator" | `hr_recruitment` | `res_groups_privilege_recruitment` |  |  |
| `hr_recruitment.group_applicant_cv_display` | "Display CV on application form" | `hr_recruitment` |  |  |  |
| `hr_recruitment.group_hr_recruitment_interviewer` | "Interviewer" | `hr_recruitment` | `res_groups_privilege_recruitment` |  | Interviewer right will give access to all job position/applications where the employee is defined. It will allow to refuse, plan meetings. |
| `hr_recruitment.group_hr_recruitment_user` | "Officer: Manage all applicants" | `hr_recruitment` | `res_groups_privilege_recruitment` |  |  |
| `project.group_project_manager` | "" | `hr_timesheet` |  |  |  |
| `hr_timesheet.group_timesheet_manager` | "Administrator" | `hr_timesheet` | `res_groups_privilege_timesheets` |  |  |
| `hr_timesheet.group_hr_timesheet_approver` | "User: all timesheets" | `hr_timesheet` | `res_groups_privilege_timesheets` |  |  |
| `hr_timesheet.group_hr_timesheet_user` | "User: own timesheets only" | `hr_timesheet` | `res_groups_privilege_timesheets` |  |  |
| `im_livechat.im_livechat_group_manager` | "Administrator" | `im_livechat` | `res_groups_privilege_live_chat` |  | The user will be able to delete support channels. |
| `im_livechat.im_livechat_group_user` | "User" | `im_livechat` | `res_groups_privilege_live_chat` |  | The user will be able to join support channels. |
| `l10n_in.group_l10n_in_reseller` | "Manage Reseller(Electronic Commerce)" | `l10n_in` |  |  |  |
| `lunch.group_lunch_manager` | "Administrator" | `lunch` | `res_groups_privilege_lunch` |  | Be able to create new products, cashmoves and to confirm or cancel orders. |
| `lunch.group_lunch_user` | "User : Order your meal" | `lunch` | `res_groups_privilege_lunch` |  |  |
| `base.group_system` | "" | `mail` |  |  |  |
| `mail.group_mail_canned_response_admin` | "Canned Response Administrator" | `mail` | `res_groups_privilege_canned_response` |  |  |
| `mail.group_mail_template_editor` | "Mail Template Editor" | `mail` |  |  |  |
| `mail.group_mail_notification_type_inbox` | "Receive notifications in the system" | `mail` |  |  |  |
| `base.group_system` | "" | `mail_group` |  |  |  |
| `mail_group.group_mail_group_manager` | "Mail Group Administrator" | `mail_group` |  |  |  |
| `maintenance.group_equipment_manager` | "Equipment Manager" | `maintenance` | `res_groups_privilege_maintenance` |  | The user will be able to manage equipment. |
| `marketing_card.marketing_card_group_manager` | "Marketing Card Manager" | `marketing_card` | `res_groups_privilege_marketing_card` |  |  |
| `marketing_card.marketing_card_group_user` | "Marketing Card User" | `marketing_card` | `res_groups_privilege_marketing_card` |  |  |
| `mass_mailing.group_mass_mailing_campaign` | "Manage Mass Mailing Campaigns" | `mass_mailing` |  |  |  |
| `mass_mailing.group_mass_mailing_user` | "User" | `mass_mailing` | `res_groups_privilege_email_marketing` |  |  |
| `mrp.group_mrp_manager` | "Administrator" | `mrp` | `res_groups_privilege_manufacturing` |  | Manage the manufacturing processes and generate reports on those processes. |
| `mrp.group_mrp_routings` | "Manage Work Order Operations" | `mrp` |  |  |  |
| `mrp.group_mrp_byproducts` | "Produce residual products" | `mrp` |  |  |  |
| `mrp.group_unlocked_by_default` | "Unlocked by default" | `mrp` |  |  |  |
| `mrp.group_mrp_workorder_dependencies` | "Use Operation Dependencies" | `mrp` |  |  |  |
| `mrp.group_mrp_reception_report` | "Use Reception Report with Manufacturing Orders" | `mrp` |  |  |  |
| `mrp.group_mrp_user` | "User" | `mrp` | `res_groups_privilege_manufacturing` |  |  |
| `point_of_sale.group_pos_manager` | "Administrator" | `point_of_sale` | `res_groups_privilege_point_of_sale` |  |  |
| `point_of_sale.group_pos_preset` | "Preset Menu" | `point_of_sale` |  |  |  |
| `point_of_sale.group_pos_user` | "User" | `point_of_sale` | `res_groups_privilege_point_of_sale` |  |  |
| `product.group_product_pricelist` | "Basic Pricelists" | `product` |  |  |  |
| `product.group_product_manager` | "Create" | `product` | `res_groups_privilege_product` |  |  |
| `product.group_product_variant` | "Manage Product Variants" | `product` |  |  |  |
| `product_expiry.group_expiry_date_on_delivery_slip` | "Include expiration dates on delivery slip" | `product_expiry` |  |  |  |
| `base.group_user` | "" | `product_matrix` |  |  |  |
| `project.group_project_manager` | "Administrator" | `project` | `res_groups_privilege_project` |  | Administrator: Can manage projects and stages, with access to reporting and configuration. |
| `project.group_project_milestone` | "Use Milestones" | `project` |  |  |  |
| `project.group_project_recurring_tasks` | "Use Recurring Tasks" | `project` |  |  |  |
| `project.group_project_stages` | "Use Stages on Project" | `project` |  |  |  |
| `project.group_project_task_dependencies` | "Use Task Dependencies" | `project` |  |  |  |
| `project.group_project_user` | "User" | `project` | `res_groups_privilege_project` |  | User: Can manage tasks in projects shared with them. |
| `base.group_user` | "" | `purchase` |  |  |  |
| `purchase.group_warning_purchase` | "A warning can be set on a product or a customer (Purchase)" | `purchase` |  |  |  |
| `purchase.group_purchase_manager` | "Administrator" | `purchase` | `res_groups_privilege_purchase` |  |  |
| `purchase.group_send_reminder` | "Send an automatic reminder email to confirm delivery" | `purchase` |  |  |  |
| `purchase.group_purchase_user` | "User" | `purchase` | `res_groups_privilege_purchase` |  |  |
| `purchase_requisition.group_purchase_alternatives` | "Manage Purchase Alternatives" | `purchase_requisition` |  |  |  |
| `sale.group_warning_sale` | "A warning can be set on a product or a customer (Sale)" | `sale` |  |  |  |
| `sale.group_discount_per_so_line` | "Discount on lines" | `sale` |  |  |  |
| `sale.group_auto_done_setting` | "Lock Confirmed Sales" | `sale` |  |  |  |
| `sale.group_proforma_sales` | "Pro-forma Invoices" | `sale` |  |  |  |
| `sale_management.group_sale_order_template` | "Quotation Templates" | `sale_management` |  |  |  |
| `base.group_user` | "" | `sale_timesheet` |  |  |  |
| `sales_team.group_sale_manager` | "Administrator" | `sales_team` | `res_groups_privilege_sales` |  | the user will have an access to the sales configuration as well as statistic reports. |
| `sales_team.group_sale_salesman_all_leads` | "User: All Documents" | `sales_team` | `res_groups_privilege_sales` |  | the user will have access to all records of everyone in the sales application. |
| `sales_team.group_sale_salesman` | "User: Own Documents Only" | `sales_team` | `res_groups_privilege_sales` |  | the user will have access to his own data in the sales application. |
| `spreadsheet_dashboard.group_dashboard_manager` | "Admin" | `spreadsheet_dashboard` | `res_groups_privilege_dashboard` |  |  |
| `stock.group_warning_stock` | "A warning can be set on a partner (Stock)" | `stock` |  |  |  |
| `stock.group_stock_manager` | "Administrator" | `stock` | `res_groups_privilege_inventory` |  |  |
| `stock.group_lot_on_delivery_slip` | "Display Serial & Lot Number in Delivery Slips" | `stock` |  |  |  |
| `stock.group_tracking_owner` | "Manage Different Stock Owners" | `stock` |  |  |  |
| `stock.group_production_lot` | "Manage Lots / Serial Numbers" | `stock` |  |  |  |
| `stock.group_stock_multi_locations` | "Manage Multiple Stock Locations" | `stock` |  |  |  |
| `stock.group_stock_multi_warehouses` | "Manage Multiple Warehouses" | `stock` |  |  |  |
| `stock.group_tracking_lot` | "Manage Packages" | `stock` |  |  |  |
| `stock.group_adv_location` | "Manage Push and Pull inventory flows" | `stock` |  |  |  |
| `stock.group_stock_lot_print_gs1` | "Print GS1 Barcodes for Lot & Serial Numbers" | `stock` |  |  |  |
| `stock.group_stock_sign_delivery` | "Require a signature on your delivery orders" | `stock` |  |  |  |
| `stock.group_reception_report` | "Use Reception Report" | `stock` |  |  |  |
| `stock.group_stock_user` | "User" | `stock` | `res_groups_privilege_inventory` |  |  |
| `stock_account.group_lot_on_invoice` | "Display Serial & Lot Number on Invoices" | `stock_account` |  |  |  |
| `survey.group_survey_manager` | "Administrator" | `survey` | `res_groups_privilege_surveys` |  |  |
| `survey.group_survey_user` | "User" | `survey` | `res_groups_privilege_surveys` |  |  |
| `uom.group_uom` | "Manage Multiple Units of Measure" | `uom` |  |  |  |
| `base.group_public` | "" | `website` |  |  |  |
| `base.group_portal` | "" | `website` |  |  |  |
| `website.group_website_designer` | "Editor and Designer" | `website` | `res_groups_privilege_website` |  |  |
| `website.group_multi_website` | "Multi-website" | `website` |  |  |  |
| `website.website_page_controller_expose` | "Public access to arbitrary exposed model" | `website` |  |  |  |
| `website.group_website_restricted_editor` | "Restricted Editor" | `website` | `res_groups_privilege_website` |  |  |
| `event.group_event_manager` | "" | `website_event` |  |  |  |
| `hr_recruitment.group_hr_recruitment_user` | "" | `website_hr_recruitment` |  |  |  |
| `base.group_user` | "" | `website_sale` |  |  |  |
| `sales_team.group_sale_manager` | "" | `website_sale` |  |  |  |
| `website_sale.group_product_price_comparison` | "Comparison Price" | `website_sale` |  |  |  |
| `website_sale.group_product_feed` | "Product Feed" | `website_sale` |  |  |  |
| `website_sale.group_show_uom_price` | "Unit of Measure Price Display for Electronic Commerce" | `website_sale` |  |  |  |
| `website_slides.group_website_slides_manager` | "Manager" | `website_slides` | `res_groups_privilege_elearning` |  |  |
| `website_slides.group_website_slides_officer` | "Officer" | `website_slides` | `res_groups_privilege_elearning` |  |  |

## 16. Every shipped record set

All 155 sets declared through declaration documents, with the number of declarations, the number of distinct records those declarations produce, and the packages that contribute. A set whose declaration count exceeds its distinct count is one that several packages extend.

| Record set | Transport name | Specified in | Declarations | Distinct records | Contributing packages |
|---|---|---|---|---|---|
| Account Cash Rounding | `account.cash.rounding` | General Ledger | 2 | 2 | `l10n_hu_edi`, `l10n_in` |
| Account Tag | `account.account.tag` | General Ledger | 519 | 517 | `account`, `l10n_at`, `l10n_au`, `l10n_cl`, `l10n_de`, `l10n_eu_oss`, `l10n_fi`, `l10n_fr_account`, `l10n_il`, `l10n_in`, `l10n_it`, `l10n_lt`, `l10n_nl`, `l10n_si`, `l10n_ua` |
| Accounting Assert Test | `accounting.assert.test` | Financial Reporting | 6 | 6 | `account_test` |
| Accounting Report | `account.report` | General Ledger | 165 | 165 | `account`, `l10n_ae`, `l10n_at`, `l10n_au`, `l10n_bd`, `l10n_be`, `l10n_bf`, `l10n_bg`, `l10n_bh`, `l10n_bj`, `l10n_bo`, `l10n_br`, `l10n_ca`, `l10n_cd`, `l10n_cf`, `l10n_cg`, `l10n_ch`, `l10n_ci`, `l10n_cl`, `l10n_cm`, `l10n_cy`, `l10n_cz`, `l10n_de`, `l10n_dk`, `l10n_do`, `l10n_dz`, `l10n_ec`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_et`, `l10n_fi`, `l10n_fr_account`, `l10n_ga`, `l10n_ge`, `l10n_gn`, `l10n_gq`, `l10n_gr`, `l10n_gw`, `l10n_hr`, `l10n_hr_kuna`, `l10n_hu`, `l10n_ie`, `l10n_il`, `l10n_in`, `l10n_it`, `l10n_jo`, `l10n_jp`, `l10n_ke`, `l10n_kh`, `l10n_km`, `l10n_kr`, `l10n_kz`, `l10n_lk`, `l10n_lt`, `l10n_lu`, `l10n_lv`, `l10n_ma`, `l10n_ml`, `l10n_mn`, `l10n_mr`, `l10n_mt`, `l10n_mu_account`, `l10n_mx`, `l10n_my`, `l10n_mz`, `l10n_ne`, `l10n_ng`, `l10n_nl`, `l10n_no`, `l10n_nz`, `l10n_om`, `l10n_ph`, `l10n_pk`, `l10n_pl`, `l10n_pt`, `l10n_ro`, `l10n_rs`, `l10n_rw`, `l10n_sa`, `l10n_se`, `l10n_sg`, `l10n_si`, `l10n_sk`, `l10n_sn`, `l10n_td`, `l10n_tg`, `l10n_th`, `l10n_tn`, `l10n_tr`, `l10n_tw`, `l10n_tz_account`, `l10n_ug`, `l10n_uk`, `l10n_us_account`, `l10n_uy`, `l10n_vn`, `l10n_za`, `l10n_zm_account` |
| Accounting Report Column | `account.report.column` | General Ledger | 275 | 275 | `account`, `l10n_ae`, `l10n_at`, `l10n_au`, `l10n_bd`, `l10n_be`, `l10n_bf`, `l10n_bg`, `l10n_bh`, `l10n_bj`, `l10n_bo`, `l10n_br`, `l10n_ca`, `l10n_cd`, `l10n_cf`, `l10n_cg`, `l10n_ch`, `l10n_ci`, `l10n_cl`, `l10n_cm`, `l10n_cy`, `l10n_cz`, `l10n_de`, `l10n_dk`, `l10n_do`, `l10n_dz`, `l10n_ec`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_et`, `l10n_fi`, `l10n_fr_account`, `l10n_ga`, `l10n_ge`, `l10n_gn`, `l10n_gq`, `l10n_gr`, `l10n_gw`, `l10n_hr`, `l10n_hr_kuna`, `l10n_hu`, `l10n_ie`, `l10n_il`, `l10n_in`, `l10n_it`, `l10n_jo`, `l10n_jp`, `l10n_ke`, `l10n_kh`, `l10n_km`, `l10n_kr`, `l10n_kz`, `l10n_lk`, `l10n_lt`, `l10n_lu`, `l10n_lv`, `l10n_ma`, `l10n_ml`, `l10n_mn`, `l10n_mr`, `l10n_mt`, `l10n_mu_account`, `l10n_mx`, `l10n_my`, `l10n_mz`, `l10n_ne`, `l10n_ng`, `l10n_nl`, `l10n_no`, `l10n_nz`, `l10n_om`, `l10n_ph`, `l10n_pk`, `l10n_pl`, `l10n_pt`, `l10n_ro`, `l10n_rs`, `l10n_rw`, `l10n_sa`, `l10n_se`, `l10n_sg`, `l10n_si`, `l10n_sk`, `l10n_sn`, `l10n_td`, `l10n_tg`, `l10n_th`, `l10n_tn`, `l10n_tr`, `l10n_tw`, `l10n_tz_account`, `l10n_ug`, `l10n_uk`, `l10n_us_account`, `l10n_uy`, `l10n_vn`, `l10n_za`, `l10n_zm_account` |
| Accounting Report Expression | `account.report.expression` | General Ledger | 6,555 | 6551 | `l10n_ae`, `l10n_at`, `l10n_au`, `l10n_bd`, `l10n_be`, `l10n_bf`, `l10n_bg`, `l10n_bh`, `l10n_bj`, `l10n_bo`, `l10n_br`, `l10n_ca`, `l10n_cd`, `l10n_cf`, `l10n_cg`, `l10n_ch`, `l10n_ci`, `l10n_cl`, `l10n_cm`, `l10n_cy`, `l10n_cz`, `l10n_de`, `l10n_dk`, `l10n_do`, `l10n_dz`, `l10n_ec`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_et`, `l10n_fi`, `l10n_fr_account`, `l10n_ga`, `l10n_ge`, `l10n_gn`, `l10n_gq`, `l10n_gr`, `l10n_gw`, `l10n_hr`, `l10n_hr_kuna`, `l10n_hu`, `l10n_ie`, `l10n_il`, `l10n_in`, `l10n_it`, `l10n_jo`, `l10n_jp`, `l10n_ke`, `l10n_kh`, `l10n_km`, `l10n_kr`, `l10n_kz`, `l10n_lk`, `l10n_lt`, `l10n_lu`, `l10n_lv`, `l10n_ma`, `l10n_ml`, `l10n_mn`, `l10n_mr`, `l10n_mt`, `l10n_mu_account`, `l10n_mx`, `l10n_my`, `l10n_mz`, `l10n_ne`, `l10n_ng`, `l10n_nl`, `l10n_no`, `l10n_nz`, `l10n_om`, `l10n_ph`, `l10n_pk`, `l10n_pl`, `l10n_pt`, `l10n_ro`, `l10n_rs`, `l10n_rw`, `l10n_sa`, `l10n_se`, `l10n_sg`, `l10n_si`, `l10n_sk`, `l10n_sn`, `l10n_td`, `l10n_tg`, `l10n_th`, `l10n_tn`, `l10n_tr`, `l10n_tw`, `l10n_tz_account`, `l10n_ug`, `l10n_uk`, `l10n_uy`, `l10n_vn`, `l10n_za`, `l10n_zm_account` |
| Accounting Report Line | `account.report.line` | General Ledger | 6,076 | 6073 | `l10n_ae`, `l10n_at`, `l10n_au`, `l10n_bd`, `l10n_be`, `l10n_bf`, `l10n_bg`, `l10n_bh`, `l10n_bj`, `l10n_bo`, `l10n_br`, `l10n_ca`, `l10n_cd`, `l10n_cf`, `l10n_cg`, `l10n_ch`, `l10n_ci`, `l10n_cl`, `l10n_cm`, `l10n_cy`, `l10n_cz`, `l10n_de`, `l10n_dk`, `l10n_do`, `l10n_dz`, `l10n_ec`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_et`, `l10n_fi`, `l10n_fr_account`, `l10n_ga`, `l10n_ge`, `l10n_gn`, `l10n_gq`, `l10n_gr`, `l10n_gw`, `l10n_hr`, `l10n_hr_kuna`, `l10n_hu`, `l10n_ie`, `l10n_il`, `l10n_in`, `l10n_it`, `l10n_jo`, `l10n_jp`, `l10n_ke`, `l10n_kh`, `l10n_km`, `l10n_kr`, `l10n_kz`, `l10n_lk`, `l10n_lt`, `l10n_lu`, `l10n_lv`, `l10n_ma`, `l10n_ml`, `l10n_mn`, `l10n_mr`, `l10n_mt`, `l10n_mu_account`, `l10n_mx`, `l10n_my`, `l10n_mz`, `l10n_ne`, `l10n_ng`, `l10n_nl`, `l10n_no`, `l10n_nz`, `l10n_om`, `l10n_ph`, `l10n_pk`, `l10n_pl`, `l10n_pt`, `l10n_ro`, `l10n_rs`, `l10n_rw`, `l10n_sa`, `l10n_se`, `l10n_sg`, `l10n_si`, `l10n_sk`, `l10n_sn`, `l10n_td`, `l10n_tg`, `l10n_th`, `l10n_tn`, `l10n_tr`, `l10n_tw`, `l10n_tz_account`, `l10n_ug`, `l10n_uk`, `l10n_uy`, `l10n_vn`, `l10n_za`, `l10n_zm_account` |
| Activity Plan | `mail.activity.plan` | Messaging and Activities | 2 | 2 | `hr` |
| Activity plan template | `mail.activity.plan.template` | Messaging and Activities | 6 | 6 | `hr`, `hr_fleet` |
| Analytic Plans | `account.analytic.plan` | Analytic Accounting | 1 | 1 | `analytic` |
| Applicant Degree | `hr.recruitment.degree` | Recruitment | 4 | 4 | `hr_recruitment` |
| ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | Fiscal Localizations | 16 | 16 | `l10n_ar` |
| Bank | `res.bank` | Contacts and Organizations | 115 | 115 | `base`, `l10n_lt`, `l10n_mx` |
| Barcode Nomenclature | `barcode.nomenclature` | Products and Catalog | 2 | 2 | `barcodes`, `barcodes_gs1_nomenclature` |
| Barcode Rule | `barcode.rule` | Products and Catalog | 39 | 39 | `barcodes`, `barcodes_gs1_nomenclature`, `point_of_sale`, `pos_loyalty`, `stock` |
| Blog | `blog.blog` | Website and Storefront | 1 | 1 | `website_blog` |
| Brand of the vehicle | `fleet.vehicle.model.brand` | Fleet | 67 | 67 | `fleet` |
| Campaign Stage | `utm.stage` | Customer Relationship Management | 1 | 1 | `utm` |
| campaign tracking parameter Campaign | `utm.campaign` | Customer Relationship Management | 1 | 1 | `hr_recruitment` |
| campaign tracking parameter Medium | `utm.medium` | Customer Relationship Management | 11 | 11 | `mass_mailing_sms`, `utm` |
| campaign tracking parameter Source | `utm.source` | Customer Relationship Management | 11 | 11 | `crm_livechat`, `utm` |
| campaign tracking parameter Tag | `utm.tag` | Customer Relationship Management | 1 | 1 | `utm` |
| Canned Response | `mail.canned.response` | Messaging and Activities | 1 | 1 | `mail` |
| Category of applicant | `hr.applicant.category` | Recruitment | 4 | 4 | `hr_recruitment` |
| Channel Member | `discuss.channel.member` | Messaging and Activities | 1 | 1 | `mail` |
| Channel/Course Groups | `slide.channel.tag.group` | Learning, Surveys and Gamification | 2 | 2 | `website_slides` |
| Channel/Course Tag | `slide.channel.tag` | Learning, Surveys and Gamification | 3 | 3 | `website_slides` |
| Chatbot Script | `chatbot.script` | Messaging and Activities | 2 | 2 | `crm_livechat`, `im_livechat` |
| Chatbot Script Answer | `chatbot.script.answer` | Messaging and Activities | 3 | 3 | `im_livechat` |
| Chatbot Script Step | `chatbot.script.step` | Messaging and Activities | 14 | 14 | `crm_livechat`, `im_livechat` |
| City | `res.city` | Contacts and Organizations | 2,850 | 2850 | `l10n_cn_city` |
| Coins/Bills | `pos.bill` | Point of Sale | 14 | 14 | `l10n_in_pos`, `point_of_sale` |
| Companies | `res.company` | Contacts and Organizations | 3 | 1 | `base`, `l10n_us`, `resource` |
| Contact | `res.partner` | Contacts and Organizations | 23 | 21 | `base`, `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_es`, `l10n_my_edi_pos`, `l10n_pe_pos`, `l10n_tw_edi_ecpay_pos`, `l10n_uy`, `l10n_vn_edi_viettel_pos`, `lunch`, `mail`, `website_partner` |
| Contract Type | `hr.contract.type` | Human Resources Core | 12 | 12 | `hr` |
| Country | `res.country` | Contacts and Organizations | 261 | 251 | `base`, `l10n_br`, `l10n_cn_city`, `l10n_eg_edi_eta`, `l10n_kr`, `l10n_pe`, `l10n_sa`, `l10n_sa_edi`, `l10n_se`, `l10n_si`, `l10n_tw` |
| Country Group | `res.country.group` | Contacts and Organizations | 19 | 19 | `account`, `base`, `l10n_ae`, `l10n_es`, `l10n_fr`, `l10n_fr_account`, `l10n_in`, `l10n_latam_base`, `l10n_nl`, `l10n_uk` |
| Country state | `res.country.state` | Contacts and Organizations | 39 | 39 | `l10n_in` |
| Croatian tax expence categories | `l10n.hr.tax.category` | Fiscal Localizations | 13 | 13 | `l10n_hr_edi` |
| Currency | `res.currency` | Multi-Currency | 186 | 173 | `base`, `l10n_au`, `l10n_cl`, `l10n_hk`, `l10n_nz`, `l10n_tw`, `l10n_uy` |
| customer relationship management Recurring revenue plans | `crm.recurring.plan` | Customer Relationship Management | 4 | 4 | `crm` |
| customer relationship management Stages | `crm.stage` | Customer Relationship Management | 4 | 4 | `crm` |
| customer relationship management Tag | `crm.tag` | Sales | 3 | 3 | `website_crm_partner_assign` |
| Delivery Price Rules | `delivery.price.rule` | Delivery and Shipping | 8 | 8 | `delivery_mondialrelay` |
| Department | `hr.department` | Human Resources Core | 1 | 1 | `hr` |
| Departure Reason | `hr.departure.reason` | Human Resources Core | 3 | 3 | `hr` |
| Digest | `digest.digest` | Messaging and Activities | 9 | 1 | `account`, `crm`, `digest`, `hr_recruitment`, `im_livechat`, `point_of_sale`, `project`, `sale_management`, `website_sale` |
| Digest Tips | `digest.tip` | Messaging and Activities | 39 | 39 | `account`, `base_automation`, `crm`, `digest`, `hr`, `hr_expense`, `hr_recruitment`, `hr_timesheet`, `im_livechat`, `mrp`, `project`, `purchase`, `sale_management`, `stock`, `website` |
| Discussion Channel | `discuss.channel` | Messaging and Activities | 3 | 2 | `mail` |
| Easily load a set of configuration options | `pos.preset` | Point of Sale | 6 | 3 | `pos_restaurant`, `pos_self_order` |
| electronic data interchange format | `account.edi.format` | Electronic Invoicing and Document Exchange | 3 | 3 | `l10n_eg_edi_eta`, `l10n_es_edi_sii`, `l10n_sa_edi` |
| Electronic Waybill Document Type | `l10n.in.ewaybill.type` | Fiscal Localizations | 17 | 17 | `l10n_in_ewaybill`, `l10n_in_ewaybill_stock` |
| Email Aliases | `mail.alias` | Messaging and Activities | 1 | 1 | `hr_expense` |
| Embedded Actions | `ir.embedded.actions` | Platform Foundation | 28 | 28 | `hr_timesheet`, `project`, `project_account`, `project_hr_expense`, `project_mrp`, `project_purchase`, `project_stock`, `sale_project` |
| Employee | `hr.employee` | Human Resources Core | 1 | 1 | `hr` |
| Event Alarm | `calendar.alarm` | Calendar and Scheduling | 7 | 7 | `calendar` |
| Event Booth Category | `event.booth.category` | Events | 8 | 3 | `event_booth`, `event_booth_sale`, `website_event_booth_exhibitor` |
| Event Question | `event.question` | Events | 3 | 3 | `event` |
| Event Sponsor Level | `event.sponsor.type` | Events | 3 | 3 | `website_event_exhibitor` |
| Event Stage | `event.stage` | Events | 4 | 4 | `event` |
| Event Track Stage | `event.track.stage` | Events | 6 | 6 | `website_event_track` |
| Fields that can be used for predictive lead scoring computation | `crm.lead.scoring.frequency.field` | Customer Relationship Management | 7 | 7 | `crm` |
| Fleet Service Type | `fleet.service.type` | Fleet | 3 | 3 | `account_fleet`, `fleet` |
| Forum | `forum.forum` | Website and Storefront | 1 | 1 | `website_forum` |
| Gamification Badge | `gamification.badge` | Learning, Surveys and Gamification | 38 | 37 | `gamification`, `website_forum`, `website_slides`, `website_slides_survey` |
| Gamification Challenge | `gamification.challenge` | Learning, Surveys and Gamification | 37 | 37 | `gamification`, `gamification_sale_crm`, `website_forum`, `website_slides` |
| Gamification generic goal for challenge | `gamification.challenge.line` | Learning, Surveys and Gamification | 41 | 41 | `gamification`, `gamification_sale_crm`, `website_forum`, `website_slides` |
| Gamification Goal Definition | `gamification.goal.definition` | Learning, Surveys and Gamification | 47 | 46 | `gamification`, `gamification_sale_crm`, `website_forum`, `website_slides`, `website_slides_survey` |
| Geo Provider | `base.geo_provider` | Platform Foundation | 2 | 2 | `base_geolocalize` |
| Group of dashboards | `spreadsheet.dashboard.group` | Spreadsheets and Dashboards | 7 | 7 | `spreadsheet_dashboard` |
| human resources Work Entry Type | `hr.work.entry.type` | Work Entries | 118 | 118 | `hr_work_entry` |
| Identification Types | `l10n_latam.identification.type` | Fiscal Localizations | 102 | 64 | `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_pe`, `l10n_uy` |
| in-app purchase Service | `iap.service` | Automation and Integration | 5 | 5 | `iap`, `l10n_in`, `partner_autocomplete`, `sms`, `snailmail` |
| Incoterms | `account.incoterms` | General Ledger | 15 | 15 | `account`, `l10n_tr_nilvera_einvoice_extended` |
| Industry | `res.partner.industry` | Contacts and Organizations | 23 | 23 | `base`, `l10n_in` |
| Inventory Locations | `stock.location` | Inventory Operations | 3 | 3 | `stock` |
| Inventory Routes | `stock.route` | Inventory Operations | 6 | 5 | `mrp`, `mrp_subcontracting`, `purchase_stock`, `sale_stock`, `stock`, `stock_dropshipping` |
| Job Platforms | `hr.job.platform` | Recruitment | 3 | 3 | `hr_recruitment` |
| l10n_ar.earnings.scale | `l10n_ar.earnings.scale` | Fiscal Localizations | 2 | 2 | `l10n_ar_withholding` |
| l10n_ar.earnings.scale.line | `l10n_ar.earnings.scale.line` | Fiscal Localizations | 16 | 16 | `l10n_ar_withholding` |
| Languages | `res.lang` | Contacts and Organizations | 6 | 6 | `base` |
| Latam Document Type | `l10n_latam.document.type` | Fiscal Localizations | 60 | 60 | `l10n_pe` |
| Livechat Channel | `im_livechat.channel` | Messaging and Activities | 1 | 1 | `im_livechat` |
| Loyalty Program | `loyalty.program` | Loyalty and Promotions | 2 | 1 | `loyalty`, `pos_loyalty` |
| Loyalty Reward | `loyalty.reward` | Loyalty and Promotions | 1 | 1 | `loyalty` |
| Loyalty Rule | `loyalty.rule` | Loyalty and Promotions | 1 | 1 | `loyalty` |
| Lunch Locations | `lunch.location` | Lunch Ordering | 1 | 1 | `lunch` |
| Lunch Product Category | `lunch.product.category` | Lunch Ordering | 4 | 4 | `lunch` |
| Lunch Supplier | `lunch.supplier` | Lunch Ordering | 1 | 1 | `lunch` |
| Mailing Contact | `mailing.contact` | Marketing and Mass Mailing | 1 | 1 | `mass_mailing` |
| Mailing List | `mailing.list` | Marketing and Mass Mailing | 1 | 1 | `mass_mailing` |
| Mailing List Subscription | `mailing.subscription` | Marketing and Mass Mailing | 1 | 1 | `mass_mailing` |
| Mailing Subscription Reason | `mailing.subscription.optout` | Marketing and Mass Mailing | 5 | 5 | `mass_mailing` |
| Maintenance Stage | `maintenance.stage` | Repair and Maintenance | 4 | 4 | `maintenance` |
| Maintenance Teams | `maintenance.team` | Repair and Maintenance | 1 | 1 | `maintenance` |
| manufacturing Workorder productivity losses | `mrp.workcenter.productivity.loss.type` | Manufacturing | 4 | 4 | `mrp` |
| Marketing Card Template | `card.template` | Marketing and Mass Mailing | 14 | 14 | `marketing_card` |
| Message | `mail.message` | Messaging and Activities | 1 | 1 | `mail` |
| Module | `ir.module.module` | Platform Foundation | 21 | 21 | `base` |
| OAuth2 provider | `auth.oauth.provider` | Identity and Access | 3 | 3 | `auth_oauth` |
| Onboarding | `onboarding.onboarding` | Automation and Integration | 1 | 1 | `account` |
| Onboarding Step | `onboarding.onboarding.step` | Automation and Integration | 5 | 5 | `account` |
| Opp. Lost Reason | `crm.lost.reason` | Customer Relationship Management | 3 | 3 | `crm` |
| Overtime Rule | `hr.attendance.overtime.rule` | Attendances and Working Time | 6 | 6 | `hr_attendance` |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | Attendances and Working Time | 2 | 2 | `hr_attendance` |
| Page | `website.page` | Website and Storefront | 5 | 5 | `website`, `website_hr_recruitment`, `website_project` |
| Paper Format Config | `report.paperformat` | Platform Foundation | 19 | 18 | `account`, `base`, `event`, `hr`, `hr_holidays`, `hr_skills`, `l10n_ch`, `l10n_din5008`, `l10n_in_ewaybill`, `l10n_sa`, `pos_self_order`, `product`, `survey`, `website_event_exhibitor` |
| Partner Activation | `res.partner.activation` | Customer Relationship Management | 3 | 3 | `website_crm_partner_assign` |
| Partner Grade | `res.partner.grade` | Customer Relationship Management | 3 | 3 | `partnership` |
| Partner Tags | `res.partner.category` | Contacts and Organizations | 21 | 21 | `l10n_tr_nilvera_einvoice` |
| Payment Method | `payment.method` | Payment Providers | 237 | 233 | `delivery`, `l10n_ec_sale`, `payment`, `payment_custom`, `payment_demo`, `website_sale_collect` |
| Payment Methods | `account.payment.method` | General Ledger | 8 | 8 | `account`, `account_check_printing`, `l10n_latam_check` |
| Payment Provider | `payment.provider` | Payment Providers | 49 | 26 | `delivery`, `payment`, `payment_adyen`, `payment_aps`, `payment_asiapay`, `payment_authorize`, `payment_buckaroo`, `payment_custom`, `payment_demo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_redsys`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `website_sale_collect` |
| Payment Terms | `account.payment.term` | General Ledger | 10 | 10 | `account` |
| Peppol clarifications used for rejection | `account.peppol.clarification` | Electronic Invoicing and Document Exchange | 21 | 21 | `account_peppol_response` |
| Picking Type | `stock.picking.type` | Inventory Operations | 1 | 1 | `repair` |
| Point of Sale Category | `pos.category` | Point of Sale | 1 | 1 | `pos_event` |
| Point of Sale Configuration | `pos.config` | Point of Sale | 1 | 1 | `pos_sms` |
| Point of Sale Note | `pos.note` | Point of Sale | 4 | 4 | `point_of_sale` |
| Post Closing Reason | `forum.post.reason` | Website and Storefront | 13 | 13 | `website_forum` |
| Product Category | `product.category` | Products and Catalog | 6 | 6 | `delivery`, `event_product`, `point_of_sale`, `product` |
| Product ribbon | `product.ribbon` | Website and Storefront | 4 | 4 | `website_sale` |
| Product Unit of Measure | `uom.uom` | Units of Measure and Packaging | 121 | 56 | `hr_expense`, `hr_timesheet`, `l10n_ar`, `l10n_cl`, `l10n_in`, `l10n_mx`, `l10n_tr_nilvera`, `l10n_us_account`, `point_of_sale`, `uom` |
| Product Variant | `product.product` | Products and Catalog | 31 | 25 | `delivery`, `delivery_mondialrelay`, `event_booth_sale`, `event_product`, `event_sale`, `hr_expense`, `l10n_cl`, `l10n_es`, `loyalty`, `point_of_sale`, `pos_discount`, `pos_event`, `pos_loyalty`, `pos_sale`, `sale_gelato`, `sale_loyalty`, `sale_timesheet`, `website_sale_collect`, `website_sale_slides` |
| Project Stage | `project.project.stage` | Projects and Tasks | 4 | 4 | `project` |
| Rank based on karma | `gamification.karma.rank` | Learning, Surveys and Gamification | 5 | 5 | `gamification` |
| Recruitment Stages | `hr.recruitment.stage` | Recruitment | 6 | 6 | `hr_recruitment` |
| Refuse Reason of Applicant | `hr.applicant.refuse.reason` | Recruitment | 6 | 6 | `hr_recruitment` |
| Removal Strategy | `product.removal` | Inventory Operations | 5 | 5 | `product_expiry`, `stock` |
| Report Layout | `report.layout` | Platform Foundation | 8 | 8 | `l10n_din5008`, `web` |
| Resource Working Time | `resource.calendar` | Attendances and Working Time | 2 | 2 | `pos_restaurant`, `resource` |
| Salary Structure Type | `hr.payroll.structure.type` | Human Resources Core | 4 | 4 | `hr` |
| Sales Team | `crm.team` | Sales | 9 | 4 | `crm`, `pos_sale`, `pos_self_order_sale`, `sales_team`, `website_sale` |
| Sales Team Member | `crm.team.member` | Sales | 1 | 1 | `sales_team` |
| Shipping Methods | `delivery.carrier` | Delivery and Shipping | 10 | 7 | `delivery`, `delivery_mondialrelay`, `sale_gelato`, `website_sale`, `website_sale_collect`, `website_sale_gelato` |
| Skill | `hr.skill` | Human Resources Core | 36 | 36 | `hr_skills` |
| Skill Level | `hr.skill.level` | Human Resources Core | 11 | 11 | `hr_skills` |
| Skill Type | `hr.skill.type` | Human Resources Core | 2 | 2 | `hr_skills` |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | Spreadsheets and Dashboards | 14 | 14 | `spreadsheet_dashboard_account`, `spreadsheet_dashboard_event_sale`, `spreadsheet_dashboard_hr_expense`, `spreadsheet_dashboard_hr_timesheet`, `spreadsheet_dashboard_im_livechat`, `spreadsheet_dashboard_pos_hr`, `spreadsheet_dashboard_pos_restaurant`, `spreadsheet_dashboard_sale`, `spreadsheet_dashboard_sale_timesheet`, `spreadsheet_dashboard_stock_account`, `spreadsheet_dashboard_website_sale`, `spreadsheet_dashboard_website_sale_slides` |
| Task Stage | `project.task.type` | Projects and Tasks | 1 | 1 | `hr_timesheet` |
| text message Templates | `sms.template` | Messaging and Activities | 6 | 6 | `calendar_sms`, `event_sms`, `hr_presence`, `pos_sms`, `stock_sms` |
| Time Off Type | `hr.leave.type` | Time Off | 132 | 73 | `hr_holidays`, `hr_holidays_attendance`, `hr_work_entry_holidays` |
| Track Karma Changes | `gamification.karma.tracking` | Learning, Surveys and Gamification | 2 | 2 | `gamification` |
| Type of a resume line | `hr.resume.line.type` | Human Resources Core | 4 | 4 | `hr_skills`, `hr_skills_survey` |
| User | `res.users` | Identity and Access | 8 | 4 | `base`, `mail`, `mail_bot`, `product` |
| User Settings | `res.users.settings` | Identity and Access | 1 | 1 | `base` |
| Vehicle Status | `fleet.vehicle.state` | Fleet | 4 | 4 | `fleet` |
| Warehouse | `stock.warehouse` | Inventory Operations | 4 | 1 | `mrp`, `purchase_stock`, `repair`, `stock` |
| Website | `website` | Website and Storefront | 4 | 1 | `website`, `website_livechat`, `website_sale`, `website_slides` |
| Website Checkout Step | `website.checkout.step` | Website and Storefront | 5 | 5 | `l10n_tw_edi_ecpay_website_sale`, `website_sale` |
| Website Configurator Feature | `website.configurator.feature` | Website and Storefront | 13 | 13 | `website` |
| Website Menu | `website.menu` | Website and Storefront | 9 | 9 | `website`, `website_blog`, `website_event`, `website_forum`, `website_hr_recruitment`, `website_sale`, `website_slides` |
| Website Snippet Filter | `website.snippet.filter` | Website and Storefront | 11 | 11 | `website_blog`, `website_event`, `website_sale` |
| Work Location | `hr.work.location` | Human Resources Core | 3 | 3 | `hr` |
| Workcenter Productivity Losses | `mrp.workcenter.productivity.loss` | Manufacturing | 7 | 7 | `mrp` |

## 17. Tabular data files

69 tabular files ship 41,336 rows. A tabular file is used where the set is large and flat: subdivisions, cities, banks, postal-code ranges, classification codes and tax offices. The rows are loaded through the generic import operation, so every rule of [`data-loading-and-exchange.md`](data-loading-and-exchange.md), section 6, applies to them, including the resolution of a link by external identifier and the refusal of a row whose link cannot be resolved.

| Package | Record set | Transport name | Rows |
|---|---|---|---|
| `base` | Country state | `res.country.state` | 2,131 |
| `base` | Languages | `res.lang` | 93 |
| `crm_iap_mine` | customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | 23 |
| `crm_iap_mine` | People Role | `crm.iap.lead.role` | 22 |
| `crm_iap_mine` | People Seniority | `crm.iap.lead.seniority` | 3 |
| `l10n_ae` | Bank | `res.bank` | 174 |
| `l10n_ar` | Latam Document Type | `l10n_latam.document.type` | 146 |
| `l10n_ar` | Country | `res.country` | 228 |
| `l10n_ar` | Currency | `res.currency` | 43 |
| `l10n_at` | Account Tag | `account.account.tag` | 950 |
| `l10n_au` | Account Tag | `account.account.tag` | 1 |
| `l10n_bd` | Account Tag | `account.account.tag` | 3 |
| `l10n_bh` | Country state | `res.country.state` | 4 |
| `l10n_br` | Brazilian city zip range | `l10n_br.zip.range` | 5,573 |
| `l10n_br` | Latam Document Type | `l10n_latam.document.type` | 31 |
| `l10n_br` | Identification Types | `l10n_latam.identification.type` | 2 |
| `l10n_br` | City | `res.city` | 5,570 |
| `l10n_cl` | Latam Document Type | `l10n_latam.document.type` | 58 |
| `l10n_cl` | Bank | `res.bank` | 32 |
| `l10n_cl` | Country | `res.country` | 167 |
| `l10n_cl` | Currency | `res.currency` | 27 |
| `l10n_co` | Identification Types | `l10n_latam.identification.type` | 13 |
| `l10n_cz` | Tax office in Czech Republic | `l10n_cz.tax_office` | 214 |
| `l10n_dk` | Account Tag | `account.account.tag` | 489 |
| `l10n_ec` | SRI Payment Method | `l10n_ec.sri.payment` | 8 |
| `l10n_ec` | Latam Document Type | `l10n_latam.document.type` | 37 |
| `l10n_ec` | Bank | `res.bank` | 459 |
| `l10n_eg_edi_eta` | Estimated Time of Arrival code for activity type | `l10n_eg_edi.activity.type` | 435 |
| `l10n_eg_edi_eta` | Estimated Time of Arrival code for the unit of measures | `l10n_eg_edi.uom.code` | 81 |
| `l10n_eg_edi_eta` | Product Unit of Measure | `uom.uom` | 19 |
| `l10n_es` | Account Tag | `account.account.tag` | 8 |
| `l10n_es_edi_facturae` | Administrative Center Role Type | `l10n_es_edi_facturae.ac_role_type` | 9 |
| `l10n_es_edi_facturae` | Product Unit of Measure | `uom.uom` | 13 |
| `l10n_fr_account` | Bank | `res.bank` | 2,797 |
| `l10n_hr_edi` | Croatian KPD Category | `l10n_hr.kpd.category` | 3,358 |
| `l10n_hu` | Bank | `res.bank` | 15 |
| `l10n_hu_edi` | Product Unit of Measure | `uom.uom` | 9 |
| `l10n_id_efaktur_coretax` | Product categorization according to E-Faktur | `l10n_id_efaktur_coretax.product.code` | 1,967 |
| `l10n_id_efaktur_coretax` | unit of measure categorization according to E-Faktur | `l10n_id_efaktur_coretax.uom.code` | 39 |
| `l10n_id_efaktur_coretax` | Product Unit of Measure | `uom.uom` | 13 |
| `l10n_ie` | Account Tag | `account.account.tag` | 81 |
| `l10n_in` | Account Tag | `account.account.tag` | 15 |
| `l10n_in` | Indian port code | `l10n_in.port.code` | 634 |
| `l10n_in` | indian section alert | `l10n_in.section.alert` | 87 |
| `l10n_it_edi` | Account Tag | `account.account.tag` | 1 |
| `l10n_it_edi` | Italian Document Type | `l10n_it.document.type` | 22 |
| `l10n_ke` | KRA defined codes that justify a given tax rate / exemption | `l10n_ke.item.code` | 284 |
| `l10n_latam_base` | Identification Types | `l10n_latam.identification.type` | 3 |
| `l10n_lb_account` | Country state | `res.country.state` | 16 |
| `l10n_lu` | Account Tag | `account.account.tag` | 60 |
| `l10n_lv` | Account Tag | `account.account.tag` | 32 |
| `l10n_mn` | Account Tag | `account.account.tag` | 36 |
| `l10n_mx` | Account Tag | `account.account.tag` | 2 |
| `l10n_my` | Account Tag | `account.account.tag` | 3 |
| `l10n_my_edi` | Malaysian Industry Classification | `l10n_my_edi.industry_classification` | 1,174 |
| `l10n_om` | Country state | `res.country.state` | 11 |
| `l10n_pe` | District | `l10n_pe.res.city.district` | 1,874 |
| `l10n_pe` | Bank | `res.bank` | 18 |
| `l10n_pe` | City | `res.city` | 196 |
| `l10n_pl` | Account Tag | `account.account.tag` | 9 |
| `l10n_pl` | Tax Office in Poland | `l10n_pl.l10n_pl_tax_office` | 400 |
| `l10n_ro` | Bank | `res.bank` | 44 |
| `l10n_ro_cpv_code` | CPV Code | `l10n_ro.cpv.code` | 9,454 |
| `l10n_se` | Account Tag | `account.account.tag` | 41 |
| `l10n_tr_nilvera_einvoice_extended` | Turkish Tax Office | `l10n_tr_nilvera_einvoice_extended.tax.office` | 1,046 |
| `l10n_tw` | City | `res.city` | 371 |
| `l10n_uy` | Latam Document Type | `l10n_latam.document.type` | 51 |
| `l10n_uz` | Account Tag | `account.account.tag` | 91 |
| `l10n_za` | Account Tag | `account.account.tag` | 16 |

## 18. Other configuration records a fresh installation carries

Beyond the sets above, the operational catalogues hold configuration records that are also shipped and also keyed by external identifier:

| Set | Records | Purpose |
|---|---|---|
| Access rights | 1,933 | One row per entity and group, granting read, write, create and delete. |
| Record rules | 576 | One row per entity and group, restricting which records that group may act on. |
| Scheduled jobs | 93 | One row per recurring task, with its interval, its next execution and the operation it invokes. |
| System parameters | 28 | One row per installation-wide setting, keyed by name. |
| Package categories | 28 | The tree by which packages are grouped in the application list. |
| Report layouts | 8 | The document layouts a company may choose. |
| Paper formats | 19 | The page sizes and margins a printable document may use. |
| Removal strategies | 5 | The rules by which stock is picked from a location: first in first out, last in first out, first expiry first out, closest location and least packages. |
| Barcode nomenclatures | 2, with 39 rules | The two barcode grammars: the default one and the international article-numbering one. |
| Geolocation providers | 2 | The two address-to-coordinates services a tenant may choose. |
| Delegated-authorization providers | 3 | The three external sign-in services a tenant may enable, each shipped with its authorization address, its scope and its validation address, and each disabled until a tenant supplies its own client identifier. |

## 19. Totals

| Measure | Count |
|---|---|
| Record sets declared through declaration documents | 155 |
| Declarations in those documents | 19,008 |
| Distinct records they produce | 18,737 |
| Tabular data files | 69 |
| Rows in tabular data files | 41,336 |
| Packages shipping country accounting templates | 127 |
| Distinct charts | 151 |
| Rows in country accounting templates | 66,368 |
| Countries | 251 |
| Country subdivisions | 2,201 |
| Cities | 8,987 |
| Country groups | 19 |
| Banks | 3,654 |
| Currencies | 173 |
| Languages | 93 |
| Units of measure | 56 |
| Decimal precisions | 7 |
| Industries | 23 |
| Payment terms | 10 |
| Payment methods of the ledger | 8 |
| Payment methods of the online checkout | 237 |
| Payment providers | 49 |
| Delivery terms | 15 |
| Account and tax tags declared | 517 |
| Account and tax tag rows in tabular files | 1,838 |
| Statements | 165 |
| Statement columns | 275 |
| Statement lines | 6,076 |
| Statement expressions | 6,555 |
| Activity types | 16 |
| Message subtypes | 103 |
| Numbering sequences | 16 |
| Privileges | 29 |
| Access groups | 140 |
| Access rights | 1,933 |
| Record rules | 576 |
| Scheduled jobs | 93 |
| System parameters | 28 |

## 20. Reconciliation notes

The target branch carried no file for this topic and the working branch carried none either: the material is assembled from the generated summary [`../references/reference-data.md`](../references/reference-data.md) and the catalogues under [`../../schemas/data/reference-data/`](../../schemas/data/reference-data/) and [`../../schemas/operational/`](../../schemas/operational/), and checked against the source tree. The following points were resolved while assembling it.

| Point | Situation | Resolution |
|---|---|---|
| Two different counts for the same set | The generated summary counts declarations, and several packages declare the same record in order to add a field to it. | Both numbers are given: the declaration count and the distinct record count. For countries the two are 261 and 251; for currencies 186 and 173; for units of measure 121 and 56. A rebuild must ship the distinct count and must accept the extra declarations as updates. |
| A set that also arrives through a tabular file | Subdivisions, languages, countries, currencies, banks, cities and units of measure appear both as declarations and as tabular rows. | Both routes are stated per set, with the count of each, because they are loaded by different code paths and a rebuild has to implement both. Tabular rows that name an existing external identifier update the record rather than creating a second one. |
| A partner title set | The brief for this document asked for shipped partner titles. | There is no title entity in this system and therefore no shipped title set. Section 8.2 states that explicitly, so that a rebuild does not create a table waiting for records that never arrive. |
| A unit of measure category set | The brief asked for units of measure and their categories. | There is no unit category entity. A unit belongs to a family by pointing at a reference unit and stating a factor, and the family is the resulting tree. Section 6 states the rule and the worked conversion. |
| Exchange rates | Currencies are shipped; rates are not. | Stated explicitly in section 4, because a rebuild that assumes a shipped rate table will silently convert at a rate of one. |
| Country accounting templates | The summary lists them as ordinary record sets. | They are template rows, not records: they are loaded into a company's chart when the chart is selected, not at installation of the package. Section 11 states the difference and gives the per-package and per-kind counts. |
| Strings inside the catalogue that carry product identity | A few shipped values in the catalogue — an electronic-mail address of the automated user, the name and the addresses of one delegated-authorization provider — were rewritten when the catalogue was generated and are no longer valid strings. | They are described by their function in sections 2 and 18 rather than quoted, because quoting a rewritten string would give a rebuild a value that cannot work. The valid third-party addresses of the other two providers are contractual and are held in the catalogue. |
