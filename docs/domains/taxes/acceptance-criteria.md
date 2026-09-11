# Taxes — Acceptance criteria

Numbered Given / When / Then scenarios with concrete numbers. An implementation that satisfies all
of them behaves identically to the specified system for the tax domain.

Unless a scenario says otherwise, assume: one company whose currency is the document currency and
whose currency has two decimal places; the rounding method "round per tax"; the company default
price inclusion "tax excluded"; a rate of one; and no fiscal position.

---

## A. Single-line computation

**A1 — Percentage excluded.**
Given a tax of twenty-one percent, price-excluded, and a line of one unit at one hundred.
When the taxes are computed.
Then the untaxed total is one hundred point zero zero, the tax amount is twenty-one point zero
zero, the total is one hundred twenty-one point zero zero, and the tax's reported base is one
hundred point zero zero.

**A2 — Percentage included.**
Given a tax of twenty-one percent, price-included, and a line of one unit at one hundred
twenty-one.
When the taxes are computed.
Then the untaxed total is one hundred point zero zero, the tax amount is twenty-one point zero
zero and the total is one hundred twenty-one point zero zero.

**A3 — Percentage included with a residue.**
Given the same tax and a line of one unit at twenty-one point five three.
When the taxes are computed.
Then the raw untaxed total is seventeen point seven nine three three eight eight four two nine, the
raw tax is three point seven three six six one one five seven, the rounded untaxed total is
seventeen point seven nine, the rounded tax is three point seven four, and seventeen point seven
nine plus three point seven four is exactly twenty-one point five three.

**A4 — Fixed tax with a quantity.**
Given a fixed tax of five hundredths per unit at sequence one, a tax of twenty percent at sequence
two, both price-excluded, and a line of seven units at fifteen.
When the taxes are computed.
Then the untaxed total is one hundred five point zero zero, the fixed tax is zero point three five,
the percentage tax is twenty-one point zero zero on a base of one hundred five point zero zero, and
the total is one hundred twenty-six point three five.

**A5 — Fixed tax affecting the base.**
Given the same two taxes but with the fixed tax flagged "affect base of subsequent taxes" and the
percentage tax flagged "base affected by previous taxes".
When the taxes are computed on seven units at fifteen.
Then the fixed tax is zero point three five on a base of one hundred five, the percentage tax is
twenty-one point zero seven on a base of one hundred five point three five, and the total is one
hundred twenty-six point four two.

**A6 — Division tax, price-excluded.**
Given a division tax of ten percent, price-excluded, and a line of one unit at one hundred eighty.
When the taxes are computed.
Then the untaxed total is one hundred eighty point zero zero, the tax is twenty point zero zero and
the total is two hundred point zero zero.

**A7 — Division tax, price-included.**
Given a division tax of ten percent, price-included, and a line of one unit at two hundred.
When the taxes are computed.
Then the untaxed total is one hundred eighty point zero zero, the tax is twenty point zero zero and
the total is two hundred point zero zero.

**A8 — Group of taxes.**
Given a Group of Taxes at sequence one whose children are a ten percent tax at sequence one and a
five percent tax at sequence two, all price-excluded, and a line of one unit at one hundred.
When the taxes are computed.
Then two results are produced, of ten point zero zero and five point zero zero, each with a base of
one hundred point zero zero; the total is one hundred fifteen point zero zero; and each result
records the group as its originator group.

**A9 — Chain with base-amount inclusion.**
Given a tax *A* of ten percent at sequence one, price-excluded, flagged "affect base of subsequent
taxes"; a tax *B* of five percent at sequence two, price-excluded, flagged "base affected by
previous taxes"; and a line of one unit at one hundred.
When the taxes are computed.
Then *A* gives ten point zero zero on a base of one hundred point zero zero, *B* gives five point
five zero on a base of one hundred ten point zero zero, the untaxed total is one hundred point zero
zero and the total is one hundred fifteen point five zero.

**A10 — The same chain with the first tax price-included.**
Given the same configuration but with *A* price-included, and a line of one unit at one hundred
ten.
Then the results are identical to A9.

**A11 — Two price-included taxes in one batch.**
Given two price-included taxes of ten percent each, neither affecting the base, and a line of one
unit at one hundred.
When the taxes are computed.
Then each tax amount is eight point three three, the untaxed total after the document-wide rounding
is eighty-three point three four and the total is exactly one hundred point zero zero.

**A12 — A fixed price-included tax.**
Given a fixed tax of five per unit, price-included, and a line of one unit at one hundred five.
Then the untaxed total is one hundred point zero zero and the tax is five point zero zero.

**A13 — A fixed price-included tax followed by a percentage price-included tax.**
Given a fixed tax of five per unit at sequence one, price-included, and a tax of twenty-one percent
at sequence two, price-included, and a line of one unit at one hundred twenty-six.
Then the untaxed total is one hundred point zero zero, the fixed tax is five point zero zero, the
percentage tax is twenty-one point zero zero and the total is one hundred twenty-six point zero
zero.

**A14 — A discount.**
Given a tax of twenty-one percent, price-excluded, and a line of three units at one hundred with a
discount of ten percent.
Then the untaxed total is two hundred seventy point zero zero and the tax is fifty-six point seven
zero.

**A15 — A discount on a price-included tax.**
Given a tax of twenty-one percent, price-included, and a line of three units at one hundred
twenty-one with a discount of ten percent.
Then the untaxed total is two hundred seventy point zero zero and the tax is fifty-six point seven
zero — identical to A14.

**A16 — A negative quantity.**
Given a tax of twenty-one percent, price-excluded, and a line of minus two units at one hundred.
Then the untaxed total is minus two hundred point zero zero and the tax is minus forty-two point
zero zero.

**A17 — A negative unit price with a fixed tax.**
Given a fixed tax of five per unit and a line of two units at minus ten.
Then the fixed tax amount is minus ten point zero zero, because the sign of the unit price is
applied.

**A18 — A quantity of zero.**
Given any tax and a line of zero units at one hundred.
Then the untaxed total, every tax amount and the total are all zero.

**A19 — No tax at all.**
Given a line of one unit at one hundred with no tax.
Then the untaxed total and the total both equal one hundred point zero zero and no tax result is
produced.

**A20 — A division tax affecting the base of a percentage tax.**
Given a division tax of ten percent at sequence one, price-excluded, flagged "affect base of
subsequent taxes"; a percentage tax of five percent at sequence two; and a line of one unit at one
hundred eighty.
Then the division tax gives twenty point zero zero on a base of one hundred eighty, the percentage
tax gives ten point zero zero on a base of two hundred, and the total is two hundred ten point zero
zero.

**A21 — A group mixing a price-excluded and a price-included child.**
Given a Group of Taxes whose children are a ten percent price-excluded tax at sequence one and a
five percent price-included tax at sequence two, and a line of one unit at one hundred five.
Then the untaxed total is one hundred point zero zero, the two tax amounts are ten point zero zero
and five point zero zero, and the total is one hundred fifteen point zero zero.

---

## B. Special modes

**B1 — Forcing tax-excluded.**
Given a tax of twenty-one percent, price-included, and a line of one unit at one hundred with the
special mode "total excluded".
Then the untaxed total is one hundred point zero zero, the tax is twenty-one point zero zero and
the total is one hundred twenty-one point zero zero — the same as supplying one hundred twenty-one
with no mode.

**B2 — Forcing tax-included.**
Given a tax of twenty-one percent, price-excluded, and a line of one unit at one hundred twenty-one
with the special mode "total included".
Then the untaxed total is one hundred point zero zero and the tax is twenty-one point zero zero —
the same as supplying one hundred with no mode.

**B3 — Batching ignores the price-inclusion flag under a special mode.**
Given a price-included ten percent tax and a price-excluded ten percent tax, neither affecting the
base, evaluated under the special mode "total included".
Then the two taxes form a **single** batch, because the batching test skips the price-inclusion
comparison whenever a special mode is active.

