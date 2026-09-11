# Taxes — Workflows

End-to-end operational sequences, step by step, with the role that performs each step, the
preconditions, and the records created or updated. Every formula referenced here is specified in
`calculations.md`; every validation in `business-rules.md`.

Roles used below:

| Role | Meaning |
|---|---|
| Accounting administrator | A user in the accounting administrator group. Creates and edits taxes, tax groups, fiscal positions and report definitions. |
| Accountant | A user in the full accounting group. Posts entries, reconciles, registers payments. |
| Invoicing user | A user in the invoicing group. Creates and posts invoices and bills; may read taxes but not change them. |
| Internal user | Any employee. May read taxes and fiscal positions only. |
| System | The application acting on its own, in a computed field, a constraint or a scheduled job. |

---

## 1. Creating a tax

**Performed by** the accounting administrator.
**Precondition** a company exists and has a fiscal country, or at least a country.

1. The administrator opens the tax list and creates a record.
2. **System** precomputes, before the record is shown:
   - the company, to the acting company;
   - the country, to the company's fiscal country, falling back to the company's country;
   - the tax group, to the first tax group of that company whose country equals the tax's country,
     falling back to the first tax group of that company with no country;
   - the invoice distribution, to one base line and one tax line of one hundred percent with no
     account and no tags;
   - the refund distribution, to the same.
3. The administrator sets the name, the tax type (sales, purchases or none), the computation kind
   and the amount.
4. **System**, when the amount is changed on a percentage or division tax to a non-zero value and
   no invoice label exists yet, proposes the label as the amount formatted with four significant
   digits followed by a percent sign.
5. **System**, when the computation kind is changed away from "group of taxes", clears the children;
   when it is changed to "group of taxes", clears the invoice label.
6. The administrator opens the definition page and sets, on each of the two distributions: the
   percentage, the account and the report tags of every line.
7. **System**, whenever a distribution line's account or kind changes, recomputes its "used in the
   tax settlement" flag as: kind is `tax`, an account is set, and that account's internal group is
   neither income nor expense.
8. The administrator opens the advanced page and may set: the invoice label, the description, the
   tax group, the analytic flag, the company, the country, the legal notes, the price-inclusion
   override, "affect base of subsequent taxes", "base affected by previous taxes" (visible only to
   a developer-mode user and only when the tax is not price-included), the exigibility (visible
   only when the company uses cash basis) and the cash basis transition account (mandatory when
   the exigibility is "based on payment").
9. **System**, when the price-inclusion is switched on in the form, proposes to switch "affect base
   of subsequent taxes" on as well.
10. On save, **System** runs every structural validation of `business-rules.md` sections 1 and 2.
11. **Records created**: one Tax, two or more Tax Distribution Lines.
12. **Side effect**: the tax becomes selectable on documents of the matching kind whose tax country
    matches the tax's country.

### 1.1 Creating a Group of Taxes

1. Create a tax whose computation kind is "group of taxes".
2. Add children. The selection list is restricted to taxes whose tax type is `none` or equal to the
   group's, and whose computation kind is not itself "group of taxes".
3. The group needs no distribution at all; when it has none, the structural validations are
   skipped for it.
4. The children keep their own distributions, accounts and tags; the group only provides the
   position in the evaluation order and a single label for the user.

### 1.2 Creating a withholding tax

*Requires the withholding capability.*

1. Create a tax whose amount is **negative**.
2. Switch "withhold on payment" on. **System** forces the exigibility back to "based on invoice"
   and the price-inclusion override to "tax excluded".
3. Optionally attach a withholding sequence so that certificate numbers are drawn automatically.
4. Leave the invoice label empty to keep the tax off printed documents.
5. The tax may be attached to products exactly like any other tax; it will be ignored on every
   document total and offered when a payment is registered.

---

## 2. Creating the report lines that create the tax grids

**Performed by** the accounting administrator (in practice this is shipped data).

1. A report definition is created or edited with a country.
2. A report line is added. The administrator either creates an expression of the tax-tags kind
   directly, or fills the shortcut field, which creates one expression labelled `tax_tags` whose
   formula is the written text.
3. **System**, on creating an expression of the tax-tags kind, looks for an Account Tag whose name
   is the formula with any leading minus sign removed, whose applicability is `taxes` and whose
   country is the report's country. If none exists it creates one.
4. **System** aligns the new tag's translations: for every installed language other than English,
   when the tag's English name is one leading character followed by a report line's English name,
   the tag's name in that language becomes that same leading character followed by that report
   line's name in that language.
