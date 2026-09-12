# Multi-Currency — Glossary

Every term this domain uses, defined. Terms are written in full words; where a term names a stored
value or a reproduced identifier, that identifier is given in code font beside it. Entries are
alphabetical.

**Account currency.** The currency optionally carried by an account (`currency_id` on Account).
When set, every journal item posted to the account must carry it; when empty, the account accepts
any currency. See [business-rules.md](business-rules.md) `MCUR-064`, `MCUR-068` and `MCUR-069`.

**Accounting rate.** See *implied rate*.

**Activity flag.** The true or false field (`active`) that decides whether a currency appears in
selection lists and in default queries. A currency whose flag is false is *archived*, not deleted:
every record that references it keeps working.

**Amount in currency.** The monetary column of a journal item (`amount_currency`) expressed in the
item's own currency, as opposed to the balance, which is expressed in the company currency.
Positive on a debit and negative on a credit.

**Applied rate.** The rate actually used to translate a document into the company currency. For an
invoice, a bill, a credit note, a debit note or a receipt it is the stored document rate, which may
have been entered by hand; for any other entry it is read from the rate table at the entry's date.

**Archived currency.** A currency whose activity flag is false. It is hidden from selection lists,
its price lists are archived with it, and a draft document expressed in it can no longer be posted.

**Average rate.** A rate type used for consolidation, equal to the day-weighted mean of the rates in
force across a reporting period. Applied to profit and loss positions. Stored selection value
`average`.

**Balance.** The monetary column of a journal item (`balance`) expressed in the company currency,
equal to the debit minus the credit. It is the column in which a journal entry must sum to zero.

**Bank account currency.** The currency in which a bank account is held, carried by the journal that
represents the bank account. Distinct both from the company currency and from the currency in which
an individual transaction may have been executed.

**Branch company.** A company that has a parent company. A branch never owns rate rows and never
chooses its own main currency: both resolve against its root company.

**Cached currency catalogue.** The map from currency identifier to currency code, symbol, symbol
position and decimal place count that the display layer uses to render a monetary value without a
round trip. Invalidated by the five keys of `MCUR-014`.

**Compact rendering.** The dashboard form of an amount, which divides by one thousand while the
absolute value is one thousand or more and appends a metric suffix, stopping at the fourth step.

**Company currency.** The main currency of a company. Every balance, debit and credit of every
journal item of that company is expressed in it, and it is the denominator of the rate an accountant
reads on a rate row. Also called the main currency and the functional currency.

**Company rate.** The rate an accountant enters on a rate row (`company_rate`): how many units of
the row's currency correspond to one unit of the company currency. Derived from the technical rate
by dividing it by the rate the company currency itself carries.

**Compensation term.** The quantity added to a normalised value before it is snapped onto the
rounding grid, equal to two raised to the power of the base-two logarithm of the absolute normalised
value minus fifty. It absorbs the accumulated error of prior floating-point operations so that a
value which lies marginally below a tie is still treated as a tie.

**Consolidation rate table.** The table built for the duration of a report that maps each company,
each period and each rate type to a factor, used to add together amounts produced by companies whose
main currencies differ.

**Conversion factor.** The multiplier that turns an amount expressed in one currency into an amount
expressed in another, equal to the rate of the target currency divided by the rate of the source
currency, both taken for the same company on the same date.

**Correction line.** The line of an exchange difference entry that sits on the account of the item
being corrected, as opposed to the counterpart line, which sits on the gain or the loss account.

**Cross-rate.** The composed factor between two currencies neither of which is the company currency,
formed by one division of the two technical rates and applied in a single multiplication.

**Cumulative translation adjustment.** The equity position that accumulates the difference between
translating a balance sheet at the closing rate and translating the corresponding result at the
average rate. This domain supplies the rate types that make it computable; presenting it belongs to
the reporting layer and to the fiscal localization that prescribes it.

**Currency.** A unit of money, with a code, a symbol, a symbol position, a rounding factor, derived
decimal places, unit and subunit labels, an activity flag and a set of dated rates. Transport name
`res.currency`.

**Currency code.** The three-letter alphabetic code of the international currency-code standard,
stored under the identifier `name` on Currency, unique across the catalogue including archived
records, and used as the currency's display name.

**Currency Rate.** A dated rate record: one currency, one date, one optional company scope and one
technical rate. Transport name `res.currency.rate`.