**B4 — Symmetry is not guaranteed with round-per-line.**
Given the rounding method "round per line" and a price whose extraction leaves a residue.
Then computing forwards and then backwards may differ by one unit of the last decimal place; only
"round per tax" with an unrounded input guarantees symmetry.

---

## C. Document-wide rounding

**C1 — Round per line on three identical lines.**
Given the rounding method "round per line", a tax of twenty-three percent price-excluded, and three
lines each of quantity twelve point one two at a unit price of twelve point one two.
Then each line reports an untaxed total of one hundred forty-six point eight nine and a tax of
thirty-three point seven eight; the document totals are four hundred forty point six seven untaxed,
one hundred one point three four of tax and five hundred forty-two point zero one in all.

**C2 — Round per tax on the same three lines.**
Given the same lines with the rounding method "round per tax".
Then the first line's balance is one hundred forty-six point nine zero and its tax is thirty-three
point seven eight; the second and third lines' balances are one hundred forty-six point eight nine
and their taxes thirty-three point seven nine; the document totals are four hundred forty point six
eight untaxed, one hundred one point three six of tax and five hundred forty-two point zero four in
all.

**C3 — The difference.**
Given C1 and C2.
Then the two methods differ by one hundredth on the untaxed amount, two hundredths on the tax and
three hundredths on the grand total.

**C4 — Round per line on three price-included lines.**
Given the rounding method "round per line", a tax of twenty-one percent price-included, and three
lines of one unit at twenty-one point five three.
Then each line reports seventeen point seven nine untaxed and three point seven four of tax, and
the document totals are fifty-three point three seven untaxed, eleven point two two of tax and
sixty-four point five nine in all.

**C5 — Round per tax on the same lines.**
Given the same lines with "round per tax".
Then the first line's balance is seventeen point eight zero and its tax three point seven three;
the other two lines' balances are seventeen point seven nine and their taxes three point seven four;
the document totals are fifty-three point three eight untaxed, eleven point two one of tax and
sixty-four point five nine in all.

**C6 — The delta lands on the first line.**
Given C5.
Then the delta of plus one hundredth is allocated to the **first** line, because the weights are
equal and the smooth distribution walks them in their original order after a stable descending
sort.

**C7 — A currency with no decimal places.**
Given a currency whose rounding step is one, a tax of ten percent price-excluded, and three lines
of one unit at one hundred five, with "round per tax".
Then each raw tax is ten point five; the target total tax is thirty-one point five, rounded to
thirty-two; the sum of the independently rounded amounts is thirty-three; the delta of minus one is
allocated to the first line, whose tax becomes ten; the other two taxes are eleven each; the
document tax is thirty-two.

**C8 — Multi-currency, deltas may land on different lines.**
Given a foreign currency, a company currency, a rate of one point two five foreign units per
company unit, a tax of twenty percent price-excluded, three lines of one unit at twenty-one point
five three, and "round per tax".
Then in the foreign currency the first line's tax is four point three zero and the others four
point three one, with no untaxed delta; in the company currency the first line's tax is three point
four five, the others three point four four, and the first line carries an untaxed delta of plus
one hundredth; the document totals are sixty-four point five nine and twelve point nine two in the
foreign currency, fifty-one point six seven and ten point three three in the company currency.

**C9 — Smooth distribution proportional to weights.**
Given a delta of three hundredths, a precision of three decimal places and three weights of zero
point four, zero point three and zero point three.
Then the allocation is twelve thousandths, nine thousandths and nine thousandths.

**C10 — Smooth distribution leftover.**
Given a delta of one hundredth, a precision of two decimal places and two equal weights.
Then the whole hundredth goes to the first weight in the sorted order and the second receives
nothing.

**C11 — Zero delta.**
Given a delta that is zero at the stated precision.
Then every allocation is zero and no weight is touched.

**C12 — Weights that sum to zero.**
Given weights that are all zero.
Then each normalised weight is one divided by the number of weights.

---

## D. Rounding primitive

**D1 — The tie is broken away from zero.**
Given the value two point six seven five and two decimal places.
Then the result is two point six eight, not two point six seven, because an epsilon scaled to the
value's magnitude lifts it above the tie.

**D2 — Negative ties.**
Given minus two point six seven five and two decimal places.
Then the result is minus two point six eight.

**D3 — Rounding to a step that is not a power of ten.**
Given one point three and a step of one half.
Then the result is one point five.

**D4 — The zero test rounds after subtracting.**
Given six thousandths and two thousandths at two decimal places.
Then the zero test on their difference reports zero, while the comparison of the two values also
reports equal; but the zero test on zero point zero zero six alone reports zero whereas the
comparison of zero point zero zero six against zero reports equal as well. The two operations
differ in general: comparison rounds each operand first, the zero test rounds the difference.

**D5 — A value of exactly zero.**
Given zero and any step.
Then the result is zero.

---

## E. Distribution and accounting

**E1 — One tax, one distribution line.**
Given a customer invoice with one line of one thousand and a sales tax of twenty-one percent whose
invoice distribution is one base line and one tax line of one hundred percent on the tax payable
account.
When the invoice is posted.
Then three journal items exist: revenue credited one thousand carrying the base tag, tax payable
credited two hundred ten carrying the tax tag with a stored base amount of minus one thousand, and
receivable debited one thousand two hundred ten.

**E2 — A tax split over two accounts.**
Given a vendor bill of one thousand with a purchase tax of twenty percent whose invoice
distribution is a base line, a tax line of fifty percent on the deductible tax account and a tax
line of fifty percent on an expense account.
Then two tax items exist, each of one hundred, each with a stored base amount of one thousand.

**E3 — The residue of a split.**
Given a tax amount of one and a distribution of thirty-three, thirty-three and thirty-four percent.
Then the three shares are zero point three three, zero point three three and zero point three four,
and their sum is exactly the tax amount. When the independently rounded shares do not add up, the
difference is given to the share with the largest absolute amount first.

**E4 — Reverse charge.**
Given a vendor bill of one thousand with a purchase tax of twenty-one percent whose invoice
distribution is a base line, a tax line of plus one hundred percent on the tax payable account
tagged *tax due*, and a tax line of minus one hundred percent on the tax deductible account tagged
*tax deductible*.
Then four journal items exist: expense debited one thousand with the base tag, tax payable credited
two hundred ten with *tax due*, tax deductible debited two hundred ten with *tax deductible*, and
payable credited one thousand. The net tax effect on the ledger is zero.

**E5 — A reverse-charge tax is never price-included.**
Given a reverse-charge tax whose price-inclusion override says "tax included", and a line of one
thousand.
Then the tax is computed as price-excluded: the base is one thousand and the two halves are plus
and minus two hundred ten.

**E6 — A distribution line with no account.**
Given a tax whose only tax distribution line has no account, on a line whose own account is
"revenue".
Then the tax journal item is posted on "revenue".

**E7 — A tax exigible on payment.**
Given a sales tax of twenty-one percent exigible on payment with a transition account "tax to
receive", and an invoice of one thousand.
When the invoice is posted.
Then the tax item sits on "tax to receive" and carries **no** report tag, and the base item carries
**no** base tag either.

**E8 — Analytic distribution on a tax item.**
Given a line with an analytic distribution and a tax that is **not** flagged analytic, whose
distribution line **is** used in the tax settlement.
Then the tax item carries **no** analytic distribution.

**E9 — Analytic distribution kept.**
Given the same line and a tax that **is** flagged analytic.
Then the tax item carries the line's analytic distribution.

**E10 — Analytic distribution kept for a non-settlement line.**
Given a line with an analytic distribution and a tax not flagged analytic, whose distribution line
is **not** used in the tax settlement (for example because it posts to an expense account).
Then the tax item carries the line's analytic distribution.

**E11 — Two lines merging into one tax item.**
Given two invoice lines with the same partner, currency, analytic distribution, account and tax.
Then one tax journal item is produced, whose amount is the sum and whose stored base amount is the
sum of the two bases.