5. The administrator can now attach the tag to a tax distribution line (workflow 1 step 6).
6. **Records created**: one Report Expression, possibly one Account Tag.

### 2.1 Renaming a grid

1. The administrator changes the formula of one or more tax-tags expressions.
2. **System**: if tags already exist for the new formula in that country, nothing happens.
   Otherwise it locates the tags of the old formula; if **every** expression that uses them is part
   of the same change, it renames the tags in place; otherwise it creates new tags. The
   distribution lines keep pointing at whatever tag record they held, so a rename is transparent
   and a split is not.

### 2.2 Deleting a report line or expression

1. **System** collects the tags matched by the deleted expressions.
2. For each tag, if no other tax-tags expression of the same country still names it:
   - the tag is removed from every tax distribution line that references it;
   - if at least one journal item still carries the tag, the tag is **archived**;
   - otherwise the tag is **deleted**.
3. **Records updated**: Tax Distribution Lines; **records archived or deleted**: Account Tags.

### 2.3 Moving a report to another country

1. The administrator changes a report's country.
2. **System**, for each tax-tags expression of that report, locates the tags matching its formula
   in the **old** country. If every report that uses those tags is being moved in the same change,
   the tags themselves are moved to the new country. Otherwise, tags are created in the new country
   if they do not already exist.

---

## 3. Putting taxes on a document line

**Performed by** an invoicing user or an accountant, or by another domain's automation.

1. A line is added to a document and a product is chosen.
2. **System** computes the line's taxes:
   - the computation is skipped entirely for a section, a subsection, a note, a payment term line,
     a cost-of-goods line, and for a line that came from an imported document;
   - it also stops unless the line has a product, or (the line is not a discount line and either
     the account proposes taxes or the line currently has none) — this is what prevents the system
     from wiping a manually chosen tax when the account proposes nothing;
   - for a **sales** document: the product's sales taxes restricted to the document's company, else
     the account's default taxes restricted to the sales kind;
   - for a **purchase** document: the product's purchase taxes restricted to the company, else the
     account's default taxes restricted to the purchase kind;
   - when the caller asked for account defaults: all of the account's default taxes;
   - otherwise: nothing for a miscellaneous entry or when the caller asked to skip the computation,
     and the account's default taxes in every other case.
3. **System** narrows the result to the line's company: it walks from the line's company up through
   its parents and keeps the first non-empty subset.
4. **System** applies the document's fiscal position, replacing each tax by the taxes it maps to
   (`calculations.md` section 11.1).
5. **Records updated**: the line's base-tax set.

### 3.1 Choosing the account

The line's account is chosen by the host domain from the product, the product category or the
journal, and is then passed through the fiscal position's account mapping
(`calculations.md` section 11.2).

### 3.2 Adapting the unit price

When the taxes change because of a fiscal position, the stored unit price may need to change too,
because a price-included tax is part of the number. The adaptation of `calculations.md` section
11.5 runs. It does nothing unless **every** original tax was price-included.

### 3.3 Changing the account by hand

When a user changes a line's account, the tax set is recomputed **only if** the new account
proposes taxes for the line's company **and** the line's product proposes none for that company.

---

## 4. Recomputing the tax journal items of a document

**Performed by** the System, inside every save of a draft entry.

The goal is to touch as few journal items as possible: a tax item that still matches is updated in
place, so that manually typed tax amounts survive edits that do not concern them.

1. **Before the change**, snapshot for every entry in the batch:
   - the entry's currency, partner, kind, currency rate and invoice date;
   - for every base item (product, early payment discount, rounding, non-deductible product): the
     base grouping key fields plus, for an invoice, the unit price, quantity, discount and
     deductibility, and for a non-invoice, the amount in document currency;
   - for every tax item: its amount in document currency, its balance and its analytic
     distribution.
2. **The change happens.**
3. For each entry still in draft, decide whether and how to recompute, in this order:

   | Condition | Decision |
   |---|---|
   | The entry is an invoice and its currency or its kind changed | recompute from scratch, ignoring the existing tax amounts |
   | A base item that carried taxes has disappeared | keep the existing tax amounts only if at least one tax item changed |
   | At least one base item changed | keep the existing tax amounts when **either** none of the changed lines carries or carried taxes, **or** the set of tax items itself changed or one of their fields is currently being written by the caller. Additionally, if the amounts are to be kept and any changed line has a non-zero amount or balance supplied explicitly, **skip the recomputation entirely** — the caller supplied a complete entry. |
   | Only the currency rate changed | recompute, keeping the tax amounts in document currency and re-deriving the company-currency balances from the new rate |
   | Nothing relevant changed | skip |

