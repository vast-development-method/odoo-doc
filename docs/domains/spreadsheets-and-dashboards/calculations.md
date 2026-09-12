# Calculations

Twenty calculation families. Each states its quantities by name, the order in which they are evaluated, what is rounded and how, and at least one worked numeric example carried to the last decimal the rule produces.

Rule references point at [`business-rules.md`](business-rules.md); the filter vocabulary is defined in [`global-filters.md`](global-filters.md); the document sections named here are specified in [`document-format.md`](document-format.md).

## 1. Conventions

| Convention | Statement |
|---|---|
| Money | Every monetary quantity produced by this domain is the sum of stored journal-item amounts in the company currency of the company the formula names. The domain performs no currency conversion of ledger amounts; the sum is carried at the stored precision and is not re-rounded |
| Rounding of sums | A sum of stored amounts is exact: no rounding step is inserted between the stored values and the cell. The presentation rounding is the number format of the cell, which shows as many decimal digits as the currency's decimal places and does not change the stored value |
| Rounding of quotients | Only two families divide: the currency-rate family (§13) and the date-serial family (§2). Both produce a decimal number and neither rounds; the cell shows it under whatever format is in force |
| Dates | A period boundary is a date without a time. A window boundary produced by a global filter is a moment with a time when the matched field carries a time, and a date when it does not (§17) |
| Empty answers | An aggregate over no record answers zero, never nothing. A lookup that cannot resolve its subject answers nothing and the cell shows the refusal message of the corresponding rule |
| Order of evaluation | Where the order matters it is stated as a numbered procedure. Where a formula is written on one line, evaluation is left to right with multiplication and division before addition and subtraction, and parentheses override both |

## 2. Dates and moments as numbers

A cell holds a date as a number. The origin of the numbering is the thirtieth of December 1899 at midnight, which is the number zero. One whole unit is one day.

### 2.1 A date

```formula
date number = ( seconds from the origin to midnight of the date ) ÷ 86400 seconds
```

The divisor is the number of seconds in a day: 24 × 60 × 60 = 86400.

**Worked example.** The thirtieth of December 1899 gives 0. The first of October 2023 is 45200 days after the origin, so the number is 45200. Carried to the last decimal the rule produces: 45200.0.

### 2.2 A moment

A moment carries a time, and the time is read in a named time zone.

1. Take the moment as stored, which is in coordinated universal time.
2. Express the same instant in the named time zone and read the offset of that zone from coordinated universal time at that instant, in seconds.
3. Add the offset to the instant's second count.
4. Subtract the origin's second count.
5. Divide by 86400 seconds.

```formula
moment number = ( seconds of the instant + offset of the zone in seconds − seconds of the origin ) ÷ 86400 seconds
```

Nothing is rounded. The fractional part is the time of day expressed as a fraction of a day.

**Worked examples.**

| Moment | Zone | Offset | Number |
|---|---|---|---|
| 30 December 1899, 00:00:00 | coordinated universal time | 0 seconds | 0.0 |
| 30 December 1899, 00:00:00 | a zone eight hours ahead | 28800 seconds | 28800 ÷ 86400 = 0.3333333333 |
| 1 October 2023, 12:00:00 | coordinated universal time | 0 seconds | 45200 + 43200 ÷ 86400 = 45200.5 |
| 1 October 2023, 12:00:00 | a zone eight hours ahead | 28800 seconds | 45200.5 + 0.3333333333 = 45200.8333333333 |

The third row in full: the instant is 45200 whole days plus 43200 seconds; 43200 ÷ 86400 = 0.5; the number is 45200.5.

## 3. The reader's locale

A locale tells the workbook how to write numbers, dates and times, where a week starts, and which character separates the arguments of a formula. It is derived from the reader's language record.

### 3.1 The eight parts

| Part | Source | Meaning |
|---|---|---|
| name | the language's name | Shown when a locale is chosen from a list |
| code | the language's code | The stable identifier of the locale |
| thousands separator | the language's thousands separator | The character grouping the integer digits |
| decimal separator | the language's decimal separator | The character before the fractional digits |
| date format | derived by §3.2 from the language's date pattern | The pattern a date cell is written with |
| time format | derived by §3.3 from the language's time pattern | The pattern a time is written with |
| formula argument separator | derived: a semicolon when the decimal separator is a comma, otherwise a comma | The character between two arguments of a formula |
| week start | the language's first day of the week, read as a number | Which day a week begins on |

The locale used for a dashboard is the one of the reader's own language, and the language falls back to the code `en_US` when the reader has none.

### 3.2 Converting a date pattern

1. Walk the source pattern from left to right and collect every escape: a percent sign followed by one character.
2. Translate each escape by this table, dropping any escape that is not in it:

| Escape | Produces | Meaning |
|---|---|---|
| `%Y` | `yyyy` | Four-digit year |
| `%y` | `yy` | Two-digit year |
| `%m` | `mm` | Two-digit month |
| `%b` | `mmm` | Abbreviated month name |
| `%B` | `mmmm` | Full month name |
| `%d` | `dd` | Two-digit day |
| `%a` | `ddd` | Abbreviated day name |
| `%A` | `dddd` | Full day name |

3. Choose the separator: the first character of the source pattern that is a slash, a hyphen or a space. When the source contains none of the three, the separator is a slash.
4. Join the translated parts with that one separator.

**Worked examples.**

| Source pattern | Result | Why |
|---|---|---|
| `%m/%d/%Y` | `mm/dd/yyyy` | First separator found is a slash |
| `%b/%a/%y` | `mmm/ddd/yy` | Abbreviated names |
| `%B/%A/%Y` | `mmmm/dddd/yyyy` | Full names |
| `%m %d %Y` | `mm dd yyyy` | First separator found is a space |
| `%m-%d-%Y` | `mm-dd-yyyy` | First separator found is a hyphen |
| `%m-%d/%Y` | `mm-dd-yyyy` | The first separator wins for every join |
| `%m.%d.%Y` | `mm/dd/yyyy` | A full stop is not one of the three, so the default slash is used |
| `%a, %Y.eko %bren %da` | `ddd yyyy mmm dd` | The literal text between escapes is dropped; the first space is the separator |
| `%w %x %Z %j %m %d %Y` | `mm dd yyyy` | Unlisted escapes are dropped |

### 3.3 Converting a time pattern

1. Walk the source and collect the escapes as above.
2. Translate by this table, dropping any escape that is not in it:

| Escape | Produces |
|---|---|
| `%H` | `hh` |
| `%I` | `hh` |
| `%M` | `mm` |
| `%S` | `ss` |

3. Note whether the source contains `%I` or `%p`; either marks a twelve-hour presentation.
4. Choose the separator: the first character of the source that is a colon or a space; when there is neither, a colon.
5. Join the translated parts with that separator, and append a space and the letter `a` when the presentation is twelve-hour.

**Worked examples.**

| Source pattern | Result |
|---|---|
| `%H:%M:%S` | `hh:mm:ss` |
| `%I:%M:%S` | `hh:mm:ss a` |
| `%H:%M:%S %p` | `hh:mm:ss a` |
| `%H %M %S` | `hh mm ss` |
| `%H %M:%S` | `hh mm ss` |
| `%H-%M-%S` | `hh:mm:ss` |
| `%H시 %M분 %S초` | `hh mm ss` |
| `%H:%M:%S %f %z` | `hh:mm:ss` |

### 3.4 The list of locales

The workbook may be given the whole list of locales, one per language record including inactive ones, each converted by the rule above.

## 4. The reading payload of a dashboard

The live branch of the dashboard reading route builds one structured answer. The four parts and the order in which they are produced:

1. **Snapshot.** The workbook text is parsed into a document. Its `settings` section is created when absent, and its `locale` key is set to the reader's locale of §3, overwriting whatever locale the workbook was stored with.
2. **Revisions.** An empty list. A dashboard is served as a finished snapshot; no revision stream is replayed on top of it.
3. **Default currency.** The currency of the reader's active company, as four parts: the code, the symbol, the number of decimal places and the position of the symbol, which is either `before` or `after`. When the company cannot be resolved this part is absent.
4. **Translation namespace.** The capability package that shipped this dashboard, read from the one external-identifier record that names this dashboard, taking one record; nothing when the dashboard was not shipped by a package. The client uses it to look up the translated form of the labels inside the workbook.

The sample branch produces a different answer with exactly two parts: the sample document under `snapshot`, and `is_sample` set to true.

**Worked example.** A dashboard whose stored workbook carries a locale with the code `fr_FR` is opened by a reader whose language is `en_US`, in a company whose currency is the euro with two decimal places and the symbol after the number. The answer carries a snapshot whose locale code is `en_US`, an empty revision list, a default currency of code `EUR`, symbol `€`, two decimal places and position `after`, and the namespace of the package that shipped the dashboard. The same dashboard opened by a reader whose language is `fr_FR` carries the code `fr_FR` instead; nothing about the stored dashboard has changed.

## 5. Period boundaries

Every accounting formula turns a period description into a first date and a last date. The description has a range type and up to four numbers: a year, a month, a quarter and a day. The company matters, because two of the four range types are anchored on the company's fiscal year.

### 5.1 The fiscal year of a company

A company carries a fiscal-year last day, a whole number, and a fiscal-year last month, a number from one to twelve. Given any date:

1. Build the candidate end: the day of the month equal to the fiscal-year last day, in the fiscal-year last month, in the year of the given date, clamped down to the last day of that month when the month is shorter.
2. When the candidate end is earlier than the given date, the fiscal year end is the same day and month one year later; otherwise the candidate end is the fiscal year end.
3. The fiscal year start is one day after the fiscal year end minus one year.

```formula
fiscal year end = the first occurrence of ( fiscal-year last day of fiscal-year last month ) that is not earlier than the given date
fiscal year start = fiscal year end − 1 year + 1 day
```

**Worked examples**, for a company whose fiscal year ends on the third of February:

| Given date | Fiscal year start | Fiscal year end |
|---|---|---|
| 5 March 2020 | 4 February 2020 | 3 February 2021 |
| 3 February 2020 | 4 February 2019 | 3 February 2020 |
| 4 February 2020 | 4 February 2020 | 3 February 2021 |

For a company whose fiscal year ends on the thirty-first of December, the fiscal year of any date in 2022 is the first of January 2022 to the thirty-first of December 2022.

### 5.2 The four range types

| Range type | First date | Last date |
|---|---|---|
| `year` | the start of the fiscal year selected in §5.3 | the end of that fiscal year |
| `month` | the first day of the given month of the given year | that first day plus one month minus one day |
| `quarter` | the first day of the month numbered ( quarter × 3 − 2 ) of the given year | that first day plus three months minus one day |
| `day` | the start of the fiscal year containing the given day | the given day itself |

The four stored range-type values are reproduced exactly.

### 5.3 Which fiscal year a year names

A year in a period description is a **fiscal** year, not a calendar year, and the rule that maps one to the other is:

1. When the company's fiscal year ends on the thirty-first of December, the year is used unchanged.
2. Otherwise the year is increased by one.
3. A reference date is built: the fiscal-year last day, clamped to the length of the month, in the fiscal-year last month, in that year.
4. The window is the fiscal year containing that reference date, by §5.1.

```formula
reference year = given year + ( 0 when the fiscal year ends on 31 December, otherwise 1 )
reference date = ( fiscal-year last day clamped to the month length ) of ( fiscal-year last month ) of ( reference year )
window = the fiscal year containing the reference date
```

**Worked example one.** A company whose fiscal year ends on the thirty-first of December, period `year` 2022. The reference year stays 2022, the reference date is 31 December 2022, and the window is 1 January 2022 to 31 December 2022.

**Worked example two.** A company whose fiscal year ends on the third of February, period `year` 2022. The reference year becomes 2023, February 2023 has 28 days so the third is unclamped, the reference date is 3 February 2023, and the window is 4 February 2022 to 3 February 2023. A journal item dated 2 January 2023 therefore counts towards the year 2022, and one dated 2 April 2022 counts too.

**Worked example three.** A company whose fiscal year ends on the twenty-ninth of February, period `year` 2022. The reference year becomes 2023; February 2023 has 28 days, so the day is clamped to 28; the reference date is 28 February 2023; the window is 1 March 2022 to 28 February 2023.

### 5.4 Worked examples for the other three range types

| Description | Company fiscal year end | First date | Last date |
|---|---|---|---|
| `month`, year 2022, month 7 | any | 1 July 2022 | 31 July 2022 |
| `month`, year 2024, month 2 | any | 1 February 2024 | 29 February 2024 |
| `quarter`, year 2022, quarter 3 | any | 1 July 2022 | 30 September 2022 |
| `quarter`, year 2022, quarter 1 | any | 1 January 2022 | 31 March 2022 |
| `day`, year 2022, month 7, day 2 | 31 December | 1 January 2022 | 2 July 2022 |
| `day`, year 2022, month 2, day 4 | 3 February | 4 February 2022 | 4 February 2022 |

The last row is the edge worth noticing: on the first day of a fiscal year, a daily period is a window of exactly one day.

## 6. The journal-item selection rule

Every accounting formula, and the cell audit action, select journal items with the same rule. It takes: a period description, an optional company identifier, either a list of account codes or a list of account-tag identifiers, an unposted flag, an optional list of partner identifiers, and a flag saying whether an empty code list falls back to the payable and receivable accounts.

### 6.1 The procedure

1. **Company.** The company is the given identifier when there is one, otherwise the acting company. Everything below is evaluated for that one company.
2. **Window.** The first date and the last date are computed by §5 for that company.
3. **Period condition.** Two alternatives joined by disjunction:
   - the item's account includes the initial balance **and** the item's date is on or before the last date;
   - the item's account does not include the initial balance **and** the item's date is on or after the first date **and** on or before the last date.

   This is what makes a balance-sheet account cumulative from the beginning of the ledger and a profit-and-loss account confined to the period.
4. **Account condition**, by exactly one of three branches:
   - **Tags supplied.** Each tag identifier is read as a whole number. A non-empty list gives: the item's account carries at least one of those tags. An empty list matches nothing at all.
   - **Codes supplied.** Empty entries are dropped. When at least one code remains, the condition is the disjunction over the codes of "the account's code begins with this code", matched exactly as written including letter case, evaluated over the accounts of the company; the matching accounts are collected and the condition becomes "the item's account is one of them". When no code remains and the fallback is off, the whole selection matches nothing and the procedure stops here. When no code remains and the fallback is on, the accounts are those whose type is `liability_payable` or `asset_receivable` in that company.
   - **Neither supplied.** Nothing matches.
5. **Posted condition.** When the unposted flag is on: the item's entry is in any state other than `cancel`. When it is off: the item's entry is in the state `posted`.
6. **Company condition.** The item's company is the company of step 1.
7. **Conjunction.** The account condition, the period condition, the company condition and the posted condition are joined by conjunction, in that order.
8. **Partner condition.** The partner identifiers are read as whole numbers and empty entries dropped. When at least one remains, a further conjunct is added: the item's partner is one of them. An empty list adds nothing, so the selection is not narrowed by partner.

The three stored values `liability_payable`, `asset_receivable`, `posted` and `cancel` are reproduced exactly.

### 6.2 What an empty code list means

| Caller | Fallback | Empty code list selects |
|---|---|---|
| Total debit, total credit, total balance | off | nothing; the answer is zero |
| Tagged balance | not applicable; the tag branch is taken | nothing when the tag list is empty |
| Residual amount | on | every payable and every receivable account of the company |
| Partner balance | on | the same |
| Cell audit action | on | the same |