**E12 — Two lines not merging.**
Given the same two lines but with different accounts.
Then the tax items still merge, because the tax distribution line has its own account; they only
fail to merge when the distribution line has **no** account, in which case the base line's account
enters the grouping key.

**E13 — A zero tax item is dropped.**
Given a tax of zero percent.
Then no tax journal item is produced, because the amount is zero in both currencies and the
grouping key carries no "keep zero line" marker.

**E14 — A group names its children's items.**
Given a Group of Taxes with two children on one invoice line.
Then two tax items exist, each recording the group in its originator-group field, while the base
item's tax set contains the **group**.

---

## F. Refunds and tag signs

**F1 — The refund distribution is used.**
Given a sales tax whose invoice distribution tags the base with *base 21* and the tax with *tax 21*,
and whose refund distribution tags them with the negated forms, and a customer credit note of one
thousand.
Then the base item is debited one thousand and carries the refund base tag, and the tax item is
debited two hundred ten and carries the refund tax tag.

**F2 — An invoice and its credit note net to zero on the return.**
Given F1 and the corresponding invoice.
Then the sales base grid receives plus one thousand from the invoice and minus one thousand from
the credit note, and the period's net base is zero.

**F3 — Both distributions must match structurally.**
Given a tax whose invoice distribution has two tax lines of fifty percent each and whose refund
distribution has one tax line of one hundred percent.
When the tax is saved.
Then it is refused with *"Invoice and credit note distribution should have the same number of lines."*

**F4 — Only the accounts and the tags may differ.**
Given a tax whose invoice and refund distributions have the same kinds and percentages in the same
order but different accounts and tags.
Then the tax saves successfully.

**F5 — The refund flag on a miscellaneous entry, sales tax, credit side.**
Given a miscellaneous entry line carrying a sales tax, with a credit amount.
Then the refund flag is **false** and the invoice distribution is used.

**F6 — The refund flag on a miscellaneous entry, sales tax, debit side.**
Given a miscellaneous entry line carrying a sales tax, with a debit amount and no credit.
Then the refund flag is **true** and the refund distribution is used.

**F7 — The refund flag on a miscellaneous entry, purchase tax, debit side.**
Given a miscellaneous entry line carrying a purchase tax, with a debit amount.
Then the refund flag is **false**.

**F8 — A line carrying both a sales and a purchase tax.**
Given a miscellaneous entry line whose taxes include both a sales tax and a purchase tax.
Then the refund flag is true when the line has no credit.

**F9 — A reversal inverts the derived flag.**
Given a miscellaneous entry that is the reversal of another entry, whose line carries taxes.
Then the flag derived from the sign is inverted.

**F10 — A reversal negates the stored manual amounts.**
Given an invoice line whose tax amount was typed by hand, so that a manual amount is stored.
When the invoice is reversed.
Then the reversal's line carries the same manual amounts with the opposite sign, and the quantity
is negated.

---

## G. Fiscal positions

**G1 — Automatic detection by country group.**
Given a fiscal position with "detect automatically" on, no country, the union country group and
"tax registration required" on; and a partner established in a member country with a valid
registration number.
Then the fiscal position is detected.

**G2 — The registration requirement blocks it.**
Given the same fiscal position and a partner in the same country with **no** registration number.
Then the fiscal position is not detected.

**G3 — A manual choice always wins.**
Given a partner carrying a manually chosen fiscal position and an automatic one that would match.
Then the manual one is returned without any automatic evaluation.

**G4 — The delivery address decides.**
Given a partner whose invoicing address is in one country and whose delivery address is in another,
and two automatic fiscal positions, one per country.
Then the one matching the **delivery** address is detected.

**G5 — Unless both parties share a registration prefix inside the union.**
Given a company and a partner both holding registration numbers whose first two characters are the
same union member code, and the partner's country equal to the company's country.
Then the **invoicing** address is used instead of the delivery address.

**G6 — Branch precedence.**
Given two matching automatic fiscal positions, one defined on the parent company and one on the
branch.
Then the branch's one wins, because candidates are sorted by their company's ancestor count
descending before their sequence.

**G7 — Sequence precedence.**
Given two matching automatic fiscal positions of the same company with sequences ten and twenty.
Then the one with sequence ten wins.

**G8 — Postal code range.**
Given a fiscal position written with a range from one hundred to nine thousand and a partner whose
postal code is five hundred seventy-five.
Then the bounds are stored as `0100` and `9000`, and the comparison is textual: `0100` is not
greater than `575` and `575` is not greater than `9000`, so the fiscal position matches.

**G8b — Postal code range, the padding matters.**
Given a fiscal position whose bounds are written as `100` and `900` — both three characters, so
the padding changes nothing — and a partner whose postal code is `1050`.
Then the textual comparison gives `100` not greater than `1050` and `1050` not greater than `900`
(because the first character `1` is less than `9`), so the fiscal position **matches** even though
one thousand fifty is numerically outside the range. The partner's own postal code is never padded;
this text-ordering behaviour is the one to reproduce.

**G9 — Tax substitution.**
Given a fiscal position carrying a replacement tax *R* that declares it replaces the domestic tax
*D*, and a line whose taxes are `[D]`.
Then the line's taxes become `[R]`.

**G10 — One tax mapping to two.**
Given two replacement taxes both declaring they replace *D*.
Then a line whose taxes are `[D]` becomes a line whose taxes are both replacements, in the order
the fiscal position lists them.

**G11 — A fiscal position with no tax at all.**
Given a fiscal position whose tax set is empty, and a line carrying a tax that belongs to some
fiscal position.
Then the mapping **removes** that tax: only taxes belonging to no fiscal position survive.

**G12 — Price adaptation when the original tax is price-included.**
Given a product priced one hundred twenty-one with a price-included tax of twenty-one percent, and
a fiscal position replacing it by a price-excluded tax of twenty-one percent.
Then the adapted unit price is one hundred point zero zero.

**G13 — No price adaptation when the original tax is price-excluded.**
Given a product priced one hundred with a price-excluded tax of fifteen percent, and a fiscal
position replacing it by a price-included tax of six percent.
Then the unit price stays one hundred, and the computation gives an untaxed total of ninety-four
point three four and a tax of five point six six.

**G14 — Account mapping.**
Given a fiscal position mapping the domestic revenue account to the export revenue account, and a
line whose account is the domestic revenue account.
Then the line's account becomes the export revenue account.

**G15 — The complete substitution.**
Given the configuration of `calculations.md` section 11.5 and a product priced one hundred
twenty-one.
Then the invoice line carries the export revenue account, the replacement tax and a unit price of
one hundred; the journal items are: export revenue credited one hundred with the intra-union base
tag, tax payable credited twenty-one with the tax-due tag, tax deductible debited twenty-one with
the deductible tag, and receivable debited one hundred.

**G16 — The tax country constraint.**
Given a document whose fiscal position has no foreign registration number and a company whose
fiscal country is one country, and a line carrying a tax of another country.
When the document is saved.
Then it is refused with *"This entry contains one or more taxes that are incompatible with your fiscal country. Check company fiscal country in the settings and tax country in taxes configuration."*

**G17 — The same with a fiscal position.**
Given a document whose fiscal position has a country different from the taxes' country.
Then it is refused with *"This entry contains taxes that are not compatible with your fiscal position. Check the country set in fiscal position and in your tax configuration."*

**G18 — A foreign registration changes the document's tax country.**
Given a fiscal position with a foreign registration number for a second country.
Then a document carrying that fiscal position has that second country as its tax country, and
taxes of that country become valid on it.

---

## H. Cash basis

**H1 — A payment of forty percent.**
Given a customer invoice of one thousand plus a twenty-one percent tax exigible on payment, total
one thousand two hundred ten, and a payment of four hundred eighty-four reconciled against it.
Then the paid percentage is zero point four zero, and one cash basis entry is created in the cash
basis journal dated on the payment date, containing four items: the base tax received account
debited four hundred with no tag, the same account credited four hundred with the base tag, the
transition account debited eighty-four with no tag, and the tax payable account credited eighty-four
with the tax tag.