4. Build the base lines and the tax lines from the entry (`calculations.md` section 1), run the
   engine, the document-wide rounding, the accounting derivation and the entry production.
5. Apply the result:
   - write the new tag set, amount and balance on each base item that needs it;
   - delete the tax items the production marked for deletion;
   - create the tax items it marked for creation, with display kind `tax`;
   - write the new amounts on the tax items it marked for update;
   - create, update or delete the non-deductible tax item.
6. Writes are grouped by (currency, new values) so that a single write statement serves many items
   and no write mixes two currencies.

---

## 5. Posting a document

**Performed by** an accountant or an invoicing user.

1. **System** validates that the entry balances, that the accounts are consistent, and that the
   taxes are consistent with the document's tax country (`business-rules.md` section 5).
2. **System** checks the tax lock date: when the entry's date falls inside a locked period and the
   entry affects the tax report, posting is refused (`business-rules.md` section 6).
3. The entry is posted. The tax items become part of the tax return through their report tags.
4. For a tax exigible on payment, the tax item sits on the **transition** account and carries **no**
   report tag, so nothing reaches the return yet.

---

## 6. Reversing a document

**Performed by** an accountant.

1. The reversal mechanism of `../general-ledger/` creates a mirror entry.
2. **System** copies the stored extra tax data of each line and negates the quantity, the manual
   untaxed totals and every manual base and tax amount, in both currencies
   (`calculations.md` section 9.3). The reversal therefore reproduces a manually adjusted tax
   exactly, with the opposite sign.
3. For a customer credit note or a vendor refund, the base lines are flagged as refunds, so the
   **refund** distribution is used: different accounts and different tags may apply
   (`calculations.md` section 8.7).
4. For a miscellaneous entry that is the reversal of another entry, the refund flag derived from
   the line's sign is **inverted** (`calculations.md` section 8.1).

---

## 7. Cash basis at reconciliation

**Performed by** an accountant, or by the System during automatic reconciliation.

**Precondition** the company has "use cash basis" on, a cash basis journal and a base tax received
account; at least one tax on the document is exigible on payment.

1. The accountant reconciles a payment (or a credit note) against the document.
2. **System** creates one or more partial reconciliations.
3. **System**, for each partial, collects the cash basis data of both sides
   (`calculations.md` section 12.1) and computes the paid percentage and the payment rate
   (section 12.2).
4. **System** creates one journal entry per partial (`accounting-effects.md` section 6), posting it
   immediately when both documents are posted and leaving it in draft otherwise.
5. **System** reconciles each new counterpart item on the transition account against the original
   tax item, when that account allows reconciliation.
6. **Records created**: one Journal Entry per partial, two Journal Items per grouped share,
   possibly an exchange difference entry.

### 7.1 Undoing it

1. The accountant unreconciles.
2. **System** deletes the partial reconciliation, which:
   - deletes every draft cash basis entry of that partial;
   - reverses and cancels every posted one, dated on the original date or on the day after the last
     violated lock date, with the reference *"Reversal of: &lt;the original number&gt;"*;
   - does the same for the partial's exchange difference entry.

---

## 8. Withholding at payment

**Performed by** an accountant.

**Precondition** the company owns at least one withholding tax matching the payment direction.

### 8.1 Through the register-payment wizard

1. The accountant selects one or more documents and opens the wizard.
2. **System** shows the withholding section only when: the company owns a matching withholding tax
   **and** the wizard will produce a single journal entry (it is editable and it is not going to
   split into one payment per document). For a batch containing refunds, the direction used to pick
   the matching taxes is inverted.
3. **System** derives the withholding lines from the base lines of the selected documents
   (`calculations.md` section 13.1) the first time the wizard is opened.
4. **System** computes, for each line: the original base and tax amounts converted into the wizard
   currency, the paid factor, the withholding base and the withholding amount.
5. **System** computes the **net amount** as the payment amount minus the sum of the withholding
   amounts.
6. The accountant may add lines, delete lines, change the base or change the amount by hand.
7. **System**, each time a line is edited, refreshes the certificate-number hints of every line
   whose kind of hint changed.
8. The accountant may need to choose an outstanding account when the payment method line has none.
9. The accountant confirms. **System** refuses when the net amount is negative:
   *"The withholding net amount cannot be negative."*
