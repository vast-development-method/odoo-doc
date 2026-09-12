# Multi-Currency — Business Rules

The complete rule catalogue of the multi-currency domain. Every rule carries a stable identifier
of the form `MCUR-nnn` which the other files of this folder cite. Every user-facing message is
reproduced exactly as it is shown, in quotation marks; a placeholder inside a message is described
in words and written in italics. A line break inside a message is written as ⏎.

A rule marked **industry-standard default** states behaviour that the system does not settle by
itself and that is filled in from established double-entry practice. A rule marked **compatibility
finding** records an observed behaviour that looks like a defect, states what a corrected
behaviour would be, and specifies the observed behaviour as the one to reproduce. Every other rule
states observed behaviour.

The numbers are grouped by subject: currency master data from `MCUR-001`, the rate table from
`MCUR-020`, conversion and arithmetic from `MCUR-040`, journal items from `MCUR-060`, the document
rate from `MCUR-080`, exchange differences from `MCUR-100`, reconciliation guards from `MCUR-130`,
bank transactions from `MCUR-150`, the company's main currency from `MCUR-170`, access from
`MCUR-190`, unrealised differences from `MCUR-200`, and presentation from `MCUR-210`. Section 13
maps every former identifier onto the current one.

---

## A. Currency master data

### MCUR-001: the currency code is unique

No two Currency records may carry the same currency code (`name`), archived records included.
Enforced by a stored uniqueness constraint.

Message: "The currency code must be unique!"

### MCUR-002: the currency code is required and is at most three characters

The currency code is mandatory and its stored length is capped at three characters, matching the
alphabetic code of the international currency-code standard. A creation or a write without a code
is refused by the required-field check of the persistence layer; a code longer than three
characters is refused by the storage.

### MCUR-003: the rounding factor is strictly positive

The rounding factor (`rounding`) must be strictly greater than zero. Enforced by a stored check
constraint.

Message: "The rounding factor must be greater than 0!"

A rounding factor of zero would make every amount unrepresentable, and a negative rounding factor
has no meaning.

### MCUR-004: the symbol is required

The symbol is mandatory. A currency without a symbol could not be rendered on any printed
document.

### MCUR-005: the decimal place count is derived and never entered

The decimal places are always derived from the rounding factor by the formula of
[calculations.md](calculations.md) section 2, and stored. A direct write to the decimal places is
overwritten by the next recomputation. Both fields are visible only to a reader holding the
technical features group.

### MCUR-006: a currency's precision may not be lowered once it has been used in accounting

A write to the rounding factor is refused when the new value is greater than the current value, or
the new value is zero, **and** the currency has already been used to round accounting entries. A
currency counts as used when at least one journal item exists whose item currency is that currency
or whose company currency is that currency.

Message: "You cannot reduce the number of decimal places of a currency which has already been used
to make accounting entries."

Raising the precision — writing a smaller rounding factor — is permitted, because previously
stored amounts remain exactly representable on the finer grid.

### MCUR-007: a currency in use by a company may not be deactivated

Writing the activity flag to false is refused when any company has that currency as its main
currency.

Message: "This currency is set on a company and therefore cannot be deactivated."

The check is skipped in two situations, both signalled through context values: during the
installation of a capability package, because a currency being attached to a company during
installation is momentarily still seen as inactive; and when the operation context carries an
explicit forced-deactivation flag, which exists so that an automated scenario can reduce a
platform to a single active currency. A rebuild must reproduce both escape hatches; without the
first, installing reference data becomes impossible.

### MCUR-008: the multi-currency capability follows the count of active currencies

After every creation, every deletion, and every write that touches the activity flag of a
Currency, the number of active currencies is counted.

| Condition | Effect |
|---|---|
| More than one active currency | The multi-currency permission group is applied to the internal-user group, which makes currency fields, currency columns and rate menus visible to every internal user. |
| One or zero active currencies | The multi-currency permission group is removed from the internal-user group. |

The grant is applied at group level, not per user. When the multi-currency group is granted and
internal users do not yet hold the price list capability group, that group is granted as well and
a default price list is created or activated for every company, because working with several
currencies requires a price list per currency.

### MCUR-009: setting a currency on a company activates it

Creating or writing a Company whose main currency is archived activates that currency
automatically, without any confirmation. A company cannot have an archived main currency.

### MCUR-010: deactivating a currency archives its price lists

When a Currency is archived, every price list whose currency is that currency is archived in the
same operation.

### MCUR-011: deleting a currency deletes its rate rows

The link from Currency Rate to Currency deletes on cascade. Deleting a currency therefore deletes
every rate row attached to it. Deleting a currency that is still referenced by any other record is
refused by the referential integrity of the persistence layer.

### MCUR-012: default ordering of currencies

Currencies are listed by activity flag descending, then currency code ascending. Active currencies
appear first, and each block is alphabetical by code.

### MCUR-013: currencies are global

A Currency record is not scoped to a company. Every company sees the same catalogue, and a change
made while working in one company is read by every other. Only rate rows may be scoped.

### MCUR-014: the cached currency catalogue is invalidated on change

A cached map from currency identifier to code, symbol, symbol position and decimal place count is
kept so that the display layer can render a monetary value without a round trip. It is invalidated
on every Currency creation, on every deletion, and on every write that touches one of five keys:
the activity flag, a key named for the digit count, the currency code, the symbol position or the
symbol.

**Compatibility finding.** No field named for the digit count exists on a currency — the decimal
places are stored under the identifier `decimal_places` — so that key never appears in a write and
never fires. The effect is harmless, because the decimal places change only together with the
rounding factor, which is not in the trigger set either. A corrected behaviour would list the
rounding factor and the decimal places in the trigger set so that a change of precision refreshes
the cached catalogue immediately rather than at the next unrelated write. A rebuild reproduces the
observed five-key trigger set.

### MCUR-015: changing the rounding factor raises an irreversibility warning in the form

While a form holds an unsaved change to the rounding factor on an already persisted currency, a
red panel is displayed reading "WARNING - This change is irreversible" followed by "You are
changing decimals in your entire database, including invoices, tax amounts, accounting amounts,
reports. This is probably not intended." The panel offers a control that opens the decimal
precision registry. The warning does not block saving; `MCUR-006` does.

### MCUR-016: a document in an archived currency cannot be posted

Posting a Journal Entry whose document currency is archived is refused.

Message: "You cannot validate a document with an inactive currency: *the currency code*"

The condition is evaluated while the entry is in draft and is surfaced on the form as a warning
before the user attempts to post.

### MCUR-017: how a currency is searched