**H2 — The transition account is reconciled.**
Given H1 and a reconcilable transition account.
Then the newly created debit of eighty-four on the transition account is reconciled against the
invoice's own tax item of two hundred ten, leaving one hundred twenty-six unreconciled.

**H3 — The settling payment.**
Given H1 followed by a payment of seven hundred twenty-six.
Then the second cash basis entry carries a base share of six hundred and a tax share forced to the
tax item's remaining residual, so that the two entries together make exactly two hundred ten
exigible; the invoice's tax item becomes fully reconciled.

**H4 — Three uneven instalments.**
Given the same invoice paid in three instalments of four hundred three point three three, four
hundred three point three three and four hundred three point three four.
Then the tax shares computed by percentage would not add up to two hundred ten; the last-partial
correction replaces the third share by the remaining residual so that the three shares sum to
exactly two hundred ten.

**H5 — No cash basis journal.**
Given a company with no cash basis journal and a reconciliation that would need one.
Then the operation is refused with *"There is no tax cash basis journal defined for the '&lt;company name&gt;' company.\nConfigure it in Accounting/Configuration/Settings"*.

**H6 — Mixed currencies.**
Given a document whose deferred lines and whose term lines are in different currencies.
Then no cash basis entry is produced at all.

**H7 — No term line.**
Given a miscellaneous entry with a deferred tax but no receivable or payable item.
Then no cash basis entry is produced.

**H8 — A draft counterpart.**
Given a reconciliation in which one of the two documents is still a draft.
Then the cash basis entry is created **in draft** and a snapshot of what it should contain is
stored on the partial reconciliation.

**H9 — Undoing the reconciliation.**
Given H1 and then an unreconciliation.
Then the posted cash basis entry is reversed and cancelled, dated on its own date; a draft one
would have been deleted instead.

**H10 — A locked period on reversal.**
Given H9 where the cash basis entry's own date falls inside a locked period and the entry affects
the tax report.
Then the reversal is dated on the day after the last violated lock date.

**H11 — Resetting a cash basis entry to draft.**
Given a posted cash basis entry.
When a user tries to reset it to draft.
Then it is refused with *"You cannot reset to draft a tax cash basis journal entry."*

**H12 — Mixing exigibilities on one line.**
Given a journal item carrying a tax exigible on payment and a tax exigible on invoice that share a
base tag.
When the item is saved.
Then it is refused with *"Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag."*

**H13 — Turning the company switch off.**
Given a company with at least one tax exigible on payment.
When the cash basis switch is cleared in the settings.
Then it is forced back on and the message
*"You cannot disable this setting because some of your taxes are cash basis. Modify your taxes first before disabling this setting."* is shown.

---

## I. Withholding

**I1 — A withholding tax is invisible on the document.**
Given a product carrying a sales tax of fifteen percent and a withholding tax of minus one percent,
and an invoice line of one thousand.
Then the invoice total is one thousand one hundred fifty; the withholding tax contributes nothing.

**I2 — Registering the payment.**
Given I1, an invoice of one thousand one hundred fifty and a withholding line with the withholding
tax and a base of one thousand.
Then the withholding amount is ten point zero zero and the net amount is one thousand one hundred
forty.

**I3 — The payment entry.**
Given I2 and a confirmation.
Then the payment's journal entry contains: outstanding receipts debited one thousand one hundred
forty, receivable credited one thousand one hundred fifty, the withholding tax account debited ten,
the withholding base account debited one thousand with no tax and no tag, and the withholding base
account credited one thousand carrying the withholding tax and its base tags.

**I4 — An instalment.**
Given the same invoice paid in two instalments of five hundred seventy-five, with the withholding
line derived from the documents.
Then the paid factor is one half, the withholding base is five hundred, the withholding amount is
five and the net amount is five hundred seventy.

**I5 — A negative net amount.**
Given a payment of five and a withholding amount of ten.
When the payment is confirmed.
Then it is refused with *"The withholding net amount cannot be negative."*

**I6 — A line with no number and no sequence.**
Given a withholding line whose tax has no sequence and whose number is empty.
When the payment entry is built.
Then it is refused with *"Please enter the withholding number for the tax &lt;the tax's name&gt;"* and
**no** sequence value is consumed for any line.

**I7 — Numbering.**
Given two withholding lines sharing a tax whose sequence has a padding of four and whose next value
is seven.
Then the hints shown are `0007` and `0008`, and the numbers actually written when the entry is
built are `0007` and `0008`.

**I8 — A base of zero.**
Given a withholding line whose base amount is zero.
Then it is refused with *"The base amount of a withholding tax line must be above 0."*

**I9 — A liquidity account on a withholding line.**
Given a withholding line whose account is the journal's default account.
Then it is refused with *"The account "&lt;the account's display name&gt;" is not valid to use on withholding lines."*

**I10 — A group or a division withholding tax.**
Given a tax flagged "withhold on payment" whose computation kind is "group of taxes".
Then saving is refused with
*"Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations."*

**I11 — Switching the flag on.**
Given a tax whose exigibility is "based on payment" and whose price-inclusion override is "tax
included".
When "withhold on payment" is switched on in the form.
Then the exigibility becomes "based on invoice" and the override becomes "tax excluded".

**I12 — A positive amount clears the flag.**
Given a withholding tax whose amount is changed to plus one.
Then the "withhold on payment" flag is cleared.

**I13 — The feature is hidden.**
Given a company owning no withholding tax matching the payment direction.
Then the withholding section is not shown at all.

**I14 — The wizard would split.**
Given a register-payment wizard that will create one payment per document.
Then the withholding section is not shown.

**I15 — A refund inverts the direction.**
Given a batch of refunds in the register-payment wizard.
Then the direction used to select the matching withholding taxes is inverted, so an outgoing
payment offers sales-kind withholding taxes.

**I16 — The refund distribution on a withholding line.**
Given a sales-kind withholding tax on an **outgoing** payment.
Then the line is treated as a refund and the tax's refund distribution is used.

---

## J. Tax identification numbers

**J1 — A valid Belgian number.**
Given the country Belgium and the number `BE0477472701`.
Then the prefix is recognised, the number is normalised to ten digits and the check
`(04774727 + 01) mod 97 = 0` succeeds.

**J2 — An invalid check digit.**
Given the country Belgium and the number `BE0477472702`.
Then the check fails and the message is
*"The VAT number [BE0477472702] for partner [&lt;the partner's name&gt;] does not seem to be valid. \nNote: the expected format is BE0477472701"*.

**J3 — The explicit "no number" marker.**
Given the number consisting of a single solidus.
Then it is accepted unchanged and no country is reported as validated.

**J4 — Any other single character.**
Given the number `X` and the error mode.
Then it is refused with *"To explicitly indicate no (valid) VAT, use '/' instead. "*.

**J5 — A doubled prefix.**
Given the country Belgium and the number `BEBE0477472701`.
Then it is refused even though stripping one prefix would validate.

**J6 — The Greek prefix.**
Given the country Greece and the number `EL123456783`.
Then the prefix maps to the Greek country code for the check, and the returned number carries the
union prefix for Greece.

**J7 — The Northern Ireland prefix.**
Given the number `XI123456782`.
Then the United Kingdom rules are applied and the prefix is retained.

**J8 — A number outside the union carrying the union prefix.**
Given a country outside the union and a number starting with the two letters of the union.
Then the number is returned unchanged and no check is run.

**J9 — Switzerland.**
Given the number `CHE-123.456.788 TVA`.
Then the nine digits are extracted, the weighted sum with the weights five, four, three, two,
seven, six, five, four is taken, the check digit is `(11 − sum mod 11) mod 11` and it must equal
the ninth digit.