10. **System** creates the payment with the outstanding account, the withholding flag and a copy of
    every withholding line (dropping the wizard link and the hint).
11. **System** builds the payment's journal entry and appends the withholding items
    (`accounting-effects.md` section 7). Certificate numbers are drawn from the sequences at this
    point, and only after every line has been checked.
12. **Records created**: one Payment, one Payment Withholding Line per wizard line, one Journal
    Entry with the extra items.

### 8.2 Directly on a payment

1. The accountant creates a payment and switches "withhold tax amounts" on.
2. The accountant adds withholding lines by hand: a tax, a base amount and, when the company has no
   default withholding base account, an account.
3. For a line typed by hand there is no source currency, so the amount is computed by running the
   engine on the typed base with only that tax and negating the result.
4. The rest is as in 8.1 from step 11.

---

## 9. Re-deriving the report tags of existing journal items

**Performed by** the accounting administrator, through a dedicated maintenance operation.
*Requires the tag-update capability.*

**Precondition** the administrator has changed the distribution or the tags of one or more taxes
and wants already-posted items to reflect the change.

1. The administrator opens the operation and picks a **starting date**. It defaults to the day
   after the company's tax lock date when there is one, otherwise today.
2. **System** warns when the chosen date is earlier than the tax lock date.
3. **System** refuses to run when any child tax belongs to more than one group of taxes:
   *"Update with children taxes that are child of multiple parents is not supported."*
4. **System** recomputes, for every journal item of the company dated on or after the starting date:
   - for a **base** item: the tags of the `base` distribution line of each of its taxes (expanding
     a group of taxes into its children), choosing the invoice or the refund distribution by the
     rule of `calculations.md` section 8.8;
   - for a **tax** item: the tags of its own distribution line.
5. **System** deletes every existing tag link of those items and inserts the new ones.
6. **Records updated**: the tag links of every touched journal item.

---

## 10. Fiscal positions

### 10.1 Creating one that maps taxes

**Performed by** the accounting administrator.

1. Create a Fiscal Position with a name.
2. Switch "detect automatically" on and fill the criteria: a country, or a country group, or a
   list of states, or a postal code range, and optionally "tax registration required".
3. **System** left-pads a fully numeric postal code bound with zeros to the length of the longer
   bound.
4. Attach the **replacement** taxes to the fiscal position through the tax's own "fiscal positions"
   field, and on each replacement tax name the domestic taxes it replaces.
5. **System** rebuilds the fiscal position's lookup table from those declarations.
6. Optionally add account mappings.
7. **Records created**: one Fiscal Position, zero or more Fiscal Position Account Mappings, and
   links on the replacement taxes.

### 10.2 Detecting one on a document

**Performed by** the System when a partner is set on a document.

1. Run the detection of `calculations.md` section 11.3 with the document's partner and its delivery
   address.
2. Write the result on the document.
3. **System** then recomputes the document's tax country: the fiscal position's country when the
   fiscal position carries a foreign registration number, the company's fiscal country otherwise.
4. **System** recomputes every line's taxes and accounts through the mapping (workflow 3).

### 10.3 Registering the company in a foreign territory

**Performed by** the accounting administrator.

1. Create a Fiscal Position, set its country and type the company's registration number for that
   territory into the foreign registration field.
2. **System** normalises and validates the number against that country's rules
   (`calculations.md` section 15), raising the standard message with the label
   *"fiscal position [&lt;the fiscal position's name&gt;]"* on failure.
3. **System** validates the rules of `entities.md` section 4.3.
4. **System** recomputes the company's list of foreign registration countries and its list of
   tax-enabled countries.
5. **System** shows a banner offering to create that country's taxes when no tax for that country
   exists yet.
6. The administrator clicks it. **System** installs the country's localization capability when it
   is not installed, instantiates that country's taxes for the company, and attaches every created
   tax to this fiscal position.
7. **Records created**: taxes, tax groups, possibly accounts; links on the fiscal position.
8. **Side effect**: documents carrying that fiscal position may now use that country's taxes, and
   that country's report tags become selectable on distribution lines.

---

## 11. Tax identification numbers

### 11.1 Entering one on a partner

**Performed by** any user who can edit the partner.

1. The user types a number.
2. **System**, while the form is open, runs the pipeline of `calculations.md` section 15.1 in
   *off* mode: the number is normalised but a failure is silent.