A typed term matches a currency for the record-name search when it is contained in the currency
code or in the full name. The single search field of the currency list matches the typed term
against the currency code, the full name, the symbol, the unit label or the subunit label; any one
of them containing the term is a match.

### MCUR-018: every shipped currency is delivered inactive

All one hundred and seventy currencies of the shipped catalogue carry an activity flag of false. A
platform therefore starts with no active currency at all and becomes multi-currency only
deliberately: through `MCUR-009`, through a country package or chart of accounts template that
activates the currency of its country, or through an explicit activation by a user. When
demonstration data is loaded, the United States dollar is activated so that monetary fields show a
symbol before any accounting package is installed.

### MCUR-019: the current rate shown on a currency is evaluated against the company in context

The current rate, the inverse rate and the rate as text are computed against a target currency
that defaults to the main currency of the company in context, at a date that defaults to today in
the reader's time zone. The rate as text is empty when the currency is that company's own main
currency, because a currency has no rate against itself.

---

## B. The rate table

### MCUR-020: rate rows belong to root companies only

A Currency Rate whose company is a branch company — a company having a parent — is refused.

Message: "Currency rates should only be created for main companies"

The consequence is that every rate lookup resolves the company to its root before searching.

### MCUR-021: an implausible rate raises a warning

When the company rate is edited in a form, the technical rate implied by the entry is compared
with the technical rate of the latest strictly earlier row of the same currency and the same
company scope. When such a row exists and the relative change exceeds twenty percent in either
direction, a non-blocking warning is shown.

Title: "Warning for *the currency code*"
Body: "The new rate is quite far from the previous rate.⏎Incorrect currency rates may cause
critical problems, make sure the rate is correct!"

The formula is in [calculations.md](calculations.md) section 9. The warning never prevents saving,
and it is raised only in the interactive form path.

A country package may add a further warning on the same trigger. The Egypt package warns when the
fiscal country of the company is Egypt and the entered inverse company rate carries more than five
significant fractional digits, with the body "Please make sure that the EGP per unit is within 5
decimal accuracy.⏎Higher decimal accuracy might lead to inconsistency with the ETA invoicing
portal!" — reproduced verbatim, including the currency code and the abbreviated service name it
contains.

### MCUR-022: one rate per currency, company scope and day

The triple of rate date, currency and company is unique. A second row for the same currency, the
same company scope and the same day is refused. A row with no company scope and a row scoped to a
company may coexist on the same day for the same currency, because an empty company is a distinct
value.

Message: "Only one currency rate per day allowed!"

### MCUR-023: the technical rate is strictly positive

The technical rate must be strictly greater than zero. Enforced by a stored check constraint.

Message: "The currency rate must be strictly positive."

A zero or negative rate would make the conversion formula undefined or sign-inverting.

### MCUR-024: precedence among the three rate fields

When more than one of the technical rate, the company rate and the inverse company rate is
supplied in the same creation or write, the supplied values are reduced before anything is stored:

1. The inverse company rate is discarded when the company rate or the technical rate is also
   supplied.
2. The company rate is discarded when the technical rate is also supplied.

The effective priority is the technical rate, then the company rate, then the inverse company
rate. The surviving value drives the other two.

### MCUR-025: an omitted technical rate carries forward

While a rate row is being edited without an explicit technical rate, the value presented by the
derived company rate is taken from the latest row of the same currency and the same company scope
whose rate date is strictly earlier, and is one when no such row exists. Writing the company rate
or the inverse company rate stores that carried-forward value as the technical rate. Once stored
it is a stored value: editing the earlier row afterwards does not change the later row. A row
saved with none of the three values supplied stores a technical rate of zero and is refused by
`MCUR-023`.

### MCUR-026: the rate date is required and defaults to today

The rate date is mandatory. On a new row it defaults to today in the reader's time zone.

### MCUR-027: asking for the preceding rate of a row with no date is an error

An explicit request for the rate preceding a row that carries no rate date is refused.

Message: "The name for the current rate is empty.⏎Please set it."

The word "name" in the message is the identifier of the date field.

### MCUR-028: the rate in force on a date

For a currency, a company and a date, the rate in force is determined as follows.

1. Only rows of that currency whose company is the company's root or is empty are candidates.
2. Among the candidates whose rate date is on or before the requested date, a row scoped to the
   root company wins over a row with no company scope; among rows of the same scope precedence,
   the greatest rate date wins.
3. The boundary is inclusive: a row dated exactly on the requested date is in force on that date.

### MCUR-029: before the first rate, the first rate applies

When a currency has rate rows but none of them is dated on or before the requested date, the
earliest row is used, with the same scope precedence. The first known rate is projected backwards
without limit.

### MCUR-030: a currency with no rate has a rate of one

When a currency has no rate row visible to the company at all, its technical rate is one. This is
what makes the company's main currency behave as the reference of rate one in a platform that
keeps no rate row for it.

### MCUR-031: changing a rate never restates a posted entry

Creating, editing or deleting a rate row changes only future conversions and the derived current
rate display. It never changes the balance or the amount in currency of a journal item that has
already been written. An item's company currency value is fixed at the moment the item is written
and can afterwards be changed only by editing the item while its entry is in draft, or by a posted
correcting entry.

### MCUR-032: the technical rate is hidden from ordinary users

The technical rate of a rate row is shown only to a reader holding the technical features group.
Ordinary accountants enter and read the company rate and the inverse company rate, whose headings
name the two currencies explicitly (`MCUR-034`).

### MCUR-033: a company-scoped rate beats a shared rate unconditionally

The company preference in `MCUR-028` and `MCUR-029` is total: *any* row that names the company and
satisfies the date condition beats *every* shared row, even a shared row with a later date. A
rebuild must not merge the two sets and then order by date. A company that wishes to override a
shared rate for a period must keep its own row current.

### MCUR-034: rate rows are ordered by date descending and searched by parsed date

Rate rows are ordered by rate date descending, then internal identifier ascending, so that the
most recent rate appears first. The display name of a rate row is its rate date rendered in the
reader's date format. A free-text search term is first parsed as a date in the reader's language
and date format and matched against the rate date; a term that cannot be parsed as a date is
matched against the technical rate instead. The two rate columns carry headings generated from the
codes of the two currencies concerned, falling back to the word "Unit" when no specific currency is
in context.

### MCUR-035: the reciprocal of a rate is guarded against zero

Before the inverse company rate is computed, a company rate that is zero or unset is forced to
one; before the company rate is computed from the inverse company rate, an inverse company rate
that is zero or unset is forced to one. The reciprocal is therefore never a division by zero.

### MCUR-036: writing a rate refreshes every displayed current rate

