# Accounting effects

## 1. The domain posts nothing

**No entity of this folder writes a Journal Entry, a Journal Item, an analytic line, a tax line or a
reconciliation.** The statement is not an omission; it follows from what the entities are.

| Entity | Why it produces no ledger entry |
|---|---|
| Automation Rule | It holds configuration only. It never creates a record of its own beyond its own log lines; the records it changes belong to other domains and post — or do not post — under their own rules. |
| Server Action as used by automations | The action is a carrier. Whatever it does is done by the record type it targets. |
| In-Application Purchase Service | A catalogue row naming a purchasable outside service and its unit. It carries no amount and no currency. |
| In-Application Purchase Account | It mirrors a balance held by the outside service. The balance is a count of service units, not money, and it is stored as text already formatted with its unit name. Nothing is owed, paid or accrued in this system when the count changes. |
| Lead Enrichment Interface, Partner Autocomplete Interface | Behaviours that call an outside service. They spend units of a balance held elsewhere. |
| Data Import Session | It loads rows into other record types. If those rows are invoices or payments, the receiving domain posts them; the session itself posts nothing. |
| Data Import Column Mapping | A remembered heading-to-field association. |
| Module Import Wizard, Module Activation Request, Module Activation Review | They install and request capability packages. |
| Recycling Model, Recycling Record | They archive or delete records. Deleting a record that carries a Journal Item is prevented by the ledger's own rules, not by this domain. |
| Privacy Log, Privacy Lookup Wizard, Privacy Lookup Wizard Line | They find, archive and delete records. The same remark applies. |
| Geocoding Provider, Geocoder | They write coordinates onto a Contact. |
| Google Service, Microsoft Service, Google Gmail Mixin, Microsoft Outlook Mixin | They hold credentials and build authentication strings. |
| Transifex Translation, Code Translation | They hold translated terms and build links. |
| Onboarding, Onboarding Step, Onboarding Progress Tracker, Onboarding Progress Step Tracker | They record which setup steps a company has completed. |
| Tour, Tour Step | They describe a walkthrough of the interface. |
| Sparse Fields Test | A reference record exercising a storage technique. |
| The cloud-storage kind of Attachment | It moves bytes from one store to another. The cost of the outside store is billed outside this system. |

**The one place money is mentioned.** The metered account carries a balance and a purchase address.
Buying more units happens on the service operator's own site, in the operator's own system. This
system holds no receivable, no payable, no prepayment and no expense for it, and it never learns the
monetary price of a unit. If an installation wishes to record that expense it does so by entering
the operator's invoice as an ordinary vendor bill in
[`../accounts-payable/`](../accounts-payable/), unconnected to the account record.

## 2. Ledger entries this domain causes indirectly

Although the domain posts nothing itself, it is the mechanism by which other domains are made to
post. Three routes exist, and a rebuild must reproduce all three faithfully because the ledger
consequences are real even though the trigger is configuration.

### 2.1 An automation rule invoking an action that posts

An Automation Rule attached to a record type of an accounting domain can run any Server Action that
record type supports, including the ones that confirm, validate or post. The ledger entry is
produced entirely by the receiving domain and is specified there.

| Configured rule | Domain that posts | Where the entry is specified |
|---|---|---|
| A Sales Order that reaches a stage runs the action that confirms it | [`../sales/`](../sales/) then [`../inventory-operations/`](../inventory-operations/) | [`../sales/accounting-effects.md`](../sales/accounting-effects.md) |
| A Customer Invoice whose due date is reached runs the action that posts it | [`../accounts-receivable/`](../accounts-receivable/) | [`../accounts-receivable/accounting-effects.md`](../accounts-receivable/accounting-effects.md) |
| A Vendor Bill that receives a tag runs the action that posts it | [`../accounts-payable/`](../accounts-payable/) | [`../accounts-payable/accounting-effects.md`](../accounts-payable/accounting-effects.md) |
| A Transfer that is validated runs an action that writes a field used in valuation | [`../inventory-valuation-and-costing/`](../inventory-valuation-and-costing/) | [`../inventory-valuation-and-costing/accounting-effects.md`](../inventory-valuation-and-costing/accounting-effects.md) |
| A Manufacturing Order that reaches a state runs the action that closes it | [`../manufacturing/`](../manufacturing/) | [`../manufacturing/accounting-effects.md`](../manufacturing/accounting-effects.md) |
| An Expense report that is approved runs the action that posts it | [`../expenses/`](../expenses/) | [`../expenses/accounting-effects.md`](../expenses/accounting-effects.md) |

**What the rebuild must preserve.** Two properties of the mechanism have ledger consequences:

1. **The rule runs inside the caller's transaction.** A rule that posts an entry posts it in the same
   transaction as the write that triggered it. If the surrounding operation is rolled back, so is the
   entry. A rebuild that ran rules asynchronously would create entries that survive a rollback.
2. **The rule runs with elevated rights.** A user who may not post an entry can nevertheless cause
   one to be posted by making the change the rule watches. The recorded author of the entry is the
   user who made the change, not the administrator who configured the rule. Any rebuild must
   reproduce both halves of that, because audit trails depend on it.
