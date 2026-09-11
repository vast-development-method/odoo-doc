# Acceptance criteria — Contacts and Organizations

Numbered Given / When / Then scenarios with concrete values. A rebuild that satisfies every
scenario below reproduces the observable behaviour of this domain.

Conventions used throughout:

- internal identifiers are written as plain numbers and are stable within a scenario;
- the "acting company" is the company on whose behalf the operation runs; where it is not stated,
  it is a company whose country is the United States;
- "the directory maintainer" is a user in the contact-creation group;
- a field written as `—` is empty.

---

# 1. Creating parties

### 1.1 An organization is created and is its own commercial entity

**Given** an empty directory
**When** a directory maintainer creates a Party with name `Deco Addict`, organization flag set, no
parent
**Then** the record is stored with internal identifier 100
**And** its complete name is `Deco Addict`
**And** its display name is `Deco Addict`
**And** its commercial entity is 100
**And** its commercial company name is `Deco Addict`
**And** its shared-party flag is set
**And** its active flag is set
**And** its address type is `contact`

### 1.2 A person with no parent

**Given** an empty directory
**When** a Party is created with name `Marc Demo`, organization flag cleared, no parent, no
free-text company name
**Then** the complete name is `Marc Demo`
**And** the commercial entity is the record itself
**And** the commercial company name is empty, because the commercial entity is not an organization
and the free-text company name is empty

### 1.3 A person with a free-text employer

**Given** an empty directory
**When** a Party is created with name `Marc Demo`, organization flag cleared, no parent, free-text
company name `Azure Interior`
**Then** the commercial company name is `Azure Interior`
**And** the complete name is `Azure Interior, Marc Demo`

### 1.4 A nameless contact is refused

**Given** an empty directory
**When** a Party is created with no name and address type `contact`
**Then** the operation fails with `Contacts require a name`

### 1.5 A nameless delivery address is accepted

**Given** an organization 100 named `Deco Addict`
**When** a Party is created with no name, address type `delivery`, parent 100
**Then** the record is stored
**And** its complete name is `Deco Addict, Delivery`

### 1.6 An invalid default address type is discarded

**Given** a creation context supplying a default address type of `shipping`
**When** a Party is created
**Then** the stored address type is empty, not `shipping` and not `contact`

### 1.7 The parent's company becomes the default

**Given** an organization 100 whose company is company 1
**When** a Party is created from a context supplying parent 100 and no company
**Then** the created Party's company is company 1

### 1.8 A website with no scheme is normalised

**Given** any creation
**When** the website is given as `example.com`
**Then** the stored value is `http://example.com`
**And** given `www.example.com/about` the stored value is `http://www.example.com/about`
**And** given `https://example.com` the stored value is unchanged

### 1.9 Setting a parent clears the free-text company name

**Given** a Party 101 with free-text company name `Azure Interior` and no parent
**When** a parent 100 is written on it
**Then** the free-text company name is empty

### 1.10 Duplicating a Party appends a suffix

**Given** a Party named `Deco Addict`
**When** it is duplicated without an explicit name
**Then** the copy's name is `Deco Addict (copy)`
**And** the copy's barcode is empty, because the barcode is not copied

---

# 2. The complete name and the display name

### 2.1 A subsidiary keeps its bare name

**Given** an organization 100 named `Azure Interior` and an organization 101 named
`Azure Interior Benelux` whose parent is 100
**Then** the complete name of 101 is `Azure Interior Benelux`, with no prefix

### 2.2 A person under a subsidiary is prefixed with the subsidiary

**Given** the hierarchy of §2.1 and a person 102 named `Brandon Freeman` whose parent is 101
**Then** the commercial entity of 102 is 101
**And** the commercial company name of 102 is `Azure Interior Benelux`
**And** the complete name of 102 is `Azure Interior Benelux, Brandon Freeman`

### 2.3 An invoicing address under a person resolves two levels up

**Given** the hierarchy of §2.2 and a nameless Party 103 of type `invoice` whose parent is 102
**Then** the commercial entity of 103 is 101
**And** the complete name of 103 is `Azure Interior Benelux, Invoice`

### 2.4 Clearing the subsidiary's organization flag re-resolves the whole subtree

**Given** the hierarchy of §2.3
**When** the organization flag of 101 is cleared
**Then** the commercial entity of 101, 102 and 103 is all 100
**And** the commercial company name of 102 becomes `Azure Interior`
**And** the complete name of 102 becomes `Azure Interior, Brandon Freeman`
**And** the complete name of 103 becomes `Azure Interior, Invoice`

### 2.5 The display name with the address and the tax number

**Given** a Party 41 with: no name; organization flag cleared; address type `delivery`; parent
`Deco Addict`; commercial company name `Deco Addict`; street `77 Santa Barbara Rd`; second street
line empty; city `Pleasant Hill`; state `California` with code `CA`; postal code `94523`; country
`United States` with the default layout; tax registration number `US12345678`
**When** the display name is read with the context keys `show_address` and `show_vat` present
**Then** the display name is exactly

```
Deco Addict, Delivery
77 Santa Barbara Rd

Pleasant Hill CA 94523
United States
 US12345678
```

with no leading or trailing whitespace, and with the trailing space that the substitution left on
the `United States` line removed by the whitespace collapse.

### 2.6 The display name with only the tax number

**Given** the Party of §2.5
**When** the display name is read with only `show_vat` present
**Then** it is `Deco Addict, Delivery - US12345678`

### 2.7 The display name with the internal identifier

**Given** an organization 14 named `Deco Addict`
**When** the display name is read with `partner_show_db_id` present
**Then** it is `Deco Addict (14)`

### 2.8 The display name with the electronic mail address

**Given** an organization named `Deco Addict` whose address is `contact@deco.example`
**When** the display name is read with `show_email` present
**Then** it is `Deco Addict <contact@deco.example>`

### 2.9 The two-part formatted display name

**Given** the Party of §2.5 with the address `deliveries@deco.example`
**When** the display name is read with `formatted_display_name` and `show_email` present
**Then** it is `Deco Addict \t --Delivery-- \t --deliveries@deco.example--`, where `\t` is one
horizontal tab character
**And** the postal address does **not** appear, even if `show_address` is also present

### 2.10 The suppression of the organization prefix

**Given** the Party of §2.2
**When** the complete name is computed with the context key
`partner_display_name_hide_company` present
**Then** it is `Brandon Freeman`, with no prefix

### 2.11 Ordering

**Given** three parties with complete names `Azure Interior` (identifier 10), `Deco Addict`
(identifier 5) and `Azure Interior` (identifier 20)
**When** they are listed with no explicit ordering
**Then** the order is: identifier 20, identifier 10, identifier 5 — complete name ascending, then
identifier descending

---

# 3. The formatted electronic mail address

### 3.1 A normal address

**Given** a Party named `Deco Addict` whose address is `Sales@Deco.Example`
**Then** the formatted address is `"Deco Addict" <sales@deco.example>`

### 3.2 Several addresses

**Given** a Party named `Deco Addict` whose address field holds
`sales@deco.example, orders@deco.example`
**Then** the formatted address is `"Deco Addict" <sales@deco.example,orders@deco.example>`