Creating or writing a Currency Rate invalidates the derived inverse rate of every Currency, so
that the current rate column of the currency list and the rate as text on a currency form refresh
at the next read.

### MCUR-037: a rate is never rounded

A technical rate, a company rate and an inverse company rate carry their full stored precision into
every computation. No currency's rounding factor is ever applied to a rate; rounding applies to
amounts only. The rate columns are *displayed* with twelve digits of which twelve are fractional
on a rate row, and with twelve digits of which six are fractional on the currency list, which is a
presentation choice and not a stored precision.

### MCUR-038: the default company of a rate row

A new rate row takes as its company the root company of the company the reader is working in. A
user may clear the field, which makes the row shared by every company.

### MCUR-039: rate rows are never archived

Currency Rate supports no activity flag. A rate row is created, edited and deleted. Deleting one
extends the validity window of the preceding row forward and leaves every posted entry untouched
(`MCUR-031`).

---

## C. Conversion, rounding and comparison

### MCUR-040: the conversion formula

An amount in a source currency is converted into a target currency for a company on a date as:

```formula
converted amount = round onto the target currency (
        amount × ( rate of the target currency ÷ rate of the source currency ) )
```

both rates being the rate in force for that company on that date by `MCUR-028`. When the source
and the target are the same currency the factor is exactly one and no lookup is performed. When
the amount is exactly zero the result is zero and no lookup is performed. Rounding may be
suppressed by the caller, in which case the raw product is returned.

### MCUR-041: rounding is half away from zero

Every monetary rounding in the platform snaps the amount onto the nearest multiple of the
currency's rounding factor and resolves an exact tie away from zero. A correction term
proportional to the magnitude of the value is added before snapping, so that a binary
representation marginally below a tie is still treated as a tie. The full algorithm is in
[calculations.md](calculations.md) section 3.

### MCUR-042: a monetary value is rounded both on assignment and on storage

A monetary value is snapped onto its currency's rounding factor when it is assigned into a record
in memory, and again when it is written to storage, where it is rendered with exactly the decimal
places of that currency. Assigning a monetary value across records whose companion currency fields
hold more than one currency is an error condition, because a single value cannot belong to two
currencies at once.

### MCUR-043: two amounts are equal when their rounded values are equal

A comparison rounds each operand first and then subtracts. Two amounts that differ by less than a
rounding step may still compare as different, because they may round onto different multiples. The
algorithm and the canonical counter-example are in [calculations.md](calculations.md) section 4.

### MCUR-044: an amount is zero when its rounded absolute value is below the rounding factor

The zero test rounds first and then compares the absolute value against the rounding factor,
strictly. An amount exactly equal to one rounding factor is not zero. The zero test rounds *after*
subtracting when it is applied to a difference, which is why testing a difference for zero and
comparing the two operands are not interchangeable.

### MCUR-045: a conversion round trip is not guaranteed to be the identity

Converting an amount into another currency and back may not return the starting amount, because
each leg rounds. No computation may rely on a round trip, and no stored foreign currency amount
may be re-derived by converting the stored company currency amount. Both amounts are stored
precisely because neither can be derived from the other.

### MCUR-046: the result of a conversion is rounded onto the target currency

The rounding applied at the end of a conversion always uses the *target* currency's rounding
factor, never the source currency's and never a generic two-decimal default.

### MCUR-047: one multiplication and one rounding

A conversion multiplies the amount by the composed factor once and rounds once. Even when neither
currency is the company currency, the conversion does **not** convert into the company currency,
round, and convert again. The worked contrast in [calculations.md](calculations.md) section 8.3
shows a one-penny divergence between the specified single step and the two-step route.

### MCUR-048: the short-circuits of a conversion

| Situation | Behaviour |
|---|---|
| The source and the target are the same currency | The factor is one without any lookup. The amount is still rounded onto that currency's rounding factor. |
| The amount is exactly zero | Zero is returned immediately; no rate lookup happens, so converting zero never fails for want of a rate. |
| One of the two currencies is empty | The missing one is taken to be the given one, which makes the conversion the identity. |
| Both currencies are empty | The operation fails: an amount cannot be converted from an unknown currency. |
| Rounding suppressed by the caller | The raw product is returned, for use where the result feeds a further computation that will round later. |

### MCUR-049: a precision is given exactly one way

The rounding routine, the comparison, the zero test and the euclidean division take their
precision **either** as a number of fractional digits **or** as a rounding factor, never both and
never neither. Supplying both, or neither, is a programming error and must fail loudly. A supplied
rounding factor must be strictly positive; a supplied digit count must be a whole number greater
than or equal to zero.

### MCUR-050: the five rounding methods and the default

Five rounding methods exist, selected by a stored selector:

| Method | Stored selector | Behaviour |
|---|---|---|
| Half away from zero | `HALF-UP` | Round onto the nearest multiple; a value exactly halfway goes away from zero. **The default, and the only method money uses unless a caller names another.** |
| Half towards zero | `HALF-DOWN` | Round onto the nearest multiple; a value exactly halfway goes towards zero. |
| Half to even | `HALF-EVEN` | Round onto the nearest multiple; a value exactly halfway goes to the nearer even multiple. The default of the language-aware presentation routine. |
| Away from zero | `UP` | Always round away from zero: any non-zero fraction of a step pushes the result to the next multiple further from zero. |
| Towards zero | `DOWN` | Always round towards zero: the fraction of a step is discarded. |

Any other selector is a programming error and must fail with a message naming the unknown method.

### MCUR-051: rendering is not rounding

The routine that renders a number as a string is a presentation routine. A rebuild must round
first and render second, and must never rely on the renderer to round. The renderer must not use a
shortest-representation conversion, because such a conversion silently drops significant digits on
large values.

### MCUR-052: the reciprocal of a rounding factor is taken from an exact table

A rounding factor smaller than one is inverted before use, and the inversion must not itself
introduce error. Thirty common factors are inverted by table lookup; any other factor is inverted
by the coefficient-and-exponent rule of [calculations.md](calculations.md) section 3.2.

### MCUR-053: an exact euclidean division is available

Splitting an amount into whole multiples of another amount uses the routine of
[calculations.md](calculations.md) section 6, which snaps both operands onto the precision grid and
scales them to whole numbers before dividing, so that the result is free of representation error.
Its postcondition is that the dividend, rounded onto the factor, equals the quotient times the
divisor plus the remainder, exactly, at that factor.

---

## D. Foreign currency amounts on journal items

### MCUR-060: the two amount columns must agree in sign

For every accountable journal item, the balance and the amount in currency must have the same sign
or one of them must be zero. Enforced by a stored database check: the display type is a section, a
subsection or a note, **or** both amounts are less than or equal to zero, **or** both amounts are
greater than or equal to zero.