3. On save, **System** runs the pipeline in *error* mode against the commercial partner's country
   and writes the normalised number back when it differs.
4. **System**, when at least one company has cross-border verification on, sends one verification
   request and stores the outcome in the "intra-community valid" flag, logging a message on the
   partner.
5. A partner whose parent carries the same number inherits the parent's flag without a request.
6. During a file import the recomputation of the flag is cancelled.

### 11.2 The delayed verification result

1. A request may come back as *pending*.
2. **System** either receives a callback on the route `/base_vat/1/webhook_update_vies` carrying a
   signed token and a status, verifies the token and applies the status to every partner with that
   number; or the daily job asks the relay for updates and applies whatever it returns.

### 11.3 Consequence on fiscal positions

A fiscal position that requires a tax registration only matches a partner whose number is present
and — when cross-border verification applies to that partner for the acting company — whose flag is
true.

---

## 12. Archiving and deleting a tax

**Performed by** the accounting administrator.

1. The administrator tries to delete a tax.
2. **System** refuses when the tax is in use — that is, when at least one journal item or one
   reconciliation model line references it:
   *"You cannot delete taxes that are currently in use. Consider archiving them instead."*
3. The administrator archives it instead by clearing the active flag. The tax disappears from
   selection lists; historical items keep pointing at it; reports keep working.
4. An archived tax is still visible in the tax list when the "inactive" filter is applied, and the
   default window action for taxes deliberately includes archived records.

### 12.1 Changing a tax that is already in use

1. **System** records the change in the tax's message history. The four tracked fields are the
   name, the tax type, the computation kind, the amount, the price-inclusion override, "affect base
   of subsequent taxes" and "base affected by previous taxes".
2. **System**, when the tax is in use, also produces a readable difference of the distribution:
   for each distribution line, renumbered per document kind, the old and new values of the
   percentage, the account, the list of tags and the settlement flag, rendered as an
   old-value/new-value pair per changed field. Added and removed lines are shown as such.
3. The raw snapshot field itself is removed from the tracked values so that the change appears only
   once.
4. **System** refuses to change the company of a tax that has journal items:
   *"You can't change the company of your tax since there are some journal items linked to it."*
5. **Existing journal items are not recomputed.** Only the maintenance operation of workflow 9
   re-derives their tags; their amounts are never re-derived.

---

## 13. Turning cash basis on and off for a company

**Performed by** the accounting administrator.

1. In the settings, switch "cash basis" on. The exigibility field becomes visible on every tax of
   that company, and the cash basis journal and base tax received account fields appear.
2. To switch it off: **System** searches the company's taxes for one whose exigibility is "based on
   payment". When one exists the switch is forced back on and a warning is shown:
   *"You cannot disable this setting because some of your taxes are cash basis. Modify your taxes first before disabling this setting."*

---

## 14. Producing the totals block of a document

**Performed by** the System on every recomputation of a document.

1. Build the base lines and run the engine and the rounding (workflow 4 steps 4).
2. Run the totals algorithm of `calculations.md` section 10 with the document's currency, the
   company and the document's cash rounding configuration when it has one.
3. Set the flag "also show the company currency" when: the company's setting says so, the company
   currency differs from the document currency, at least one tax group is involved, and the
   document is a sales document.
4. The block is not produced at all for a non-invoice entry, because such an entry may mix
   currencies.

---

## 15. Loading a chart of accounts template

**Performed by** the System when a company's chart of accounts is installed.

**Precondition** the company has no accounting data yet, or the template is being reloaded onto a
company whose data still matches the template.

1. The template's tax groups are created, each with its country and its settlement accounts.
2. The template's taxes are created. Each tax record in the template carries, on its first row,
   the name, the description, the invoice label, the amount, the tax type, the tax group, the
   fiscal positions it belongs to and the taxes it replaces; and, on that row and the rows that
   follow it, one distribution line each, given as a document kind, a percentage, a line kind and
   an account.
3. **System** suspends the cash basis transition account check while the template loads, so that a
   deferred tax may be created before its transition account exists.
4. The template's fiscal positions are created with their detection criteria and their sequence.
5. The links between fiscal positions and taxes, and between a replacement tax and the taxes it
   replaces, are established from the two columns on the tax rows.
6. **System** recomputes the company's domestic fiscal position.
7. **Records created**: Tax Groups, Taxes, Tax Distribution Lines, Fiscal Positions, Fiscal
   Position Account Mappings.

Country-specific templates, and the country catalogue of what each ships, belong to
`../fiscal-localizations/`.