### 3.3 A malformed address

**Given** a nameless Party whose address field holds `not-an-address`
**Then** the formatted address is `"False" <not-an-address>`

### 3.4 An empty address

**Given** a Party whose address field is empty
**Then** the formatted address is empty

---

# 4. Address synchronization

### 4.1 A contact child inherits the organization's address

**Given** an organization 100 with street `77 Santa Barbara Rd`, city `Pleasant Hill`, state
`California`, postal code `94523`, country `United States`, and tax registration number
`US11223344`
**When** a Party 101 is created with name `Douglas Fletcher`, parent 100, address type `contact`,
and every address field left empty
**Then** 101's street is `77 Santa Barbara Rd`, its city `Pleasant Hill`, its state `California`,
its postal code `94523`, its country `United States`
**And** 101's tax registration number is `US11223344`
**And** 100's address is unchanged

### 4.2 An invoicing child does not inherit the address but does inherit the tax number

**Given** the organization 100 of §4.1
**When** a Party 102 is created with no name, parent 100, address type `invoice`, street
`PO Box 1201`, city `Pleasant Hill`, state `California`, postal code `94523`, country
`United States`
**Then** 102's street stays `PO Box 1201`
**And** 102's tax registration number is `US11223344`

### 4.3 The organization moves; the contact follows, the invoicing address does not

**Given** the state after §4.1 and §4.2
**When** 100 is written with street `215 Vine Street`, second street line `Suite 400`, city
`Walnut Creek`, postal code `94596` (the state and country unchanged)
**Then** 101 has street `215 Vine Street`, second street line `Suite 400`, city `Walnut Creek`,
postal code `94596`, state `California`, country `United States`
**And** 102 still has street `PO Box 1201`, second street line empty, city `Pleasant Hill`, postal
code `94523`

### 4.4 Editing a contact's address moves the organization

**Given** the state after §4.3
**When** 101 is written with street `215 Vine Street, Building B`
**Then** 100's street becomes `215 Vine Street, Building B`
**And** every other `contact`-type child of 100 receives the same street
**And** 102's street is unchanged

### 4.5 A partly filled parent address clears the child's other fields

**Given** an organization 200 with **only** the city `Brussels` filled and every other address
field empty
**When** a Party 201 is created with parent 200, address type `contact`, street `Rue Neuve 1`,
postal code `1000`
**Then** 201's city becomes `Brussels`
**And** 201's street becomes empty and its postal code becomes empty, because the parent's address
map is non-empty and therefore carries **all six** fields including the empty ones

### 4.6 A completely empty parent address pushes nothing

**Given** an organization 300 with every address field empty
**When** a Party 301 is created with parent 300, address type `contact`, street `Rue Neuve 1`,
city `Brussels`, postal code `1000`, country `Belgium`
**Then** the inherit step pushes nothing, because the parent's address map is empty
**And** the upward push then fires, because 301's address differs from 300's
**And** 300 ends with street `Rue Neuve 1`, city `Brussels`, postal code `1000`, country `Belgium`

### 4.7 Changing the address type to contact overwrites the address

**Given** the state after §4.3, where 102 is a `delivery`-style independent address with street
`PO Box 1201`
**When** 102 is given the name `Accounts Payable` and its address type is changed to `contact`
**Then** 102's address becomes 100's address — street `215 Vine Street`, second street line
`Suite 400`, city `Walnut Creek`, postal code `94596`
**And** the former street `PO Box 1201` is lost

### 4.8 A subsidiary does not inherit its parent's address

**Given** an organization 100 with a filled address
**When** an organization 104 is created with parent 100 and address type `contact`, with an empty
address
**Then** 104's address stays empty: the inherit step applies to any `contact`-type child, so 104
**does** inherit — the address is inherited, and it is the **commercial** fields that 104 does not
inherit, because 104 is its own commercial entity
**And** 104's tax registration number stays empty

> This scenario is stated deliberately because the two rules differ: the address push keys on the
> address type, the commercial push keys on the organization flag.

### 4.9 The controlled city travels with the address

**Given** the extended-address behaviour installed, a country `United States` with the enforcement
flag set, and a controlled city `New York` in that country
**When** an organization is created with that country and that controlled city, and then a
`contact`-type child is created under it
**Then** the child's country is `United States` and its controlled city is `New York`

---

# 5. Commercial-field synchronization

### 5.1 The tax number typed on a child travels up and then down

**Given** an organization 100 with tax registration number `US11223344`, a `contact`-type child
101 and an `invoice`-type child 102, all three carrying `US11223344`
**When** 101 is written with tax registration number `US99887766`
**Then** 100's tax registration number becomes `US99887766`
**And** 102's tax registration number becomes `US99887766`
**And** 101's is `US99887766`

### 5.2 The company registration number travels down only

**Given** the hierarchy of §5.1
**When** 101 is written with company registration number `0477472701`
**Then** 100's company registration number stays empty
**And** 102's stays empty
**And** only 101 carries the value

### 5.3 The company registration number typed on the organization travels down

**Given** the hierarchy of §5.1
**When** 100 is written with company registration number `0477472701`
**Then** 101 and 102 both receive `0477472701`

### 5.4 Empty commercial values are never pushed

**Given** an organization 100 with an empty industry, and a child 101 whose industry is
`Manufacturing`
**When** 100's tax registration number is changed
**Then** 101's industry stays `Manufacturing`, because only the commercial entity's **non-empty**
values are pushed on an inherit, and the downstream push is restricted to the fields that actually
changed

### 5.5 An organization child blocks the commercial push

**Given** an organization 100 with tax registration number `US11223344`, an organization child 104
with tax registration number `BE0477472701`, and a person 105 under 104
**When** 100's tax registration number is changed to `US99887766`
**Then** 104's stays `BE0477472701`
**And** 105's stays whatever it was, because the push skips the organization child

---

# 6. Address resolution

Hierarchy used in §6.1 to §6.4:

| Identifier | Name | Organization | Parent | Address type |
|---|---|---|---|---|
| 100 | `Deco Addict` | yes | — | `contact` |
| 101 | `Douglas Fletcher` | no | 100 | `contact` |
| 102 | — | no | 100 | `invoice` |
| 103 | — | no | 100 | `delivery` |
| 104 | `Deco Addict Belgium` | yes | 100 | `contact` |
| 105 | — | no | 104 | `delivery` |

### 6.1 Resolving from a contact reaches the siblings

**When** the addresses `invoice` and `delivery` are resolved for 101
**Then** the result is exactly `{contact: 101, invoice: 102, delivery: 103}`

### 6.2 Resolving from inside a subsidiary stops at the subsidiary

**When** the address `invoice` is resolved for 105
**Then** the result is exactly `{contact: 104, invoice: 104}`
**And** 102 is **not** returned

### 6.3 Resolving from the organization

**When** the address `delivery` is resolved for 100
**Then** the result is exactly `{contact: 100, delivery: 103}`

### 6.4 A lone Party is its own every address

**Given** a Party 200 with no parent and no children
**When** the address `invoice` is resolved for 200
**Then** the result is `{contact: 200, invoice: 200}`