**J10 — Switzerland with the wrong suffix.**
Given the same number with the three-letter English abbreviation of the tax as the suffix.
Then it is refused.

**J11 — Norway.**
Given the number `NO123456785` and the weights three, two, seven, six, five, four, three, two.
Then the check is `11 − (sum mod 11)`, mapped to zero when it is eleven, and a value of ten makes
the number invalid.

**J12 — Peru.**
Given an eleven-digit number and the weights five, four, three, two, seven, six, five, four, three,
two.
Then the check is `11 − (sum mod 11)`, with ten mapped to zero and eleven mapped to one.

**J13 — Venezuela.**
Given the number `V-12345678-1`.
Then the kind digit for the letter V is one, the checksum is `1 × 4` plus the weighted sum with
three, two, seven, six, five, four, three, two, the check digit is `11 − (checksum mod 11)` mapped
to zero when it exceeds nine, and it must equal the last digit.

**J14 — Taiwan with a seven in the seventh position.**
Given an eight-digit number whose seventh digit is seven.
Then two sums are computed — one counting the seventh product as one and one counting it as zero —
and the number is valid when either is divisible by five.

**J15 — Taiwan otherwise.**
Given an eight-digit number whose seventh digit is not seven.
Then the number is valid when the digit sum of the eight products is divisible by five.

**J16 — Uzbekistan depends on the partner kind.**
Given a nine-digit number on a company partner: valid. Given the same number on an individual
partner: invalid, because fourteen digits are required.

**J17 — Germany accepts two kinds.**
Given a nine-digit registration number satisfying the recursive modulus eleven over ten: valid.
Given a regional tax number matching one of the sixteen regional patterns: also valid.

**J18 — Cross-border verification is pending.**
Given a company with the verification switch on and a partner whose number the relay reports as
pending.
Then the flag is false and the message `The VIES check is pending. The status will be updated soon.`
is logged.

**J19 — The callback resolves it.**
Given J18 and a later callback carrying a verifying token and the status *valid*.
Then the flag becomes true and the message
*"The Intra-Community validity has been updated to: valid."* is logged on every partner with that
number.

**J20 — A bad callback token.**
Given a callback whose token does not verify.
Then a warning is logged and nothing changes.

**J21 — Import suppresses the verification.**
Given ten thousand partners created by a file import.
Then no verification request is sent at all.

**J22 — A fiscal position requiring a registration.**
Given a fiscal position with "tax registration required" and a partner whose number is present but
whose cross-border flag is false, in a situation where the verification applies.
Then the fiscal position does not match.

**J23 — A child partner inherits.**
Given a partner whose parent carries the same number.
Then the child copies the parent's flag and no request is sent.

---

## K. Report tags

**K1 — Creating an expression creates a tag.**
Given a report whose country is set, and a report line.
When an expression of the tax-tags kind with the formula `55` is created.
Then an Account Tag named `55`, applicability `taxes`, with that country, is created.

**K2 — A negated formula reuses the same tag.**
Given K1 and a second expression whose formula is `-55` in the same country.
Then no new tag is created; the existing tag `55` is matched and its "negate balance" flag reads
true for that expression.

**K3 — Renaming.**
Given one expression naming the tag `55` and no other.
When its formula is changed to `56`.
Then the tag is renamed to `56` in place and every distribution line keeps pointing at it.

**K4 — Splitting.**
Given two expressions naming the tag `55`.
When only one of them is changed to `56`.
Then a **new** tag `56` is created and the old one is untouched.

**K5 — Deleting with items.**
Given a tag carried by at least one journal item.
When its last expression is deleted.
Then the tag is removed from every distribution line and **archived**.

**K6 — Deleting without items.**
Given a tag carried by no journal item.
When its last expression is deleted.
Then the tag is removed from every distribution line and **deleted**.

**K7 — A shipped cash-flow tag.**
When one of the three shipped cash-flow tags is deleted.
Then it is refused with *"You cannot delete this account tag (&lt;tag name&gt;), it is used on the chart of account definition."*

**K8 — Base tags flow from an affecting tax into the affected tax's item.**
Given a tax *A* flagged "affect base of subsequent taxes" whose base tags are *base A*, and a tax
*B* accepting to be affected whose base tags are *base B*, both on one line.
Then *A*'s tax journal item carries, in addition to its own distribution tags, the base tags of
*B*.

**K9 — Product tags.**
Given a product carrying an account tag whose applicability is `products`.
Then that tag appears on **both** the base journal item and every tax journal item of the line.

**K10 — A deferred tax stamps nothing.**
Given a tax exigible on payment, and a document posted but not reconciled.
Then neither the base item nor the tax item carries any tag from that tax.

---

## L. Structural validations

**L1 — Duplicate name.**
Given an existing sales tax named "21%" with no scope and country *X*.
When a second sales tax with the same name, no scope and country *X* is saved in the same company
tree.
Then it is refused with *"Tax names must be unique!"* followed by one dash line per duplicate.

**L2 — Duplicate name is allowed for the type "none".**
Given two taxes of type `none` with the same name.
Then both save successfully.

**L3 — Tax group country mismatch.**
Given a tax whose country is *X* and a tax group whose country is *Y*.
Then it is refused with *"The tax group must have the same country_id as the tax using it."*

**L4 — A non-reconcilable transition account.**
Given a tax exigible on payment whose transition account does not allow reconciliation.
Then it is refused with *"The cash basis transition account needs to allow reconciliation."*

**L5 — A cycle in a group.**
Given a group whose children eventually include itself.
Then it is refused with *"Recursion found for tax “&lt;the tax's name&gt;”."*

**L6 — A nested group.**
Given a group one of whose children is itself a group.
Then it is refused with *"Nested group of taxes are not allowed."*

**L7 — A child with an incompatible scope.**
Given a group whose scope is "goods" and a child whose scope is "services".
Then it is refused with
*"The application scope of taxes in a group must be either the same as the group or left empty."*

**L8 — Two base lines in a distribution.**
Then it is refused with
*"Invoice and credit note distribution should each contain exactly one line for the base."*

**L9 — No tax line in a distribution.**
Then it is refused with
*"Invoice and credit note repartition should have at least one tax repartition line."*

**L10 — A mismatched percentage between the two distributions.**
Then it is refused with
*"Invoice and credit note distribution should match (same percentages, in the same order)."*

**L11 — Positive factors not adding to one hundred.**
Given a distribution with tax lines of sixty and thirty percent.
Then it is refused with
*"Invoice and credit note distribution should have a total factor (+) equals to 100."*

**L12 — Negative factors not adding to minus one hundred.**
Given a distribution with tax lines of plus one hundred and minus fifty percent.
Then it is refused with
*"Invoice and credit note distribution should have a total factor (-) equals to 100."*

**L13 — A group with no distribution at all.**
Given a Group of Taxes whose invoice and refund distributions are both empty.
Then every structural rule is skipped and the tax saves.

**L14 — Changing the company of a used tax.**
Then it is refused with
*"You can't change the company of your tax since there are some journal items linked to it."*

**L15 — Deleting a used tax.**
Then it is refused with
*"You cannot delete taxes that are currently in use. Consider archiving them instead."*

**L16 — Duplicating a tax.**
Given a tax named "21%".
When it is duplicated.
Then the copy is named "21% (copy)" and its distribution lines, including their tags, are copied.

**L17 — A postal code range with only one bound.**
Then it is refused with
*"Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From"."*

**L18 — A foreign registration with no country.**
Then it is refused with
*"The country of the foreign VAT number could not be detected. Please assign a country to the fiscal position."*

**L19 — A foreign registration inside the fiscal country with no state.**
Given the fiscal country has states.
Then it is refused with
*"You cannot create a fiscal position with a foreign VAT within your fiscal country without assigning it a state."*

**L20 — A country outside the chosen country group.**
Then it is refused with
*"You cannot create a fiscal position with a country outside of the selected country group."*