---

## 16. Consumers of the engine outside the accounting documents

Every consumer follows the same four-step contract. An implementer must reproduce the contract,
not each consumer's own code.

```
1. Build a base line per taxable amount, choosing the sign, the special mode, the rate and the
   refund flag appropriate to the consumer.
2. Add the tax details to every base line at once, passing the company.
3. Round the tax details of all the base lines together, passing the company and, when the records
   already exist, the existing tax lines.
4. Read whatever is needed: the totals block, the aggregated amounts, or the accounting entries.
```

Steps two and three must see **every** base line of the document at the same time; calling them
per line reproduces the "round per line" behaviour even when the company asked for "round per tax".

### 16.1 An order

**Performed by** the sales or purchasing domain whenever an order line changes.

1. One base line per order line, with the order's currency, the line's price, quantity and
   discount, no special mode and the order's rate.
2. Steps two and three.
3. The totals block is produced and shown; no accounting entry is produced.
4. When the order is invoiced, the invoice lines carry the same prices and taxes, so the invoice's
   totals block agrees with the order's to the cent.

### 16.2 An expense

**Performed by** the expenses domain whenever the amount, the quantity or the taxes change.

1. One base line for the expense, with the vendor as partner, the special mode **total included**
   and the expense's own currency rate.
2. Steps two and three.
3. The tax amount shown on the expense is the total with taxes minus the untaxed total.
4. The default taxes of an expense are the product's **purchase** taxes restricted to the
   expense's company.

The "total included" mode is what makes an employee's typed amount mean "what I actually paid",
whatever the taxes' own price-inclusion says.

### 16.3 A bank statement line matched by a reconciliation model

**Performed by** the reconciliation component.

1. The model's line proposes an amount and a set of taxes.
2. One base line is built with the sign of the statement line.
3. Steps two, three and four; the resulting tax items are added to the statement line's entry.

### 16.4 A point-of-sale order

**Performed by** the point-of-sale client while offline, and again by the server on
synchronisation.

1. The client received, at session opening, every tax it may need, with the fields listed in
   `interfaces.md` section 3.5.
2. The client runs the mirrored engine on its own order lines.
3. On synchronisation the server rebuilds the base lines from the stored order and runs the same
   steps.
4. The two results must be identical. Any divergence between the two implementations of a step
   marked *mirrored* shows up here first.

### 16.5 A loyalty or promotion reward line

**Performed by** the promotions domain.

1. The discountable base lines are prepared, which drops every fixed and custom-formula tax into
   its own tax-free base line.
2. The reduction to a target amount is run with the reward's amount kind and amount.
3. The result is frozen into manual amounts, so that a later recomputation of the document cannot
   move the reward by a cent.

### 16.6 A down payment deduction

**Performed by** the sales domain when a final invoice deducts a previous down payment.

1. The original lines are prepared with no computation key.
2. The down-payment lines are prepared with the computation key `down_payment` and a negative
   amount.
3. Steps two and three see both sets at once, but the two redistribution passes group by the key,
   so each subset rounds on its own.

---

## 17. Diagnosing a wrong total

A checklist for an implementer whose numbers do not match.

| Symptom | Most likely cause |
|---|---|
| The document total is off by one cent, and the per-line subtotals look right | The delta of `calculations.md` section 7.6 is not being added to the base line's balance. |
| The sum of the tax journal items differs from the tax shown in the totals block | The residue redistribution of `calculations.md` section 8.2 is missing. |
| A price-included tax is counted twice | The untaxed total is being taken from the raw base instead of the first result's base. |
| Two price-included taxes give a different base from the specified one | The two taxes are being cascaded instead of batched. |
| A fixed tax changes the result of a price-included tax in the wrong direction | The fixed tax is not being evaluated in the first pass, or the propagation table of section 5.1 is being applied with the effective rather than the configured price-inclusion flag. |
| A credit note's tax lands on the wrong account | The refund distribution is not being used. |
| The tax return double-counts a base | Two taxes of different exigibility share a base tag on one line, which the validation of `business-rules.md` section 5.2 should have refused. |
| The total changes when the document is saved | The two implementations of the engine disagree on a *mirrored* step. |
| A down payment deduction moves the final invoice's total | The computation key is not taking part in the redistribution groupings. |
| Amounts in the company currency drift from the document currency | The two currencies are not being rounded and redistributed independently, or the conversion is being done as a multiplication instead of a division by the rate. |