### 6.5 The contact entry is always present

**When** only the address `delivery` is requested for any Party
**Then** the result also carries a `contact` entry

---

# 7. Address rendering

### 7.1 A Belgian address with the company line

**Given** a Party whose commercial company name is `Deco Addict`, street
`Chaussée de Namur 40`, second street line empty, postal code `1367`, city `Ramillies`, no state,
country `Belgium` with the layout `%(street)s\n%(street2)s\n%(zip)s %(city)s\n%(country_name)s`
**When** the address is rendered with the company line
**Then** the result is exactly

```
Deco Addict
Chaussée de Namur 40

1367 Ramillies
Belgium
```

### 7.2 The same address without the company line

**Then** the result is exactly

```
Chaussée de Namur 40

1367 Ramillies
Belgium
```

### 7.3 The default layout with a state

**Given** the same Party with the country `United States`, the state `California` code `CA`, and
the default layout
**When** the address is rendered without the company line
**Then** the result is exactly

```
Chaussée de Namur 40

Ramillies CA 1367
United States
```

### 7.4 A layout that puts the country first

**Given** the same Party with the country `China`, whose layout is
`%(country_name)s, %(zip)s\n%(state_name)s %(city)s %(street)s %(street2)s`
**When** the address is rendered without the company line
**Then** the result is exactly two lines: `China, 1367`, then a line consisting of a space, then
`Ramillies Chaussée de Namur 40`, then a trailing space

### 7.5 A layout with an unknown key is refused

**When** a country's layout is written as `%(street)s\n%(unknown_key)s`
**Then** the write fails with `The layout contains an invalid format key`

### 7.6 The input-form reordering for a Belgian acting company

**Given** an acting company whose country is `Belgium`
**When** a Party form is built
**Then** the address inputs appear in the order: street, second street line, postal code, city,
state, country

### 7.7 The input-form reordering for a United States acting company

**Given** an acting company whose country is `United States`
**Then** the address inputs appear in the order: street, second street line, city, state, postal
code, country

---

# 8. Street decomposition

Every row is a scenario: **Given** the extended-address behaviour installed, **when** the street
is set to the input, **then** the three parts take the stated values; and **when** the three parts
are written back, **then** the street becomes the stated recomposition.

| # | Input street | Street name | House number | Door number | Recomposed street |
|---|---|---|---|---|---|
| 8.1 | *(empty)* | *(empty)* | *(empty)* | *(empty)* | *(empty)* |
| 8.2 | `Place Royale` | `Place Royale` | *(empty)* | *(empty)* | `Place Royale` |
| 8.3 | `Chaussee de Namur 40a - 2b` | `Chaussee de Namur` | `40a` | `2b` | `Chaussee de Namur 40a - 2b` |
| 8.4 | `Chaussee de Namur 1` | `Chaussee de Namur` | `1` | *(empty)* | `Chaussee de Namur 1` |
| 8.5 | `40 Chaussee de Namur` | `Chaussee de Namur` | `40` | *(empty)* | `Chaussee de Namur 40` |
| 8.6 | `Chaussee de Namur, 40 - Apt 2b` | `Chaussee de Namur` | `40` | `Apt 2b` | `Chaussee de Namur 40 - Apt 2b` |
| 8.7 | `header Chaussee de Namur, 40 trailer ` | `header Chaussee de Namur` | `40` | *(empty)* | `header Chaussee de Namur 40` |
| 8.8 | `1600 Pennsylvania Ave NW, Apt 4B` | `Pennsylvania Ave NW` | `1600` | `Apt 4B` | `Pennsylvania Ave NW 1600 - Apt 4B` |
| 8.9 | `10, Rue de la Paix` | `Rue de la Paix` | `10` | *(empty)* | `Rue de la Paix 10` |
| 8.10 | `Calle Gran Vía, 42, 3º Dcha` | `Calle Gran Vía` | `42` | `3º Dcha` | `Calle Gran Vía 42 - 3º Dcha` |
| 8.11 | `Jean-Baptiste-Lebas 12 - A-3` | `Jean-Baptiste-Lebas` | `12` | `A-3` | `Jean-Baptiste-Lebas 12 - A-3` |
| 8.12 | `Jean-Baptiste-Lebas, 12 / A-3` | `Jean-Baptiste-Lebas` | `12` | `A-3` | `Jean-Baptiste-Lebas 12 - A-3` |
| 8.13 | `1-7-1 Nagatacho, Chiyoda-ku, Apt 3` | `Nagatacho, Chiyoda-ku` | `1-7-1` | `Apt 3` | `Nagatacho, Chiyoda-ku 1-7-1 - Apt 3` |

### 8.14 A multi-line street

**Given** the street set to a line feed, then `Cl 53`, then a line feed, then ` # 43 - 81`
**Then** the street name is `Cl 53`, a line feed, ` #`; the house number is `43`; the door number
is `81`
**And** the recomposition is `Cl 53`, a line feed, ` # 43 - 81`

### 8.15 A trailing word is lost

**Given** the street `header Chaussee de Namur, 40 trailer ` (scenario 8.7)
**When** the three parts are written back
**Then** the street becomes `header Chaussee de Namur 40` and the word `trailer` is gone, because
a door number must contain at least one digit

---

# 9. Tax registration numbers

### 9.1 A Belgian number is normalised and prefixed

**Given** a Party whose country is `Belgium`
**When** the tax registration number is written as `be 0477.472.701`
**Then** the stored value is `BE0477472701`
**And** no failure is raised

### 9.2 A bare Belgian number acquires no prefix from the country alone

**Given** a Party whose country is `Belgium`
**When** the number is written as `0477472701`
**Then** the routine finds no alphabetic prefix, so the prefixed country stays empty and the code
to check is `BE`
**And** the stored value is `0477472701`

### 9.3 The marker for "no tax number"

**When** the number is written as `/`
**Then** it is accepted and stored as `/`

### 9.4 Any other single character is refused

**When** the number is written as `X` with validation in raising mode
**Then** the operation fails with `To explicitly indicate no (valid) VAT, use '/' instead. `

### 9.5 A doubled prefix is refused

**Given** a Party whose country is `Belgium`
**When** the number is written as `BEBE0477472701`
**Then** the operation fails with the standard failure message

### 9.6 A Belgian number on a French Party is validated as Belgian

**Given** a Party whose country is `France`
**When** the number is written as `BE0477472701`
**Then** the prefixed country is `BE`, the code to check is `BE`, and the value is accepted

### 9.7 A Belgian number on a Swiss Party triggers the union-wide retry

**Given** a Party whose country is `Switzerland`
**When** the number is written as `BE0477472701`
**Then** the first attempt validates it as Swiss and fails
**And** the retry validates it against Belgium and succeeds
**And** the stored value is `BE0477472701`

### 9.8 A Greek number keeps the special prefix

**Given** a Party whose country is `Greece`
**When** the number is written as `EL123456783`
**Then** the stored value keeps the prefix `EL`, not `GR`

### 9.9 The Swiss check digit

