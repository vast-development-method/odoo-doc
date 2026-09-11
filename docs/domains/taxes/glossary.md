# Taxes — Glossary

Every term used in this folder, defined in full. Terms are listed alphabetically. A term in
**bold** inside a definition is itself defined here.

---

**Account mapping.** A row of a **fiscal position** that says: wherever the account on the left
would be used, use the account on the right instead. Applied to a document line's account before
the entry is built.

**Account tag.** A label attachable to accounts, to **tax distribution lines** or to products.
A tag whose applicability is *taxes* is a **tax grid**. Tags are per country, never per company.

**Accounting derivation.** The step of the **engine** that turns a computed tax amount into the
data an accounting entry needs: which **distribution line** produced what share, on which account,
with which **tax grids**, under which **grouping key**. Specified in `calculations.md` section 8.

**Advance tax payment account.** An account on a **tax group** on which payments made to the
authorities before the return is filed are posted, so that the periodic settlement can take them
into account.

**Affect base of subsequent taxes.** A flag on a **tax**: when set, this tax's amount is added to
the base of every later tax that accepts being affected. See **base affected by previous taxes**.

**Aggregation.** The operation that regroups the **tax details** of one or many **base lines**
under a caller-supplied key and returns, per key, twenty-four amounts: base, tax and untaxed total,
each in *raw*, *rounded* and *target* form, each in the document currency and in the company
currency.

**Amount.** On a **tax**, the number the computation kind interprets: a percentage for a percentage
or a division tax, a monetary amount per unit for a fixed tax, and nothing for a group or a custom
formula. Stored with four decimal places.

**Base affected by previous taxes.** A flag on a **tax**: when clear, this tax ignores whatever
earlier taxes added to the base. Visible only in developer mode and only on a tax that is not
price-included.

**Base amount.** For one tax on one line, the amount the tax was computed on, after any **extra
base** and after the removal of the **price-included** amounts of its own **batch**. It is what the
tax return reports as the taxable base, and it is stored on the tax journal item.

**Base journal item.** An ordinary journal item of a document that carries at least one tax in its
base-tax set. The tax domain sets its **tax grids** and its amounts but never creates it.

**Base line.** The engine's representation of one taxable amount: a set of values holding the
taxes, the price, the quantity, the discount, the currency, the rate, the sign, the refund flag and
a handful of extras. Every consumer of the engine converts its own records into base lines.

**Batch.** A maximal run of consecutive **taxes**, taken in reverse evaluation order, that must be
solved together — typically several **price-included** percentage taxes extracted from one price by
a single division. The membership test is given in `calculations.md` section 3.3.

**Cash basis.** The regime in which a tax becomes due to the authorities only when the document is
paid. See **exigibility** and **transition account**.

**Cash basis entry.** The journal entry the tax domain creates at each partial reconciliation of a
document carrying a tax exigible on payment. It moves the paid share of the tax from the
**transition account** to the real tax account and stamps the **tax grids**.

**Cash basis journal.** The company-level journal in which **cash basis entries** are created.

**Cash rounding.** A feature of the invoicing domain that adjusts a document's grand total to a
multiple of a chosen step, either by adding a dedicated line or by adjusting the largest tax
amount. The tax domain reports the resulting delta in the **totals block**.

**Check digit.** A digit or letter appended to a **tax identification number** and derived
arithmetically from the rest of it, so that a mistyped number can be detected.

**Company currency.** The currency the ledger is kept in. Every amount the engine produces exists
in both the document currency and the company currency, rounded independently.

**Computation key.** An optional label on a **base line** that partitions a document into
independent rounding subsets, so that a down payment deduction or a global discount cannot lend a
cent to the ordinary lines. It takes part in both redistribution groupings.

**Computation kind.** The selection on a **tax** that says how its amount is derived: *group of
taxes*, *fixed*, *percentage*, *percentage tax included* (the **division tax**), or *custom
formula*.

**Country group.** A named set of countries used by a **fiscal position** to match a whole region
at once, and used by the number-validation pipeline to recognise the union prefixes.

**Cross-border verification.** The check of a partner's **tax identification number** against the
union's central register, performed through a relay service. Its outcome is the flag
*intra-community valid*.