**L21 — A second foreign registration for the same country.**
Given an existing fiscal position with a foreign registration number for country *X* and a new one
with a **different** number for the same country.
Then it is refused with *"A fiscal position with a foreign VAT already exists in this country."*

**L22 — The same number twice.**
Given the same number for the same country on two fiscal positions.
Then both save, because the rule only forbids a **different** number.

**L23 — A duplicate account mapping.**
Then it is refused with
*"An account fiscal position could be defined only one time on same accounts."*

**L24 — An off-balance account with taxes.**
Then it is refused with *"An Off-Balance account can not have taxes"*.

**L25 — Deductibility on a customer document.**
Given a customer invoice line whose deductibility is ninety percent.
Then it is refused with *"Only vendor bills allow for deductibility of product/services."*

**L26 — Deductibility out of range.**
Given a vendor bill line whose deductibility is one hundred ten.
Then it is refused with *"The deductibility must be a value between 0 and 100."*

---

## M. Locking

**M1 — The tax lock date blocks a change.**
Given a company whose tax lock date is the thirty-first of March, and a posted entry dated the
fifteenth of March that carries a tax.
When any item of that entry is changed.
Then it is refused with *"The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: &lt;the violated lock dates&gt;."*

**M2 — An entry that does not affect the tax report.**
Given the same lock date and a posted entry dated the fifteenth of March whose items carry no tax,
no originator tax and no tax-applicability tag.
Then the change is allowed.

**M3 — A lock exception does not help.**
Given a user holding a lock exception for that period.
Then the tax lock check still refuses, because it consults the hard lock date.

---

## N. Totals block

**N1 — One tax group.**
Given one line of one hundred with a twenty-one percent tax whose group has no preceding subtotal.
Then the block has one subtotal named *"Untaxed Amount"* with a base of one hundred and a tax of
twenty-one; the grand total is one hundred twenty-one; and the "same base" flag is true.

**N2 — Two tax groups with different bases.**
Given two lines, one of one hundred with tax group *A* and one of fifty with tax group *B*.
Then the block has one subtotal with two tax groups, whose displayed bases are one hundred and
fifty; the "same base" flag is false because the set of displayed bases has three members — one
hundred, fifty and the block's untaxed amount of one hundred fifty.

**N3 — A preceding subtotal.**
Given two lines of one hundred each, the first with a tax *X* of ten percent whose group has no
preceding subtotal, the second with a tax *Y* of five percent whose group's preceding subtotal is
"Total excluding surcharge".
Then the block has two subtotals: *"Untaxed Amount"* with a base of two hundred and a tax of ten,
and *"Total excluding surcharge"* with a base of two hundred ten and a tax of five; the grand total
is two hundred fifteen.

**N4 — Fixed taxes show no base.**
Given a tax group all of whose contributing taxes are fixed.
Then the group's displayed base is explicitly absent and the row shows only the group name and the
tax amount.

**N5 — Price-included division taxes show the inclusive base.**
Given a tax group all of whose contributing taxes are price-included division taxes, and one line
of two hundred.
Then the group's displayed base is two hundred — the untaxed total of one hundred eighty plus the
tax of twenty — while its plain base amount is one hundred eighty.

**N6 — Cash rounding, add a rounding line.**
Given one line of nine point nine nine with a twenty-one percent tax, a cash rounding step of five
hundredths and the method "half away from zero", with the "add a rounding line" strategy.
Then the block reports an untaxed amount of nine point nine nine, a tax of two point one zero, a
cash rounding amount of one hundredth and a grand total of twelve point one zero.

**N7 — Cash rounding, adjust the biggest tax.**
Given the same with the "adjust the biggest tax" strategy.
Then the single tax group's tax becomes two point one one, the block reports a tax of two point one
one and a grand total of twelve point one zero, and no cash rounding amount is reported.

**N8 — Cash rounding with no tax at all.**
Given a document with no tax and the "adjust the biggest tax" strategy.
Then the rounding is abandoned and the delta reset to zero.

**N9 — Excluding a tax group from the block.**
Given a block with two tax groups and a request to exclude one.
Then the excluded group's tax amount moves into the base amount of its subtotal and of the block,
its tax amount is removed from both, and the group disappears; a subtotal left with no group is
removed too.

**N10 — No tax at all.**
Given a document with one line of one hundred and no tax.
Then the block has one subtotal named *"Untaxed Amount"* with no tax group, a base of one hundred
and a grand total of one hundred; the "has tax groups" flag is false.

---

## O. Custom formula taxes

**O1 — A simple formula.**
Given a custom-formula tax whose formula multiplies the unit price by one tenth, and a line of one
unit at two hundred.
Then the tax amount is twenty point zero zero.

**O2 — A formula reading the base.**
Given a formula that multiplies the base by five hundredths, on a line of two units at one hundred.
Then the base handed to the formula is two hundred and the tax is ten point zero zero.

**O3 — A formula with a bound.**
Given a formula taking the smaller of the unit price multiplied by one tenth and the constant
fifteen, on a line of one unit at two hundred.
Then the tax is fifteen point zero zero.

**O4 — A forbidden construct.**
Given a formula containing an assignment.
Then saving is refused with *"Invalid AST node: &lt;the construct's name&gt;"*.

**O5 — An unknown identifier.**
Given a formula naming something other than the five permitted names.
Then saving is refused with *"Unknown identifier: &lt;the name&gt;"*.

**O6 — A relational field.**
Given a formula reading a relational field of the product.
Then saving is refused with *"Field '&lt;the field name&gt;' is not accessible"*.

**O7 — A division by zero.**
Given a formula that divides by a product field whose value is zero.
Then the tax amount is zero rather than an error.

**O8 — A custom-formula tax behaves like a fixed tax.**
Given a custom-formula tax and a price-included percentage tax in the same line.
Then the custom-formula tax is computed in the first pass, before the extraction, exactly as a
fixed tax would be.

**O9 — A custom-formula tax cannot be discounted.**
Given a global discount on a document containing a custom-formula tax.
Then that tax is excluded from the proportional reduction, exactly like a fixed tax.

**O10 — Uninstalling the capability.**
When the custom formula capability is uninstalled.
Then every tax whose computation kind was "custom formula" becomes a percentage tax and is
archived.

---

## P. Manual amounts and computation keys

**P1 — A manual tax amount is kept.**
Given an invoice line of one hundred with a twenty-one percent tax whose tax amount the user typed
as twenty point five zero.
When the invoice is saved and reopened.
Then the tax amount is still twenty point five zero.

**P2 — Changing the price discards it.**
Given P1 and a change of the unit price to one hundred ten.
Then the stored manual amount is **not** reloaded, because the stored unit price no longer matches,
and the tax amount is recomputed as twenty-three point one zero.

**P3 — Changing the currency rate rescales it.**
Given P1 in a foreign currency with a rate of one point two five, and a change of the rate to one
point five.
Then the manual amount in document currency is kept and the company-currency amount is recomputed
by dividing by the ratio of the two rates.

**P4 — Changing the set of taxes discards it.**
Given P1 and the addition of a second tax to the line.
Then the stored manual amounts are not reloaded, because the number of flattened taxes no longer
matches.

**P5 — A down payment deduction.**
Given a final invoice whose original lines total one thousand with a twenty-one percent tax and
whose down-payment lines total minus three hundred with the computation key `down_payment`.
Then the two subsets are rounded independently, so the invoice shows exactly seven hundred untaxed
and one hundred forty-seven of tax, and no cent leaks between the subsets.

**P6 — A global discount.**
Given a document of one thousand with a twenty-one percent tax and a global discount of ten
percent.
Then negative lines carrying the key `global_discount` are produced whose totals are exactly minus
one hundred untaxed and minus twenty-one of tax, and the result is frozen into manual amounts so
that a later recomputation cannot move it.

---

## Q. Maintenance and lifecycle

**Q1 — Archiving a tax.**
Given a tax used on posted invoices.
When it is archived.
Then it disappears from selection lists, the posted invoices are untouched and every report still
works.