**Current rate.** Two meanings, distinguished by context. On a currency, the derived field (`rate`)
giving the rate in force today against the main currency of the company in context. In
consolidation, a rate type equal to the rate in force at the end of a reporting period, applied to
balance sheet positions, with the stored selection value `current`.

**Decimal places.** The number of fractional digits used when a monetary amount in a currency is
stored, rendered or spelled (`decimal_places`). Derived from the rounding factor and never entered
directly.

**Decimal precision registry.** A separate registry of named precisions for non-monetary decimals,
owned by the platform foundation. It has no bearing on monetary rounding and is frequently confused
with it.

**Decimal separator.** The character the reader's language places between the whole part and the
fractional part of a rendered number (`decimal_point` on Language).

**Denormalise.** The second half of the rounding routine: multiply the rounded whole number of steps
by the rounding factor, or, when the factor was inverted because it is smaller than one, divide by
the inverted factor.

**Document currency.** The currency written on a document (`currency_id` on Journal Entry) and
carried by every journal item generated from it. The currency the counterparty sees.

**Document rate.** The stored rate on an invoice, a bill, a credit note, a debit note or a receipt
(`invoice_currency_rate`), expressed as units of the document currency per one unit of the company
currency, from which every line balance of the document is derived. Recomputed automatically on
every change of currency, company or date, and overridable by the accountant.

**Exchange difference entry.** A journal entry generated automatically during a matching, in the
company's exchange journal, that writes off the residual left by a rate movement. Its correction
line sits on the account being reconciled and its counterpart line on the company's gain or loss
exchange account.

**Exchange difference instruction.** The working value produced by the matching algorithm that names
an item, a column — the company currency column or the document currency column — and an amount to
be written off. An instruction whose amount is zero at the relevant precision produces no line.

**Exchange journal.** The journal of type general configured on the company
(`currency_exchange_journal_id`) in which exchange difference entries are created.

**Exchange-line matching.** A matching in which one of the two items is an exchange difference
correction line, detected by the two items sharing a document currency while at least one of them
has no residual left in it. In this mode both matched amounts in the document currencies are forced
to zero.

**Expected rate.** The rate the rate table gives for a document's currency at the document's rate
date (`expected_currency_rate`). The value the document rate is reset to whenever the document's
currency, company or date changes, and the value the rate refresh operation restores.

**Fallback rate.** The rate returned when a currency has rate rows but none of them is dated on or
before the requested date: the earliest row's rate, projected backwards without limit. Distinct from
the last resort of one, which applies when the currency has no visible row at all.

**Foreign currency.** Any currency other than the company currency, seen from that company's point
of view.

**Full reconciliation.** A record grouping a set of journal items that close each other completely,
together with the partial matchings that connect them. Created only when every item of the connected
group passes the closure test.

**Functional currency.** See *company currency*.

**Gain exchange account.** The income account credited when a rate movement produces a gain
(`income_currency_exchange_account_id` on Company).

**Grouping pattern.** The list of group sizes, read from the decimal separator outwards, that
decides where the reader's language inserts a thousands separator (`grouping` on Language). A zero
element repeats the preceding size until the digits are exhausted; a negative element stops the
grouping.

**Historical rate.** A rate type used for consolidation, equal to the rate in force at the date of
each individual movement and valid from that date until the next rate change. Applied to balance
sheet positions that must keep their transaction-date valuation. Stored selection value
`historical`.

**Implied rate.** The rate recovered from a posted journal item by dividing its amount in currency by
its balance, in absolute value: the rate at which the item was recorded, to within the rounding of
both columns. The reconciliation algorithm prefers it to the rate table whenever a posted item's own
valuation must be respected.

**Import mark.** A matching number beginning with the letter `I`, written on an item imported with a
matching key whose real matching is deferred until every entry carrying the same mark is posted.

**International currency-code standard.** The standard that assigns each currency a three-letter
alphabetic code and a three-digit numeric code. The alphabetic code is stored under `name` and the
numeric code under `iso_numeric`.

**Inverse company rate.** The reciprocal of the company rate (`inverse_company_rate`): how many units
of the company currency correspond to one unit of the row's currency.

**Item currency.** The currency of a journal item (`currency_id` on Journal Item), in which its
amount in currency, its subtotals and its foreign residual are expressed.