**When** the number `CHE-123.456.788 TVA` is checked
**Then** the weighted sum over the digits 1 2 3 4 5 6 7 8 with the weights 5 4 3 2 7 6 5 4 is 168
**And** the check digit is the remainder of (11 minus the remainder of 168 divided by 11) divided
by 11, which is 8
**And** the ninth digit is 8, so the number is valid

### 9.10 The Norwegian check digit

**When** the number `123456785` is checked as Norwegian
**Then** the weighted sum with the weights 3 2 7 6 5 4 3 2 is 138
**And** the check digit is 11 minus 6, which is 5
**And** the ninth digit is 5, so the number is valid

### 9.11 A Norwegian number whose check digit computes to ten is invalid

**When** the arithmetic of §9.10 yields a check of 10
**Then** the number is invalid regardless of its ninth digit

### 9.12 The Uruguayan number

**When** the number `219999830019` is checked
**Then** the first two digits `21` lie between `01` and `22`
**And** digits three to eight `999983` are not all zero
**And** digits nine to eleven are `001`
**And** the weighted sum with the weights 4 3 2 9 8 7 6 5 4 3 2 is 310
**And** the check digit is the non-negative remainder of minus 310 divided by 11, which is 9
**And** the twelfth digit is 9, so the number is valid

### 9.13 The Venezuelan number

**When** the number `V-12.345.678-1` is checked
**Then** the kind digit for `V` is 1
**And** the checksum is 1 times 4 plus the weighted sum 138, that is 142
**And** the check digit is 11 minus 10, which is 1
**And** the stated check digit is 1, so the number is valid

### 9.14 A Venezuelan number with mixed punctuation is refused

**When** the number `V-12345.678-1` is checked
**Then** the pattern fails because full stops are only permitted when the hyphen form is used
consistently, and the number is invalid

### 9.15 An Uzbek number depends on the Party's shape

**Given** a Party whose organization flag is **set** and whose country is `Uzbekistan`
**When** the number `123456789` is written
**Then** it is accepted, because an organization's number is nine digits
**But given** the same number on a Party whose organization flag is cleared
**Then** it is refused, because a person's number is fourteen digits

### 9.16 A country with no check accepts anything

**Given** a Party whose country has neither a system-defined check nor a library module
**When** any number is written
**Then** it is accepted unchanged

### 9.17 The check runs against the commercial entity's country

**Given** an organization 100 whose country is `Belgium`, and a `delivery`-type child 103 whose
country is `France`
**When** a tax registration number is written on 103
**Then** it is validated against `Belgium`, the commercial entity's country

### 9.18 The failure message for a named Party

**Given** a Party named `Deco Addict` whose country is `Belgium`
**When** the number `BE0000000000` is written and fails
**Then** the message begins with a line feed and reads
`The VAT number [BE0000000000] for partner [Deco Addict] does not seem to be valid. ` followed by
a line feed and `Note: the expected format is BE0477472701`

### 9.19 The suppression flag

**Given** the reading context carries `no_vat_validation`
**When** an invalid number is written
**Then** it is stored normalised and no failure is raised

---

# 10. Duplicate detection

### 10.1 A twin with the prefixed spelling is found

**Given** a Party A with country `Greece` and number `EL123456783`
**When** a Party B is created with country `Greece` and number `123456783`
**Then** B's tax-registration twin is A, because the search looks for `123456783`, `GR123456783`
and `EL123456783`

### 10.2 A twin with the unprefixed spelling is found

**Given** a Party A with country `Belgium` and number `0477472701`
**When** a Party B is created with country `Belgium` and number `BE0477472701`
**Then** B's twin is A, because the search looks for both spellings

### 10.3 An archived twin is reported

**Given** an **archived** Party A with number `BE0477472701`
**When** a Party B is created with the same number
**Then** B's twin is A

### 10.4 A child never reports a twin

**Given** an organization 100 with number `BE0477472701` and a child 101 that inherited it
**Then** 101's twin is empty, because a Party with a parent never reports one

### 10.5 A descendant is never a twin

**Given** an organization 100 with number `BE0477472701` and a child 101 carrying the same number
**Then** 100's twin is empty, because descendants are excluded from the search

### 10.6 A one-character number is never checked

**Given** a Party whose number is `/`
**Then** its twin is empty

### 10.7 Duplicates are warnings, not refusals

**Given** a twin exists
**When** the Party is saved
**Then** the save succeeds and the warning block is shown

---

# 11. Telephone numbers

### 11.1 A Belgian national number rendered internationally

**Given** a Party whose country is `Belgium` (code `BE`, calling code thirty-two)
**When** the telephone `02 290 34 90` is formatted in international presentation form
**Then** the result is `+32 2 290 34 90`

### 11.2 The same number in strict international form

**Then** the result is `+3222903490`

### 11.3 The same number in national form

**Then** the result is `02 290 34 90`, because the parsed calling code equals the assumed one

### 11.4 A foreign number is never rendered nationally

**Given** the assumed country `Belgium`
**When** `+33 1 42 68 53 00` is formatted with the national form requested
**Then** the result is the international presentation form `+33 1 42 68 53 00`, because the parsed
calling code thirty-three differs from the assumed thirty-two

### 11.5 The double-zero repair

**Given** the assumed country `Belgium`
**When** `0032 2 290 34 90` is formatted
**Then** the first parse reports "too long", the two zeros are stripped, a plus sign is prepended,
and the result is `+32 2 290 34 90`

### 11.6 The missing-plus repair

**Given** the assumed country `Belgium`
**When** `32 2 290 34 90` is formatted
**Then** the first parse reports "too long", a plus sign is prepended, and the result is
`+32 2 290 34 90`

### 11.7 An impossible number fails with the right message

**Given** the assumed country `France`
**When** `02 290 34 90` is formatted with raising enabled
**Then** the operation fails with `Impossible number 02 290 34 90: not enough digits.`

### 11.8 The country resolution order

**Given** a record with no country field of its own but a Party reference whose country is
`Belgium`
**Then** the assumed country is `Belgium`
**And given** neither a country field nor a Party reference, the assumed country is the acting
company's country

### 11.9 Searching by number ignores punctuation

**Given** a Party whose telephone is stored as `02/290.34.90`
**When** a search is run for `2290`
**Then** the Party is found, because both sides have every character in the class "whitespace,
backslash, full stop, solidus, parentheses, hyphen" removed

### 11.10 The plus and double-zero prefixes are equivalent in a search

**Given** a Party whose telephone is stored as `0032485112233`
**When** a search is run for `+32485112233`
**Then** the Party is found
**And** the converse also holds

### 11.11 A short search term is refused

**When** a search is run for `12`
**Then** the operation fails with
`Please enter at least 3 characters when searching a Phone number.`

### 11.12 The form rewrites the number

**Given** a Party form with the country `Belgium`
**When** the operator types `02 290 34 90` into the telephone input and leaves it
**Then** the input shows `+32 2 290 34 90`

---

# 12. Blocking telephone numbers

### 12.1 Blocking normalises

**When** the number `02/290.34.90` is blocked with the acting user's country being `Belgium`
**Then** the stored number is `+3222903490`

### 12.2 Blocking twice is idempotent

**Given** the number of §12.1 is already blocked and active
**When** it is blocked again
**Then** no second record is created and the existing record is returned