**Q2 — The tax list shows archived taxes.**
Given an archived tax.
When the taxes menu is opened.
Then the archived tax is listed, because the window action deliberately includes inactive records.

**Q3 — Changing a used tax logs a readable difference.**
Given a tax in use whose first invoice distribution line's account is changed.
Then the message history shows *Invoice repartition line 1* with the old account name, an arrow and
the new account name, annotated with the word *Account*.

**Q4 — Changing an unused tax logs nothing about the distribution.**
Given a tax that is not in use.
Then no distribution difference is written.

**Q5 — Re-deriving the tags of existing items.**
Given posted entries dated from the first of April and a change to a tax's tags.
When the maintenance operation is run with a starting date of the first of April.
Then every base item's tags are rebuilt from the `base` distribution line of each of its taxes and
every tax item's tags from its own distribution line, for entries dated on or after that date.

**Q6 — A child tax in two groups.**
Given a child tax that belongs to two different Groups of Taxes.
When the maintenance operation is run.
Then it is refused with
*"Update with children taxes that are child of multiple parents is not supported."*

**Q7 — The default starting date.**
Given a company whose tax lock date is the thirty-first of March.
When the maintenance operation is opened.
Then the starting date defaults to the first of April.

**Q8 — A warning for an earlier date.**
Given the same company and a starting date of the first of March.
Then a warning is displayed, but the operation is still allowed.

**Q9 — Creating a foreign registration.**
Given a fiscal position with a country for which no tax exists and a foreign registration number.
Then a banner offers to create that country's taxes; clicking it installs the localization
capability if needed, instantiates the taxes and attaches each of them to the fiscal position.

**Q10 — Creating a tax writes no "created" message.**
When a tax is created.
Then no automatic "created" entry appears in its message history and the creator is not subscribed
to its thread.

---

## R. Ordering, flattening and batching

**R1 — Flattening order.**
Given taxes named *G*, *B*, *E* and *C* whose sequences order them alphabetically, where *B* is a
Group of Taxes containing *A*, *D* and *F* whose sequences also order them alphabetically.
Then the evaluation order is *A*, *D*, *F*, *C*, *E*, *G*.

**R2 — The group's own sequence positions its children.**
Given a Group of Taxes at sequence one containing a child at sequence nine, and an ordinary tax at
sequence five.
Then the child is evaluated **before** the ordinary tax, because the group's sequence decides the
group's position.

**R3 — Ties are broken by identifier.**
Given two taxes with the same sequence.
Then the one with the smaller identifier is evaluated first.

**R4 — An unsaved tax sorts first.**
Given a saved tax and an unsaved one with the same sequence.
Then the unsaved one is evaluated first, because it has no identifier.

**R5 — Two price-included percentage taxes batch together.**
Given two price-included ten percent taxes, neither affecting the base.
Then they form one batch and are extracted by dividing by one point two, not twice by one point
one.

**R6 — An affecting tax breaks the batch.**
Given the same two taxes but with the earlier one flagged "affect base of subsequent taxes" and
the later one accepting it.
Then two batches of one are formed and the taxes cascade.

**R7 — An affecting tax does not break the batch when the next tax refuses to be affected.**
Given the same two taxes with the earlier one flagged "affect base" and the later one **not**
accepting to be affected.
Then the fourth batching condition holds and the two taxes form one batch.

**R8 — Different computation kinds never batch.**
Given a percentage tax and a division tax adjacent in the order.
Then they form two batches.

**R9 — The special mode collapses the inclusion test.**
Given a price-included and a price-excluded percentage tax, adjacent, neither affecting the base,
under the special mode "total included".
Then they form one batch.

**R10 — Filtering keeps the group link.**
Given a Group of Taxes with two children and a filter that removes one of them.
Then the surviving child still records the group as its originator group.

---

## S. Engine internals

**S1 — The untaxed total is the first result's base.**
Given a line with one price-included tax and one price-excluded tax.
Then the untaxed total is the base of the **first** result in evaluation order, not the raw base.

**S2 — The extra base for the amount is frozen once the amount is known.**
Given a tax *X* evaluated before a tax *Y*, where *Y*'s amount is already known when *X*'s
propagation runs.
Then *Y*'s base for **reporting** is updated but the base *Y*'s amount was computed on is not.

**S3 — Downstream taxes are recorded.**
Given a tax *A* flagged "affect base of subsequent taxes" and a tax *B* accepting it.
Then *A*'s result records *B* as a downstream tax, and the journal item produced for *A* carries
*B* in its base-tax set.

**S4 — The subsequent-tax stack only collects taxes that accept being affected.**
Given a tax *A* flagged "affect base" and a tax *B* **not** accepting to be affected.
Then *A*'s downstream tax list does **not** contain *B*.

**S5 — The batch total includes the reverse-charge halves.**
Given a price-included reverse-charge tax in a batch.
Then the amount subtracted from the base is the sum of the batch's amounts **plus** the sum of the
reverse-charge halves of those members that have one — which is zero for a symmetric reverse
charge.

**S6 — The company-currency amount is a division.**
Given a rate of one point two five foreign units per company unit and a foreign amount of one
hundred.
Then the company amount is eighty.

**S7 — Round per line rounds the raw base first.**
Given the rounding method "round per line", a quantity of twelve point one two and a unit price of
twelve point one two.
Then the raw base is one hundred forty-six point eight nine **before** any tax is computed.

**S8 — Round per tax does not round the raw base.**
Given the same with "round per tax".
Then the raw base is one hundred forty-six point eight nine four four.

**S9 — Merging tax details shifts the base of a tax missing from the second block.**
Given two tax details blocks, the first carrying a fixed tax the second does not carry.
Then, after the merge, the second block's untaxed totals are added to that fixed tax's base
amounts, so that the merged base stays meaningful.

**S10 — Splitting a base line preserves the total.**
Given a base line with a tax amount of one and weights of one third each.
Then the three pieces' tax amounts add back to exactly one.

**S11 — Reducing lines to a target amount hits the target exactly.**
Given a document whose grand total is one thousand two hundred ten and a request for a fixed
amount of three hundred.
Then the resulting lines' grand total is exactly three hundred point zero zero.

**S12 — Aggregating with an empty tax set calls the grouping function once.**
Given a base line with no tax.
Then the grouping function is called once with an empty tax result, so the line still contributes
to the untaxed amount of its group.

**S13 — The analytic average of an aggregation.**
Given a line of one thousand distributed one hundred percent to an analytic account and a line of
minus one hundred distributed fifty percent to the same account, aggregated together.
Then the aggregate's distribution for that account is one hundred five point five six percent.

**S14 — The analytic average of an aggregation totalling zero.**
Given lines whose raw untaxed totals cancel exactly.
Then every analytic percentage becomes one hundred.

---

## T. Multi-company

**T1 — A branch sees its parent's taxes.**
Given a parent company owning a tax and a branch.
Then a user acting for the branch sees that tax.

**T2 — A parent does not see its branch's taxes.**
Given a tax owned by the branch.
Then a user acting only for the parent does not see it.

**T3 — Filtering taxes by company walks upwards.**
Given a tax set containing one tax owned by the parent and one owned by the branch, and a request
to filter for the branch.
Then only the branch's tax is returned; when the branch owns none, the parent's is returned.

**T4 — A tax may not be used outside its company tree.**
Given a tax of company *A* and an attempt to change its company to *B* while journal items of *A*
reference it.
Then it is refused.

**T5 — A distribution line with no company is visible everywhere.**
Given a distribution line whose company is empty.
Then it passes the record rule for every company.

**T6 — Account tags are not scoped by company.**
Given a tag created for a country.
Then every company sees it, subject only to the tag chooser's country restriction on a
distribution line.

**T7 — The display name shows the company when asked.**
Given more than one active company and a request to append the company.
Then the tax's display name ends with the company's display name in parentheses.

**T8 — The display name shows a foreign country.**
Given a tax whose country differs from the fiscal country of the first accessible branch of its
company.
Then the display name ends with that country's two-letter code in parentheses.