Message: "The amount expressed in the secondary currency must be positive when account is debited
and negative when account is credited. If the currency is the same as the one from the company,
this amount must strictly be equal to the balance."

### MCUR-061: a journal item carries a debit or a credit, never both

Enforced by a stored check: the display type is a section, a subsection or a note, **or** the
product of the debit and the credit is zero.

Message: "Wrong credit or debit value in accounting entry!"

### MCUR-062: a non-accountable line carries no amount and no account

Enforced by a stored check: the display type is **not** a section, a subsection or a note, **or**
the amount in currency, the debit and the credit are all zero and the account is empty.

Message: "Forbidden balance or account on non-accountable line"

### MCUR-063: both amount columns are always written together

Because `MCUR-060` is a stored statement that reads both columns, a write that touches either the
balance or the amount in currency must send both columns to storage in the same statement. A
replacement implementation must either do the same or defer the sign check to the end of the
transaction; writing one column alone would fail the check on an intermediate state that the
second write is about to correct.

### MCUR-064: the account may force the currency of the item

When the account of a journal item carries a currency, and that account currency differs from the
company currency, and it differs from the item currency, the write is refused.

Message: "The account selected on your journal entry forces to provide a secondary currency. You
should remove the secondary currency on the account."

**Compatibility finding.** A branch exempting the journal's own default account and the journal's
suspense account from this check exists, but it is evaluated *after* the check has already been
made, so it exempts nothing: an item on the journal's default account or on its suspense account
is refused like any other. A corrected behaviour would evaluate the exemption first, so that the
two accounts a journal keeps aligned with its own currency by `MCUR-067` cannot be refused by a
rule about the currency they were given. A rebuild reproduces the observed behaviour: the check
applies to every accountable item.

### MCUR-065: the amount in currency is derived from the balance and the item rate

When no amount in currency has been supplied for an item, it is derived as the balance multiplied
by the item rate and rounded onto the item currency. When the item currency equals the company
currency and the item does not belong to an invoice-like entry, the amount in currency is forced
equal to the balance.

### MCUR-066: editing the amount in currency re-derives the balance

In a form, when the user edits the amount in currency or the item currency:

- when the item currency equals the company currency and the balance differs from the amount in
  currency, the balance is set equal to the amount in currency;
- when the item currency differs from the company currency, the entry is not invoice-like, and the
  balance is not write-protected, the balance is set to the amount in currency divided by the item
  rate, rounded onto the company currency.

For an invoice-like entry the balance is never re-derived this way, because it is governed by the
document rate of section E.

### MCUR-067: a journal with a currency forces it on its accounts

Setting a currency on a Journal propagates it to the journal's default account and to the bank
account record linked to the journal. Every entry of that journal then takes that currency as its
document currency. The display name of such a journal is suffixed with the currency code in
parentheses when the currency differs from the company's main currency.

### MCUR-068: an account currency must agree with the journal currency

An account that is the default account of a journal, or the payment account of an inbound or
outbound payment method line of a journal, must carry the same currency as that journal whenever
the journal's currency is set and differs from the company's main currency.

Message: "The foreign currency set on the journal '*journal display name*' and the account
'*account display name*' must be the same."

### MCUR-069: an account currency may not be set once items in another currency exist

Writing a currency on an Account is refused when at least one journal item already exists on that
account whose item currency is neither empty nor the new value.

Message: "You cannot set a currency on this account as it already has some journal entries having a
different foreign currency."

### MCUR-070: the currency of an item follows its entry

The item currency is derived as follows, and may then be overridden by the user on a miscellaneous
entry:

1. a cost-of-goods-sold line always takes the company currency;
2. a line of an invoice, a bill, a credit note, a debit note or a receipt takes the entry's
   document currency;
3. otherwise the line keeps whatever currency it already has, falling back to the main currency of
   the company.

### MCUR-071: the currency of an entry

The document currency of a Journal Entry is derived, in order of precedence, from the transacted
currency of the originating bank statement line, then the journal's currency, then the currency
already on the entry, then the main currency of the journal's company. A user may override it
while the entry is in draft; the override survives, and every line follows by `MCUR-070`.

### MCUR-072: the company currency column is the one that must balance

**Industry-standard default.** A Journal Entry is balanced when the sum of the balances over its
accountable items is zero at the company currency's precision. The sum of the amounts in currency
is *not* required to be zero, because different items of one entry may carry different document
currencies, as a bank transaction in three currencies does. Where every item of an entry shares a
single document currency, the document currency column also sums to zero as a consequence of the
way the balances are derived from a single document rate, but that is a property of the derivation
and not a validated constraint.

### MCUR-073: residual amounts exist only on reconcilable and cash-like accounts

The two residual amounts are computed only for an item whose account allows reconciliation, or
whose account type is cash or credit card. On any other account both residuals are forced to zero
and the reconciled flag is forced to false.

### MCUR-074: matched foreign amounts are rounded inside the aggregation

When the residual amount in currency is computed, the sums of the matched amounts in the debit and
credit currencies are each rounded to the number of decimal places of the currency concerned
**inside** the aggregation, before being subtracted. A rebuild that sums first and rounds at the
end can differ by one unit in the last place over a chain of many matchings.

### MCUR-075: which of the two amounts is derived

The rule is stated in [entities.md](entities.md) section 4.1: the amount in currency is derived
when only the balance was supplied; the balance is derived when only the amount in currency was
supplied; both are kept when both were supplied; and two forcings override those cases — the
equality forcing for an item in the company currency on a non-invoice entry, and the re-derivation
of the balance on an invoice-like entry whenever the amount in currency, the item rate or the
document type changes.

---

## E. The document rate on invoices, bills, credit notes, debit notes and receipts

### MCUR-080: the document rate must be strictly positive

For a document that is an invoice, a bill, a credit note, a debit note or a receipt, with a
company set and a document currency different from the company's main currency, the document rate
must be strictly greater than zero.

Message: "The currency rate must be strictly positive."

Typing zero or a negative value in a form raises the message and leaves the previous value in
place.

### MCUR-081: the document rate is recomputed on every change that can move it

The document rate is reset to the expected rate whenever the document currency, the company
currency, the company, the invoice date or the taxable supply date changes on such a document. The
expected rate is the rate from the company currency to the document currency at the document's
rate date.

### MCUR-082: a manually entered document rate survives posting