### 12.3 Blocking a previously unblocked number reactivates it

**Given** an archived blocked-number record for `+3222903490`
**When** the number is blocked
**Then** the record's active flag is set and no new record is created

### 12.4 Unblocking a number that was never blocked creates an archived record

**When** `+3222903490` is unblocked and no record exists
**Then** a record is created with the active flag **cleared**

### 12.5 A reason becomes a note

**When** `+3222903490` is unblocked through the dialogue with the reason
`Asked to receive our next newsletters`
**Then** a rich-text paragraph `Unblock Reason: Asked to receive our next newsletters` appears in
the record's history

### 12.6 An unparseable number aborts the batch

**When** a batch containing `+3222903490` and `nonsense` is blocked
**Then** the whole operation fails with
`<the formatting error> Please correct the number and try again.` and neither record is created

### 12.7 Order is preserved

**When** a batch of three numbers is blocked, of which the second already exists
**Then** the returned records are in the order the caller listed them

### 12.8 Uniqueness

**When** two records with the same normalised number are created directly
**Then** the second fails with `Number already exists`

---

# 13. Number formatting

### 13.1 German, fixed point, grouped

**Given** the language German with grouping `[3,0]`, decimal separator the comma and thousands
separator the full stop
**When** the value one million two hundred and thirty-four thousand five hundred and sixty-seven
point eight nine one is formatted with the specification `%.2f` and grouping requested
**Then** the result is `1.234.567,89`

### 13.2 English (United States), same value

**Then** the result is `1,234,567.89`

### 13.3 French, same value

**Given** the thousands separator is a non-breaking space
**Then** the result is `1` then a non-breaking space then `234` then a non-breaking space then
`567,89`

### 13.4 Hindi, the four-group case

**Given** the language Hindi with grouping `[3,2,0]`, decimal separator the full stop and
thousands separator the comma
**When** the value twelve million three hundred and forty-five thousand six hundred and
seventy-eight point nine is formatted with `%.2f` and grouping requested
**Then** the result is `1,23,45,678.90`

### 13.5 A negative value keeps its sign outside the grouping

**Given** German
**When** minus one million two hundred and thirty-four thousand five hundred and sixty-seven point
eight nine one is formatted with `%.2f` and grouping requested
**Then** the result is `-1.234.567,89`

### 13.6 A value smaller than one group

**Given** German
**When** twelve point five is formatted with `%.2f` and grouping requested
**Then** the result is `12,50` with no separator

### 13.7 An integer conversion

**Given** German
**When** one million two hundred and thirty-four thousand five hundred and sixty-seven is
formatted with `%d` and grouping requested
**Then** the result is `1.234.567`

### 13.8 Grouping switched off

**Given** German
**When** the value of §13.1 is formatted with `%.2f` and grouping **not** requested
**Then** the result is `1234567,89`

### 13.9 An integer conversion with grouping switched off

**Given** German
**When** one million two hundred and thirty-four thousand five hundred and sixty-seven is
formatted with `%d` and grouping **not** requested
**Then** the result is `1234567`, completely untouched

### 13.10 A bad specification

**When** the specification does not begin with a percent sign
**Then** the operation fails with
`format() must be given exactly one %char format specifier`

### 13.11 An inactive language

**When** formatting is requested for a language that is not active
**Then** the operation fails with `The language <the language's name> is not installed.`

---

# 14. Bank accounts

### 14.1 Sanitisation

| Input | Sanitised |
|---|---|
| `BE71 0961 2345 6769` | `BE71096123456769` |
| `be71-0961-2345-6769` | `BE71096123456769` |
| `123-4567890-02` | `123456789002` |
| `A_1` | `A_1` |

### 14.2 Uniqueness per holder

**Given** a Party 100 holding an account `BE71 0961 2345 6769`
**When** a second account `be71-0961-2345-6769` is created for 100
**Then** the operation fails with
`The combination Account Number/Partner must be unique.`
**But** the same number may be created for a different Party

### 14.3 Searching by any spelling

**Given** the account of §14.2
**When** a search is run on the account number for `BE71 0961 2345 6769`
**Then** the account is found
**And** a search for `be710961-2345-6769` also finds it

### 14.4 The holder name defaults and follows

**Given** a Party named `Deco Addict` holding an account whose holder name is `Deco Addict`
**When** the Party is renamed `Deco Addict SA`
**Then** the account's holder name becomes `Deco Addict SA`
**But given** the holder name had been set to `Deco Addict Treasury`
**Then** it is left unchanged

### 14.5 Outgoing payments are disallowed by default

**When** an account is created with no explicit permission
**Then** the "send money" flag is cleared
**And** the colour is 1
**And** setting the flag changes the colour to 10

### 14.6 Deleting archives

**When** an account is deleted
**Then** the row still exists with the active flag cleared

### 14.7 The sanitised number cannot be written directly

**When** a payload writes only the sanitised number as `BE71096123456769`
**Then** the account number becomes `BE71096123456769` and the sanitised number is recomputed from
it

### 14.8 Find or create returns the closest match

**Given** a Party 100 whose commercial entity is itself, a child 101, and an account
`BE71 0961 2345 6769` held by 101
**When** the find-or-create procedure is called with that number, the Party 100 and the acting
company
**Then** the existing account held by 101 is returned and nothing is created

### 14.9 Find or create prefers the exact holder

**Given** the same number held by both 100 and 101
**When** the procedure is called with the Party 100
**Then** the account held by 100 is returned

### 14.10 Find or create refuses to create for one's own company

**Given** the Party of the acting company
**When** the procedure is called with a number that does not exist and creation for one's own
company is not allowed
**Then** the operation fails with
`Please add your own bank account manually: <the account number> (<the Party's display name>)`

### 14.11 A created account is not payable

**When** the procedure creates an account
**Then** the "send money" flag is cleared

---

# 15. Merging

### 15.1 The destination is the oldest active Party

**Given** three parties: A active created on the eleventh of April two thousand and twenty-three; B
active created on the second of September two thousand and twenty-one; C archived created on the
thirtieth of January two thousand and twenty-four
**When** a merge is started with no explicit destination
**Then** the destination is B

### 15.2 An explicit destination wins

**Given** the same three
**When** the destination is set to A
**Then** A survives and B and C are the sources

### 15.3 Four parties are refused

**When** four parties are merged
**Then** the operation fails with
`For safety reasons, you cannot merge more than 3 contacts together. You can re-open the wizard several times if needed.`

### 15.4 A parent and a child are refused

**Given** a Party 100 and its child 101
**When** they are merged
**Then** the operation fails with `You cannot merge a contact with one of his parent.`

### 15.5 Two user accounts are refused

**Given** two parties each carrying a user account, one of them archived
**When** they are merged
**Then** the operation fails with
`You cannot merge contacts linked to more than one user even if only one is active.`

### 15.6 Different addresses are refused for a non-administrator

**Given** two parties with different electronic mail addresses
**When** a directory maintainer merges them
**Then** the operation fails with
`All contacts must have the same email. Only the Administrator can merge contacts with different emails.`
**But when** an administrator merges them
**Then** the merge proceeds