**Custom formula tax.** A **tax** whose amount is produced by evaluating a single arithmetic
expression against a restricted context. Evaluated in the first pass, like a **fixed tax**.

**Deductibility.** A percentage on a vendor document line saying how much of the tax may be
reclaimed. The non-recoverable share produces a separate journal item. Not to be confused with a
**distribution line** posting part of a tax to an expense account.

**Deferred tax.** A tax whose **exigibility** is "based on payment"; between posting and payment
its amount sits on the **transition account** and carries no **tax grid**.

**Delta.** The correction the document-wide rounding allocates to one **base line**, so that the
sum of the line balances equals the correctly rounded document total. The accounting balance of a
base line is its rounded untaxed total **plus** its delta.

**Display base.** The base a **tax group** shows in the **totals block**, which is not always its
plain base amount: it is absent for a group of fixed taxes only, and it is the tax-inclusive amount
for a group of price-included **division taxes**.

**Distribution.** See **tax distribution**.

**Distribution line.** See **tax distribution line**.

**Division tax.** A **tax** whose amount is the stated percentage **of the total including the
tax**. Price-excluded, its amount is the base times the percentage divided by one minus the batch
percentage. Price-included, its amount is simply the price times the percentage.

**Document currency.** The currency of the document, in which the price is expressed and in which
the customer or the supplier is invoiced.

**Domestic fiscal position.** The company's own **fiscal position**, computed as the first one
whose country is the company's country, or whose country is empty and whose country group contains
the company's country, ordered by country then by sequence. It is what makes a **tax** count as
*domestic*.

**Doubling checksum.** The check-digit scheme in which every second character's value, counted from
the right, is doubled and the two digits of the product added together. Used by a dozen countries
and, over an alphabet of thirty-six characters, by one.

**Down payment.** A partial invoice raised in advance and later deducted from the final invoice.
The deduction lines carry the **computation key** `down_payment`.

**Early payment discount line.** A line reducing the tax of a document without touching its untaxed
amount, carrying the same taxes as the lines it reduces, with the special type *early payment* and
the special mode *total excluded*.

**Engine.** The pure function that turns a list of **base lines** and a company into rounded tax
amounts, accounting data and a **totals block**. Duplicated on the client side; the two copies must
agree exactly.