When a document has no invoice date at the moment of posting, the invoice date is set to today.
For a sale document that would normally re-trigger `MCUR-081` and discard the manual rate. To
prevent that, the rate is treated as manual when it differs from the expected rate at the
document's creation date, and in that case the rate field is write-protected while the invoice
date is assigned. A rate that equals the expected rate is not treated as manual and is recomputed
at the new invoice date. A purchase document is unaffected, because posting one without a document
date is refused with "The Bill/Refund date is required to validate this document."

### MCUR-083: changing the document rate reapplies it to the company currency column only

When the document rate changes on a draft document, the amounts in the document currency of every
base line and every tax line are preserved, and only the company currency balances are re-derived
from the new rate. The tax amounts are not recomputed from the tax rules; they are re-translated.

### MCUR-084: the document rate is not copied when a document is duplicated

The document rate is excluded from the values carried over by a duplication. The copy recomputes
the expected rate at its own rate date.

### MCUR-085: the rate refresh operation

The rate refresh operation sets the document rate back to the expected rate of the document's
current rate date, discarding a manual override. It is available while the document is in draft
and is refused on a posted document by the general ledger's posted-entry protection.

### MCUR-086: the rate on a line of a non-invoice entry

For a line of a miscellaneous entry, a payment entry or a bank transaction entry, the applicable
rate is read from the rate table at the entry's invoice date when it has one, otherwise its
accounting date, otherwise today. There is no stored document rate on such an entry.

### MCUR-087: the rate date of a document

The rate date of a document is its invoice date when one is set, and today in the reader's time
zone otherwise. Both the expected rate and, through `MCUR-081`, the applied rate are evaluated at
that date.

### MCUR-088: the expected rate of a document with no currency

When a document carries no currency at all, its expected rate is exactly one.

---

## F. Exchange difference entries

### MCUR-100: when an exchange difference arises

An exchange difference instruction is produced during a matching whenever, after the matched
amounts have been computed, one of the two matched items retains a residual that the matching
cannot consume:

- when the matching is measured in the company currency, a residual left in a document currency on
  a fully matched side;
- when the matching is measured in a document currency, a residual left in the company currency on
  a fully matched side, or a residual needed to keep the ratio between the two columns consistent
  on a partially matched side.

The exact formulas are in [calculations.md](calculations.md) section 14.

### MCUR-101: no exchange difference is produced when explicitly suppressed

The computation is skipped when the operation context requests it. Two distinct suppressions exist
and both must be honoured: one that suppresses differences for the current operation, and one that
suppresses them for the current operation **and** every reconciliation it triggers. The second is
what prevents an exchange difference entry from generating an exchange difference entry of its own
when its correction line is matched against the item it repairs.

### MCUR-102: the exchange journal must be configured

Creating an exchange difference entry without an exchange journal configured on the company is
refused, and the whole reconciliation is abandoned.

Message: "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to
manage automatically the booking of accounting entries related to differences between exchange
rates."

### MCUR-103: the loss account must be configured

Message: "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage
automatically the booking of accounting entries related to differences between exchange rates."

### MCUR-104: the gain account must be configured

Message: "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage
automatically the booking of accounting entries related to differences between exchange rates."

Both account checks are performed for every company whose journal takes part in the batch, before
any entry is created, so that a misconfiguration fails the whole reconciliation rather than
leaving it half done.

### MCUR-105: the account is chosen by the sign of the instruction

A **positive** instruction amount — an excess of debit that must be written off — uses the company's loss
exchange account. A **zero or negative** instruction amount uses the company's gain exchange
account. The sign of the instruction amount, and nothing else, chooses the account. Stated once
more because it reads as counter-intuitive: a positive repair removes company-currency value from a
*debit* item, which is a loss; a negative repair removes company-currency value from a *credit*
item, which is a gain.

### MCUR-106: the date of an exchange difference entry

The date is the greater of the two matched items' accounting dates, passed through the exchange
journal's accounting-date rule, and then raised to the accounting date of every item submitted
with an instruction. The full rule, including the lock-date displacement and the numbering reset
period, is in [calculations.md](calculations.md) section 16.

### MCUR-107: the exchange difference entry is posted only when both matched entries are posted

The generated entry is posted immediately, without the soft posting delay, when both matched items
belong to posted entries. Otherwise it is left in draft and will be posted, or discarded, together
with the draft entry it belongs to.

### MCUR-108: the exchange difference entry is always tax-exigible

The generated entry is flagged always tax-exigible, meaning its lines are never deferred by the
cash basis mechanism, and it never carries a tax. An exchange difference is a realised result and
is recognised immediately.

### MCUR-109: the label of every exchange difference line

Every line of the generated entry carries the label "Currency exchange rate difference".

### MCUR-110: the exchange difference entry is linked to the matching

Each generated entry is linked to exactly one partial matching through the matching's exchange
difference entry field. When several matchings are produced in the same batch, each generated
entry is attached to the first matching that involves one of the items it corrects and that does
not yet carry an entry; no matching ever carries two entries.

### MCUR-111: undoing a matching reverses its exchange difference entry

Deleting a partial matching that carries an exchange difference entry:

1. collects the linked exchange difference entry, together with any cash basis entry generated by
   the same matching;
2. collects the payments in the paid state whose signed amount equals the matched amount, so that
   they can be returned to the in-process state afterwards;
3. deletes the matchings first, which is what breaks the recursion, then deletes the Full
   Reconciliation records that covered them;
4. reverses every collected entry that is not in draft, in cancelling mode, with the reversal date
   computed by [calculations.md](calculations.md) section 17 and the reversal's internal reference
   set to "Reversal of: *the entry number of the reversed entry*";
5. deletes outright every collected entry that is still in draft;
6. recomputes the matching numbers of every item that remains connected;
7. returns the collected payments to the in-process state.

### MCUR-112: an exchange difference line carries the other column at zero

When the correction is expressed in the company currency column, the two generated lines carry the
corrected item's currency together with an amount in currency of zero, except when the corrected
item is itself in the company currency, in which case the amount in currency equals the
correction. The correction changes only the company currency valuation and must leave the document
currency residual untouched. The sign check of `MCUR-060` is satisfied because one of the two
columns is zero.

When the correction is expressed in the document currency column — which happens when the matching
was measured in the company currency — the two generated lines carry the document currency amount
and a balance of zero.

### MCUR-113: matching an exchange difference line consumes no document currency residual

When an exchange difference correction line is matched against the item it corrects, the matching
is detected as an exchange-line matching: both items share a document currency but at least one of
them has no residual left in it. Both matched amounts in the document currencies are then forced
to zero, and the matched amount in the company currency is the whole residual. The detection rule
is in [calculations.md](calculations.md) section 13.2.

### MCUR-114: a rounding artefact does not produce an exchange difference