### 15.7 Fewer than two parties is a silent no-operation

**When** one Party is merged
**Then** nothing happens and no failure is raised

### 15.8 References are rewritten

**Given** party 501 `Deco Addict` (active, created 2021-03-04) and party 502 `Deco Addict SA`
(active, created 2023-08-19), both with the address `contact@deco.example`; invoice
`INV/2022/00013` with three Journal Items on 501; invoice `INV/2024/00007` with two Journal Items
on 502; sales order `S00021` on 501 in all three of its Party columns; sales order `S00038` on 502
in all three; twelve messages on 501 and four on 502
**When** they are merged
**Then** the destination is 501
**And** `INV/2024/00007` and its two Journal Items point at 501
**And** `S00038` points at 501 in all three columns
**And** 501's history holds sixteen messages
**And** 502 no longer exists

### 15.9 Field values: the destination wins where it has a value

**Given** the parties of §15.8 where 502 has telephone `+32 2 290 34 90`, tax registration number
`BE0477472701`, reference `C0087`, language `en_US`; and 501 has telephone empty, tax registration
number empty, reference `C0042`, language `fr_FR`, street `Chaussée de Namur 40`
**When** they are merged
**Then** 501 ends with telephone `+32 2 290 34 90`, tax registration number `BE0477472701`,
reference `C0042`, language `fr_FR` and street `Chaussée de Namur 40`

### 15.10 Bank accounts are deduplicated by the sanitised number

**Given** 501 holding `BE71 0961 2345 6769`, and 502 holding both `BE71 0961 2345 6769` and
`BE68 5390 0754 7034`
**When** they are merged
**Then** 501 holds two accounts: the one it already had and `BE68 5390 0754 7034`
**And** 502's duplicate account row no longer exists

### 15.11 Tags collide harmlessly

**Given** 501 tagged `Prospect` and 502 tagged `Prospect` and `Wholesaler`
**When** they are merged
**Then** 501 is tagged `Prospect` and `Wholesaler`, each once

### 15.12 A company-dependent value is merged with the destination winning

**Given** 502 whose barcode for company 1 is `DA-502` and 501 whose barcode is empty
**When** they are merged
**Then** 501's barcode for company 1 is `DA-502`
**But given** 501's barcode for company 1 had been `DA-501`
**Then** it stays `DA-501`

### 15.13 The grouping query excludes rows with an empty grouping value

**Given** three parties: two named `Deco Addict` and `deco addict`, both with the address
`contact@deco.example`; and one named `Deco Addict` with no address
**When** a duplicate search groups by the electronic mail address and the name
**Then** exactly one group is produced, containing the first two

### 15.14 No criterion is refused

**When** a duplicate search is started with no criterion ticked
**Then** the operation fails with `You have to specify a filter for your selection.`

### 15.15 An exclusion filter drops a group

**Given** a group of two parties, one of which carries accounting entries, and the exclusion
filter for accounting entries switched on
**When** the duplicate search runs
**Then** the group is not offered

### 15.16 No accounting amount changes

**Given** the merge of §15.8
**Then** no journal entry is created, no amount changes, no reconciliation is broken and no
document changes state

---

# 16. Avatars

### 16.1 The generated initials avatar

**Given** a Party named `Deco Addict` created on the fifteenth of March two thousand and
twenty-four at nine hours thirty minutes zero seconds Coordinated Universal Time, with no stored
image and no user account, whose address type is `contact`
**When** the avatar is computed
**Then** the seed is `Deco Addict1710495000.0`
**And** the digest's first four hexadecimal characters are `4f1f`
**And** the hue is seventy-nine times three hundred and sixty divided by two hundred and
fifty-five, which rounds to 112
**And** the saturation is thirty-one times thirty divided by two hundred and fifty-five plus
forty, which rounds to 44
**And** the colour text is `hsl(112, 44%, 45%)`
**And** the initial is `D`
**And** the image document is exactly the single line given in
[calculations.md](calculations.md) §12.3 step 9

### 16.2 An organization with no image gets the company placeholder

**Given** a Party with the organization flag set, no stored image and no user account
**Then** the avatar is the shipped company picture, not a generated initial

### 16.3 A delivery address gets the truck placeholder

**Given** a Party with address type `delivery`, no stored image and no user account
**Then** the avatar is the shipped truck picture

### 16.4 An invoicing address gets the bill placeholder

**Then** the avatar is the shipped bill picture

### 16.5 An "other" address gets the puzzle placeholder

**Then** the avatar is the shipped puzzle-piece picture

### 16.6 A Party with an internal user always gets initials

**Given** a Party with the organization flag set, no stored image, but a **non-shared** user
account
**Then** the avatar is the generated initials image, not the company placeholder

### 16.7 A stored image wins

**Given** any Party with a stored image
**Then** every avatar size is the stored image resized

### 16.8 A nameless Party with no image

**Given** a Party with no name, no image, no user account and address type `other`
**Then** the avatar is the puzzle-piece placeholder
**And given** the same Party with address type `contact`, the avatar is the grey placeholder,
because the generated image requires a name

---

# 17. Archival and deletion

### 17.1 A Party with an active user cannot be archived

**Given** a Party carrying an active user account
**When** a user with write access on user accounts archives it
**Then** the operation fails with a redirecting warning whose message is
`You cannot archive contacts linked to an active user.\nYou first need to archive their associated user.\n\nLinked active users : <the names>` and whose button is labelled `Go to users`

### 17.2 The same without write access on user accounts

**Then** the operation fails with
`You cannot archive contacts linked to an active user.\nAsk an administrator to archive their associated user first.\n\nLinked active users :\n<the names>`

### 17.3 Archiving keeps every reference

**Given** an archived Party referenced by a posted invoice
**Then** the invoice still names it and every balance still aggregates it

### 17.4 A referenced country cannot be deleted

**Given** a Party whose country is `Belgium`
**When** `Belgium` is deleted
**Then** the deletion is refused by the storage engine

### 17.5 A Company with active users cannot be archived

**Given** a Company with three active users whose own company it is
**When** it is archived
**Then** the operation fails with
`The company <the company name> cannot be archived because it is still used as the default company of 3 users.`

### 17.6 Archiving a Company archives its branches

**Given** a Company with two branches and no active users anywhere
**When** it is archived
**Then** both branches are archived

---

# 18. Languages

### 18.1 The locale code cannot be changed

**When** an existing language's locale code is written to a different value
**Then** the operation fails with `Language code cannot be modified.`

### 18.2 A language used by an active user cannot be deactivated

**Then** the operation fails with
`Cannot deactivate a language that is currently used by users.`

### 18.3 A language used by an active Party cannot be deactivated

**Then** the operation fails with
`Cannot deactivate a language that is currently used by contacts.`

### 18.4 The setup language can never be deactivated

**Given** a language used by an archived user account
**Then** the operation fails with
`You cannot archive the language in which the application was setup as it is used by automated processes.`

### 18.5 The base language cannot be deleted

**Then** the operation fails with `Base Language 'en_US' can not be deleted.`

### 18.6 An active language cannot be deleted

**Then** the operation fails with
`You cannot delete the language which is Active!\nPlease de-activate the language first.`