**Evaluation order.** The order in which taxes are computed: flattened (groups replaced by their
children at the group's own position), sorted by sequence then identifier, then three passes —
fixed and custom formula backwards, price-included backwards, price-excluded forwards — followed by
a base pass backwards.

**Exigibility.** When a tax becomes due to the authorities: *based on invoice*, meaning at posting;
or *based on payment*, meaning at reconciliation. See **cash basis**.

**Extra base.** The amount another tax added to, or removed from, this tax's base. Two accumulators
exist per tax: one used to compute the amount (frozen once the amount is known) and one used to
report the base (always updated).

**Fiscal country.** The country whose tax reports a company files. Defaults to the company's
country. Determines which taxes and which **tax grids** are offered by default.

**Fiscal position.** A rule set substituting taxes and accounts according to who the counterpart is
and where the goods or services are delivered. It also carries the company's **foreign
registration** for a territory.

**Fixed tax.** A **tax** whose amount is a fixed monetary amount per unit of quantity, multiplied
by the quantity and carrying the sign of the unit price. Evaluated first, so it can move the base
of a **price-included** batch.

**Flattening.** Replacing every **group of taxes** by its children, at the group's own position in
the sequence order, while remembering which group each child came from.

**Foreign registration.** A **tax identification number** the company holds in a territory other
than its **fiscal country**, stored on a **fiscal position** and validated against that territory's
rules. It makes that territory's taxes and tax grids available.

**Grouping key.** The tuple that decides whether two computed tax results share one journal item.
The base part is partner, currency, analytic distribution, account and base taxes; the full key adds
the **distribution line**, the originator group, the resolved account, the downstream taxes and the
**tax grids**.

**Group of taxes.** A **tax** whose computation kind is *group of taxes*: it produces no amount of
its own and is replaced by its children during **flattening**. Nesting is forbidden.

**Included in price.** See **price-included**.

**Intra-community valid.** The stored flag holding the outcome of the **cross-border
verification**.

**Legal notes.** Rich text on a **tax** that must be printed on documents carrying it. A document's
notes are the concatenation, in encounter order and without duplicates, of the notes of every tax
it uses.

**Manual amount.** A base or tax amount pinned by the caller instead of being derived. Stored in
the **stored extra tax data** together with a snapshot of the inputs it was captured against, and
reloaded only when every one of those inputs still matches.

**Modulus eleven over ten.** A recursive check-digit scheme, standardised internationally, in which
a running value starts at five and, for each digit, becomes the digit plus twice the running value
modulo eleven, all modulo ten; the number is valid when the final value is one.

**Modulus ninety-seven over ten.** A check-digit scheme in which each character is replaced by the
decimal writing of its value in base thirty-six, the whole is read as an integer, and the remainder
modulo ninety-seven must be one.

**Negate balance.** The flag derived on an **account tag** from the leading minus sign of the report
expression that names it: it tells the report to show the opposite of the tagged balance.

**Net amount.** On a payment carrying **withholding lines**, the amount that actually reaches the
bank: the payment amount minus the sum of the withheld amounts.

**Non-deductible line.** An extra pair of base lines and one tax item produced on a vendor document
when a product line declares a **deductibility** below one hundred percent.

**Originator group of taxes.** The field on a tax journal item recording the **group of taxes** the
producing tax came from, when it came from one.

**Originator tax.** The field on a tax journal item recording the tax whose **distribution line**
produced it. Its presence is what makes an item a tax item.

**Paid factor.** The ratio applied to a **withholding line**'s original base when only part of the
documents is being paid: the amount being paid divided by the full amount still due, multiplied by
the full amount divided by the documents' total.

**Payment rate.** The rate used by the **cash basis** mechanism to convert a share expressed in the
document currency back into the company currency: either the ratio of the reference line's amounts,
or, when the two reconciled lines are in different currencies, the conversion rate at the payment
date.

**Preceding subtotal.** A label on a **tax group** that makes the **totals block** show a subtotal
carrying that label **before** the group, with a base equal to the untaxed amount plus every tax
accumulated so far.

**Price-excluded.** A **tax** that is added on top of the stated price.

**Price-included.** A **tax** that is already contained in the stated price and must be extracted
from it. Determined by the tax's own override, falling back to the company default.

**Raw amount.** An amount before any rounding. Structured document formats report raw per-line
amounts, because they carry more decimals than a currency does.

**Reverse charge.** A **tax** whose invoice **distribution** contains a tax line with a negative
factor, typically plus one hundred and minus one hundred percent, so that the ledger effect is nil
while two **tax grids** are stamped. Always evaluated as price-excluded.

**Rounding method.** The company-level choice between *round per line* — round the base and every
tax amount of each line as they are computed — and *round per tax* — compute everything unrounded
and round once per tax across the whole document, then redistribute the difference.

**Sequence** (on a tax). The integer that orders taxes for evaluation. A **group of taxes**
contributes its own sequence for its children's position.

**Sign.** The plus one or minus one that turns an engine amount into an accounting balance. It
comes from the host document: a customer invoice's product line is a credit, a vendor bill's a
debit.

**Smooth distribution.** The allocation of a rounding difference over a list of weights, in whole
units of the last decimal place, biggest weight first, with the leftover units handed out one each
in the same order.

**Special mode.** An override of what the supplied price means: *total excluded* (the price is the
amount without any tax) or *total included* (the price is the grand total). Also makes the batching
test ignore the price-inclusion flag.

**Special type.** A marker on a **base line** identifying it as something other than an ordinary
product line: *early payment*, *cash rounding*, *non deductible*, *global discount* or *down
payment*.

**Stored extra tax data.** The structured document kept on a journal item holding the **computation
key**, the **manual amounts** and a snapshot of the price, discount, quantity, currency and rate
they were captured against. It is what makes a manually adjusted tax survive a save and a reversal.

**Subsequent taxes.** The taxes that come after a given tax in **evaluation order** and accept being
affected by it. Recorded on the tax's result and copied onto the produced journal item's base-tax
set, so that a recomputation reapplies them.

**Target amount.** In the redistribution passes, the amount the rounded figures must add up to: the
**manual amount** when one is set, and the **raw amount** otherwise.

**Tax.** The complete definition of one tax: its amount, its **computation kind**, its price
inclusion, its sequence, its scope, its **exigibility**, its base-chaining flags, its **tax group**,
its country and its two **tax distributions**.

**Tax country.** The country whose taxes a document may carry: the **fiscal position**'s country
when the fiscal position holds a **foreign registration**, and the company's **fiscal country**
otherwise.

**Tax details.** The block attached to a **base line** after the computation, holding the four
totals, the two **deltas** and one entry per produced tax result with its raw and rounded base and
tax amounts in both currencies.

**Tax distribution.** The ordered list of **distribution lines** of one **tax** for one document
kind. Every tax has two: one for invoices and bills, one for credit notes and refunds. They must
have the same kinds and the same percentages in the same order.

**Tax distribution line.** One share of a **tax**: a percentage, a kind (*base* or *of tax*), an
account, a set of **tax grids** and a settlement flag. A *base* line produces no journal item; it
only supplies the tags of the **base journal item**.

**Tax grid.** An **account tag** whose applicability is *taxes*. It is the mechanism by which an
amount reaches a line of the tax return.

**Tax group.** The presentation and settlement grouping of taxes: it names the line of the
**totals block** the tax appears on and holds the three accounts the periodic settlement uses.

**Tax identification number.** The number identifying a partner or a company to a tax
administration. Normalised and checked on write against the country's own rules.

**Tax journal item.** A journal item produced by a **distribution line** of kind *of tax*. It
carries the originator tax, the distribution line, the base amount and the **tax grids**.

**Tax label.** The short text printed next to a tax on a document: the invoice label when set,
otherwise the tax's name — except for a **withholding tax**, whose label never falls back to the
name.

**Tax line.** In the engine's vocabulary, the representation of an **existing** tax journal item
supplied so that the engine can decide which items to keep, update, delete or create. Not to be
confused with a **tax journal item**, which is the record itself.

**Tax lock date.** The company-level date at or before which no posted entry affecting the tax
report may be changed. The check consults the hard lock date, so a lock exception does not help.

**Tax report.** A report definition whose expressions use the **tax tags** engine. The evaluation
of those expressions at rendering time is not performed by the packages covered here.

**Tax result.** One entry of the **tax details**: a tax, its base amount, its tax amount, whether it
was treated as price-included, whether it is the **reverse charge** half, its batch, its group and
its downstream taxes.

**Tax scope.** An optional restriction of a **tax** to goods or to services.

**Tax settlement flag.** The flag *used in the tax settlement* on a **distribution line**: computed
as true when the line is of kind *of tax*, has an account, and that account's internal group is
neither income nor expense. It also governs whether the tax journal item keeps the line's analytic
distribution.

**Tax tags engine.** The report expression kind whose formula names a **tax grid**. Creating such
an expression creates the tag; deleting the last one archives or deletes it.

**Tax type.** The selection on a **tax** saying where it can be picked: *sales*, *purchases* or
*none*. A tax of type *none* can only be used inside a **group of taxes**.

**Totals block.** The structured value a document exposes so that a screen or a printed page can
show the untaxed amount, one row per **tax group**, optional intermediate subtotals, the **cash
rounding** delta and the grand total, in both currencies.

**Transition account.** The account on which a **deferred tax** sits between posting and payment.
It must allow reconciliation.

**Union prefix.** The two-letter prefix of a **tax identification number** identifying the issuing
member country. Two prefixes differ from the country code they denote: the one used for Greece and
the one used for Northern Ireland.

**Untaxed total.** The amount of a **base line** before taxes, taken as the **base amount** of the
first tax result in evaluation order, or the raw base when there is no tax.

**Withholding line.** A line on a payment or on the register-payment wizard declaring that part of
the payment is retained as tax: a withholding **tax**, a base amount, an amount, an account, an
analytic distribution and a certificate number.

**Withholding sequence.** The optional numbering attached to a **withholding tax**, used to draw
certificate numbers. A value is drawn only when the payment entry is built and only after every
line has been checked.

**Withholding tax.** A **tax** with a negative amount and the flag *withhold on payment*. It is
filtered out of every ordinary document computation and applied when a payment is registered.

---

## Terms deliberately avoided

The following words are used loosely in accounting practice and are **not** used in this folder,
because each of them means at least two different things:

| Word | What it could mean here | What this folder says instead |
|---|---|---|
| "repartition" | the split of a tax over accounts | **tax distribution** |
| "grid" alone | a tag, a report line, or a column | **tax grid** for the tag, *report line* for the line |
| "base" alone | the taxable amount, the untaxed total, or the base journal item | **base amount**, **untaxed total**, **base journal item** |
| "tax line" | the record, the engine's input, or the distribution line | **tax journal item**, **tax line**, **tax distribution line** |
| "included" | price-included, or included in a total | **price-included** |
| "rounding" | the currency step, the method, or the cash rounding feature | *rounding step*, **rounding method**, **cash rounding** |
| "exigible" alone | due, or already posted to the tax account | **exigibility** with its two values |

---

## How to read the formulas in this folder

Formulas are written in fenced blocks labelled `formula`. They use only the symbols
× ÷ + − = and parentheses, plus the named operations below. They are not code and are not tied to
any language.

| Notation | Meaning |
|---|---|
| `round_to_currency( v )` | `round_to_step( v , the currency's rounding step , half away from zero )` |
| `round_to_document_currency( v )` | the same with the document currency's step |
| `round_to_company_currency( v )` | the same with the company currency's step |
| `round_to_step( v , s , method )` | the primitive of `calculations.md` section 2.1 |
| `a mod b` | the remainder of a Euclidean division whose result carries the sign of the divisor, so that minus three modulo eleven is eight |
| `floor( v )` | the largest integer not greater than *v* |
| `truncate( v )` | the integer part of *v*, discarding the fraction, towards zero |
| `sign( v )` | minus one when *v* is negative, plus one otherwise |
| `\| v \|` | the absolute value of *v* |
| `weighted_sum( number , weights )` | the sum of each weight times the digit it is paired with, paired from the left unless the text says otherwise |
| `d1 … dn` | the characters of a number read left to right, numbered from one |
| `batch_percentage( B )` | the sum of the amounts of the taxes in the batch *B*, divided by one hundred |
| `doubling_checksum( s )` | the primitive of `calculations.md` section 15.2.2 |
| `alphabet[ i ]` | the character at position *i* of the stated alphabet, counted from zero |

A step written as *"round per line"* or *"round per tax"* always refers to the company's rounding
method; a step written as *"half away from zero"* or *"half to even"* always refers to the
tie-breaking rule of the rounding primitive. The two are independent.

---

## Additional terms

**Batch percentage.** The sum of the amounts of every tax in a **batch**, divided by one hundred.
It is the number a price-included percentage extraction divides by, and the number a price-excluded
division tax divides by after being subtracted from one.

**Combined all-in-one computation.** The single operation that prepares a base line, computes it,
derives its accounting data and returns everything at once, for a caller that has only one line
and no document. It reports the raw per-result base rather than the rounded one, and it adds a
"total posted to no account" figure.

**Effective price inclusion.** The price-inclusion flag the engine actually uses for a tax: false
for a **reverse charge**, true under the mode *total included*, false under the mode *total
excluded*, and the tax's own flag otherwise. Distinct from the tax's configured flag, which is
what the propagation table of `calculations.md` section 5.1 consults.

**Fallback mapping.** The approximate base-to-tax pairing used when the exact one finds nothing,
described in `calculations.md` section 16.4.

**Hard lock date.** The lock date that lock exceptions do not relax. The tax lock check consults
it, so a user holding an exception is still refused.

**Mirrored step.** A step of the engine that exists in both implementations and must produce
identical numbers. Listed in `interfaces.md` section 9.1.

**Running-total allocation.** The technique of allocating a total over an ordered list by rounding
the cumulative allocation and taking differences, so that the parts always add back to the total.
Used by the base-to-tax reconstruction of `calculations.md` section 16.

**Source item.** In the base-to-tax reconstruction, the journal item a contribution came from:
either the base item itself, or the tax item of a base-affecting tax.

**Tail test.** The condition, in the base-to-tax reconstruction, that a base item's
affecting-tax list must end with the tax item's originator tax followed by the tax item's own
affecting-tax list.

**Target amount.** See the entry in the main list; note that under a **manual amount** the target
is the manual value, which is what pins the document total.

**Withheld amount.** The positive amount a **withholding line** retains, obtained by negating the
engine's negative tax amount.