When a matching is measured in a foreign currency and each side's translation of the matched
foreign amount into the company currency falls inside the interval the other side's translation
stands for, the two sides disagree only by a rounding artefact. The matched amount is then snapped
onto the smaller of the two outstanding company-currency residuals, and both sides adopt it, so
that neither a spurious exchange difference entry nor a spurious open residual is produced. The
four comparisons that make up the condition are in [calculations.md](calculations.md) section
13.4.

### MCUR-115: the residuals are decremented after the exchange handling

After the exchange instructions have been produced, and regardless of whether any was, the running
residuals are decremented: the debit side loses the matched amount in the company currency and the
matched amount in the debit currency, and the credit side gains the matched amount in the company
currency and the matched amount in the credit currency. A side is declared finished, and dropped
from the matching loop, when **both** of its remaining amounts are zero at their own currencies'
precisions.

### MCUR-116: the currency a matching is measured in

The reconciliation currency is the debit item's currency when that currency is not the company
currency and both sides can offer a residual in it; otherwise the credit item's currency under the
same condition; otherwise the company currency. If the chosen currency is missing from either
side's offer, the pairing is abandoned: the side that lacks it is marked as having nothing left
and the loop advances to the next item on that side.

### MCUR-117: the date of the exchange entry is raised by every submitted line

The candidate date computed by `MCUR-106` is raised to the accounting date of **every** item
submitted with the batch of instructions, including an item whose instruction is afterwards
skipped for being zero at its own precision. A rebuild that raises the date only for the items
that produce lines can date the entry one period earlier.

### MCUR-118: the correction line inherits the matching of the item it corrects

The correction line of a generated entry carries the account, the counterparty and the currency of
the item it corrects, inherits that item's full reconciliation link, and is marked as reconciling
against it, which is what triggers the second partial matching of `MCUR-113`. The counterpart line
carries the same counterparty and the same currency, and the analytic distribution supplied by the
caller when one was supplied.

---

## G. Guards on any reconciliation involving currencies

These guards are owned by
[../payments-and-bank-reconciliation/README.md](../payments-and-bank-reconciliation/README.md);
they are restated here because the currency behaviour depends on them.

### MCUR-130: items already reconciled may not be reconciled again

Message: "You are trying to reconcile some entries that are already reconciled."

An item that is not reconciled but carries a partial matching number is exempt, because it may
still receive further matchings.

### MCUR-131: cancelled entries may not be reconciled

Message: "You can not reconcile cancelled entries."

### MCUR-132: all items of one matching batch share one account

Message: "Entries are not from the same account: *the comma-separated account display names*"

### MCUR-133: all items of one matching batch share one root company

Message: "Entries don't belong to the same company: *the comma-separated company display names*"

A single root company is required because the rate table is resolved against the root company, and
two root companies may hold different rates for the same currency on the same date.

### MCUR-134: the account must allow reconciliation

Message: "Account *the account display name* does not allow reconciliation. First change the
configuration of this account to allow it."

Cash accounts and credit card accounts are exempt.

### MCUR-135: a batch is split by currency before matching

Within a batch, items are grouped by their item currency and each group is matched on its own
before the remainder is matched across groups. This keeps same-currency matchings, which are exact
in the document currency, separate from cross-currency matchings, which are exact only in the
company currency.

### MCUR-136: the matching order

Within a group, items are ordered by maturity date, falling back to the accounting date when there
is none, then by currency, then by amount in currency, then by balance. A reduced ordering that
stops after the currency is used when the operation context requests it. When a group spans
several counterparties, the items are additionally ordered by counterparty, so that the items of
one counterparty close among themselves before any cross-counterparty matching occurs.

### MCUR-137: the payment's rate wins over the rate table

When an item expressed in the company currency is matched against an item originating from a
payment or a bank transaction that is expressed in a foreign currency, the mirror rate used to
express the company-currency item in the foreign currency is the *payment's* implied rate, not the
rate table's rate for that date. The settlement value recorded by the bank is therefore the value
that closes the receivable, and the whole rate movement lands in the exchange difference entry
rather than being split between a rate difference and a residual. A rate explicitly forced by the
payment registration screen takes precedence even over the payment's implied rate.

### MCUR-138: a payment follows the matchings of its counterpart item

Creating a partial matching sets a linked payment from the in-process state to the paid state when
the payment has no outstanding account and its signed amount equals the matched amount in the
payment's currency, compared at that currency's precision. Deleting such a matching returns the
payment to the in-process state.

---

## H. Bank transactions in a foreign currency

### MCUR-150: the transacted currency must differ from the bank account currency

Message: "The foreign currency must be different than the journal one: *the bank account currency
code*"

### MCUR-151: an amount in currency requires a transacted currency

Message: "You can't provide an amount in foreign currency without specifying a foreign currency."

### MCUR-152: a transacted currency requires an amount in currency

Message: "You can't provide a foreign currency without specifying an amount in 'Amount in
Currency' field."

### MCUR-153: a redundant transacted currency supplied on creation is silently dropped

When a statement line is created with a journal and a transacted currency, and the supplied
currency equals the journal's effective currency, the transacted currency is cleared and the
amount in currency is set to zero instead of `MCUR-150` being raised. This makes a bulk import
tolerant of a feed that always states the currency.

### MCUR-154: the amount in currency is derived when it is not supplied

When a transacted currency is set, a date is known and no amount in currency has been entered, the
amount in currency is derived by converting the amount from the bank account currency to the
transacted currency at the line's date. An amount already entered is never overwritten.

### MCUR-155: a suspense account is required

Generating the journal items of a statement line without a counterpart account and without a
suspense account on the journal is refused.

Message: "You can't create a new statement line without a suspense account set on the *journal
display name* journal."

### MCUR-156: the company currency amount comes from the bank account currency

The company currency value of a bank transaction is derived from the amount in the bank account
currency, not from the amount in the transacted currency, except when the bank account currency is
itself the company currency or when the transacted currency is the company currency. The bank
moved a known quantity of the bank account currency, and the ledger must reflect that quantity.

### MCUR-157: counterpart amounts use the bank's implied rates

When an item is matched against a bank transaction, the counterpart amounts are derived from the
rates implied by the transaction itself, computed from the three amounts the transaction carries,
rather than from the rate table. This guarantees that the transaction side reconciles to exactly
zero and pushes the whole rate movement onto the document side.

---

## I. The main currency of a company

### MCUR-170: a branch shares its root company's currency

The main currency is delegated from the root company. Every branch must carry the same value as
its root; the field is shown read-only on a branch's form.