### 18.7 The last language cannot be removed

**When** the only language would be removed
**Then** the operation fails with `At least one language must be active.`

### 18.8 Activating a long code shortens it and lengthens another

**Given** an inactive language with locale code `fr_FR` and address-bar code `fr`, and an inactive
language with locale code `fr_BE` and address-bar code `fr_BE`
**When** `fr_BE` is activated
**Then** `fr_FR`'s address-bar code becomes `fr_FR`
**And** `fr_BE`'s address-bar code becomes `fr`

### 18.9 A forbidden format directive is refused

**When** a date format containing a forbidden directive is written
**Then** the operation fails with
`Invalid date/time format directive specified. Please refer to the list of allowed directives, displayed when you edit a language.`

### 18.10 A mixed clock format is corrected

**When** a time format containing both a twenty-four-hour directive and a morning/afternoon
directive is entered in a form
**Then** the twenty-four-hour directive is replaced by the twelve-hour one
**And** a notification appears with the title
`Using 24-hour clock format with AM/PM can cause issues.` and the message
`Changing to 12-hour clock format instead.`

---

# 19. Currencies

### 19.1 A currency used by a Company cannot be deactivated

**Then** the operation fails with
`This currency is set on a company and therefore cannot be deactivated.`

### 19.2 Choosing an inactive currency for a Company activates it

**Given** an inactive currency
**When** it is written as a Company's currency
**Then** its active flag becomes set

### 19.3 The multi-currency group follows the active count

**Given** exactly one active currency
**Then** the multi-currency group is not granted
**When** a second currency is activated
**Then** the group is granted to every internal user
**When** the second is deactivated again
**Then** the group is withdrawn

### 19.4 The rounding factor must be positive

**When** a currency is written with a rounding factor of zero
**Then** the operation fails with `The rounding factor must be greater than 0!`

---

# 20. Companies

### 20.1 Creating a Company creates its Party

**When** a Company named `My Company` is created with no Party
**Then** a Party named `My Company` is created first, with the organization flag set
**And** the Company's Party is that record
**And** the creating user and the built-in super-user are both in the Company's accepted-users
list

### 20.2 A branch inherits the currency

**Given** a root Company whose currency is the euro
**When** a branch is created with no currency
**Then** its currency is the euro

### 20.3 A branch with a different currency is refused

**When** a branch is written with the dollar while its root uses the euro
**Then** the operation fails with
`The Currency of a subsidiary must be the same as it's root company.`

### 20.4 Changing the root's currency changes every branch

**Given** a root with two branches
**When** the root's currency is changed
**Then** both branches receive the new currency

### 20.5 The hierarchy cannot be changed

**When** a parent is written on an existing Company
**Then** the operation fails with `The company hierarchy cannot be changed.`

### 20.6 A Company cannot be duplicated

**Then** the operation fails with
`Duplicating a company is not allowed. Please create a new company instead.`

### 20.7 The name must be unique

**Then** a second Company with the same name fails with `The company name must be unique!`

### 20.8 The address writes back to the Company's own Party

**Given** a Company whose Party has a `contact`-type child with a divergent address
**When** the Company's street is written
**Then** the Company's **own** Party's street changes
**And** the displayed address is re-resolved from the `contact` address, which is the child's

### 20.9 The colour index

**Given** a root Company with identifier 7 whose Party's colour index is 0
**Then** the Company's colour index is 7
**And given** identifier 14, the colour index is 2
**And when** a colour index is written on a branch, the root's Party receives it

### 20.10 A Party that represents a Company must belong to it

**Given** the Party of Company X
**When** its company is written as Company Y
**Then** the operation fails with
`The company assigned to this partner does not match the company this partner represents.`

### 20.11 A Party's company must agree with its users

**Given** a Party with two user accounts belonging to different companies
**When** a company is written on it
**Then** the operation fails with
`The selected company is not compatible with the companies of the related user(s)`

### 20.12 Writing a Party's company cascades

**Given** a Party with two children
**When** its company is written
**Then** both children receive the same company

---

# 21. Countries, states and country groups

### 21.1 The code is upper-cased

**When** a country is created with the code `be`
**Then** the stored code is `BE`

### 21.2 Name and code uniqueness

**Then** a second country with the same name fails, and a second with the same code fails

### 21.3 A state code is unique only inside its country

**Given** a state coded `CA` in `United States`
**When** a state coded `CA` is created in `Canada`
**Then** the creation succeeds
**But when** a second `CA` is created in `United States`
**Then** it fails with `The code of the state must be unique by country!`

### 21.4 Searching countries by a two-letter term prefers the code

**Given** the countries `Belgium` (code `BE`), `Belize` (code `BZ`) and `Benin` (code `BJ`)
**When** a name search is run for `BE`
**Then** `Belgium` appears first

### 21.5 A state's display name and the reverse search

**Given** the state `California` in `United States`
**Then** its display name is `California (US)`
**And** a search for `California (US)` finds it
**And** a search for `California (United States)` also finds it

### 21.6 The state display name under the formatted context

**Then** it is `California \t --US--`

### 21.7 A country group code is upper-cased and unique

**When** a group is created with the code `sepa`
**Then** the stored code is `SEPA`
**And** a second group with that code fails with `The country group code must be unique!`

### 21.8 The flag path

**Given** the country `Belgium` with code `BE`
**Then** the flag path is the flag directory followed by `be.png`
**And given** the country `Réunion` with code `RE`, the path is the flag directory followed by
`fr.png`, because `RE` is one of the ten mapped overrides
**And given** `Antarctica` with code `AQ`, the path is empty

### 21.9 The country group codes of a country with no group

**Then** the computed list is a one-element list containing the empty string, not an empty list

---

# 22. Cities

### 22.1 The display name

**Given** a city `Brussels` with postal code `1000`
**Then** its display name is `Brussels (1000)`
**And given** no postal code, it is `Brussels`

### 22.2 Choosing a city cascades

**Given** a controlled city `New York` in `United States` with postal code `10001` and state
`New York`
**When** it is chosen on a Party
**Then** the Party's free-text city becomes `New York`, its postal code `10001` and its state
`New York`

### 22.3 Clearing a city on a saved record clears the three

**When** the controlled city is cleared on a saved Party
**Then** the free-text city, the postal code and the state all become empty

### 22.4 Changing the country clears the city

**Given** a Party with a controlled city in `United States`
**When** the country is changed to `Belgium`
**Then** the controlled city is cleared

---

# 23. Import

### 23.1 A state from the wrong country is repaired

**Given** an import row naming the country `Canada` and a state whose country is `United States`
but whose code is `ON`
**When** the import runs
**Then** the row's state becomes the `ON` state of `Canada`

### 23.2 A state with no counterpart is dropped

**Given** an import row naming the country `Belgium` and a state coded `CA` in `United States`
**When** the import runs
**Then** the row's state becomes empty

### 23.3 The verification service is not called during an import

**Given** a Company with verification switched on
**When** ten thousand parties with tax registration numbers are imported
**Then** no call is made to the verification service

### 23.4 Geocoding does not run during an import

**When** the resolution operation is called with the import flag in the context and no force flag
**Then** it returns immediately without contacting a provider