This is rule [SD-055](business-rules.md#sd-055).

### 6.3 Worked example

A company whose fiscal year ends on the thirty-first of December holds two accounts: `sp1234566`, of type income, which does not include the initial balance, and `sp1234577`, of type expense, which does not either. One entry dated 2 April 2022 debits `sp1234566` by 500.00 and credits `sp1234577` by 500.00. A request for the period `year` 2022, codes `sp1234566` and `sp1234577`, unposted included, selects:

- window 1 January 2022 to 31 December 2022;
- period condition: both accounts exclude the initial balance, so only items dated within the window;
- account condition: the two accounts whose codes begin with those two strings;
- posted condition: entries not cancelled;
- company condition: the one company.

Both items are selected.

A second request with the code `sp1234` alone selects the same two items, because both codes begin with `sp1234`. A third request with the codes `sp1234` and `sp1234` again selects the same two items: duplicate prefixes cannot double-count, because the condition selects accounts, not items, and an account is selected once.

## 7. Total debit, total credit and total balance

### 7.1 The three cell formulas

| Formula name | Answer |
|---|---|
| `ODOO.DEBIT` | The sum of the debit amounts of the selected journal items |
| `ODOO.CREDIT` | The sum of the credit amounts of the selected journal items |
| `ODOO.BALANCE` | The debit sum minus the credit sum, both taken over the same selection |

All three take the same five arguments, in this order: account codes as one text, the period, a year offset defaulting to zero, an optional company identifier, and an unposted flag defaulting to false.

### 7.2 Preparing the arguments

1. The account-code text is split on commas, each part is trimmed of surrounding blank characters, and the parts are sorted. Sorting has no effect on the answer; it makes two requests that differ only in the order of their codes identical, so that they share one round trip.
2. The offset is read as a number, defaulting to zero.
3. The period is parsed by §10.
4. The company identifier is read as a number, or left unset when the argument is absent.
5. The unposted flag is read as a logical value.
6. The offset is added to the period's year. When the resulting year is below 1900, the cell shows the message of rule [SD-048](business-rules.md#sd-048) and nothing is requested.

```formula
requested year = year of the parsed period + offset
```

### 7.3 The sums

```formula
total debit = sum over the selected journal items of ( debit amount )
total credit = sum over the selected journal items of ( credit amount )
total balance = total debit − total credit
```

A selection that matches no item gives zero for each sum, not nothing. No rounding step is applied: the sums are exact sums of stored amounts.

### 7.4 The number format

The cell's number format is the format derived from the currency of the company named in the arguments, by §13.3. When no company was named and no default currency is attached to the workbook, or the derivation yields nothing, the format is the reproduced pattern `#,##0.00`.

### 7.5 Worked examples

Starting records: the company of §6.3, with the entry of 2 April 2022 (debit 500.00 on `sp1234566`, credit 500.00 on `sp1234577`), and a second company holding accounts `sp99887755` and `sp99887766` with an entry dated 2 February 2022 for 1500.00.

| Request | Total debit | Total credit | Total balance |
|---|---|---|---|
| Codes `sp1234566`, period `year` 2022, unposted included | 500.00 | 0.00 | 500.00 − 0.00 = 500.00 |
| Codes `sp1234566` and `sp1234577`, same period | 500.00 | 500.00 | 500.00 − 500.00 = 0.00 |
| Code `sp1234`, same period | 500.00 | 500.00 | 0.00 |
| Codes `sp1234` and `sp1234`, same period | 500.00 | 500.00 | 0.00 |
| Code `sp1234566`, period `year` 2021 | 0.00 | 0.00 | 0.00 |
| Code `10000000000`, period `year` 2022 | 0.00 | 0.00 | 0.00 |
| No code at all, period `year` 2022 | 0.00 | 0.00 | 0.00 |
| Code `sp1234566` for the first company and `sp99887755` for the second, named explicitly | 500.00 and 1500.00 respectively | 0.00 and 0.00 | 500.00 and 1500.00 |

**A mixed example that shows the initial-balance rule.** Change `sp1234566` to the type `asset_receivable`, so that it now includes the initial balance, and add an entry dated 2 July 2000 debiting `sp1234566` by 555.00 and crediting `sp1234577` by 555.00. For the period `year` 2022 with both codes:

```formula
total debit  = 555.00 ( the 2000 item on the balance-sheet account, still inside "on or before 31 December 2022" )
             + 500.00 ( the 2022 item on the same account )
             = 1055.00
total credit = 500.00 ( the 2022 item on the profit-and-loss account )
             = 500.00
total balance = 1055.00 − 500.00 = 555.00
```

The 555.00 credit of the year 2000 is excluded because it sits on an account that does not include the initial balance and 2 July 2000 is outside the window.

**A shifted fiscal year.** Set the company's fiscal year to end on the third of February, and add an entry dated 2 January 2023 debiting `sp1234566` by 1000.00. For the period `year` 2022, the window is 4 February 2022 to 3 February 2023, and:

```formula
total debit = 500.00 ( 2 April 2022 ) + 1000.00 ( 2 January 2023 ) = 1500.00
```

**Quarter, month and day.** Add an entry dated 2 July 2022 debiting and crediting `sp1234566` by 777.00, with the account left as an income account. Then:

| Period | Total debit | Total credit |
|---|---|---|
| `quarter` 3 of 2022 | 777.00 | 777.00 |
| `month` 7 of 2022 | 666.00 for an entry of 666.00 on the same day | 666.00 |
| `day` 2 July 2022, account of type income | 555.00 for an entry of 555.00 that day | 555.00 |

With the account changed to `asset_receivable`, so that it includes the initial balance, the same three periods each add the 500.00 debit of 2 April 2022 that precedes them:

| Period | Total debit | Total credit |
|---|---|---|
| `quarter` 3 of 2022 | 777.00 + 500.00 = 1277.00 | 777.00 |
| `month` 7 of 2022 | 777.00 + 500.00 = 1277.00 | 777.00 |
| `day` 2 July 2022 | 777.00 + 500.00 = 1277.00 | 777.00 |
| `year` 2025 with no item after 2022 | 500.00 | 0.00 |

**The posted flag.** With `sp1234566` of type `asset_receivable` holding the 500.00 debit of 2 April 2022 in a posted entry:

| Added entry | Unposted included | Total debit | Total credit |
|---|---|---|---|
| A cancelled entry of 10000000.00 both ways | no | 0.00 | 0.00 |
| The same cancelled entry | yes | 500.00 | 0.00 |
| A draft entry of 888.00 both ways, then posted | no | 888.00 | 888.00 |
| The same, with the earlier draft 500.00 entry still unposted | yes | 888.00 + 500.00 = 1388.00 | 888.00 |

A cancelled entry is excluded even when unposted entries are asked for: the unposted flag widens the selection to every state except `cancel`, it does not widen it to every state.

**The first day of a shifted fiscal year.** With the fiscal year ending on the third of February, entries of 111.00 dated 3 February 2022 and 423.00 dated 4 February 2022, both on `sp1234566`: the period `day` 4 February 2022 has the window 4 February 2022 to 4 February 2022 and gives a total debit of 423.00 and a total credit of 423.00. The entry of the third of February belongs to the previous fiscal year and is excluded.

## 8. Residual amount

### 8.1 The formula

`ODOO.RESIDUAL` answers the sum of the residual amounts of the selected journal items. Its five arguments are the same as those of §7, except that the account codes and the period are both optional.

```formula
residual amount = sum over the selected journal items of ( residual amount of the item )
```

The selection is the one of §6 **with the payable-and-receivable fallback on**, so a call with no code covers every payable and every receivable account of the company. A missing or unreadable period is replaced by the current calendar year before parsing. A selection matching no item answers zero. When the lookup itself answers nothing, the cell shows the message of rule [SD-050](business-rules.md#sd-050).

### 8.2 Worked example

Starting records, in one company whose fiscal year ends on the thirty-first of December:

| Record | Date | Effect |
|---|---|---|
| Entry A | 2 February 2022 | Debit 1500.00 on the receivable account, credit 1500.00 on the revenue account |
| Entry B | 2 February 2023 | Debit 2500.00 on the expense account, credit 2500.00 on the payable account |
| Payment C, received from a customer | 10 February 2022 | 150.00, which credits the receivable account |
| Payment D, paid to a customer | 10 February 2023 | 250.00, which debits the receivable account |

All four are posted. Payment C is reconciled against Entry A's receivable item, so that item's residual falls from 1500.00 to 1350.00 and Payment C's own item has a residual of 0.00. Payment D is unreconciled and carries a residual of 250.00; Entry B's payable item is unreconciled and carries a residual of −2500.00.

| Request | Arithmetic | Answer |
|---|---|---|
| `year` 2023, no code, posted only | 1500.00 − 150.00 − 2500.00 + 250.00 | −900.00 |
| `year` 2023, code of the receivable account, posted only | 1500.00 − 150.00 + 250.00 | 1600.00 |
| `year` 2022, no code, unposted included | 1500.00 − 150.00 | 1350.00 |
| `quarter` 4 of 2022, no code, posted only | 1500.00 − 150.00 | 1350.00 |
| `quarter` 1 of 2023, no code, posted only | 1500.00 − 150.00 − 2500.00 + 250.00 | −900.00 |
| `day` 1 February 2022, no code, posted only | no item in the window | 0.00 |
| `day` 2 February 2022, no code, posted only | 1500.00 − 150.00 | 1350.00 |

The last two rows are the ones that show the cumulation rule at work. Both payable and receivable accounts include the initial balance, so the window's first date is irrelevant to them: only the last date matters. On 1 February 2022 nothing has yet been booked; on 2 February 2022 Entry A has, and its residual already reflects the reconciliation with a payment dated eight days later, because a residual is a current property of the item and not a property of the window.

## 9. Partner balance

### 9.1 The formula

`ODOO.PARTNER.BALANCE` answers the sum of the balances of the selected journal items, narrowed to a list of partners. Its six arguments are: partner identifiers as one text, account codes as one text, the period, a year offset, an optional company identifier and an unposted flag.

```formula
partner balance = sum over the selected journal items of ( debit amount − credit amount )
```

1. The partner text is split on commas and each part is read as a number, then the numbers are sorted.
2. Empty entries are dropped. **When nothing remains, the answer is zero and no ledger query is made at all**; the formula does not fall back to every partner. This is rule [SD-053](business-rules.md#sd-053).
3. Otherwise the selection of §6 is built with the payable-and-receivable fallback on and with the partner condition, and the balances are summed.

A missing or unreadable period is replaced by the current calendar year. When the lookup answers nothing, the cell shows the message of rule [SD-051](business-rules.md#sd-051).

### 9.2 Worked example

Starting records, one company, fiscal year ending on the thirty-first of December:

| Record | Date | Partner | Effect |
|---|---|---|---|
| Entry A, posted | 2 February 2022 | Partner A | Debit 1500.00 receivable, credit 1500.00 revenue |
| Entry B, posted | 2 February 2023 | Partner A | Debit 2500.00 expense, credit 2500.00 payable |
| Entry C, draft | 2 February 2023 | Partner B | A copy of Entry B |

| Request | Arithmetic | Answer |
|---|---|---|
| Partner A, `year` 2023, no code, posted only | 1500.00 − 2500.00 | −1000.00 |
| Partner A, `year` 2023, receivable code, posted only | 1500.00 | 1500.00 |
| Partner A, `year` 2022, no code, unposted included | 1500.00 | 1500.00 |
| Partner A, `quarter` 4 of 2022, no code, unposted included | 1500.00 | 1500.00 |
| Partner A, `quarter` 1 of 2023, no code, unposted included | 1500.00 − 2500.00 | −1000.00 |
| Partner A, `day` 1 February 2022, no code, unposted included | nothing in the window | 0.00 |
| Partner A, `day` 2 February 2022, no code, unposted included | 1500.00 | 1500.00 |
| Partner B, `year` 2023, no code, posted only | Entry C is a draft | 0.00 |
| Empty partner list, any period | not queried | 0.00 |

The revenue and expense items never enter the sum when no code is given, because the fallback selects only payable and receivable accounts.

## 10. Reading the period argument

The period argument of the six accounting formulas is a text or a date, and it is read by trying four shapes in a fixed order. The first that succeeds wins.

### 10.1 The precedence

| Order | Shape | Recognised when | Produces |
|---|---|---|---|
| 1 | Quarter | The trimmed text matches the letter `q` or `Q`, a digit from 1 to 4, a slash and four digits | range type `quarter`, with that year and that quarter |
| 2 | Month | Either the argument is a number whose format contains a month marker and no day marker — in which case the number is read as a date and its year and month are taken — or the trimmed text matches an optional leading zero, a month number from 1 to 12, a slash and four digits | range type `month`, with that year and that month |
| 3 | Year | The argument read as a number is below 3000 | range type `year`, with that number as the year |
| 4 | Day | Anything else that reads as a number | range type `day`, with the year, month and day of the date that number denotes |

An argument that fails all four — because reading it as a number raises — makes the cell show the message of rule [SD-047](business-rules.md#sd-047), whose text quotes the three accepted written shapes.

### 10.2 Why the threshold is 3000

A bare number below 3000 is treated as a year, so that an author may write `2022` and mean the year 2022. A number of 3000 or more is treated as a date serial number; 3000 is the eighteenth of March 1908, so the convention costs the author nothing they could otherwise have expressed.

### 10.3 Worked examples

| Argument | Shape chosen | Result |
|---|---|---|
| `Q1/2022` | Quarter | `quarter`, year 2022, quarter 1 |
| `q4/2019` | Quarter | `quarter`, year 2019, quarter 4 |
| `12/2022` | Month | `month`, year 2022, month 12 |
| `07/2022` | Month | `month`, year 2022, month 7 |
| `2022` | Year | `year`, year 2022 |
| a cell holding the date 21 December 2022 with a full date format | Day | `day`, year 2022, month 12, day 21 |
| a cell holding a date with a month-and-year format only | Month | `month`, with that year and month |
| `44916`, a date serial number | Day | `day`, 21 December 2022 |
| `2999` | Year | `year`, year 2999 |
| `3000` | Day | `day`, 18 March 1908 |

### 10.4 The offset and the lower bound

After the period has been read, the year offset is added to its year. The check of rule [SD-048](business-rules.md#sd-048) is made **after** the addition, immediately before the request, so an offset that drags a valid year below 1900 is refused as well.

**Worked example.** Period `2022`, offset −200: the requested year is 2022 − 200 = 1822, which is below 1900, and the cell shows the refusal naming 1822. Period `2022`, offset −122: the requested year is 1900, which is accepted.

## 11. Tagged balance

### 11.1 The formula

`ODOO.BALANCE.TAG` answers the sum of the balances of the journal items whose account carries at least one of the given account tags. Its five arguments are: tag identifiers as one text, the period, a year offset, an optional company identifier and an unposted flag.

1. The tag text is split on commas, each part read as a number, and the numbers sorted.
2. Empty entries are dropped. When nothing remains the answer is zero and no ledger query is made; rule [SD-054](business-rules.md#sd-054).
3. Otherwise the selection of §6 is built through the tag branch — there is **no** payable-and-receivable fallback for this formula — and the balances are summed.

```formula
tagged balance = sum over the selected journal items of ( debit amount − credit amount )
```

A missing or unreadable period is replaced by the current calendar year. When the lookup answers nothing, the cell shows the message of rule [SD-052](business-rules.md#sd-052).

### 11.2 Worked example

Starting records, one company, fiscal year ending on the thirty-first of December. Tag one is carried by the receivable account and by the revenue account; tag two is carried by the expense account.

| Record | Date | Effect |
|---|---|---|
| Entry A, posted | 1 January 2025 | Debit 100.00 receivable, debit 50.00 revenue, credit 150.00 expense |
| Entry B, draft | 1 January 2024 | Debit 100.00 receivable, credit 100.00 expense |

| Request | Arithmetic | Answer |
|---|---|---|
| `year` 2025, tag one, posted only | 100.00 (receivable, cumulative to 31 December 2025) + 50.00 (revenue, inside 2025) | 150.00 |
| `year` 2025, tags one and two, posted only | 150.00 + ( −150.00 ) | 0.00 |
| `year` 2024, tag one, posted only | the posted entry is dated 2025, outside "on or before 31 December 2024" | 0.00 |
| `year` 2024, tag one, unposted included | the draft receivable item of 100.00 | 100.00 |

## 12. Fiscal-year boundaries and account groups

### 12.1 The two fiscal-year formulas

| Formula name | Arguments | Answer |
|---|---|---|
| `ODOO.FISCALYEAR.START` | a day, an optional company identifier | The first date of the fiscal year containing that day |
| `ODOO.FISCALYEAR.END` | the same | The last date of that fiscal year |

The day argument is read as a date. The company is the given one, or the acting company when none is given. The boundaries are those of §5.1. Both answers are returned as date serial numbers by §2.1, and the cell's number format is the reader's date format.

When the company identifier names no company, the lookup answers nothing and the cell shows the message of rule [SD-049](business-rules.md#sd-049).

**Worked examples**, for a company whose fiscal year ends on the third of February:

| Day | Start | End |
|---|---|---|
| 5 March 2020 | 4 February 2020 | 3 February 2021 |
| 3 February 2020 | 4 February 2019 | 3 February 2020 |
| 4 February 2020 | 4 February 2020 | 3 February 2021 |

**Worked example with two companies.** The acting company's fiscal year ends on the seventh of June; a second company's ends on the third of February. Asked for the day 4 February 2020, the second company answers 4 February 2020 to 3 February 2021 and the acting company answers 8 June 2019 to 7 June 2020. Asked with a company identifier that names no company, the answer for that request alone is nothing, and the other requests in the same round trip are unaffected.

### 12.2 The account-group formula

`ODOO.ACCOUNT.GROUP` takes one account type and answers the codes of the accounts of that type in the acting company, joined by commas with no space.

1. The accounts of the acting company are grouped by account type.
2. For each requested type, the codes of its accounts are collected.
3. A type with no account gives an empty list, and therefore an empty text.
4. The answers keep the order in which the types were requested, whatever order the grouping produced.

The eighteen account types, with the label each carries in the argument helper:

| Stored value | Label |
|---|---|
| `asset_receivable` | "Receivable" |
| `asset_cash` | "Bank and Cash" |
| `asset_current` | "Current Assets" |
| `asset_non_current` | "Non-current Assets" |
| `asset_prepayments` | "Prepayments" |
| `asset_fixed` | "Fixed Assets" |
| `liability_payable` | "Payable" |
| `liability_credit_card` | "Credit Card" |
| `liability_current` | "Current Liabilities" |
| `liability_non_current` | "Non-current Liabilities" |
| `equity` | "Equity" |
| `equity_unaffected` | "Current Year Earnings" |
| `income` | "Income" |
| `income_other` | "Other Income" |
| `expense` | "Expenses" |
| `expense_depreciation` | "Depreciation" |
| `expense_direct_cost` | "Cost of Revenue" |
| `off_balance` | "Off-Balance Sheet" |

**Worked examples.**

| Request | Answer |
|---|---|
| No type at all | an empty list |
| `income_other`, with one such account whose code is `450000` | one entry, the list holding `450000` |
| `income_other`, after every such account has been deleted | one entry, an empty list |
| A value that is not an account type | one entry, an empty list |
| `income_other`, with three such accounts whose codes are `123`, `450000` and `789` | one entry holding those three codes |
| `income` then `income_other` | two entries, the first for `income` and the second for `income_other`, in that order |

The typical use is to feed the answer straight into the account-code argument of §7: a cell holding the group's codes is referred to by a balance formula, so that adding an account of that type to the chart of accounts widens the balance without the workbook being edited.

## 13. Currency rates and the currency number format

### 13.1 The rate formula

`ODOO.CURRENCY.RATE` takes a source currency code, a target currency code, an optional date and an optional company identifier, and answers the rate that converts one unit of the source into the target.

1. When either code is empty, the answer is nothing.
2. Each code is looked up among the currencies by name, including currencies that are not active.
3. When either lookup finds nothing, the answer is nothing.
4. The company is the given one, or the acting company.
5. The date is the given one, or today in the acting user's time zone.
6. The answer is the conversion rate from the source to the target for that company on that date.

```formula
rate from source to target = ( rate of the target against the company's reference currency on the date )
                           ÷ ( rate of the source against the company's reference currency on the date )
```

When the answer is nothing the cell shows the message of rule [SD-038](business-rules.md#sd-038).

### 13.2 Worked examples

A company whose currency is the euro. Rates against it: the dollar at 1.5 and the Canadian dollar at 1.2, both in force now; on 11 November 2021 the dollar at 1.8 and the Canadian dollar at 1.9.

| Request | Arithmetic | Answer |
|---|---|---|
| Dollar to euro, no date | 1 ÷ 1.5 | 0.6666666667 |
| Euro to dollar, no date | 1.5 ÷ 1 | 1.5000000000 |
| Dollar to Canadian dollar, no date | 1.2 ÷ 1.5 | 0.8000000000 |
| Dollar to euro on 11 November 2021 | 1 ÷ 1.8 | 0.5555555556 |
| Euro to dollar on 11 November 2021 | 1.8 ÷ 1 | 1.8000000000 |
| Dollar to Canadian dollar on 11 November 2021 | 1.9 ÷ 1.8 | 1.0555555556 |
| Any pair where a code names no currency | not computed | nothing, and the cell shows the refusal |
| Any pair where a code is empty | not computed | nothing |

Nothing is rounded; the figures above are shown to ten decimal places because that is where the repeating expansions become legible.

**Company argument.** With a second company whose reference currency differs and which holds its own rate of 0.5 for the dollar, the dollar-to-euro rate asked for that company is 1 ÷ 0.5 = 2.0000000000, while the same question asked without a company uses the acting company and gives 0.6666666667.

**Time-zone sensitivity.** The date defaults to *today in the acting user's time zone*, and a rate created without an explicit date is dated the same way. At the instant 21:00 coordinated universal time on 1 January 2020, a user in a zone eleven hours ahead is already on 2 January, so a rate that user created is dated 2 January and is the one that user's formulas find; a user in coordinated universal time is still on 1 January and finds the rate dated 1 January. Two readers of the same workbook can therefore see two different rates at the same instant. This is recorded as a **compatibility finding**: a corrected behaviour would resolve the default date in the company's time zone rather than the reader's, so that a financial figure does not depend on where the reader sits. The observed behaviour is specified here because a rebuild that changed it would produce different numbers for the same workbook.

### 13.3 The number format of a currency

A currency gives four parts: a code, a symbol, a number of decimal places and a position, which is `before` or `after`. The number format is built from three of them — the symbol, the position and the decimal places — as follows:

1. The number pattern is `#,##0`, followed by a full stop and as many zeros as the number of decimal places, when that number is not zero.
2. The symbol expression is the symbol wrapped in the marker `[$` and the closing bracket.
3. When the position is `before`, the symbol expression precedes the number pattern; otherwise it follows it.

```formula
number pattern = "#,##0" + ( "." + as many "0" as the decimal places, when the decimal places are not zero )
format = symbol expression + number pattern   ( position before )
format = number pattern + symbol expression   ( position after )
```

**Worked examples.**

| Currency | Decimal places | Position | Format |
|---|---|---|---|
| Symbol `€` | 2 | `after` | `#,##0.00[$€]` |
| Symbol `$` | 2 | `before` | `[$$]#,##0.00` |
| Symbol `¥` | 0 | `before` | `[$¥]#,##0` |
| Symbol `د.ك` | 3 | `after` | `#,##0.000[$د.ك]` |

The company-currency lookup that feeds this rule answers the four parts for a given company, or nothing when the identifier names no company, in which case the cell shows the message of rule [SD-039](business-rules.md#sd-039).

**Worked example of the lookup.** A company whose currency is the euro answers code `EUR`, symbol `€`, two decimal places, position `after`. A company whose currency is the dollar answers code `USD`, symbol `$`, two decimal places, position `before`. An identifier naming no company answers nothing.

## 14. The number format of a list cell

A cell reading a field of a data-bound list takes its number format from the field's type.

| Field type | Format | Note |
|---|---|---|
| Whole number | `0` | No thousands separator |
| Decimal number | `#,##0.00` | Two decimal places, whatever the field's own precision |
| Monetary amount | the format of §13.3, built from the currency of the record at that position | Falls back to `#,##0.00` when the currency cannot be determined |
| Date | the reader's date format | From the locale of §3 |
| Date and time | the reader's date format, a space, the reader's time format | From the same locale |
| Text and long text | `@` | Forces the value to be treated as text, so that a value that looks like a number keeps its leading zeros |
| Every other type | none | The cell inherits whatever format it was given by hand |

### 14.1 Determining the currency of a monetary cell

1. Walk the field path down to the record that carries the monetary field.
2. When the walk yields exactly one record, take that record's currency field.
3. When it yields several records, collect their currency identifiers: one distinct identifier means that currency; more than one means the currency cannot be determined and the fallback format is used.
4. When it yields nothing, no currency is available and the fallback format is used.

**Worked example.** A list over orders shows the column "untaxed amount", a monetary field whose currency field names the order's currency. Row three is an order in dollars with two decimal places and the symbol before, so the cell's format is `[$$]#,##0.00`. Row four is an order in euros, so that cell's format is `#,##0.00[$€]`. A column reached through a one-to-many path whose records mix the two currencies is formatted `#,##0.00` with no symbol at all.

## 15. The value of a list cell

A cell reading a field of a data-bound list shows a converted value, not the stored one.

### 15.1 Reaching the record

1. Split the field path on full stops.
2. Walk every segment except the last, starting from the record at the requested position. A segment applied to a list of records yields the concatenation of what each of them yields.
3. When the walk yields nothing, the cell is the empty text.
4. When the walk yields exactly one record, the last segment is read on it and converted once.
5. When the walk yields several records, the last segment is read on each, each is converted, and the results are joined by a comma and a space. The joined text is not localised: it is a plain concatenation.

### 15.2 Converting the value

| Field type | Cell value |
|---|---|
| Link to one record | the display name of the linked record, or the empty text when it has none |
| Link to several records, in either direction | the display names, joined by a comma and a space, dropping the ones that are absent |
| Stored option | the label of the option whose stored value matches; the empty text when no option matches |
| Logical | true when the stored value is truthy, false otherwise |
| Date | the date serial number of §2.1, computed from the date written as year, month and day; the empty text when there is no date |
| Date and time | the moment number of §2.2, computed from the moment written as year, month, day, hour, minute and second; the empty text when there is no moment |
| Free-form properties | the names of the properties, joined by a comma and a space |
| Free-form structured value | a refusal; the cell shows the message of rule [SD-033](business-rules.md#sd-033) |
| Monetary amount, decimal number, whole number | the number itself; the empty text when there is none |
| Everything else | the stored value, or the empty text when it is falsy |

### 15.3 Worked example

A list over orders, five rows fetched. Row two is an order whose customer is a contact named "Deco Addict", whose confirmation moment is the fourteenth of March 2026 at 09:30:00 as delivered, whose state is the stored option `sale` labelled "Sales Order", whose untaxed amount is 1234.5, whose tag list holds two tags named "Priority" and "Retail", and whose delivery date is empty.

| Column | Cell value | Cell format |
|---|---|---|
| customer | "Deco Addict" | none |
| confirmation moment | 46095.3958333333 | the reader's date format, a space, the reader's time format |
| state | "Sales Order" | none |
| untaxed amount | 1234.5 | the currency format of the order's currency |
| tags | "Priority, Retail" | none |
| delivery date | the empty text | the reader's date format |

The confirmation moment in full: the fourteenth of March 2026 is 46095 days after the origin; 09:30:00 is 9 × 3600 + 30 × 60 = 34200 seconds; 34200 ÷ 86400 = 0.3958333333; the number is 46095.3958333333.

### 15.4 Positions and headers

A value cell names a row position counting from one, and the position is turned into an index counting from zero by subtracting one. A position beyond what has been fetched widens the fetched window and the cell shows the loading marker until the wider window arrives; the growth rule is in [`state-machines.md`](state-machines.md) §6.4.

A header cell names a field and may carry a third argument. When that argument is a non-empty text it is the header; otherwise the header is the field's own label as the entity declares it. A field the entity does not have, or that the reader may not read, raises rule [SD-032](business-rules.md#sd-032).

## 16. Pivot dimensions, measures and values

### 16.1 Reading a dimension name

A dimension is named by a text of the form field name, optionally a colon and a granularity, optionally preceded by a number sign.

1. When the text contains a colon, the part before it is the field name and the part after it is the granularity.
2. A leading number sign marks a **positional** reference: the group is addressed by its rank inside its parent rather than by its value.
3. The field name is looked up on the pivot's entity. A field the entity does not have raises rule [SD-035](business-rules.md#sd-035).
4. A date field or a date-and-time field with no granularity receives the granularity `month`.
5. The dimension's full name is the field name, and, when there is a granularity, a colon and that granularity.

**Worked examples.**

| Text | Field name | Granularity | Positional |
|---|---|---|---|
| `create_date:month` | `create_date` | `month` | no |
| `create_date` | `create_date` | `month`, supplied by default | no |
| `#partner_id` | `partner_id` | none | yes |
| `partner_id` | `partner_id` | none | no |
| `date_order:year` | `date_order` | `year` | no |

### 16.2 The granularities

| Kind of field | Granularities offered, in order |
|---|---|
| Date | `year`, `quarter_number`, `quarter`, `month_number`, `month`, `iso_week_number`, `week`, `day_of_month`, `day`, `day_of_week` |
| Date and time | the same ten, then `hour_number`, `minute_number`, `second_number` |

The stored values are reproduced exactly. The values ending in `_number` denote the ordinal within the containing period — the quarter of the year, the month of the year, the week of the year under the international week-numbering standard, the day of the month, the day of the week, the hour of the day, the minute of the hour, the second of the minute — while the plain names denote the period itself, so that `month` groups by "March 2026" and `month_number` groups by "3".

### 16.3 Which field may serve as what

| Role | Conditions, all of which must hold |
|---|---|
| Measure | The field is a whole number, a decimal number or a monetary amount **and** declares an aggregator, or the field is a link to one record; **and** it is not the identifier field; **and** its name contains no full stop, so a path through a relation cannot be a measure; **and** it is stored |
| Dimension | The field is groupable **and** its type has a normalisation rule, so that two group values can be compared |
| Carrier of a workbook-defined grouping | The field is groupable, is not itself such a grouping, and is a link to one record, a text, a list of links in either direction, or a stored option |

A workbook-defined grouping may not be aggregated by counting distinct values.

### 16.4 The aggregators

| Field type | Aggregators offered |
|---|---|
| Whole number, decimal number, monetary amount | `max` "Maximum", `min` "Minimum", `avg` "Average", `sum` "Sum", `count_distinct` "Count Distinct", `count` "Count" |
| Date, date and time | `max`, `min`, `count_distinct`, `count` |
| Logical | `count_distinct`, `count`, `bool_and` "Boolean And", `bool_or` "Boolean Or" |
| Text | `count_distinct`, `count` |
| Link to one record | `count_distinct`, `count` |
| Reference to any record | `count_distinct`, `count` |

The stored aggregator values and the labels are both reproduced.

### 16.5 Addressing a weekly group

A cell addressing a group at the granularity `week` supplies the group as a text made of the week number, a slash and a four-digit year. Anything else raises rule [SD-036](business-rules.md#sd-036), whose message quotes a correct example built from the current year.

**Worked example.** In the year 2026 the message's example reads `"52/2026"`. A cell supplying `2026-W12` is refused and the message names the value that was given.

### 16.6 When a change reloads the data

A change to a pivot's definition reloads its records only when the change can alter the answer.

| Change | Reload |
|---|---|
| A row dimension or a column dimension added, removed, reordered or re-granulated | yes |
| The record selection changed | yes |
| The reading context changed | yes |
| The entity changed | yes |
| A workbook-defined grouping added, removed or redefined | yes |
| A measure added or removed | yes |
| A fetched measure's field or aggregator changed | yes |
| A measure that is computed inside the workbook added, removed or changed | no |
| Measures reordered or re-labelled | no |
| The sort column changed | no |
| A group collapsed or expanded | no |

## 17. Turning a filter period into a window

A date filter's value is turned into a first moment and a last moment. The element's own period offset shifts the window; its unit is the unit of the value's own shape.

Throughout this section, *now* is the moment at which the workbook is evaluated, in the reader's own zone. *Start of a unit* means the first instant of it; *end of a unit* means the last instant of it, which is one millisecond before the first instant of the next.

### 17.1 Fixed shapes

| Shape | Procedure | Offset unit |
|---|---|---|
| `year` | Take now, set its year to the value's year, add the offset in years; the window is that whole year | years |
| `month` | Take now, set its year and its month, add the offset in months; the window is that whole month | months |
| `quarter` | Take now, set its year, set its month to ( quarter × 3 − 2 ), add the offset in quarters; the window is that whole quarter | quarters |
| `range` | The first moment is the start of the day named as the first end; the last moment is the end of the day named as the last end; either may be absent | none; the offset is ignored |

Two degenerate cases:

1. A value with no year at all produces **no window**: neither a first nor a last moment, so the element is not narrowed.
2. A `month` value with no month, or a `quarter` value with no quarter, falls back to the whole year of the given year, and the offset is ignored.

Where setting the month would land on a day that month does not have — the thirty-first of a thirty-day month — the day is clamped to the last day of the target month. This does not change the window, because the window is snapped to the whole month, quarter or year; it is stated so that a rebuild does not fail on the arithmetic. **Industry-standard default.**

**Worked examples**, with now = 15 March 2026, 14:30:

| Value | Offset | First moment | Last moment |
|---|---|---|---|
| `year` 2025 | 0 | 1 January 2025, 00:00:00.000 | 31 December 2025, 23:59:59.999 |
| `year` 2025 | −1 | 1 January 2024, 00:00:00.000 | 31 December 2024, 23:59:59.999 |
| `month` 2026-03 | 0 | 1 March 2026, 00:00:00.000 | 31 March 2026, 23:59:59.999 |
| `month` 2026-03 | −1 | 1 February 2026, 00:00:00.000 | 28 February 2026, 23:59:59.999 |
| `quarter` 2 of 2026 | 0 | 1 April 2026, 00:00:00.000 | 30 June 2026, 23:59:59.999 |
| `quarter` 2 of 2026 | +1 | 1 July 2026, 00:00:00.000 | 30 September 2026, 23:59:59.999 |
| `range` 2026-02-10 to 2026-02-20 | any | 10 February 2026, 00:00:00.000 | 20 February 2026, 23:59:59.999 |
| `range` with only a first end 2026-02-10 | any | 10 February 2026, 00:00:00.000 | absent |
| `range` with only a last end 2026-02-20 | any | absent | 20 February 2026, 23:59:59.999 |

### 17.2 The nine relative periods

Let *tomorrow* mean the start of the day after now.

| Period | First moment | Last moment | Offset unit |
|---|---|---|---|
| `today` | start of today, plus the offset in days | end of today, plus the offset in days | days |
| `yesterday` | start of the day before today, plus the offset in days | end of the day before today, plus the offset in days | days |
| `month_to_date` | start of this month, plus the offset in months | end of today, plus the offset in months | months |
| `last_month` | start of the month before ( now plus the offset in months ) | end of that same month | months |
| `year_to_date` | start of this year, plus the offset in years | end of today, plus the offset in years | years |
| `last_7_days` | tomorrow minus 7 days, plus 7 × the offset in days | end of today, plus 7 × the offset in days | weeks of seven days |
| `last_30_days` | tomorrow minus 30 days, plus 30 × the offset in days | end of today, plus 30 × the offset in days | blocks of thirty days |
| `last_90_days` | tomorrow minus 90 days, plus 90 × the offset in days | end of today, plus 90 × the offset in days | blocks of ninety days |
| `last_12_months` | start of the month twelve months before tomorrow, plus 12 × the offset in months | end of the month before the month of tomorrow, plus 12 × the offset in months | blocks of twelve months |

Note that the "last *n* days" windows are inclusive of today and count backwards from tomorrow, so "last 7 days" spans exactly seven days ending today.

**Worked examples**, with now = 15 March 2026, 14:30, offset zero:

| Period | First moment | Last moment |
|---|---|---|
| `today` | 15 March 2026, 00:00:00.000 | 15 March 2026, 23:59:59.999 |
| `yesterday` | 14 March 2026, 00:00:00.000 | 14 March 2026, 23:59:59.999 |
| `last_7_days` | 9 March 2026, 00:00:00.000 | 15 March 2026, 23:59:59.999 |
| `last_30_days` | 14 February 2026, 00:00:00.000 | 15 March 2026, 23:59:59.999 |
| `last_90_days` | 16 December 2025, 00:00:00.000 | 15 March 2026, 23:59:59.999 |
| `month_to_date` | 1 March 2026, 00:00:00.000 | 15 March 2026, 23:59:59.999 |
| `last_month` | 1 February 2026, 00:00:00.000 | 28 February 2026, 23:59:59.999 |
| `year_to_date` | 1 January 2026, 00:00:00.000 | 15 March 2026, 23:59:59.999 |
| `last_12_months` | 1 March 2025, 00:00:00.000 | 28 February 2026, 23:59:59.999 |

The `last_7_days` row in full: tomorrow is 16 March 2026 at 00:00; 16 March minus 7 days is 9 March at 00:00; the last moment is the end of today. The window covers 9, 10, 11, 12, 13, 14 and 15 March: seven days.

The `last_90_days` row in full: 16 March 2026 minus 15 days is 1 March 2026, minus a further 28 days (February 2026 has 28) is 1 February 2026, minus a further 31 days is 1 January 2026, minus the remaining 16 days is 16 December 2025.

**The same nine with offset −1:**

| Period | First moment | Last moment |
|---|---|---|
| `today` | 14 March 2026, 00:00:00.000 | 14 March 2026, 23:59:59.999 |
| `yesterday` | 13 March 2026, 00:00:00.000 | 13 March 2026, 23:59:59.999 |
| `last_7_days` | 2 March 2026, 00:00:00.000 | 8 March 2026, 23:59:59.999 |
| `last_30_days` | 15 January 2026, 00:00:00.000 | 13 February 2026, 23:59:59.999 |
| `last_90_days` | 17 September 2025, 00:00:00.000 | 15 December 2025, 23:59:59.999 |
| `month_to_date` | 1 February 2026, 00:00:00.000 | 15 February 2026, 23:59:59.999 |
| `last_month` | 1 January 2026, 00:00:00.000 | 31 January 2026, 23:59:59.999 |
| `year_to_date` | 1 January 2025, 00:00:00.000 | 15 March 2025, 23:59:59.999 |
| `last_12_months` | 1 March 2024, 00:00:00.000 | 28 February 2025, 23:59:59.999 |

### 17.3 From a window to a record selection

1. When the matched field is a date, each moment is written as a date, losing the time.
2. When the matched field is a date and time, each moment is written as a moment with its time.
3. The conditions are:

| Moments present | Condition |
|---|---|
| both | the field is at least the first moment **and** at most the last moment |
| first only | the field is at least the first moment |
| last only | the field is at most the last moment |
| neither | no condition; the element is not narrowed |

**Worked example.** The filter is set to `month` 2026-03, one pivot has offset 0 and another has offset −1, and both match a date field named as their invoice date. The first pivot is narrowed to invoice dates between 1 March 2026 and 31 March 2026; the second to invoice dates between 1 February 2026 and 28 February 2026. One filter, two periods, side by side.

### 17.4 The rendered value of a date filter

A date filter's display value is two cells, one under the other: the first moment written as a date, then the last moment written as a date, each carrying the reader's date format. A missing moment renders as an empty cell.

**Worked example.** With the window 1 March 2026 to 31 March 2026 and a reader whose date format is `mm/dd/yyyy`, the two cells hold the date serial numbers of those two dates, and are shown as `03/01/2026` and `03/31/2026`.

## 18. Naming a duplicate

When a Spreadsheet Dashboard is duplicated and the caller supplies no name, the copy's name is built from the original's:

```formula
copy name = original name + " (copy)"
```

The pattern is the reproduced text `%s (copy)`, where the placeholder stands for the original's name. A caller that supplies a name suppresses the rule entirely, and the supplied name is used unchanged.

**Worked examples.**

| Original name | Supplied name | Copy's name |
|---|---|---|
| "a dashboard" | none | "a dashboard (copy)" |
| "a dashboard" | "a copy" | "a copy" |
| "a dashboard (copy)" | none | "a dashboard (copy) (copy)" |

The rule does not deduplicate: duplicating a copy appends the suffix again.

## 19. Freezing a workbook

Freezing turns a live dashboard into a self-contained snapshot that a reader outside the application can open. It is the algorithm behind sharing.

### 19.1 Waiting for the data

1. Load the outline data of every map used by a chart of the workbook.
2. Load every data source: each data-bound chart, each data-bound pivot, each list.
3. Re-evaluate every cell.
4. Walk every cell of every sheet. When any of them holds the loading marker, wait for the next arrival of data from any data source and go back to step 3. When none does, continue.

The loop terminates because every data source ends in one of the settled states of [`state-machines.md`](state-machines.md) §6.2, and a settled source never produces the loading marker again.

### 19.2 Transforming the document

The document is exported and then rewritten sheet by sheet.

1. For each cell whose stored content begins with an equals sign **and**, once parsed, calls at least one function whose name begins with `ODOO.`, or begins with `_t`, or begins with `PIVOT`:
   1. When the cell belongs to a pivot that is not data-bound, leave it alone and move on.
   2. Replace the content by the text of the cell's evaluated value. A value that is the empty text becomes the formula that yields an empty text, so that the cell stays a cell rather than becoming blank.
   3. When the evaluated cell carries a number format, register that format in the workbook's format table and point the cell at it.
   4. When the formula spills across a region, do the same for every cell of that region.
2. For each cell holding a link whose target is one of the three application prefixes, replace the content by the same label pointing at the neutralised target.
3. For each figure:
   - a chart figure whose kind begins with `odoo_`, or whose kind is the geographic kind, becomes an image figure holding a picture of the chart drawn at the figure's own width and height on a white background;
   - a carousel figure containing at least one such chart becomes an image figure holding a picture of the first chart of the carousel, at the carousel's own size;
   - every other figure is untouched.
4. Drop every data-bound pivot definition from the workbook's pivot section, keeping the pivots that read cell ranges.
5. Empty the list section completely.
6. Append the filter sheet of [`document-format.md`](document-format.md) §9.
7. For each global filter, write its current display value into the definition as text: take every cell of the display value, format each with its own number format under the reader's locale, drop the ones that format to nothing, and join the rest by a comma and a space.

### 19.3 Registering a format

A format is registered by looking for it among the formats the workbook already has and reusing its key when it is there. When it is not, the new key is one more than the largest key in use, and one when the table is empty.

```formula
new format key = ( largest existing key, or 0 when the table is empty ) + 1
```

**Worked example.** A workbook whose format table holds the keys 1, 2 and 5 gains a new format under the key 6.

### 19.4 Deciding whether anything changed

A reader who opens the share control twice in one session should get the same address twice unless something changed. The comparison has three parts, and any one of them being different is enough:

1. The workbook's revision marker differs from the one recorded at the last share of this session.
2. Any cell of the appended filter sheet differs from the corresponding cell recorded at the last share.
3. The locale code differs from the one recorded at the last share.

When none differs, nothing is sent and the previously obtained address is shown again. When any differs, the three records are updated and the share proceeds.

**Worked example.** A reader shares a dashboard, then changes a filter from `year` 2025 to `year` 2026 and shares again. The revision marker is unchanged, because setting a filter value does not change the stored workbook; but the filter sheet's value cell now holds different dates, so the second part differs and a second Dashboard Share is created. If instead the reader shares twice with nothing in between, the second attempt sends nothing and reuses the first address.

### 19.5 Worked example of the whole transformation

A dashboard with one sheet holding:

| Cell | Before | After |
|---|---|---|
| `A1` | `=ODOO.LIST.HEADER(1,"name")` | `Customer` |
| `A2` | `=ODOO.LIST(1,1,"name")` | `Deco Addict` |
| `B2` | `=ODOO.LIST(1,1,"amount_untaxed")` | `1234.5`, with the format `[$$]#,##0.00` registered |
| `A3` | `=ODOO.LIST(1,2,"name")`, evaluating to nothing | `=""` |
| `C1` | `=ODOO.BALANCE("400","2026")` | `15000` |
| `D1` | a link labelled "Open orders" targeting a menu | the same label targeting the neutralised address |
| one bar chart figure | a data-bound chart definition | an image figure of the same width and height |

Afterwards the workbook's list section is empty, its data-bound pivot definitions are gone, a sheet named "Active Filters" has been appended, and each filter definition carries its rendered value.

## 20. Extending a serialised workbook without parsing it

A consumer that has to add a handful of keys to a large serialised document may do so at the text level.

1. Trim the document of surrounding blank characters.
2. Remove its final closing brace.
3. When what remains is not the empty document — that is, when the trimmed document was not exactly an opening brace followed by a closing brace — append a comma.
4. For each pair in turn: append a quotation mark, the key, a quotation mark, a colon, and the already-serialised value. Append a comma between consecutive pairs, and none after the last.
5. Append a closing brace.

The values must already be serialised; the rule performs no conversion of its own.

**Worked examples.**

| Document | Pairs | Result |
|---|---|---|
| `{}` | none | `{}` |
| `{}` | `key` with the value `{}` | `{"key":{}}` |
| `{}` | `key` with the value `[]` | `{"key":[]}` |
| `{}` | `key` with the value `"value"` | `{"key":"value"}` |
| `{"a": 1}` | `key` with the value `"value"` | `{"a": 1,"key":"value"}` |
| `{"a": 1}` | `key` with the value `{"b": 2}` | `{"a": 1,"key":{"b": 2}}` |
| `{"a": {}}` | `key` with the value `{"b": 2}` | `{"a": {},"key":{"b": 2}}` |
| `{"a": 1}` | `key` with the value `[]` | `{"a": 1,"key":[]}` |
| `{"a": []}` | `key` with the value `[]` | `{"a": [],"key":[]}` |
| `{}` | `key1` with the value `1`, then `key2` with the value `2` | `{"key1":1,"key2":2}` |
| a line break, `{}`, a line break | the same two pairs | `{"key1":1,"key2":2}` |

The second-to-last row shows why step 3 tests the document rather than its length: an empty document must not gain a leading comma. The last row shows that the trimming of step 1 happens before every other step.
