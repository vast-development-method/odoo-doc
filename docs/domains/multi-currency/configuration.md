# Multi-Currency — Configuration

Every setting, parameter, default record, master data prerequisite, permission group, menu and
piece of scheduled work this domain needs, with its type, its default and its effect. The shipped
currency catalogue and the shipped demonstration rate table are published here in full, because
they are reference data rather than arithmetic; [calculations.md](calculations.md) keeps only the
summary of them that its formulas need.

Contents:

1. [Company settings](#1-company-settings)
2. [The automatic rate retrieval capability](#2-the-automatic-rate-retrieval-capability)
3. [Master data prerequisites](#3-master-data-prerequisites)
4. [The shipped currency catalogue](#4-the-shipped-currency-catalogue)
5. [The decimal precision registry](#5-the-decimal-precision-registry)
6. [Journal and account configuration](#6-journal-and-account-configuration)
7. [Permission groups and the access matrix](#7-permission-groups-and-the-access-matrix)
8. [Menus and navigation](#8-menus-and-navigation)
9. [Scheduled work](#9-scheduled-work)
10. [Configuration checklist](#10-configuration-checklist)
11. [Reconciliation notes](#11-reconciliation-notes)

---

# 1. Company settings

These values live on the Company record and are surfaced on the accounting settings screen. They
are company-dependent: a platform holding several companies holds a separate value for each.

## 1.1 The Currencies block

| Setting label | Field | Type | Default | Effect |
|---|---|---|---|---|
| Main Currency | `currency_id` (main currency) on Company | Required link to Currency | The main currency of the company of the user creating the company; proposed from the company's country when a country is chosen | The currency in which the balance, the debit and the credit of every journal item of the company are expressed, and the denominator of every rate an accountant reads. Governed by `MCUR-170` to `MCUR-173`. The selector lists archived currencies as well, because choosing one activates it (`MCUR-009`). |
| Currencies | Not a field; a control that opens the currency catalogue | Navigation | Not applicable | Opens the currency catalogue with archived records included. |
| Automatic Currency Rates | `module_currency_rate_live` (automatic currency rates) on the settings screen | True or false | Off | Installs or removes the capability package that fetches rates from an external rate service. Shown only to holders of the multi-currency permission group. |

## 1.2 The Default Accounts block, exchange difference entries

Shown only to holders of the multi-currency permission group, and only within the part of the
settings screen reserved for holders of the accounting user group. Its heading is "Exchange
difference entries:".

| Setting label | Field | Type | Default | Effect |
|---|---|---|---|---|
| Journal | `currency_exchange_journal_id` (exchange difference journal) on Company | Link to Journal, restricted to journals whose type is general | Set by the chart of accounts template | The journal in which every realised exchange difference entry is created. Missing: `MCUR-102`. |
| Gain | `income_currency_exchange_account_id` (gain exchange account) on Company | Link to Account, restricted to accounts whose internal group is income | Set by the chart of accounts template | Credited when a rate movement produces a gain. Missing: `MCUR-104`. |
| Loss | `expense_currency_exchange_account_id` (loss exchange account) on Company | Link to Account, restricted to accounts whose type is expense or other expense | Set by the chart of accounts template | Debited when a rate movement produces a loss. Missing: `MCUR-103`. |

## 1.3 The Customer Invoices block, currency presentation

| Setting label | Field | Type | Default | Effect |
|---|---|---|---|---|
| Display the total amount of an invoice in letters | `display_invoice_amount_total_words` (display invoice total in words) on Company | True or false | Off | When on, the printed invoice shows the document total spelled out by the algorithm of [calculations.md](calculations.md) section 25, using the document currency's unit and subunit labels. |
| Taxes are also displayed in local currency on invoices | `display_invoice_tax_company_currency` (display invoice tax in company currency) on Company | True or false | On | When on, the printed invoice shows every tax amount converted into the company currency in addition to the document currency. |

## 1.4 The settings screen fields

The settings screen is a transient record whose currency-related fields are bound to the values
above: `currency_id`, `currency_exchange_journal_id`, `income_currency_exchange_account_id`,
`expense_currency_exchange_account_id`, `display_invoice_tax_company_currency`,
`module_currency_rate_live`, and the read-only helper `group_multi_currency` (multi-currency group
held), which exists only so that the blocks above can be shown or hidden. The full list is in
[entities.md](entities.md) section 13.

---

# 2. The automatic rate retrieval capability

Fetching rates from an external service is an optional capability package that the setting
`module_currency_rate_live` installs. The platform without it keeps its rate table by hand
(workflow 2 of [workflows.md](workflows.md)); with it, the scheduled job of section 9 writes rate
rows automatically (workflow 3).

**Industry-standard default.** The reviewed material settles that the setting installs the
capability and that the capability writes ordinary Currency Rate rows subject to the same three
guards as a manual entry. It does not settle the list of services the package offers or the exact
wording of its labels. The contract below is therefore stated as the industry-standard resolution:
a replacement may offer any set of services behind it, and must keep the four settings and the
service contract, because the rest of this domain depends on them.

| Setting | Type | Default | Effect |
|---|---|---|---|
| Service | A selection over the rate services the platform can reach | A public central-bank reference rate service | Chooses which external service is queried. Each service publishes a mapping from currency code to a value expressed as units of that currency per one unit of a base currency; the package translates that answer into the company rate of `MCUR-024`. |
| Interval | A selection over the closed list: manually, daily, weekly, monthly | Manually | How often the scheduled job fetches rates for this company. Manually disables the scheduled fetch entirely; only the "Update now" control then writes rates. |
| Next Run | A date | Today, when the interval is first set to a value other than manually | The next moment at which the scheduled job fetches rates for this company. Advanced by the interval after each scheduled run, and **not** advanced by an interactive "Update now". |
| Update now | Not a field; a control next to Next Run | Not applicable | Fetches rates for the company immediately. |

**The rate service contract.** A rate service is an interface with a single operation: given a base
currency code, a set of currency codes and a date, it returns a mapping from currency code to a
decimal value meaning "units of this currency per one unit of the base currency". A service that
cannot answer for a code omits it, and the currency concerned keeps its previous rate. A service
failure leaves the rate table untouched, is logged, and is not retried within the same run.

---

# 3. Master data prerequisites

| Prerequisite | Owned by | Why this domain needs it |
|---|---|---|
| The currency catalogue | This domain, shipped as reference data | Without a Currency record no monetary amount can be rounded, compared or displayed. |
| At least one Company with a main currency | [../contacts-and-organizations/README.md](../contacts-and-organizations/README.md) | Every conversion resolves against a company, and every rate row against a root company. |
| At least one Language | [../contacts-and-organizations/README.md](../contacts-and-organizations/README.md) | The decimal separator, the thousands separator and the grouping pattern of a formatted amount come from the reader's language. |
| A journal whose type is general | [../general-ledger/README.md](../general-ledger/README.md) | Required as the exchange difference journal. |
| An income account and an expense account | [../general-ledger/README.md](../general-ledger/README.md) | Required as the gain and the loss exchange accounts. |
| A chart of accounts | [../general-ledger/README.md](../general-ledger/README.md) | Loading a chart of accounts template pre-fills the exchange journal and the two exchange accounts, and activates the currency of its country. |
| One price list per active currency | [../pricing-and-pricelists/README.md](../pricing-and-pricelists/README.md) | Recommended; a default price list is created automatically for every company when the multi-currency capability is first granted (`MCUR-008`). |

---

# 4. The shipped currency catalogue

One hundred and seventy Currency records are shipped as reference data, covering the alphabetic and
the numeric codes of the international currency-code standard. They are loaded with the flag that
prevents a later upgrade from overwriting them, so an accountant's local edits are never lost.

| Property | Shipped value |
|---|---|
| `active` (activity flag) | False for **every** shipped currency. A platform therefore starts with no active currency and becomes multi-currency only deliberately (`MCUR-018`). |
| `name` (currency code) | The three-letter alphabetic code. |
| `iso_numeric` (international standard numeric code) | The numeric code, stored as a whole number. The shipped data writes it with leading zeros where the standard does, so the value written as `032` is stored as thirty-two; the leading zeros are presentational. |
| `full_name` (full name) | The currency's name in words. |
| `symbol` (symbol) | The customary sign. |
| `position` (symbol position) | `after` unless the shipped record states `before`. |
| `rounding` (rounding factor) | One hundredth unless the shipped record states otherwise. |
| `currency_unit_label`, `currency_subunit_label` (currency unit label, currency subunit label) | The customary unit and subunit names. |
| `decimal_places` (decimal places) | Never shipped: derived from the rounding factor on every write, by the formula of [calculations.md](calculations.md) section 2. |

## 4.1 Currencies whose rounding factor is not one hundredth

| Rounding factor | Derived decimal places | Number of shipped currencies | Codes |
|---|---|---|---|
| One hundredth | 2 | 142 | Every shipped currency that does not appear in one of the three rows below. The complete list, with every other shipped property, is section 4.2. |
| One | 0 | 18 | `XPF` the CFP franc, `VND` the Vietnamese dong, `JPY` the Japanese yen, `KRW` the South Korean won, `XOF` the West African CFA franc, `XAF` the Central African CFA franc, `UGX` the Ugandan shilling, `CLP` the Chilean peso, `BYR` the Belarusian ruble of the older denomination, `BIF` the Burundian franc, `KMF` the Comorian franc, `DJF` the Djiboutian franc, `GNF` the Guinean franc, `ISK` the Icelandic krona, `VUV` the Vanuatu vatu, `TWD` the New Taiwan dollar, `RWF` the Rwandan franc and `PYG` the Paraguayan guarani. Fourteen of the eighteen declare the factor as `1` and four — `XPF`, `VND`, `JPY` and `TWD` — declare it as `1.00`, which is numerically identical and yields the same zero decimal places. |
| One thousandth | 3 | 7 | `BHD` the Bahraini dinar, `IQD` the Iraqi dinar, `JOD` the Jordanian dinar, `KWD` the Kuwaiti dinar, `LYD` the Libyan dinar, `TND` the Tunisian dinar and `OMR` the Omani rial. |
| One ten-thousandth | 4 | 3 | `CLF` the Chilean unit of account known as the development unit, `UYW` the Uruguayan indexed pension unit and `UYI` the Uruguayan peso in indexed units. |

A replacement that hard-codes two decimal places anywhere produces wrong amounts for twenty-eight of
the shipped currencies.

## 4.2 The complete shipped catalogue

The one hundred and seventy shipped Currency records, in the order in which they are loaded. Every
one of them carries an activity flag of false, so that column is not repeated. A cell reading
"none" means the shipped record leaves that field empty. The symbol position column carries the
value the record states, with `after` shown where the record states none, because `after` is the
field default. The decimal places column is derived from the rounding factor and is shown for
completeness; it is not independent data. Loading this table, with the activity flag false on every
row, reproduces the reference catalogue exactly.

| Code | Numeric code | Full name | Symbol | Symbol position | Rounding factor | Decimal places | Unit label | Subunit label |
|---|---|---|---|---|---|---|---|---|
| `USD` | 840 | United States dollar | `$` | `before` | `0.01` | 2 | Dollars | Cents |
| `VEF` | 937 | Venezuelan bolívar fuerte | `Bs.F` | `after` | `0.01` | 2 | Bolivar | Centimos |
| `CAD` | 124 | Canadian dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `CHF` | 756 | Swiss franc | `CHF` | `before` | `0.01` | 2 | Franc | Centimes |
| `BRL` | 986 | Brazilian real | `R$` | `before` | `0.01` | 2 | Real | Centavos |
| `CNY` | 156 | Chinese yuan | `¥` | `before` | `0.01` | 2 | Yuan | Fen |
| `CNH` | none | Chinese yuan - Offshore | `¥` | `before` | `0.01` | 2 | Yuan | Fen |
| `COP` | 170 | Colombian peso | `$` | `before` | `0.01` | 2 | Peso | Centavos |
| `CZK` | 203 | Czech koruna | `Kč` | `after` | `0.01` | 2 | Koruna | Halers |
| `DKK` | 208 | Danish krone | `kr` | `before` | `0.01` | 2 | Krone | Ore |
| `HUF` | 348 | Hungarian forint | `Ft` | `after` | `0.01` | 2 | Forint | Filler |
| `IDR` | 360 | Indonesian rupiah | `Rp` | `before` | `0.01` | 2 | Rupiah | Sen |
| `LVL` | 840 | Latvian lats | `Ls` | `after` | `0.01` | 2 | Lats | Santims |
| `NOK` | 578 | Norwegian krone | `kr` | `before` | `0.01` | 2 | Krone | Ore |
| `XPF` | 953 | CFP franc | `XPF` | `after` | `1.00` | 0 | Franc | Centimes |
| `PAB` | 590 | Panamanian balboa | `B/.` | `after` | `0.01` | 2 | Balboa | Centesimo |
| `PLN` | 985 | Polish złoty | `zł` | `after` | `0.01` | 2 | Zloty | Groszy |
| `SEK` | 752 | Swedish krona | `kr` | `after` | `0.01` | 2 | Krona | Ore |
| `ARS` | 032 | Argentine peso | `$` | `before` | `0.01` | 2 | Peso | Centavos |
| `INR` | 356 | Indian rupee | `₹` | `before` | `0.01` | 2 | Rupees | Paise |
| `AUD` | 036 | Australian dollar | `$` | `before` | `0.01` | 2 | Dollars | Cents |
| `UAH` | 980 | Ukraine Hryvnia | `₴` | `after` | `0.01` | 2 | Hryvnia | Kopiyka |
| `VND` | 704 | Vietnamese đồng | `₫` | `after` | `1.00` | 0 | Dong | Xu |
| `HKD` | 344 | Hong Kong dollar | `$` | `before` | `0.01` | 2 | Dollars | Cents |
| `JPY` | 392 | Japanese yen | `¥` | `before` | `1.00` | 0 | Yen | Cen |
| `BGN` | 975 | Bulgarian lev | `лв` | `after` | `0.01` | 2 | Lev | Stotinki |
| `LTL` | 840 | Lithuanian litas | `Lt` | `after` | `0.01` | 2 | Litas | Centas |
| `RON` | 946 | Romanian leu | `lei` | `after` | `0.01` | 2 | Leu | Bani |
| `HRK` | 191 | Croatian kuna | `kn` | `after` | `0.01` | 2 | Kuna | Lipa |
| `RUB` | 643 | Russian ruble | `руб` | `after` | `0.01` | 2 | Ruble | Kopek |
| `TRY` | 949 | Turkish lira | `₺` | `after` | `0.01` | 2 | Lira | Kurus |
| `KRW` | 410 | South Korean won | `₩` | `before` | `1` | 0 | Won | Chon |
| `MXN` | 484 | Mexican peso | `$` | `before` | `0.01` | 2 | Pesos | Centavos |
| `MYR` | 458 | Malaysian ringgit | `RM` | `before` | `0.01` | 2 | Ringgit | Sen |
| `NZD` | 554 | New Zealand dollar | `$` | `before` | `0.01` | 2 | Dollars | Cents |
| `PHP` | 608 | Philippine peso | `₱` | `after` | `0.01` | 2 | Peso | Centavos |
| `SGD` | 702 | Singapore dollar | `S$` | `before` | `0.01` | 2 | Dollars | Cents |
| `ZAR` | 710 | South African rand | `R` | `before` | `0.01` | 2 | Rand | Cents |
| `CRC` | 188 | Costa Rican colón | `₡` | `before` | `0.01` | 2 | Colon | Centimos |
| `MUR` | 480 | Mauritian rupee | `Rs` | `after` | `0.01` | 2 | Rupee | Cents |
| `XOF` | 952 | CFA franc BCEAO | `CFA` | `after` | `1` | 0 | Franc | Centimes |
| `XAF` | 950 | CFA franc BEAC | `FCFA` | `after` | `1` | 0 | Franc | Centimes |
| `UGX` | 800 | Ugandan shilling | `USh` | `after` | `1` | 0 | Shilling | Cents |
| `HNL` | 340 | Honduran lempira | `L` | `before` | `0.01` | 2 | Lempiras | Centavos |
| `CLP` | 152 | Chilean peso | `$` | `before` | `1` | 0 | Peso | Centavos |
| `UYU` | 858 | Uruguayan peso | `$` | `after` | `0.01` | 2 | Peso | Centesimos |
| `AFN` | 971 | Afghan afghani | `Afs` | `after` | `0.01` | 2 | Afghani | Puls |
| `AOA` | 973 | Angolan kwanza | `Kz` | `after` | `0.01` | 2 | Kwanza | Centimos |
| `XCD` | 951 | East Caribbean dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `AMD` | 051 | Armenian dram | `դր.` | `after` | `0.01` | 2 | Dram | Luma |
| `AWG` | 533 | Aruban florin | `Afl.` | `after` | `0.01` | 2 | Guilder | Cents |
| `AZN` | 944 | Azerbaijani manat | `₼` | `after` | `0.01` | 2 | Manat | Qapik |
| `BSD` | 044 | Bahamian dollar | `B$` | `after` | `0.01` | 2 | Dollars | Cents |
| `BHD` | 048 | Bahraini dinar | `BD` | `after` | `0.001` | 3 | Dinar | Fils |
| `BDT` | 050 | Bangladeshi taka | `৳` | `after` | `0.01` | 2 | Taka | Paisa |
| `BBD` | 052 | Barbados dollar | `Bds$` | `after` | `0.01` | 2 | Dollars | Cents |
| `BYR` | 974 | Belarusian ruble | `BR` | `after` | `1` | 0 | Ruble BYR | Kapeyka |
| `BYN` | 974 | Belarusian ruble | `Br` | `after` | `0.01` | 2 | Rubles | Kopeks |
| `BZD` | 084 | Belize dollar | `BZ$` | `after` | `0.01` | 2 | Dollars | Cents |
| `BMD` | 060 | Bermudian dollar | `BD$` | `after` | `0.01` | 2 | Dollars | Cents |
| `BTN` | 064 | Bhutanese ngultrum | `Nu.` | `after` | `0.01` | 2 | Ngultrum | Chhertum |
| `BOB` | 068 | Boliviano | `Bs.` | `after` | `0.01` | 2 | Boliviano | Centavos |
| `BAM` | 977 | Bosnia and Herzegovina convertible mark | `KM` | `after` | `0.01` | 2 | Mark | Fening |
| `BWP` | 072 | Botswana pula | `P` | `after` | `0.01` | 2 | Pula | Thebe |
| `BIF` | 108 | Burundian franc | `FBu` | `after` | `1` | 0 | Franc | Centime |
| `KHR` | 116 | Cambodian riel | `៛` | `after` | `0.01` | 2 | Riel | Sen |
| `KYD` | 136 | Cayman Islands dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `KMF` | 174 | Comorian franc | `CF` | `after` | `1` | 0 | Franc | Centime |
| `CDF` | 976 | Congolese franc | `Fr` | `after` | `0.01` | 2 | Franc | Centime |
| `CUP` | 192 | Cuban peso | `$` | `after` | `0.01` | 2 | Peso | Centavos |
| `ANG` | 532 | Netherlands Antillean guilder | `ƒ` | `before` | `0.01` | 2 | Guilder | Cents |
| `XCG` | none | Caribbean Guilder | `Cg` | `after` | `0.01` | 2 | Guilder | Cents |
| `DJF` | 262 | Djiboutian franc | `Fdj` | `after` | `1` | 0 | Franc | Centime |
| `DOP` | 214 | Dominican peso | `RD$` | `before` | `0.01` | 2 | Pesos | Centavos |
| `EGP` | 818 | Egyptian pound | `LE` | `after` | `0.01` | 2 | Pound | Piastres |
| `SVC` | 222 | Salvadoran Colon | `¢` | `after` | `0.01` | 2 | Colones | Centavo |
| `ERN` | 232 | Eritrean nakfa | `Nfk` | `after` | `0.01` | 2 | Nakfa | Cents |
| `ETB` | 230 | Ethiopian birr | `Br` | `after` | `0.01` | 2 | Birr | Cents |
| `FKP` | 238 | Falkland Islands pound | `£` | `after` | `0.01` | 2 | Pound | Penny |
| `FJD` | 242 | Fiji dollar | `FJ$` | `after` | `0.01` | 2 | Dollars | Cents |
| `GEL` | 981 | Georgian lari | `ლ` | `after` | `0.01` | 2 | Lari | Tetri |
| `GIP` | 292 | Gibraltar pound | `£` | `after` | `0.01` | 2 | Pound | Penny |
| `GNF` | 324 | Guinean franc | `FG` | `after` | `1` | 0 | Franc | Centime |
| `GYD` | 328 | Guyanese dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `HTG` | 332 | Haitian gourde | `G` | `after` | `0.01` | 2 | Gourde | Centime |
| `ISK` | 352 | Icelandic króna | `kr` | `after` | `1` | 0 | Krona | Aurar |
| `IRR` | 364 | Iranian rial | `﷼` | `after` | `0.01` | 2 | Dinar | Fils |
| `IQD` | 368 | Iraqi dinar | `ع.د` | `after` | `0.001` | 3 | Dinar | Fils |
| `ILS` | 376 | Israeli new shekel | `₪` | `before` | `0.01` | 2 | Shekel | Agorot |
| `JMD` | 388 | Jamaican dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `JOD` | 400 | Jordanian dinar | `د.ا` | `after` | `0.001` | 3 | Dinar | Fils |
| `KZT` | 398 | Kazakhstani tenge | `₸` | `after` | `0.01` | 2 | Tenge | Tiin |
| `KES` | 404 | Kenyan shilling | `KSh` | `after` | `0.01` | 2 | Shilling | Cents |
| `KWD` | 414 | Kuwaiti dinar | `د.ك` | `after` | `0.001` | 3 | Dinar | Fils |
| `KGS` | 417 | Kyrgyzstani som | `лв` | `after` | `0.01` | 2 | Som | Tyiyn |
| `LAK` | 418 | Lao kip | `₭` | `after` | `0.01` | 2 | Kip | Att |
| `LBP` | 422 | Lebanese pound | `ل.ل` | `after` | `0.01` | 2 | Pound | Piastres |
| `LSL` | 426 | Lesotho loti | `M` | `after` | `0.01` | 2 | Loti | Sente |
| `LRD` | 430 | Liberian dollar | `L$` | `after` | `0.01` | 2 | Dollars | Cents |
| `LYD` | 434 | Libyan dinar | `ل.د` | `after` | `0.001` | 3 | Dinar | Dirham |
| `MOP` | 446 | Macanese pataca | `MOP$` | `after` | `0.01` | 2 | Pataca | Sin |
| `MKD` | 807 | Macedonian denar | `ден` | `after` | `0.01` | 2 | Denar | Deni |
| `MGA` | 969 | Malagasy ariary | `Ar` | `after` | `0.01` | 2 | Ariary | Iraimbilanja |
| `MWK` | 454 | Malawian kwacha | `MK` | `after` | `0.01` | 2 | Kwacha | Tambala |
| `MVR` | 462 | Maldivian rufiyaa | `Rf` | `before` | `0.01` | 2 | Rufiyaa | Laari |
| `MRO` | 478 | Mauritanian ouguiya (old) | `UM` | `after` | `0.01` | 2 | Ouguiya | Khoums |
| `MRU` | 478 | Mauritanian ouguiya | `UM` | `after` | `0.01` | 2 | Ouguiya | Khoums |
| `MDL` | 498 | Moldovan leu | `L` | `after` | `0.01` | 2 | Leu | Ban |
| `MNT` | 496 | Mongolian tögrög | `₮` | `after` | `0.01` | 2 | Tugrik | Mongo |
| `MAD` | 504 | Moroccan dirham | `DH` | `after` | `0.01` | 2 | Dirham | Centimes |
| `BND` | 096 | Brunei dollar | `B$` | `before` | `0.01` | 2 | Dollars | Cents |
| `DZD` | 012 | Algerian dinar | `DA` | `after` | `0.01` | 2 | Dinar | Centimes |
| `GHS` | 936 | Ghanaian cedi | `GH¢` | `after` | `0.01` | 2 | Cedi | Pesewas |
| `GMD` | 270 | Gambian dalasi | `D` | `after` | `0.01` | 2 | Dalasi | Butut |
| `MZN` | 943 | Mozambican metical | `MT` | `after` | `0.01` | 2 | Metical | Centavo |
| `MMK` | 104 | Myanmar kyat | `K` | `after` | `0.01` | 2 | Kyat | Pya |
| `NAD` | 516 | Namibian dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `NPR` | 524 | Nepalese rupee | `₨` | `after` | `0.01` | 2 | Rupee | Paisa |
| `ALL` | 008 | Albanian lek | `L` | `after` | `0.01` | 2 | Lek | Qindarke |
| `NIO` | 558 | Nicaraguan córdoba | `C$` | `after` | `0.01` | 2 | Cordoba | Centavos |
| `NGN` | 566 | Nigerian naira | `₦` | `after` | `0.01` | 2 | Naira | Kobo |
| `KPW` | 408 | North Korean won | `₩` | `after` | `0.01` | 2 | Won | Chon |
| `ZIG` | none | Zimbabwe Gold | `ZiG` | `after` | `0.01` | 2 | ZiGs | none |
| `ZMW` | 967 | Zambian kwacha | `ZK` | `after` | `0.01` | 2 | Kwacha | Ngwee |
| `YER` | 886 | Yemeni rial | `﷼` | `after` | `0.01` | 2 | Rial | Fils |
| `EUR` | 978 | Euro | `€` | `after` | `0.01` | 2 | Euros | Cents |
| `VUV` | 548 | Vanuatu vatu | `VT` | `after` | `1` | 0 | Vatu | none |
| `UZS` | 860 | Uzbekistan som | `лв` | `after` | `0.01` | 2 | Som | Tiyin |
| `AED` | 784 | United Arab Emirates dirham | `AED` | `after` | `0.01` | 2 | Dirham | Fils |
| `TMT` | 934 | Turkmenistan manat | `T` | `after` | `0.01` | 2 | Manat | Tenge |
| `TND` | 788 | Tunisian dinar | `DT` | `after` | `0.001` | 3 | Dinar | Millimes |
| `TTD` | 780 | Trinidad and Tobago dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `TOP` | 776 | Tongan paʻanga | `T$` | `after` | `0.01` | 2 | Paanga | Seniti |
| `THB` | 764 | Thai baht | `฿` | `after` | `0.01` | 2 | Baht | Satang |
| `TZS` | 834 | Tanzanian shilling | `TSh` | `after` | `0.01` | 2 | Shilling | Senti |
| `TJS` | 972 | Tajikistani somoni | `TJS` | `after` | `0.01` | 2 | Somoni | Diram |
| `TWD` | 901 | New Taiwan dollar | `NT$` | `after` | `1.00` | 0 | Dollars | Cents |
| `SYP` | 760 | Syrian pound | `£` | `after` | `0.01` | 2 | Pound | Piastrp |
| `SZL` | 748 | Swazi lilangeni | `E` | `after` | `0.01` | 2 | Lilangeni | Cents |
| `SRD` | 968 | Surinamese dollar | `$` | `after` | `0.01` | 2 | Dollars | Cents |
| `SDG` | 938 | Sudanese pound | `ج.س.` | `after` | `0.01` | 2 | none | none |
| `LKR` | 144 | Sri Lankan rupee | `Rs` | `after` | `0.01` | 2 | Rupee | Cents |
| `SSP` | 728 | South Sudanese pound | `£` | `after` | `0.01` | 2 | Pounds | Piasters |
| `GBP` | 826 | Pound sterling | `£` | `before` | `0.01` | 2 | Sterling | Penny |
| `SOS` | 706 | Somali shilling | `Sh.` | `after` | `0.01` | 2 | Shillings | Senti |
| `SBD` | 090 | Solomon Islands dollar | `SI$` | `after` | `0.01` | 2 | Dollars | Cents |
| `SLL` | 694 | Sierra Leonean leone | `Le` | `after` | `0.01` | 2 | Leone | Cents |
| `SLE` | 694 | Sierra Leonean leone | `Le` | `after` | `0.01` | 2 | Leone | Cents |
| `SCR` | 690 | Seychellois rupee | `SR` | `after` | `0.01` | 2 | Rupee | Cents |
| `RSD` | 941 | Serbian dinar | `din.` | `after` | `0.01` | 2 | Dinar | Para |
| `SAR` | 682 | Saudi riyal | `SR` | `after` | `0.01` | 2 | Riyal | Halala |
| `STD` | 678 | São Tomé and Príncipe dobra | `Db` | `after` | `0.01` | 2 | Dobra | Centimo |
| `WST` | 882 | Samoan tālā | `WS$` | `after` | `0.01` | 2 | Tala | Sene |
| `SHP` | 654 | Saint Helena pound | `£` | `after` | `0.01` | 2 | Pound | Penny |
| `RWF` | 646 | Rwandan franc | `RF` | `after` | `1` | 0 | Franc | Santime |
| `QAR` | 634 | Qatari riyal | `QR` | `after` | `0.01` | 2 | Riyal | Dirham |
| `PEN` | 604 | Peruvian sol | `S/` | `before` | `0.01` | 2 | Soles | Centimos |
| `PYG` | 600 | Paraguayan guaraní | `₲` | `after` | `1` | 0 | Guarani | Centimos |
| `PGK` | 598 | Papua New Guinean kina | `K` | `after` | `0.01` | 2 | Kina | Toea |
| `PKR` | 586 | Pakistani rupee | `Rs.` | `before` | `0.01` | 2 | Rupee | Paisa |
| `OMR` | 512 | Omani rial | `ر.ع.` | `after` | `0.001` | 3 | Rial | Baisa |
| `CVE` | 132 | Cape Verdean escudo | `$` | `after` | `0.01` | 2 | Escudo | Centavo |
| `COU` | 970 | Unidad de Valor Real | `$` | `before` | `0.01` | 2 | Peso | centavo |
| `CLF` | 990 | Unidad de Fomento | `$` | `before` | `0.0001` | 4 | Peso | Centavos |
| `CUC` | 931 | Cuban convertible peso | `$` | `after` | `0.01` | 2 | Cuban convertible peso | none |
| `GTQ` | 320 | Guatemalan Quetzal | `Q` | `after` | `0.01` | 2 | Quetzales | Centavo |
| `VES` | 937 | Venezuelan bolívar soberano | `Bs` | `after` | `0.01` | 2 | none | none |
| `UYW` | 858 | Unidad previsional | `$` | `after` | `0.0001` | 4 | peso | centésimo |
| `UYI` | 940 | Uruguay Peso en Unidades Indexadas | `$` | `after` | `0.0001` | 4 | Peso | centésimo |
| `STN` | 678 | São Tomé and Príncipe dobra | `Db` | `after` | `0.01` | 2 | Dobra | cêntimo |

### 4.2.1 Symbol position across the catalogue

Thirty-three of the shipped currencies place the symbol **before** the amount — `ANG`, `ARS`,
`AUD`, `BND`, `BRL`, `CHF`, `CLF`, `CLP`, `CNH`, `CNY`, `COP`, `COU`, `CRC`, `DKK`, `DOP`, `GBP`,
`HKD`, `HNL`, `IDR`, `ILS`, `INR`, `JPY`, `KRW`, `MVR`, `MXN`, `MYR`, `NOK`, `NZD`, `PEN`, `PKR`,
`SGD`, `USD` and `ZAR` — and the remaining one hundred and thirty-seven place it **after**. A
currency that states no position takes the field default, which is `after`.

### 4.2.2 Fields the shipped data leaves empty

- Three records carry no numeric code: `CNH` the offshore Chinese yuan, `XCG` the Caribbean
  guilder and `ZIG` the Zimbabwe gold currency.
- Two records carry no unit label: `SDG` the Sudanese pound and `VES` the Venezuelan bolivar of the
  newer denomination.
- Five records carry no subunit label: `CUC` the Cuban convertible peso, `SDG`, `VES`, `VUV` the
  Vanuatu vatu and `ZIG`.

Loading them with an empty value for those fields is correct; substituting a guessed value is not,
because the field is used verbatim when an amount is spelled in words
([calculations.md](calculations.md) section 25).

### 4.2.3 The numeric code is not unique

Two pairs of shipped records share a numeric code while remaining distinct currencies with distinct
alphabetic codes:

| Pair | Numeric code | Difference |
|---|---|---|
| `VEF` "Venezuelan bolívar fuerte" and `VES` "Venezuelan bolívar soberano" | 937 | Two successive denominations of the same national currency. |
| `BYR` "Belarusian ruble" with a rounding factor of one and `BYN` "Belarusian ruble" with a rounding factor of one hundredth | 974 | Two successive denominations, differing in precision. |

Both members of each pair are shipped and neither may be dropped. The consequence for a replacement
is that the numeric code must **not** carry a uniqueness constraint; only the currency code is
unique (`MCUR-001`).

## 4.3 Currencies contributed or activated by country packages

Country packages add three currencies that are units of account rather than circulating money, and
several of them activate currencies on installation. All of this is owned by
[../fiscal-localizations/README.md](../fiscal-localizations/README.md) and is listed here so that
the catalogue is complete.

| Code | Contributed by | Symbol | Symbol position | Rounding factor | Decimal places | Unit label | Other shipped values |
|---|---|---|---|---|---|---|---|
| `UF` | The Chile package | `UF` | `after` | `0.01` | 2 | Development Unit | The Chile short name reads "Development Unit", and in Latin American Spanish "Unidad de Fomento". |
| `UTM` | The Chile package | `UTM` | `after` | `0.01` | 2 | Monthly Tax Unit | The Chile short name reads "Monthly Tax Unit", and in Latin American Spanish "Unidad Tributaria Mensual". |
| `OTR` | The Chile package | `OTR` | `after`, by the field default | `1` | 0 | none | The Chile currency code reads `900` and the Chile short name reads `OTR`. It is the catch-all entry for a currency the Chilean statutory documents do not name. |

| Package | Currencies it activates on installation |
|---|---|
| Australia | `AUD`, and the common trading currencies `NZD`, `JPY`, `INR` and `GBP` |
| New Zealand | `NZD`, and the common trading currencies `AUD`, `JPY`, `INR` and `GBP` |
| Hong Kong | `CNY`, activated together with its chart of accounts template |
| Taiwan | `TWD`, whose symbol position is at the same time overridden from `after` to `before` |
| Uruguay | `UYI` |

Any chart of accounts template also activates the currency of its own country, through the company
whose main currency it sets (`MCUR-009`).

## 4.4 Demonstration data

When demonstration data is loaded, two additions are made.

1. **The United States dollar is activated.** The reasoning is that a platform without
   demonstration data must not start multi-currency: activating one currency by default would, as
   soon as a country's chart of accounts activated its own currency, leave two currencies active
   and grant the multi-currency capability unintentionally. With demonstration data the company's
   currency must be active so that monetary fields show a symbol even before any accounting
   capability is installed.
2. **One hundred and sixty-three Currency Rate rows are created**, listed in full below. Each is
   created only when a row with the same external identifier does not already exist, and none is
   ever updated by a later upgrade. One hundred and fifty-seven of them are dated 1 January 2010;
   the remaining six carry the date at which the currency concerned came into existence or was
   redenominated, so that the demonstration data does not claim a rate for a currency that did not
   yet exist. None of them states a company, so each takes the default company scope of a rate row,
   which is the root company of the loading context; in a freshly loaded platform that is the
   single main company. The Indonesian rupiah is the only currency with two demonstration rows.

**The complete demonstration rate table**, in load order. The rate column is the technical rate
(`rate`), which in a platform whose main currency carries no rate row is numerically the same as
the company rate.

| Currency code | Rate date | Technical rate |
|---|---|---|
| `USD` | `2010-01-01` | `1.0` |
| `VEF` | `2010-01-01` | `5.864` |
| `CAD` | `2010-01-01` | `1.3388` |
| `CHF` | `2010-01-01` | `1.3086` |
| `BRL` | `2010-01-01` | `2.2344` |
| `CNY` | `2010-01-01` | `8.7556` |
| `COP` | `2010-01-01` | `2933.8378` |
| `CZK` | `2010-01-01` | `26.5634` |
| `DKK` | `2010-01-01` | `7.4445` |
| `HUF` | `2010-01-01` | `271.5621` |
| `IDR` | `2009-01-01` | `14352.00` |
| `IDR` | `2010-01-01` | `11796.39` |
| `LVL` | `2010-01-01` | `0.7086` |
| `NOK` | `2010-01-01` | `7.8668` |
| `XPF` | `2010-01-01` | `119.331742` |
| `PAB` | `2010-01-01` | `1.2676` |
| `PLN` | `2010-01-01` | `4.1005` |
| `SEK` | `2010-01-01` | `10.3004` |
| `ARS` | `2010-01-01` | `5.0881` |
| `INR` | `2010-01-01` | `59.9739` |
| `AUD` | `2010-01-01` | `1.4070` |
| `UAH` | `2010-01-01` | `10.1969` |
| `VND` | `2010-01-01` | `26330.01` |
| `HKD` | `2010-01-01` | `11.1608` |
| `JPY` | `2010-01-01` | `133.62` |
| `BGN` | `2010-01-01` | `1.9558` |
| `LTL` | `2010-01-01` | `3.4528` |
| `RON` | `2010-01-01` | `4.2253` |
| `HRK` | `2010-01-01` | `7.2936` |
| `RUB` | `2010-01-01` | `43.16` |
| `TRY` | `2010-01-01` | `2.1411` |
| `KRW` | `2010-01-01` | `1662.37` |
| `MXN` | `2010-01-01` | `18.6664` |
| `MYR` | `2010-01-01` | `4.8887` |
| `NZD` | `2010-01-01` | `1.9764` |
| `PHP` | `2010-01-01` | `66.1` |
| `SGD` | `2010-01-01` | `2.0126` |
| `ZAR` | `2010-01-01` | `10.5618` |
| `CRC` | `2010-01-01` | `691.3153` |
| `MUR` | `2010-01-01` | `40.28` |
| `XOF` | `2010-01-01` | `655.957` |
| `XAF` | `2010-01-01` | `655.957` |
| `UGX` | `2010-01-01` | `3401.91388` |
| `HNL` | `2010-01-01` | `25` |
| `CLP` | `2010-01-01` | `710` |
| `UYU` | `2010-01-01` | `28.36` |
| `UYI` | `2023-09-13` | `5.7778` |
| `AFN` | `2010-01-01` | `59.33` |
| `AOA` | `2010-01-01` | `117.080` |
| `XCD` | `2010-01-01` | `3.32` |
| `AMD` | `2010-01-01` | `506.02` |
| `AWG` | `2010-01-01` | `0.45` |
| `AZN` | `2010-01-01` | `0.96` |
| `BSD` | `2010-01-01` | `1.23` |
| `BHD` | `2010-01-01` | `0.46` |
| `BDT` | `2010-01-01` | `100.59` |
| `BBD` | `2010-01-01` | `2.46` |
| `BYR` | `2010-01-01` | `10228.19` |
| `BYN` | `2016-07-01` | `2.23151` |
| `BZD` | `2010-01-01` | `2.33` |
| `BMD` | `2010-01-01` | `1.23` |
| `BTN` | `2010-01-01` | `67.81` |
| `BOB` | `2010-01-01` | `8.50` |
| `BAM` | `2010-01-01` | `1.96` |
| `BWP` | `2010-01-01` | `9.45` |
| `BIF` | `2010-01-01` | `1736.73` |
| `KHR` | `2010-01-01` | `5054.53` |
| `KYD` | `2010-01-01` | `1.007` |
| `KMF` | `2010-01-01` | `492.23` |
| `CDF` | `2010-01-01` | `1112.80` |
| `CUP` | `2010-01-01` | `1.226` |
| `ANG` | `2010-01-01` | `2.11` |
| `XCG` | `2025-01-15` | `1.80302` |
| `DJF` | `2010-01-01` | `222.22` |
| `DOP` | `2010-01-01` | `48.09` |
| `EGP` | `2010-01-01` | `7.46` |
| `SVC` | `2010-01-01` | `10.74` |
| `ERN` | `2010-01-01` | `18.89` |
| `ETB` | `2010-01-01` | `21.94` |
| `FKP` | `2010-01-01` | `0.78` |
| `FJD` | `2010-01-01` | `2.22` |
| `GEL` | `2010-01-01` | `2.018` |
| `GIP` | `2010-01-01` | `0.786` |
| `GNF` | `2010-01-01` | `8835.85` |
| `GYD` | `2010-01-01` | `247.31` |
| `HTG` | `2010-01-01` | `51.69` |
| `ISK` | `2010-01-01` | `158.70` |
| `IRR` | `2010-01-01` | `15059.97` |
| `IQD` | `2010-01-01` | `1432.27` |
| `ILS` | `2010-01-01` | `4.89` |
| `JMD` | `2010-01-01` | `108.94` |
| `JOD` | `2010-01-01` | `0.87` |
| `KZT` | `2010-01-01` | `184.09` |
| `KES` | `2010-01-01` | `103.43` |
| `KWD` | `2010-01-01` | `0.35` |
| `KGS` | `2010-01-01` | `57.93` |
| `LAK` | `2010-01-01` | `9847.72` |
| `LBP` | `2010-01-01` | `1853.94` |
| `LSL` | `2010-01-01` | `10.06` |
| `LRD` | `2010-01-01` | `90.28` |
| `LYD` | `2010-01-01` | `1.54` |
| `MOP` | `2010-01-01` | `9.81` |
| `MKD` | `2010-01-01` | `60.29` |
| `MGA` | `2010-01-01` | `2775.86` |
| `MWK` | `2010-01-01` | `329.79` |
| `MVR` | `2010-01-01` | `18.89` |
| `MRO` | `2010-01-01` | `362.23` |
| `MDL` | `2010-01-01` | `15.27` |
| `MNT` | `2010-01-01` | `1643.57` |
| `MAD` | `2010-01-01` | `10.9962499` |
| `BND` | `2010-01-01` | `1.54` |
| `DZD` | `2010-01-01` | `99.65` |
| `GHS` | `2010-01-01` | `2.39` |
| `GMD` | `2010-01-01` | `38.89` |
| `MZN` | `2010-01-01` | `33.65` |
| `MMK` | `2010-01-01` | `1080.82` |
| `NAD` | `2010-01-01` | `10.06` |
| `NPR` | `2010-01-01` | `108.33` |
| `ALL` | `2010-01-01` | `138.00` |
| `NIO` | `2010-01-01` | `28.97` |
| `NGN` | `2010-01-01` | `196.77` |
| `KPW` | `2010-01-01` | `1105.24376765` |
| `ZIG` | `2024-04-08` | `14.69` |
| `ZMW` | `2010-01-01` | `11.61` |
| `YER` | `2010-01-01` | `264.784483` |
| `EUR` | `2010-01-01` | `1.2834` |
| `VUV` | `2010-01-01` | `114.024315363` |
| `UZS` | `2010-01-01` | `2331.3093` |
| `AED` | `2010-01-01` | `4.51253195` |
| `TMT` | `2010-01-01` | `4.31` |
| `TND` | `2010-01-01` | `34.0661408` |
| `TTD` | `2010-01-01` | `8.61926302` |
| `TOP` | `2010-01-01` | `31.1217` |
| `THB` | `2010-01-01` | `38.4070124` |
| `TZS` | `2010-01-01` | `1947.06815` |
| `TJS` | `2010-01-01` | `6.15` |
| `TWD` | `2010-01-01` | `36.8329536` |
| `SYP` | `2010-01-01` | `78.8338` |
| `SZL` | `2010-01-01` | `10.07` |
| `SRD` | `2010-01-01` | `9.17930` |
| `SDG` | `2010-01-01` | `3.1999` |
| `LKR` | `2010-01-01` | `163.552526` |
| `SSP` | `2010-01-01` | `5.528` |
| `GBP` | `2010-01-01` | `0.784221994` |
| `SOS` | `2010-01-01` | `1993.71` |
| `SBD` | `2010-01-01` | `8.6605` |
| `SLL` | `2010-01-01` | `5320.43478` |
| `SLE` | `2023-06-08` | `22.5847` |
| `SCR` | `2010-01-01` | `18.2587287` |
| `RSD` | `2010-01-01` | `117.381295` |
| `SAR` | `2010-01-01` | `4.58898972` |
| `STD` | `2010-01-01` | `24105.86` |
| `WST` | `2010-01-01` | `2.8628` |
| `SHP` | `2010-01-01` | `0.7856` |
| `RWF` | `2010-01-01` | `752.57` |
| `QAR` | `2010-01-01` | `4.45500218` |
| `PEN` | `2010-01-01` | `3.20731573` |
| `PYG` | `2010-01-01` | `5343.66812` |
| `PGK` | `2010-01-01` | `2.5903895` |
| `PKR` | `2010-01-01` | `115.97432` |
| `OMR` | `2010-01-01` | `0.472921728` |
| `CVE` | `2010-01-01` | `0.61` |
| `GTQ` | `2010-01-01` | `11.2020` |

Eight shipped currencies receive no demonstration rate row at all and therefore convert at the
fallback rate of one under `MCUR-030` until an accountant records a rate: `CNH` the offshore
Chinese yuan, `MRU` the Mauritanian ouguiya of the newer denomination, `COU` the Colombian real
value unit, `CLF` the Chilean development unit, `CUC` the Cuban convertible peso, `VES` the
Venezuelan bolivar of the newer denomination, `UYW` the Uruguayan indexed pension unit and `STN`
the São Tomé and Príncipe dobra of the newer denomination.

The six rows that are not dated 1 January 2010 are:

| Currency code | Rate date | Why |
|---|---|---|
| `IDR` | `2009-01-01` | The second, earlier row of the only currency that carries two demonstration rows. |
| `UYI` | `2023-09-13` | The date from which the demonstration data claims a rate for the Uruguayan indexed unit. |
| `BYN` | `2016-07-01` | The redenomination of the Belarusian ruble. |
| `XCG` | `2025-01-15` | The introduction of the Caribbean guilder. |
| `ZIG` | `2024-04-08` | The introduction of the Zimbabwe gold currency. |
| `SLE` | `2023-06-08` | The redenomination of the Sierra Leonean leone. |

---

# 5. The decimal precision registry

A separate registry of named precisions exists for non-monetary decimals. It is owned by
[../platform-foundation/README.md](../platform-foundation/README.md) and is described here because
the currency form links to it and because it is frequently confused with currency precision.

| Aspect | Rule |
|---|---|
| Record shape | A usage name, unique, and a digit count, required, defaulting to two. |
| Lookup | A lookup for an unregistered usage returns two. Lookups are cached, and the cache is invalidated on every creation, write and deletion. |
| Behaviour during accounting computations | While an internal recursion guard is active, the usage "Discount" returns thirteen and the usage "Product Unit" returns ten, so that intermediate results are not truncated prematurely. |
| Warning on reduction | Lowering a digit count raises a non-blocking warning whose title is "Warning for " followed by the usage name and whose body is quoted in [calculations.md](calculations.md) section 1. |
| Relationship to currencies | None. A monetary amount is rounded by its currency's rounding factor, never by a registry entry (`MCUR-005`). |

---

# 6. Journal and account configuration

| Setting | Entity | Type | Default | Effect |
|---|---|---|---|---|
| Currency | `currency_id` (journal currency) on Journal | Link to Currency | Empty | When set, every entry of the journal takes this currency as its document currency, the journal's default account and the bank account record linked to the journal are forced to the same currency, and the journal's display name is suffixed with the currency code in parentheses when the currency differs from the company's main currency (`MCUR-067`). When empty, the journal accepts the company's main currency and any document currency. |
| Account Currency | `currency_id` (account currency) on Account | Link to Currency | Empty | When set, every journal item posted to the account must carry that currency (`MCUR-064`). When empty, the account accepts any currency. Also governed by `MCUR-068` and `MCUR-069`. |

A bank account held in a foreign currency is configured by creating a journal whose type is bank,
setting its currency to the foreign currency, and letting the journal propagate that currency to
its default account. The counterpart accounts of the journal's inbound and outbound payment method
lines must carry the same currency (`MCUR-068`).

---

# 7. Permission groups and the access matrix

| Group | Members | What it grants or controls in this domain |
|---|---|---|
| Public visitor | Unauthenticated visitors | Read on Currency and on Currency Rate. Needed to render prices on public pages. |
| Portal user | External users with a portal account | Read on Currency and on Currency Rate. Needed to render portal documents. |
| Internal user | Every employee account | Read on Currency and on Currency Rate. |
| Settings Administration | Administrators | Create, read, write and delete on Currency and on Currency Rate. Access to the accounting settings screen. |
| Accounting Manager | Accountants responsible for the configuration | Create, read, write and delete on Currency and on Currency Rate. Access to the Currencies menu, and to the exchange journal and exchange account settings. |
| Accounting User | Accountants recording documents | Read on Currency and Currency Rate. May select a document currency, may override a document rate, may reconcile, and therefore may cause an exchange difference entry to be created. |
| Accounting Read-Only | Auditors and reviewers | Read on everything, no write. |
| Multi-currency | Granted to and revoked from the internal-user group automatically by `MCUR-008` | Controls the **visibility** of currency fields, not the **right** to change them: the document currency selector, the amount in currency column, the residual amount in currency column, the document rate field, the exchange difference settings block and the automatic rate setting are shown only to holders. |
| Multi-company | Granted by hand | Controls the visibility of the company column of a rate row, whose placeholder reads "Visible to all". |
| Technical features | Granted by hand, typically to administrators | Controls the visibility of the rounding factor, the decimal places and the technical rate column of a rate row. |

## 7.1 Access matrix

| Entity | Public visitor | Portal user | Internal user | Accounting Manager | Settings Administration |
|---|---|---|---|---|---|
| Currency | read | read | read | read, create, write, delete | read, create, write, delete |
| Currency Rate | read | read | read | read, create, write, delete | read, create, write, delete |

No record-level rule restricts Currency or Currency Rate: every row is visible to every reader. The
company scope of a rate row decides which rate a conversion **uses**, not which rows may be
**read**.

---

# 8. Menus and navigation

| Menu path | Target | Visible to |
|---|---|---|
| Accounting, Configuration, Accounting, Currencies | The currency catalogue, opened with archived records included | Holders of the accounting manager group, because the Configuration menu itself is reserved to them |
| Accounting, Configuration, Settings, Currencies block | The main currency, a control that opens the currency catalogue, and the automatic rate setting | Holders of the settings administration group |
| Accounting, Configuration, Settings, Default Accounts, Exchange difference entries | The exchange journal, the gain account and the loss account | Holders of the accounting user group, when the multi-currency group is held |
| The contextual action "Show Currency Rates", offered on a currency form | The rate list filtered on that currency, with the currency pre-filled on new rows | Every reader of Currency Rate |

---

# 9. Scheduled work

| Job | Frequency | Behaviour |
|---|---|---|
| Automatic currency rate update | Runs on the platform's scheduler; each company is processed when its own next run moment has been reached | Workflow 3 of [workflows.md](workflows.md). Fetches rates for every active currency other than the company's main currency, writes or updates one Currency Rate per currency for today and for the company's root, and advances the company's next run moment by the configured interval. A service failure is logged and leaves the rate table untouched. The job exists only when the automatic rate retrieval capability is installed. |

No other scheduled work belongs to this domain. Exchange difference entries are never produced by a
scheduled job; they are produced synchronously by a reconciliation.

---

# 10. Configuration checklist

A platform that must handle foreign currencies is configured in this order.

1. Confirm the company's main currency, and confirm that no journal item exists yet, because the
   choice can never be changed afterwards (`MCUR-171`).
2. Review the rounding factor of every currency that will be activated **before** any document is
   recorded in it, because the factor can never be coarsened once the currency has been used
   (`MCUR-006`).
3. Activate every currency that will be used, from the currency catalogue (workflow 1 of
   [workflows.md](workflows.md)).
4. Confirm that activating the second currency granted the multi-currency capability and, through
   it, the price list capability.
5. For each active currency, record at least one rate row, or install the automatic rate retrieval
   capability and choose a service and an interval.
6. Confirm the exchange journal, the gain account and the loss account. Without all three, the
   first reconciliation that crosses a rate movement fails and writes nothing.
7. Create one price list per active currency in which products will be sold.
8. For each bank account held in a foreign currency, create a journal of that currency and confirm
   that its default account and its payment method accounts carry the same currency.
9. Decide the two presentation settings: taxes shown in the company currency on printed invoices,
   and the total spelled out in letters.
10. Confirm that every company that will report together shares a root, or plan the consolidation
    rate types of [calculations.md](calculations.md) section 20.

---

# 11. Reconciliation notes

1. **Where the catalogue lives.** Both drafts carried a complete table of the one hundred and
   seventy shipped currencies, one ordered alphabetically and one in load order, and the two agreed
   on every value of every row. The catalogue is published once here, in load order, with the
   derived decimal places added as a column; the alphabetical draft's two summaries — by rounding
   factor and by symbol position — are kept as sections 4.1 and 4.2.1.
2. **The country-package currencies.** Only one draft listed the three units of account contributed
   by the Chile package and the currencies the Australian, New Zealand, Hong Kong, Taiwanese and
   Uruguayan packages activate. They are kept, in section 4.3, with the values each record ships,
   including the Taiwanese package's override of the symbol position of `TWD`.
3. **The demonstration rate table.** Only one draft carried it. It is kept in full, in load order,
   in section 4.4, together with the reason each of the six rows that are not dated 1 January 2010
   carries the date it does.
4. **Field identifiers.** One draft named the settings by a canonical full name, the other by the
   stored identifier. Every setting here carries the stored identifier in code font with its full
   name in words, matching [entities.md](entities.md).
5. **The automatic rate retrieval capability.** One draft specified the service selector, the four
   interval values and the service contract as observed behaviour. The reviewed material settles
   that the setting installs an optional capability package and that the package writes ordinary
   rate rows, but not the list of services; section 2 therefore marks the contract as an
   **industry-standard default** while keeping every setting the draft described, because the
   scheduled job of section 9 and workflow 3 depend on them.
6. **Rule identifiers.** The former `MCUR-RULE-nnn` citations of one draft are written here in the
   consolidated `MCUR-nnn` scheme; the mapping is in [business-rules.md](business-rules.md)
   section 13.