---

## U. Defaulting taxes on a line

**U1 — From the product on a sales document.**
Given a product with a sales tax and an account with a different default tax, on a customer
invoice.
Then the line takes the **product's** sales tax.

**U2 — From the account when the product has none.**
Given a product with no sales tax and an account with a default sales tax.
Then the line takes the account's tax, restricted to the sales kind.

**U3 — Nothing is wiped when the account proposes nothing.**
Given a line with a manually chosen tax, no product, and an account proposing no tax.
Then the manual tax survives, because the computation only runs when the line has a product, or
the account proposes taxes, or the line currently has none.

**U4 — A discount line is left alone.**
Given a line whose display kind is a discount and no product.
Then no tax is computed for it.

**U5 — Section, note and payment-term lines are skipped.**
Given a section, a subsection, a note, a payment term line or a cost-of-goods line.
Then no tax is computed at all.

**U6 — An imported line is skipped.**
Given a line marked as coming from an imported document.
Then no tax is computed.

**U7 — A miscellaneous entry takes nothing by default.**
Given a miscellaneous entry line with no product.
Then no tax is proposed unless the caller explicitly asks for the account's default taxes.

**U8 — The fiscal position maps the defaults.**
Given a document with a fiscal position that replaces the product's tax.
Then the line's taxes are the replacements, not the product's.

**U9 — Changing the account recomputes only under a condition.**
Given a line whose product proposes a tax for the company, and a change of account to one that
also proposes taxes.
Then the taxes are **not** recomputed, because the condition requires the product to propose none.

---

## V. Printing and presentation

**V1 — The tax column.**
Given a line carrying two taxes whose labels are `21%` and `Eco`.
Then the tax column prints `21%, Eco`.

**V2 — A tax with no label.**
Given a tax whose invoice label is empty and whose name is "Withholding", flagged "withhold on
payment".
Then nothing is printed for it, because a withholding tax's label never falls back to the name.

**V3 — A tax with no label that is not a withholding tax.**
Given a tax whose invoice label is empty and whose name is "Reduced".
Then "Reduced" is printed, because the label falls back to the name.

**V4 — The subtotal column under a tax-included default.**
Given the company default "tax included".
Then the line's subtotal column prints the total **with** taxes.

**V5 — The company-currency totals block.**
Given a sales document in a foreign currency, the company setting on, and at least one tax group.
Then a second totals block in the company currency is printed.

**V6 — The same on a purchase document.**
Then the second block is **not** printed, because the flag is restricted to sales documents.

**V7 — The fiscal position note.**
Given a document whose fiscal position has a note.
Then the note is printed after the totals.

**V8 — The tax legal notes.**
Given two lines carrying the same tax, which has legal notes.
Then the notes are printed once, not twice, because the taxes are collected without duplicates.

**V9 — The payment receipt.**
Given a payment carrying two withholding lines.
Then a table with the four columns Tax, Withholding number, Base and Amount is printed before the
ordinary content, with the base and the amount as absolute values.

**V10 — The product price hint.**
Given a product priced one hundred with a price-excluded tax of twenty-one percent.
Then the hint reads *"(= 121.00 Incl. Taxes)"* — only the first fragment, because the untaxed total
equals the price.

**V11 — The product price hint with a price-included tax.**
Given a product priced one hundred twenty-one with a price-included tax of twenty-one percent.
Then the hint reads *"(= 100.00 Excl. Taxes)"*.

**V12 — The product price hint with a withholding tax.**
Given a product priced one hundred with a price-excluded tax of twenty-one percent and a
withholding tax of minus one percent.
Then the hint reads *"(= 121.00 Incl. Taxes, 1.00 Tax Withheld)"*.

**V13 — No fragment at all.**
Given a product with no tax.
Then the hint is a single space.

---

## W. Reconstructing the base-to-tax mapping

**W1 — A simple entry.**
Given a posted invoice with one base item of one thousand carrying one tax of twenty-one percent,
and the resulting tax item of two hundred ten.
When the mapping is reconstructed.
Then one row exists, pairing the base item with the tax item, with a base amount of one thousand
and a tax amount of two hundred ten, and with the base item as the source item.

**W2 — Two base items, one tax item.**
Given two base items of one thousand and two thousand carrying the same tax, and one tax item of
six hundred thirty.
Then two rows exist, with base amounts of one thousand and two thousand and tax amounts of two
hundred ten and four hundred twenty.

**W3 — A tax affecting the base.**
Given the illustrating entry of `calculations.md` section 16 — three base items of one thousand,
two thousand and three thousand, a tax *ten affecting base* and two subsequent taxes.
Then nine rows exist, exactly as tabulated in section 16.5, and each tax item's tax amounts add
back to its own balance.

**W4 — The tail test excludes the wrong base items.**
Given the same entry.
Then the tax item produced by the affecting tax on base item one is paired **only** with base item
one, because base items two and three carry a different subsequent tax.

**W5 — The fallback.**
Given a posted entry whose tax configuration was changed afterwards, so that the exact pairing
finds nothing for a tax item.
Then the fallback pairs that tax item with every base item of the same entry and the same currency
carrying its originator tax, with each base item's whole balance.

**W6 — The fallback switched off.**
Given the same entry and a caller that switched the fallback off.
Then the orphan tax item produces no row at all.

**W7 — A fixed tax contributes by quantity.**
Given a base item with a quantity of seven and a fixed tax.
Then the contribution used to allocate the tax amount is the absolute quantity carrying the sign of
the balance, not the balance.

**W8 — The exigible column.**
Given a tax item of a tax exigible on payment, on the original invoice.
Then the row's exigible column is false. Given the corresponding item of the cash basis entry, the
column is true.

**W9 — The row identifier.**
Given a row pairing tax item forty-two, base item seven and source item seven.
Then the row's identifier is the three identifiers joined by hyphens in that order.

---

## X. Regression fixtures

Ten compact fixtures that together exercise every branch of the engine. An implementation that
reproduces all ten is very unlikely to be wrong anywhere.

| # | Configuration | Input | Expected untaxed / tax / total |
|---|---|---|---|
| X1 | one tax, twenty-one percent, price-excluded | one unit at 100.00 | 100.00 / 21.00 / 121.00 |
| X2 | one tax, twenty-one percent, price-included | one unit at 121.00 | 100.00 / 21.00 / 121.00 |
| X3 | fixed 0.05 per unit at sequence one, twenty percent at sequence two, both price-excluded, the fixed one affecting the base | seven units at 15.00 | 105.00 / 21.42 / 126.42 |
| X4 | division ten percent, price-excluded | one unit at 180.00 | 180.00 / 20.00 / 200.00 |
| X5 | group of ten percent and five percent, price-excluded | one unit at 100.00 | 100.00 / 15.00 / 115.00 |
| X6 | two price-included ten percent taxes in one batch | one unit at 100.00 | 83.34 / 16.66 / 100.00 |
| X7 | twenty-three percent price-excluded, round per line | three lines of 12.12 × 12.12 | 440.67 / 101.34 / 542.01 |
| X8 | the same, round per tax | the same | 440.68 / 101.36 / 542.04 |
| X9 | twenty-one percent price-included, round per tax | three lines of one unit at 21.53 | 53.38 / 11.21 / 64.59 |
| X10 | ten percent price-excluded, a currency with no decimal places, round per tax | three lines of one unit at 105 | 315 / 32 / 347 |

For X8, X9 and X10 the per-line split matters as much as the totals:

| Fixture | Line 1 | Line 2 | Line 3 |
|---|---|---|---|
| X8 balances | 146.90 | 146.89 | 146.89 |
| X8 taxes | 33.78 | 33.79 | 33.79 |
| X9 balances | 17.80 | 17.79 | 17.79 |
| X9 taxes | 3.73 | 3.74 | 3.74 |
| X10 taxes | 10 | 11 | 11 |