**Journal currency.** The currency optionally carried by a journal (`currency_id` on Journal). When
set, every entry of the journal takes it as its document currency and the journal's own accounts are
forced to it.

**Last resort of one.** The value a rate lookup returns when a currency has no rate row visible to
the company at all. It is what makes a company's main currency behave as the reference of rate one
in a platform that keeps no rate row for it.

**Lock date displacement.** The rule that moves an entry's date forward to the day after the latest
violated lock date when the date the algorithm computed falls in a locked period.

**Loss exchange account.** The expense account debited when a rate movement produces a loss
(`expense_currency_exchange_account_id` on Company).

**Main currency.** See *company currency*.

**Matching number.** The short reference written on every journal item that takes part in a matching
(`matching_number`): the identifier of the full reconciliation when the group closed, the letter `P`
followed by an identifier while the group is only partially matched, or a mark beginning with `I`
for an item imported with a matching key.

**Mirror rate.** The rate used to express a journal item recorded in the company currency as if it
had been recorded in the counterpart item's foreign currency, so that the two can be matched in that
currency. Chosen in order of preference: a rate forced by the payment registration screen, the
counterpart's implied rate when the counterpart is a payment or a bank transaction, and the rate
table otherwise.

**Monetary field.** A decimal field that names a companion field holding its currency. It is rounded
onto that currency's rounding factor both when it is assigned in memory and when it is written to
storage.

**Multi-currency permission group.** The permission group granted to, and revoked from, the
internal-user group automatically according to the number of active currencies. It controls the
*visibility* of currency fields and columns, never the *right* to change a value.

**Non-breaking space.** The space character used between a currency symbol and the number, and in
place of every ordinary space produced by the grouping, so that a line break can never separate them.

**Normalise.** The first half of the rounding routine: divide the value by the rounding factor, or,
when the factor is smaller than one, multiply by the accurately inverted factor. The result is the
value counted in whole rounding steps plus a fraction.

**Partial matching.** A record linking one debit journal item to one credit journal item and carrying
the matched amount in the company currency and the matched amount in each of the two items' own
currencies, all three always positive. Transport name `account.partial.reconcile`.

**Payment state.** The stored selection field of a document (`payment_state`) that says how far it
has been settled. The test that drives it reads the document's residual at the **document
currency's** precision.

**Rate.** See *technical rate*, *company rate* and *inverse company rate*, which are three views of
one quantity.

**Rate as text.** The derived one-line statement of a currency's current rate (`rate_string`): the
digit one, the evaluation currency's code, an equals sign, the current rate to six decimal digits
and this currency's code. Empty when the currency is the main currency of the company in context.

**Rate date.** For a rate row, the first day on which the rate applies, stored under the identifier
`name`. For a document, the date at which the rate table is consulted: the invoice date when one is
set, and today otherwise.

**Rate row.** A Currency Rate record: one currency, one date, one optional company scope, one rate.

**Rate service.** An external service that publishes exchange rates, queried by the scheduled rate
update job. Its contract is one operation: given a base currency code, a set of currency codes and a
date, return units of each currency per one unit of the base currency.

**Rate table.** Two different things, distinguished by context. In the ledger sense, the collection
of Currency Rate records. In the reporting sense, the consolidation rate table.

**Realised exchange difference.** The gain or loss recognised when a foreign currency balance is
settled, arising from the difference between the rate at which it was recorded and the rate at which
it was settled. Produced automatically by a matching.

**Reconciliation currency.** The currency in which a single matching is measured: the debit item's
foreign currency when both sides can offer a residual in it, otherwise the credit item's foreign
currency under the same condition, otherwise the company currency.

**Reference of rate one.** See *reference unit of account*.

**Reference unit of account.** The abstract unit against which technical rates are expressed. Its
rate is one by definition. In a platform that keeps no rate row for the company currency, the
company currency and the reference unit coincide.

**Residual amount.** The part of a journal item's balance not yet consumed by matchings
(`amount_residual`), expressed in the company currency.

**Residual amount in currency.** The part of a journal item's amount in currency not yet consumed by
matchings (`amount_residual_currency`), expressed in the item's own currency.

**Root company.** The company at the top of a company hierarchy. Rate rows belong to root companies
only, and every lookup made for a branch resolves against its root.

**Rounding factor.** The grid onto which every amount in a currency is snapped, stored on the
currency (`rounding`). It need not be a power of ten. Also called the rounding step.

