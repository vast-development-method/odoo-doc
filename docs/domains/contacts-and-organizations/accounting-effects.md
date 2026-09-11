# Accounting effects of the Contacts and Organizations domain

**This domain produces no journal entries.**

Creating, editing, archiving, deleting or merging a Party; creating or editing a Bank Account, a
Bank, a Country, a State, a Country Group, a City, a Currency, a Language, an Industry or a Party
Tag; blocking or unblocking a telephone number; geocoding an address; publishing a public page —
none of these writes a single accounting line, changes a single balance, or moves a single amount.

The domain is nonetheless *decisive* for accounting, because it supplies the facts from which
other domains derive their entries. This file states, precisely, which of those facts come from
here, so that a rebuild knows what contract the accounting domains depend on.

---

# 1. What this domain supplies to the accounting domains

## 1.1 The counterparty of every accounting line

Every Journal Item carries a Party. The value written there is, in almost every flow, the
**commercial entity** of the Party named on the source document, not the Party itself. So an
invoice addressed to a delivery address under a subsidiary produces Journal Items whose Party is
the subsidiary.

Consequences a rebuild must preserve:

| Consequence | Why it follows from this domain |
|---|---|
| The receivable balance of a group is split per subsidiary, not aggregated | because each subsidiary is its own commercial entity |
| Two invoices, one addressed to a person and one to that person's delivery address, land on the same receivable balance | because both resolve to the same commercial entity |
| Reconciling a payment against an invoice works across those two documents | because reconciliation matches on the Party of the Journal Item, which is the same commercial entity |
| Changing a Party's organization flag changes which commercial entity **future** documents resolve to, but never rewrites existing Journal Items | because the commercial entity is stored on the Party and copied onto the document at creation time |

## 1.2 The country and state that select the tax treatment

The fiscal position that decides which taxes apply to a document is selected from the Party's
country and state, and from the presence and validity of its tax registration number. This domain
supplies all four facts:

- the country and state on the Party (and, through the address-resolution algorithm, on the
  delivery address when the tax rules key on the place of supply);
- the tax registration number, normalised and validated as specified in
  [calculations.md](calculations.md) §8;
- the country groups the country belongs to, which is how a customs union is recognised;
- the intra-community validity flag, where the verification behaviour is installed.

See [Taxes](../taxes/README.md) for how those four are consumed.

## 1.3 The currency in which the books are kept

Every Company names a currency, and that currency is the one in which its Journal Items are
balanced. This domain owns the Company entity, the constraint that a branch shares its root's
currency, and the rule that choosing a Company's currency activates it.

## 1.4 The bank account a payment is made to or from

The Bank Account entity, its sanitised number, its holder, its outgoing-payment permission flag and
its find-or-create procedure all live here.
[Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) produces the
entries; this domain guarantees:

- that two accounts of the same holder cannot carry the same number;
- that an account created from an incoming document is **not** payable until a human allows it;
- that deleting an account archives it, so that a posted payment never loses its account
  reference;
- that an account number written in any spelling resolves to the same account.

## 1.5 The language a document is rendered in

A Party's language decides the language of its invoices, its payment reminders and its statements.
This is presentation only: no amount changes. The **number formatting** of those documents comes
from the reader's language through the algorithm in [calculations.md](calculations.md) §10 — the
grouping and the separators are the language's, while the number of decimal places and the symbol
position are the currency's.

## 1.6 The credit standing of a counterparty

Where a credit limit exists, it is held on the **commercial entity** and the exposure is
aggregated across the whole subtree. That aggregation is a direct consequence of the commercial
entity rule specified here; the limit itself and its enforcement belong to
[Accounts Receivable](../accounts-receivable/README.md).

---

# 2. Operations of this domain that touch accounting data without producing entries

## 2.1 Changing a Party's company

Writing the company on a Party cascades to every child. It does **not** touch any existing Journal
Item: an entry posted for company A keeps its Party even after that Party is reassigned to company
B. The Party may then become invisible to company A's users under the multi-company record rule,
while the entry remains. A rebuild must not attempt to repair this: the accounting record is the
authority, and the Party's company is metadata.

## 2.2 Changing a Party's tax registration number

Writing the number pushes it up to the parent and down to every non-organization descendant (see
[calculations.md](calculations.md) §3). It does **not** alter any posted document. Documents carry
their own copy of the number at the moment they are issued, or re-read it at print time depending
on the document; either way, no accounting amount changes.

## 2.3 Archiving a Party

Archiving hides the Party from default searches. Every Journal Item that references it keeps the
reference, every balance still aggregates it, and every report that groups by Party still shows
it — reports read the stored reference, not a search. Archiving is therefore safe at any time and
has no accounting consequence.