3. **The recursion guard applies.** A rule that posts an entry and thereby changes the watched field
   again does not post a second entry
   ([`business-rules.md#aut-017`](business-rules.md#aut-017)).

### 2.2 Importing accounting records from a file

A Data Import Session loads rows into whatever record type it was pointed at. When that record type
is an accounting one, the ordinary creation rules of that domain apply in full, including their
ledger effects.

| Imported record type | Effect | Where specified |
|---|---|---|
| Journal Entry and its Journal Items | The entry is created in the state the file gives; a file that sets the posted state posts it, with all the balance and period checks of the ledger | [`../general-ledger/`](../general-ledger/) |
| Customer Invoice | Created as a draft unless the file posts it | [`../accounts-receivable/`](../accounts-receivable/) |
| Vendor Bill | The same | [`../accounts-payable/`](../accounts-payable/) |
| Opening balances | Loaded as ordinary Journal Entries | [`../general-ledger/`](../general-ledger/) |
| Bank statement lines | Created for later reconciliation | [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/) |

**Two properties with ledger consequences.**

- **A trial run posts nothing.** The trial run is wrapped in a savepoint that is always rolled back,
  and the whole cache is cleared afterwards. A rebuild must guarantee that a trial run leaves no
  entry, no sequence consumption that survives, and no cached identifier.
- **A batched import commits between batches only through the caller.** The loader is given a batch
  size and reports where to resume; each submission is its own transaction. A run interrupted
  between batches therefore leaves the entries of the completed batches posted. A rebuild must
  either reproduce that or document a different guarantee, because an accountant resuming an import
  must know which rows are already in the ledger.

### 2.3 Deleting or archiving records that carry ledger consequences

Data recycling and privacy handling both delete records with elevated rights.

- **Archiving** never touches the ledger. An archived Contact keeps its Journal Items; the entries
  stay posted and stay reconciled.
- **Deleting** is refused by the ledger itself whenever the record is referenced by a posted Journal
  Item, by the platform's own referential rules. A Recycling Model pointed at a record type used in
  posted entries therefore fails on those records rather than silently unbalancing the ledger, and
  the failure surfaces as the platform's own message about a record that is still referenced.
- **Privacy deletion** of a Contact behaves the same way. The wizard lists the Contact and every
  record that mentions it, including its Journal Items when it has any; deleting the Contact while
  posted items reference it is refused. The handling procedure of
  [`workflows.md`](workflows.md) §20 therefore ends, for a customer with a trading history, in
  archiving rather than deletion, and the Privacy Log records exactly that.

**Where the rules live.** The referential protection of posted entries is specified in
[`../general-ledger/business-rules.md`](../general-ledger/business-rules.md); the retention
consequences for a Contact are specified in
[`../contacts-and-organizations/business-rules.md`](../contacts-and-organizations/business-rules.md).

## 3. Currency

The domain holds no monetary amount, so it performs no currency conversion and consults no rate. Two
near-misses are worth stating explicitly so that a rebuild does not invent a conversion:

1. **The metered balance** is a count of service units. It is formatted with the service's unit name
   — for example `143 Credits` — and is never a currency amount. It is not rounded to a currency
   precision but to four decimal places, or to a whole number for services that count in whole units
   ([`calculations.md`](calculations.md) §4.1).
2. **The import's currency-symbol stripping** recognises a currency symbol only in order to remove
   it from a number before parsing ([`calculations.md`](calculations.md) §5.9). It does not record
   the currency, and the value is stored on the target field exactly as the target domain would
   store a typed value. If a monetary column of an imported file is in a currency other than the
   record's, the value is loaded as if it were in the record's currency; the correction belongs to
   the importing user, not to the import.

The rules for converting between currencies, when another domain needs them, are in
[`../multi-currency/`](../multi-currency/).

## 4. Analytic distribution

The domain writes no analytic line. An Automation Rule may of course carry an action that writes an
analytic distribution onto a record of another domain; the meaning of that distribution and the
lines it produces are specified in [`../analytic-accounting/`](../analytic-accounting/).

## 5. Taxes

The domain computes no tax and holds no tax field. The nearest contact point is the enrichment of a
company, which may return a tax registration number; that number is validated and, when it does not
pass, silently emptied ([`calculations.md`](calculations.md) §10.3). A tax registration number is an
identifier, not a tax; the tax consequences of a registration are specified in
[`../taxes/`](../taxes/).

## 6. Audit trail obligations this domain carries

Although it posts nothing, the domain holds three records that an auditor will ask for, and a
rebuild must keep them:

| Record | What it proves |
|---|---|
| The webhook call log | That an outside system asked for a change, when, and with what payload. Written only when the rule asks for logging; each line carries the rule identifier as its origin so that the whole history of one rule can be read at once. |
| The message thread on an Automation Rule | Who changed the rule's name, watched record type, trigger, delay, delay mode, delay unit or trigger date field, and when. Those are the tracked fields. |
| The Privacy Log | That a person's data was found and what was done with it, with the name and the address masked so that the proof does not itself hold the data. One log per handling session. |

None of these is a ledger record, and none of them is reconciled against anything. They are retained
under the retention rules of the installation, described in
[`../../overview/security-model.md`](../../overview/security-model.md).