**Rounding method.** One of the five ways a tie is resolved: half away from zero (`HALF-UP`), half
towards zero (`HALF-DOWN`), half to even (`HALF-EVEN`), away from zero (`UP`) and towards zero
(`DOWN`). Money uses half away from zero unless a caller names another.

**Rounding step.** See *rounding factor*.

**Rounding tolerance.** The device by which the matching algorithm recognises that the difference
between two translations of the same foreign amount is a rounding artefact rather than a genuine
rate movement, and collapses them onto a single value so that no spurious exchange difference is
produced.

**Sanitising rule.** The precedence applied when more than one of the three rate representations is
supplied in the same write: the technical rate beats the company rate, which beats the inverse
company rate.

**Sign invariant.** The stored check that the balance and the amount in currency of a journal item
never disagree in sign: either both are less than or equal to zero, or both are greater than or
equal to zero.

**Soft posting.** The deferred posting path an ordinary entry may take. An exchange difference entry
whose two matched items both belong to posted entries is posted immediately, without it.

**Step function of time.** The reading of the rate table under which a row is valid from its own date
onwards until a later row of the same scope supersedes it, and the earliest row is projected
backwards without limit.

**Subunit label.** The plural name of one hundredth — more generally of one rounding step — of a
currency unit: cents, centimes, fils (`currency_subunit_label`). Used when an amount is spelled in
words.

**Suspense account.** The holding account on which the counterpart of an unmatched bank transaction
sits until the transaction is matched.

**Symbol position.** Whether a currency's symbol is printed before or after the numeric part of an
amount (`position`, with the stored values `before` and `after`).

**Tax-exigible entry.** An entry flagged so that its lines are never deferred by the cash basis
mechanism. Every exchange difference entry carries the flag and never carries a tax.

**Technical features group.** The permission group that reveals the rounding factor, the decimal
places and the technical rate column, none of which an ordinary accountant needs.

**Technical rate.** The rate stored on a rate row (`rate`): how many units of the row's currency
correspond to one unit of the reference unit of account. Hidden from ordinary users, who read and
enter the company rate instead.

**Thousands separator.** The character the reader's language inserts between digit groups
(`thousands_sep` on Language). It may be empty, which suppresses the separator while keeping the
grouping positions.

**Three-way comparison.** The operation that rounds two amounts onto a currency and returns minus
one, zero or one according to which is the greater. It rounds each operand *before* subtracting,
which is what distinguishes it from the zero test.

**Transacted currency.** The currency in which a bank transaction was actually executed, when it
differs from the bank account currency (`foreign_currency_id` on Bank Statement Line).

**Unit label.** The plural name of one whole unit of a currency: dollars, euros, francs
(`currency_unit_label`). Used when an amount is spelled in words.

**Unrealised exchange difference.** The gain or loss that would arise if an open foreign currency
balance were settled at the reporting date's rate. Recognised only by a deliberate adjustment entry,
which is always reversed in the following period.

**Zero test.** The operation deciding whether an amount is materially nil at a currency's precision:
true when the rounded absolute value is strictly below the rounding factor. Distinct from comparing
two amounts, because it rounds *after* subtracting rather than before.

---

## Reconciliation notes

1. **Provenance.** One draft carried a glossary of the terms it used; the other carried none but
   introduced several terms in its prose. Every entry of the first draft is kept, and the terms the
   second used without defining — the compensation term, normalising and denormalising, the last
   resort of one, the step function of time, the lock date displacement, the sanitising rule, the
   sign invariant, the import mark, the cached currency catalogue, the cross-rate, the correction
   line, the exchange difference instruction, the grouping pattern, the non-breaking space, the
   root company, the branch company, the rounding method, the decimal precision registry, the
   compact rendering, the consolidation rate table, the archived currency, the activity flag, the
   payment state, the tax-exigible entry, soft posting, the technical features group, the
   multi-currency permission group, the rate as text, the account currency, the item currency, the
   journal currency and the international currency-code standard — are defined here as well.
2. **Identifiers beside terms.** Where a term names a stored value, the identifier is given in code
   font beside the definition, so that the glossary can be read against
   [entities.md](entities.md) without a second lookup.
3. **The two meanings of "current rate" and of "rate table".** Both drafts used each phrase in two
   senses. The entries above keep both senses and say which context selects which.