Message: "The *field label* of a subsidiary must be the same as it's root company." The placeholder
is the translated label of the field being written, which for this field is "Currency". The
message is reproduced as it is shown, including its grammatical slip.

Selecting a parent company in a form copies the parent's currency onto the record immediately.

### MCUR-171: the main currency cannot change once entries exist

Writing a different main currency on a Company is refused when at least one journal item exists for
the root company or for any company below it in the hierarchy, whether active or archived.

Message: "You cannot change the currency of the company since some journal items already exist"

The check is deliberately broad: changing a main currency would invalidate every stored balance,
and no restatement is offered. A company that must change its functional currency closes its
books, creates a new company, and carries the balances over with an opening entry.

### MCUR-172: choosing a country proposes its currency

Selecting a country on a Company form sets the company's main currency to the currency associated
with that country. The user may then change it, subject to `MCUR-171`.

### MCUR-173: the default main currency of a new company

A new Company takes as its main currency the main currency of the company of the user creating it.

---

## J. Access and visibility

### MCUR-190: everybody may read currencies and rates

Read access to Currency and to Currency Rate is granted to internal users, to portal users and to
unauthenticated visitors. No monetary value could be rendered without it.

### MCUR-191: only administrators and accounting managers may write

Creation, writing and deletion on Currency and on Currency Rate are granted to the system
administration group and to the accounting manager group. Every other group has read access only.

### MCUR-192: currency fields are shown only when more than one currency is active

Fields and columns that only matter in a multi-currency setting — in particular the document
currency selector, the amount in currency column, the company column of a rate row and the
document rate field — are shown only to holders of the multi-currency permission group, which is
itself granted and revoked automatically by `MCUR-008`. The group controls *visibility*, not the
*right* to change a value.

### MCUR-193: the company column of a rate row is shown only in a multi-company setting

The company field of a rate row is visible only to holders of the multi-company permission group,
with the placeholder text "Visible to all" when it is empty.

---

## K. Unrealised gains and losses

### MCUR-200: open foreign currency balances are revalued by a reversing adjustment

**Industry-standard default.** A balance still open at a reporting date, expressed in a currency
other than the company currency, carries an unrealised gain or loss equal to the difference
between its company currency book value and its value at the reporting date's rate. The system
does not post that difference automatically; it is produced by an adjustment operation the
accountant runs from the unrealised gain and loss report, with the following properties.

1. The report lists every open item on every balance sheet account whose items carry a foreign
   currency, grouped by account and currency, showing for each group the balance in the foreign
   currency, the book value in the company currency, the value at the reporting date's rate, and
   the adjustment needed to move from the former to the latter.
2. The accountant may substitute a manual rate per currency for the reporting date instead of the
   rate table's rate; a banner then offers to reset to the rate table's value.
3. The adjustment operation asks for a journal, an expense account, an income account, a reporting
   date and a reversal date.
4. It posts one entry at the reporting date debiting or crediting each account for its adjustment
   amount, with the opposite side on the expense account when the total adjustment is a loss and
   on the income account when it is a gain, and with a zero amount in the document currency on
   every line so that the foreign currency position is unchanged.
5. It posts a reversing entry at the reversal date, which is by convention the first day of the
   following period, so that the adjustment does not survive into the period in which the balance
   is actually settled and would otherwise be double counted against the realised difference.
6. After posting, the adjustment column of the report reads zero for every group.

The distinction from a realised difference is exact: a realised difference is produced by a
*matching*, is never reversed while the matching stands, and hits the configured gain or loss
account of `MCUR-105`; an unrealised difference is produced by a *reporting date*, is always
reversed, and hits the accounts chosen for that adjustment.

---

## L. Presentation: formatting and spelling

### MCUR-210: the symbol is placed as the currency prescribes

A formatted amount carries its currency's symbol before the number when the symbol position is
`before`, and after the number when it is `after`. In both cases exactly one non-breaking space
separates the symbol from the number, and the space is emitted even when the currency has no
symbol. Inside the number, every ordinary space produced by the grouping is replaced by a
non-breaking space, and every minus sign is followed by a zero-width non-breaking space, so that a
line break can never separate the sign from the digits.

### MCUR-211: the grouping pattern of the reader's language is applied

The whole part of a formatted amount is grouped with the reader's thousands separator according to
the reader's grouping pattern, read from the decimal separator outwards by the rule of
[entities.md](entities.md) section 14.1, and the decimal point is then replaced by the reader's
decimal separator.

### MCUR-212: trailing zeros are suppressed only when the caller asks

A formatted amount carries exactly the decimal places of its currency. When the caller asks for
trailing zeros to be suppressed, a run of trailing zeros is removed together with the decimal
separator that precedes it, so that a whole amount shows no fractional part.

### MCUR-213: a negative zero is presented without a sign

A currency's format operation adds a positive zero to the amount before formatting, so that an
amount which rounds to a negative zero is presented as a plain zero and never as a minus sign in
front of nothing. A rebuild must reproduce this: a report showing a minus sign in front of nothing
is treated as a defect.

### MCUR-214: an amount is spelled with the currency's own unit and subunit labels

An amount written out in words uses the currency's unit label and, when a fraction remains, the
word "and" and the currency's subunit label. The fractional part is read as a whole number of
subunits, not as a fraction, because the subunit label already supplies the scale. When the
reader's language has no spelling rules available, English is used; when no spelling facility is
available at all, the result is the empty string and a warning is logged. The algorithm is in
[calculations.md](calculations.md) section 25.

### MCUR-215: the compact rendering stops at the fourth metric step

The compact rendering used on dashboards divides by one thousand while the absolute value is one
thousand or more, stepping through the suffixes none, `k`, `M`, `G` and stopping at `T`. Larger
values keep the last suffix and grow the number. The symbol is attached immediately before the
number when the symbol position is `before`, and after a single ordinary space when it is `after`.

---

## 13. Mapping of former rule identifiers

Two independently written drafts of this folder existed before this catalogue was consolidated.
One of them carried rule identifiers of the form `MCUR-RULE-nnn`; the other carried no rule
identifiers at all and stated its rules inside numbered sections of its entity and calculation
documents. The table below maps every former reference onto the current identifier, so that a
citation taken from either draft can be resolved.