## 2.4 Merging two parties

This is the only operation of the domain that rewrites accounting data, and it rewrites **only the
Party column**.

What changes:

- the Party column of every row in every table that has a foreign key to the Party table,
  including the invoice table, the Journal Item table, the payment table, the bank-statement-line
  table, the sales-order table (in all three of its Party columns) and the purchase-order table;
- the bank accounts, which are moved or deduplicated;
- the polymorphic references: attachments, followers, activities, messages and external
  identifiers.

What does **not** change:

- no amount, in any currency;
- no account;
- no date;
- no debit or credit side;
- no tax line and no tax amount;
- no reconciliation: a partial or full reconciliation that linked two Journal Items keeps linking
  the same two items, and the fact that they now share a Party rather than differing in Party has
  no effect on the reconciliation records;
- no journal, no sequence, no document number;
- no state: a posted entry stays posted, a draft stays draft.

What becomes possible after the merge that was not possible before:

- reconciling a receivable line of the former source against a payment of the former destination,
  because reconciliation requires the two lines to share a Party;
- a single aged-balance row instead of two;
- a single credit exposure instead of two.

What may be **lost** by the merge:

- a row that would violate a uniqueness constraint after the rewrite is deleted rather than moved.
  In an accounting context the candidates are rows in join tables and rows with a uniqueness
  constraint involving the Party column. A rebuild must check its own schema: any table where the
  Party participates in a uniqueness constraint will lose the source's rows where the destination
  already has an equivalent one. The Journal Item table has no such constraint, so no accounting
  line is ever lost.

Worked example: see [calculations.md](calculations.md) §11.10, which traces two invoices, two
sales orders, two bank accounts and a tag set through a merge, listing exactly which references
move.

## 2.5 Renaming a Party

Renaming rewrites the account holder name on every bank account of that Party whose holder name
still equalled the old name. Payment files generated *after* the rename therefore carry the new
name. Payment files already generated are unaffected. No entry changes.

## 2.6 Deleting a bank account

The deletion is turned into an archive precisely so that no posted payment loses its account. A
rebuild that implements a real deletion will break the audit trail.

---

# 3. What a rebuild must *not* do

- Do **not** create an opening balance, a write-off or an adjustment when parties are merged. The
  merge is a metadata operation.
- Do **not** post anything when a Party's tax registration number changes, even if that change
  would have altered the tax treatment of a past document. Correcting a past document is a
  deliberate act in [Accounts Receivable](../accounts-receivable/README.md) or
  [Accounts Payable](../accounts-payable/README.md), never an automatic consequence of editing a
  Party.
- Do **not** revalue anything when a Company's currency is activated. Activation is a visibility
  flag, not a functional-currency change; changing a Company's functional currency is out of scope
  for this domain and is refused while the branch constraint holds.
- Do **not** derive a receivable or payable account from anything in this domain. Account selection
  belongs to [General Ledger](../general-ledger/README.md) and to the receivable and payable
  domains; this domain supplies only the counterparty identity.

---

# 4. Summary table

| Event in this domain | Journal entry | Effect elsewhere |
|---|---|---|
| Create a Party | none | a counterparty becomes selectable |
| Attach a Party to a parent | none | the commercial entity of the subtree changes for **future** documents |
| Change the organization flag | none | same |
| Change the address | none | future documents print a different address; the geocoding coordinates are reset |
| Change the tax registration number | none | future documents carry a different number; the fiscal-position selection may change for future documents |
| Change the country or state | none | the fiscal-position selection may change for future documents |
| Change the language | none | future documents are rendered in the new language |
| Archive or unarchive a Party | none | the Party disappears from or returns to default searches; balances unaffected |
| Delete a Party | none | refused while any reference exists |
| Merge parties | none | the Party column of every referencing row is rewritten; reconciliation across the former two parties becomes possible |
| Create, edit or archive a Bank Account | none | the account becomes available to, or unavailable to, payment registration |
| Delete a Bank Account | none | the account is archived instead |
| Create or edit a Company | none | a new set of books exists, keyed to the chosen currency |
| Archive a Company | none | the company and every branch leave the company switcher; their books remain |
| Activate or deactivate a Currency | none | the currency becomes selectable; the multi-currency group is granted or withdrawn |
| Activate or deactivate a Language | none | documents may or may not be rendered in it |
| Block or unblock a telephone number | none | automated messaging is suppressed or resumed |
| Geocode an address | none | two decimal coordinates are stored |
| Publish a Party's public page | none | a public web page becomes reachable |