### 23.5 The tax-number check still runs during an import

**Given** an import row with an invalid tax registration number
**When** the import runs
**Then** that row fails with the standard failure message

---

# 24. Geocoding

### 24.1 A successful resolution

**Given** a Party with street `Chaussée de Namur 40`, postal code `1367`, city `Ramillies`,
country `Belgium`
**When** the resolution runs with the first provider
**Then** the query string is `Chaussée de Namur 40, 1367 Ramillies, Belgium`
**And** on a hit the latitude and longitude are written and the resolution date becomes today in
the acting user's time zone

### 24.2 The two-stage retry

**Given** a first attempt that returns nothing
**Then** a second attempt is made with the query string `1367 Ramillies, Belgium` — the street
dropped
**And** only if that also returns nothing is the Party counted as failed

### 24.3 The failure notification

**Given** two parties named `Test A` and `Deco Addict` that both fail
**Then** a danger notification is pushed with the title `Warning` and the message
`No match found for Test A, Deco Addict address(es).`

### 24.4 Moving the address resets the coordinates

**Given** a Party with a latitude and a longitude
**When** its city is written and neither coordinate is written in the same operation
**Then** both coordinates become zero
**And** the resolution date is **not** cleared

### 24.5 Writing the coordinates with the address does not reset them

**When** a write carries the city **and** both coordinates
**Then** the coordinates keep the written values

### 24.6 The second provider without a key

**When** the second provider is configured with no key and a resolution is attempted
**Then** the operation fails with a message beginning
`API key for GeoCoding (Places) required.`

### 24.7 A provider with no implementation

**When** the configured provider's technical name has no caller
**Then** the operation fails with
`Provider <the technical name> is not implemented for geolocation service.`

---

# 25. Public pages

### 25.1 An unpublished Party is not served

**Given** a Party whose published flag is cleared
**When** a public visitor requests its page
**Then** the not-found response is returned

### 25.2 A published Party is served

**Given** a Party whose published flag is set
**When** a public visitor requests its page at the canonical address
**Then** the page is rendered with the Party's image, contact block, display name and full
description

### 25.3 A non-canonical address redirects

**Given** a published Party whose canonical slug differs from the one requested
**Then** the visitor is redirected to the canonical address

### 25.4 An editor sees an unpublished page

**Given** a visitor holding the restricted-editor right
**When** an unpublished Party's page is requested
**Then** the page is served

### 25.5 Publishing leaves a history entry

**When** the published flag is set
**Then** an entry appears under the subtype `Partner published`
**And when** it is cleared, under `Partner unpublished`

---

# 26. Access and visibility

### 26.1 An internal user may read but not create

**Given** a user in the internal-user group only
**When** a Party is created
**Then** the operation is refused by the access-rights check

### 26.2 A directory maintainer may create

**Given** a user in the contact-creation group
**Then** creation, update and deletion of parties, tags, banks and bank accounts all succeed

### 26.3 A portal visitor sees only their own subtree

**Given** a portal visitor whose commercial entity is 100
**When** they list parties
**Then** they see 100 and its descendants and nothing else
**And** any attempt to write is refused

### 26.4 The multi-company rule on a Party

**Given** a user whose allowed companies are {1}
**And** parties: A with company 1, B with company 2, C with no company, D with company 2 but whose
shared-party flag is cleared because it carries an internal user
**When** the user lists parties
**Then** they see A, C and D — D because the rule's first clause exempts parties of internal users

### 26.5 The multi-company rule on a bank account

**Given** a user whose allowed companies are {1}
**And** accounts: one whose company is 1, one whose company is 2, one with no company
**Then** the user sees the first and the third

### 26.6 Editing another internal user's Party

**Given** a directory maintainer with **no** write access on user accounts
**When** they write on the Party of another internal user
**Then** the write is refused by the post-write access check

### 26.7 The blocked list is administrator-only

**Given** a directory maintainer
**When** they list blocked numbers
**Then** the operation is refused

---

# 27. The merge wizard's states

### 27.1 Opening from a selection starts in the selection state

**Given** two parties selected in the list
**When** the merge action is chosen
**Then** the wizard's state is `selection`, the party list holds both, and the destination is the
oldest active one

### 27.2 Opening from the menu starts in the option state

**Then** the state is `option` and the criteria are shown

### 27.3 Merging the last group finishes

**Given** one group remaining
**When** it is merged
**Then** the state becomes `finished`, the current group is cleared and the party list is emptied

### 27.4 Skipping deletes the group without merging

**When** a group is skipped
**Then** its parties are unchanged and the group no longer exists

### 27.5 Merging an empty party list finishes immediately

**Given** the state `selection` with an empty party list
**When** the merge button is pressed
**Then** the state becomes `finished` and nothing is merged

### 27.6 The automatic merge commits per group

**Given** five groups and a failure in the third
**When** the automatic merge runs
**Then** the first two are merged and committed and the remainder are not

---

# 28. Round-trip and consistency checks

### 28.1 The complete name survives a rename of the organization

**Given** an organization 100 named `Deco Addict` with a child 101 named `Douglas Fletcher`
**When** 100 is renamed `Deco Addict SA`
**Then** 101's commercial company name becomes `Deco Addict SA`
**And** 101's complete name becomes `Deco Addict SA, Douglas Fletcher`

### 28.2 Re-parenting recomputes the whole subtree

**Given** 100 with child 101 and grandchild 102
**When** 101 is re-parented under a different organization 200
**Then** 101's and 102's commercial entity become 200
**And** their commercial company names and complete names are recomputed

### 28.3 A cycle is refused at every level

**Given** 100 → 101 → 102
**When** 100 is given the parent 102
**Then** the operation fails with `You cannot create recursive Partner hierarchies.`

### 28.4 A tag cycle is refused

**When** a tag is given a descendant as its parent
**Then** the operation fails with `You can not create recursive tags.`

### 28.5 A tag's display name joins the whole chain

**Given** a tag `Consulting` whose parent is `Services`
**Then** its display name is `Services / Consulting`
**And** a search for `Services` also matches parties tagged `Services / Consulting`

### 28.6 The address rendering is stable across a save

**Given** a Party whose rendered address is the five-line result of §7.1
**When** it is saved and re-read
**Then** the rendered address is identical

### 28.7 Formatting a number and re-parsing it is stable

**Given** the number `+32 2 290 34 90`
**When** it is formatted to the strict international form and the result is re-parsed with the
same assumed country
**Then** the international presentation form is again `+32 2 290 34 90`

### 28.8 Sanitising a sanitised account number is idempotent

**Given** `BE71096123456769`
**When** it is sanitised again
**Then** the result is unchanged

### 28.9 The time-zone offset changes with the season

**Given** a Party whose time zone is `Europe/Brussels`
**When** the offset is read on the fifteenth of March
**Then** it is `+0100`
**When** it is read on the fifteenth of July
**Then** it is `+0200`

### 28.10 The generated avatar is stable once saved

**Given** a saved Party
**When** its avatar is computed twice
**Then** the two results are byte-identical
**But** the avatar of the same Party before it was saved differs, because the seed lacked the
creation timestamp