| Former identifier or section reference | Current identifier |
|---|---|
| `MCUR-RULE-001` | `MCUR-001` |
| `MCUR-RULE-002` | `MCUR-002` |
| `MCUR-RULE-003` | `MCUR-003` |
| `MCUR-RULE-004` | `MCUR-004` |
| `MCUR-RULE-005` | `MCUR-005` |
| `MCUR-RULE-006` | `MCUR-006` |
| `MCUR-RULE-007` | `MCUR-007` |
| `MCUR-RULE-008` | `MCUR-008` |
| `MCUR-RULE-009` | `MCUR-009` |
| `MCUR-RULE-010` | `MCUR-010` |
| `MCUR-RULE-011` | `MCUR-011` |
| `MCUR-RULE-012` | `MCUR-012` |
| `MCUR-RULE-013` | `MCUR-013` |
| `MCUR-RULE-014` | `MCUR-014` |
| `MCUR-RULE-015` | `MCUR-015` |
| `MCUR-RULE-016` | `MCUR-016` |
| `MCUR-RULE-020` to `MCUR-RULE-032` | `MCUR-020` to `MCUR-032`, number for number |
| `MCUR-RULE-040` to `MCUR-RULE-046` | `MCUR-040` to `MCUR-046`, number for number |
| `MCUR-RULE-060` to `MCUR-RULE-072` | `MCUR-060` to `MCUR-072`, number for number |
| `MCUR-RULE-080` to `MCUR-RULE-086` | `MCUR-080` to `MCUR-086`, number for number |
| `MCUR-RULE-100` to `MCUR-RULE-113` | `MCUR-100` to `MCUR-113`, number for number |
| `MCUR-RULE-130` to `MCUR-RULE-137` | `MCUR-130` to `MCUR-137`, number for number |
| `MCUR-RULE-150` to `MCUR-RULE-157` | `MCUR-150` to `MCUR-157`, number for number |
| `MCUR-RULE-170` to `MCUR-RULE-173` | `MCUR-170` to `MCUR-173`, number for number |
| `MCUR-RULE-190` to `MCUR-RULE-193` | `MCUR-190` to `MCUR-193`, number for number |
| `MCUR-RULE-200` | `MCUR-200` |
| Entities, "Currency", ordering and display | `MCUR-012`, `MCUR-017`, `MCUR-019` |
| Entities, "Currency", uniqueness and validation | `MCUR-001`, `MCUR-003`, `MCUR-007` |
| Entities, "Currency", the multi-currency permission group | `MCUR-008` |
| Entities, "Currency", cache invalidation | `MCUR-014`, `MCUR-036` |
| Entities, "Currency Rate", the sanitising rule | `MCUR-024` |
| Entities, "Currency Rate", validation | `MCUR-020`, `MCUR-022`, `MCUR-023`, `MCUR-027` |
| Entities, "Currency Rate", the large-movement warning | `MCUR-021` |
| Entities, "Currency Rate", deriving a missing rate | `MCUR-025` |
| Entities, "Currency Rate", company rate arithmetic | `MCUR-035` |
| Entities, "Currency Rate", ordering and display | `MCUR-034`, `MCUR-037` |
| Entities, "Currency Rate", multi-company behaviour | `MCUR-020`, `MCUR-033`, `MCUR-038`, `MCUR-039` |
| Entities, "Company", validation | `MCUR-170`, `MCUR-171` |
| Entities, "Journal Entry", validation | `MCUR-016`, `MCUR-080`, `MCUR-087`, `MCUR-088` |
| Entities, "Journal Item", the derivation of the two amounts | `MCUR-065`, `MCUR-066`, `MCUR-075` |
| Entities, "Journal Item", the sign invariant | `MCUR-060`, `MCUR-061`, `MCUR-062`, `MCUR-063`, `MCUR-064` |
| Entities, "Journal Item", the residual computation | `MCUR-073`, `MCUR-074` |
| Entities, "Partial Reconciliation", validation | `MCUR-138` and [entities.md](entities.md) section 5 |
| Entities, "Language", formatting aspects | `MCUR-211` |
| Calculations, the five rounding methods | `MCUR-050` |
| Calculations, the rounding routine | `MCUR-041`, `MCUR-049`, `MCUR-052` |
| Calculations, the zero test | `MCUR-044` |
| Calculations, the three-way comparison | `MCUR-043` |
| Calculations, exact euclidean division | `MCUR-053` |
| Calculations, rendering a number as a string | `MCUR-051` |
| Calculations, the rate lookup | `MCUR-028`, `MCUR-029`, `MCUR-030`, `MCUR-033` |
| Calculations, conversion between two currencies | `MCUR-040`, `MCUR-046`, `MCUR-047`, `MCUR-048` |
| Calculations, the two amounts of a journal item | `MCUR-065`, `MCUR-075` |
| Calculations, residual arithmetic | `MCUR-073`, `MCUR-074` |
| Calculations, choosing the reconciliation currency | `MCUR-116`, `MCUR-137` |
| Calculations, the partial amounts | `MCUR-114` |
| Calculations, the exchange difference amounts | `MCUR-100`, `MCUR-101`, `MCUR-105`, `MCUR-115` |
| Calculations, formatting an amount for a reader | `MCUR-210`, `MCUR-211`, `MCUR-212`, `MCUR-213`, `MCUR-215` |
| Calculations, writing an amount in words | `MCUR-214` |
| Calculations, the catalogue of shipped currencies | `MCUR-018` |

## 14. Reconciliation notes

1. **Numbering.** The former `MCUR-RULE-nnn` identifiers have been shortened to `MCUR-nnn` with
   their numbers preserved, and the rules that one draft stated only as prose inside its entity
   and calculation sections have been given numbers in the free ranges of the same grouped scheme:
   `MCUR-017` to `MCUR-019`, `MCUR-033` to `MCUR-039`, `MCUR-047` to `MCUR-053`, `MCUR-073` to
   `MCUR-075`, `MCUR-087`, `MCUR-088`, `MCUR-114` to `MCUR-118`, `MCUR-138` and `MCUR-210` to
   `MCUR-215`. Section 13 maps every former reference.
2. **The exemption in `MCUR-064`.** One draft stated that the account-currency check is skipped for
   the journal's own default account and suspense account. The exemption branch exists but is
   evaluated after the check, so it exempts nothing; the rule now records the observed behaviour
   and marks the vestigial branch as a compatibility finding.
3. **The carry-forward in `MCUR-025`.** Both drafts stated that an omitted technical rate is stored
   as the carried-forward value. The rule now states where the carry-forward is visible, when it
   becomes a stored value, and what happens when none of the three rate fields is supplied.
4. **The invalidation keys in `MCUR-014`.** One draft listed four trigger keys and the other five.
   Five is correct, and one of them names a field that does not exist; the rule records that as a
   compatibility finding.
5. **Messages.** Where the two drafts quoted a message differently, the text shown by the system is
   reproduced: "Missing foreign currencies on partials having ids: " keeps its abbreviation, and
   the subsidiary message of `MCUR-170` keeps its placeholder and its grammatical slip.
