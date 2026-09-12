# Accounting effects

## The reasoned statement: this domain posts nothing

**The fleet domain creates no journal entry and no journal item of its own.** No operation in this folder — registering a vehicle, assigning a driver, recording an odometer reading, recording a service, recording a contract, cancelling a contract, archiving a vehicle, sending a message to a driver, releasing a vehicle at a departure, or deriving either analysis result set — writes anything to the ledger.

That is a deliberate design, and the reasoning is worth stating because it is the single most important fact a rebuild has to get right about this domain:

1. **The monetary fields of this domain are records of fact, not postings.** The cost of a Vehicle Service, the activation cost of a Vehicle Contract and its recurring cost are figures a fleet officer records so that the fleet can be analysed. None of them is an accounting document. A contract with a recurring cost of four hundred a month does not create four hundred of expense every month; it states what the agreement charges, so that the Fleet Analysis Report can spread it across months. The actual expense reaches the ledger when the supplier's bill arrives and is entered as a vendor bill, which belongs to [accounts payable](../accounts-payable/).
2. **The catalogue value, purchase value and residual value of a Vehicle are equally records of fact.** They are not an asset register. Nothing in this domain depreciates a vehicle, capitalises it, or creates an asset. A company that wants a vehicle on its balance sheet records the acquisition as a vendor bill and manages the asset through the general ledger's own asset handling; the vehicle record here is the operational counterpart, not the accounting one.
3. **The direction of the one bridge that exists is from the ledger into the fleet, not the other way.** The accounting bridge does not turn a Vehicle Service into a journal entry. It turns a *posted journal item* into a Vehicle Service. The ledger is the source and the fleet is the consumer.

**The consequence for a rebuild.** A rebuild that implements this domain need implement no posting logic at all. What it must implement is the vehicle dimension on journal items, the creation of services from posted vendor bill lines, the protection of those services, and the carrying of the dimension through the accrual assistant. Those four things are specified below.

**A double-entry test.** Every event of this domain leaves the trial balance unchanged. Creating a contract with a recurring cost of four hundred changes no account balance. Archiving a vehicle changes no account balance. Deriving the Fleet Analysis Report changes no account balance. Only posting a vendor bill does, and that posting is the general ledger's own.

---

## Part one: the vehicle dimension on journal items

### 1.1 What the dimension is

The accounting bridge adds one reference to the Journal Item: `vehicle_id`, the Vehicle, indexed with an index that skips empty values. It is a plain analytic-style dimension: it carries no amount, no account and no rule. Its only purposes are to let the ledger be read by vehicle and to let the fleet consume the ledger.

Two companion fields accompany it and are specified in [entities.md](entities.md) section 15: `need_vehicle`, a flag that always yields false and exists as an extension point for localizations that must make the dimension mandatory on particular accounts; and `vehicle_log_service_ids`, the back reference to the generated Vehicle Service.

### 1.2 Where the dimension is offered

| Screen | Behaviour |
|---|---|
| The item lines of a Journal Entry form | Shown as an optional, hidden-by-default column immediately after the account column. The column is entirely hidden unless the entry's kind is a vendor bill, a vendor credit note or a vendor receipt. It is required on a line whose vehicle-required flag is true and whose entry is a vendor bill or a vendor credit note; the shipped rule never makes the flag true. |
| The invoice lines of a Journal Entry form | The same, with the same conditions. |
| The general Journal Item list | Shown as an optional, hidden-by-default column immediately after the label column, on entries of every kind. |
| The Vehicle form | A counter button labelled "Bills" appears when the count is greater than zero and opens the purchase entries of the vehicle; the count is derived by [calculations.md](calculations.md), C-23. |

### 1.3 What the dimension does not do

- It does not select an account. The account of a bill line is chosen by the ordinary rules of [accounts payable](../accounts-payable/) from the product, the fiscal position and the vendor.
- It does not change any amount, any tax or any currency conversion.
- It does not create an analytic distribution. The analytic dimension and the vehicle dimension are independent; see part four for the extension point that connects them in a localization.
- It does not restrict the entry. A journal item may name a vehicle of any company, and no company consistency check is declared on the reference.

---

## Part two: the ledger entry that triggers a Vehicle Service

The single event that connects the ledger to this domain is the **first posting of a vendor bill that carries a vehicle on a product line**. The journal entry itself is produced entirely by [accounts payable](../accounts-payable/) and [general ledger](../general-ledger/); it is itemised here so that a reader of this folder can see exactly which item feeds the fleet, and so that the amount the fleet takes is unambiguous.

### 2.1 The items of the triggering entry

Consider a vendor bill for one repair, entered against a vendor, with one product line naming a vehicle, one tax and one payment term. Its items are:

| # | Item | Journal | Account selection rule | Debit or credit | Amount formula | Currency and rate | Date | Counterparty | Analytic distribution | Tax treatment | Reconciled against |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | The expense line | The purchase journal of the bill | The expense account of the product, adjusted by the fiscal position of the vendor; when the product names none, the default purchase expense account of the company | **Debit** | quantity × unit price × (1 − discount ÷ 100), converted to the company currency | The document currency of the bill, with the amount also recorded in that currency; the rate is the one in force on the bill's accounting date | The accounting date of the bill | The vendor | Taken from the analytic distribution model that matches the line, when one applies | Carries the taxes named on the line; its amount is the untaxed base | Not reconciled |
| 2 | The tax line, one per tax repartition line | The same purchase journal | The account named on the tax's repartition line | **Debit** for a deductible input tax | the untaxed base of item 1 × the tax rate, rounded by the rule of [taxes](../taxes/) | As item 1 | As item 1 | The vendor | Inherited from the base line when the repartition line says so | Is itself the tax | Not reconciled |
| 3 | The payable line | The same purchase journal | The payable account of the vendor | **Credit** | the sum of items 1 and 2 | As item 1 | As item 1 | The vendor | None | None | Reconciled against the eventual payment, by [payments and bank reconciliation](../payments-and-bank-reconciliation/) |

**Only item 1 feeds the fleet.** Items 2 and 3 are skipped, item 2 because its display kind is not a product line and item 3 for the same reason. The skip conditions are enumerated in [business-rules.md](business-rules.md), rule FLT-041.

### 2.2 What the fleet takes from item 1

| What the service records | Where it comes from |
|---|---|
| Service kind | The shipped Fleet Service Type named "Vendor Bill", of the service category |
| Vehicle | The vehicle named on item 1 |
| Vendor | The partner of item 1, which is the bill's vendor |
| Description | The label of item 1 |
| Journal item link | Item 1 itself |
| Cost | The **debit** of item 1, in the company currency |
| Date | Today, the day of posting — **not** the bill's accounting date; see compatibility finding FLT-C23 in [business-rules.md](business-rules.md) |
| Progress stage | `new` |
| Company | The current company at the moment of posting |

**The amount taken is the company-currency debit.** That matters in three ways:

1. **It is net of tax on a fully deductible tax.** The debit of item 1 is the untaxed base, so a fleet cost report built on it reports costs excluding recoverable tax.
2. **It includes non-deductible tax when the tax is non-deductible.** A tax whose repartition sends part of the tax to the expense account raises the debit of item 1, and the fleet cost follows. That is correct: a non-recoverable tax is a real fleet cost.
3. **It is already converted.** On a bill in a currency other than the company's, the debit is the converted amount at the rate of the bill's accounting date. This domain performs no conversion. The worked arithmetic is in [calculations.md](calculations.md), C-22, worked example 2.

### 2.3 The message logged

One message is posted to the discussion thread of each new service, whose body is "Service Vendor Bill: %s", with the single placeholder replaced by a link to the journal entry, rendered as the entry's name. The message is the audit trail that connects the fleet record to the ledger record; a rebuild must reproduce it because support procedures use it to find the bill from the service.

### 2.4 What happens on a vendor credit note

A vendor credit note may carry the vehicle dimension on its lines — the column is offered — but posting one creates **no** Vehicle Service, because skip condition three of rule FLT-041 restricts the creation to vendor bills. A credit note that reverses a fleet cost therefore reduces the ledger but leaves the fleet's own cost record untouched.

**Compatibility finding note.** This is recorded as an observation rather than a defect. Creating a service with a negative cost from a credit note would make the Fleet Analysis Report net out, which is arguably what a reader wants; but it would also create a service record for a document on which no work was done, and the shipped behaviour is the more conservative one. A rebuild that changes it must say so, because the fleet cost totals would change for every installation that issues credit notes against vehicle costs.

### 2.5 Idempotence and reversal

| Event | Ledger effect | Fleet effect |
|---|---|---|
| The bill is posted for the first time | Items 1, 2 and 3 are created and posted | One Vehicle Service is created per qualifying item |
| The bill is reset to draft | The items return to draft; nothing is removed | Nothing. The service stays, and its state mirror shows the entry as a draft |
| The amount of item 1 is changed and the bill is posted again | Item 1 carries the new amount | No second service is created. The existing service's cost follows the new debit |
| The vehicle on item 1 is changed and the bill is posted again | Item 1 carries the new vehicle | The existing service follows the new vehicle; no second service is created |
| The vehicle on item 1 is cleared | Item 1 carries no vehicle | The service is **deleted**, with the deletion guard bypassed |
| Item 1 is deleted | The item is removed | The service is **deleted**, with the deletion guard bypassed |
| The bill is cancelled | The entry's state becomes cancelled | Nothing. The service stays and continues to appear in the Fleet Analysis Report, while the vehicle's bill count drops, because that count excludes cancelled entries. This asymmetry is worth reproducing exactly |
| The bill is reversed by a credit note | A reversing entry is posted | Nothing, because a credit note creates no service and no service is removed |

---

## Part three: carrying the dimension through an accrual period change

The general ledger offers an assistant that moves the recognition of a journal item, or part of it, from one accounting period to another. It is owned by [general ledger](../general-ledger/); this domain changes exactly one thing about it.

### 3.1 What the assistant produces

Given one source journal item, a percentage, a target date, a journal and an accrual account, the assistant produces **two** journal entries:

| Entry | Date | Item | Account | Debit or credit | Amount |
|---|---|---|---|---|---|
| The reversal entry | The source item's own date | 1 | The **source item's account** | The mirror of the source: credit where the source debits, debit where it credits | percentage ÷ 100 × the source item's debit, and the same of its credit, each rounded to the company currency's decimal places |
| The reversal entry | The source item's own date | 2 | The accrual account, which is the expense accrual account for an expense item and the revenue accrual account for a revenue item | The mirror of item 1 | The same amounts, mirrored |
| The recognition entry | The target date | 3 | The **source item's account** | The same direction as the source | The same amounts as item 1, in the source's direction |
| The recognition entry | The target date | 4 | The accrual account | The mirror of item 3 | The same amounts, mirrored |

Every one of the four items carries the source item's currency, the source item's amount in that currency scaled by the same percentage and rounded to that currency's decimal places, the source item's partner, and the source item's analytic distribution. Their labels are produced by the assistant's own cut-off label rule.

### 3.2 What this domain adds

When the source journal item names a vehicle, every generated item **whose account equals the source item's account** also names that vehicle. In the table above that is items 1 and 3. Items 2 and 4, which sit on the accrual account, do **not** receive the vehicle.

```formula
generated item carries the vehicle  when the generated item's account = the source item's account
                                    and the source item names a vehicle
```

**Why the accrual items are excluded.** The accrual account is a balance-sheet holding account. Attributing a vehicle to it would make a fleet cost report count the same cost twice — once on the expense account and once on the holding account — for every period a cost is deferred across.

**Consequence.** After a period change, the expense continues to be attributable to the vehicle in both periods: negatively in the original period and positively in the target period. A report that sums the vehicle dimension across all periods therefore still reports the original amount once.

### 3.3 Worked example

A vendor bill dated 1 September 2026 carries one expense item of 100.00 in the company currency, on the expense account, naming a vehicle. Sixty per cent of it is moved to 10 September 2026 through the assistant, using a miscellaneous journal and an expense accrual account.

| Entry | Date | Account | Debit | Credit | Vehicle |
|---|---|---|---|---|---|
| Reversal | 2026-09-01 | Expense account | — | 60.00 | **the vehicle** |
| Reversal | 2026-09-01 | Expense accrual account | 60.00 | — | none |
| Recognition | 2026-09-10 | Expense account | 60.00 | — | **the vehicle** |
| Recognition | 2026-09-10 | Expense accrual account | — | 60.00 | none |

The vehicle's total on the expense account is 100.00 − 60.00 + 60.00 = 100.00, unchanged. Its total in the September period before the tenth is 40.00, and 60.00 from the tenth onwards.

**No Vehicle Service is created by any of these four items**, for two reasons: none of them is on an entry whose kind is a vendor bill — they are miscellaneous entries — and their display kind is not a product line.

---

## Part four: the analytic extension point

The Vehicle publishes one rule whose sole purpose is to give a localization a stable name for a vehicle when the vehicle is used as an analytic dimension:

```formula
analytic name of a vehicle = the licence plate of the vehicle
analytic name of a vehicle = the text "No plate"   when the plate is empty
```

Nothing inside this repository calls it. It exists so that a country package which must post fleet costs to a per-vehicle analytic account — several jurisdictions require a benefit-in-kind computation per vehicle — has one agreed way to name that account, and so that a payroll package which charges a company vehicle to an employee can name the same account the same way.

**A rebuild must keep it.** Removing an unused rule that other packages override is the one refactoring that breaks a localization silently. The name of the rule and the text it falls back to are both part of the contract.

---

## Part five: ledger effects this domain triggers indirectly, by domain

| Owning domain | The event | The ledger effect | Where it is specified |
|---|---|---|---|
| [accounts payable](../accounts-payable/) | A vendor bill for a repair, a lease instalment, an insurance premium or a fuel purchase is entered and posted, with a vehicle named on one or more product lines | The three items of section 2.1 | [accounts payable](../accounts-payable/) |
| [general ledger](../general-ledger/) | The period of a fleet expense item is changed through the accrual assistant | The four items of section 3.1, of which two carry the vehicle | [general ledger](../general-ledger/) |
| [payments and bank reconciliation](../payments-and-bank-reconciliation/) | The vendor is paid | A payment entry whose counterpart reconciles the payable item of section 2.1 | [payments and bank reconciliation](../payments-and-bank-reconciliation/) |
| [taxes](../taxes/) | The bill carries value added tax | The tax items of section 2.1, and the tax report lines they feed | [taxes](../taxes/) |
| [multi-currency](../multi-currency/) | The bill is in a currency other than the company's | The conversion of every item to the company currency, and the exchange difference recognised at payment | [multi-currency](../multi-currency/) |
| [analytic accounting](../analytic-accounting/) | An analytic distribution model matches a fleet expense line | Analytic lines mirroring the expense item | [analytic accounting](../analytic-accounting/) |
| [financial reporting](../financial-reporting/) | A reader analyses expenses by vehicle | No further posting; the vehicle dimension is read from the journal items | [financial reporting](../financial-reporting/) |

## Part six: what a rebuild must guarantee

1. **No posting.** No operation of this domain writes to the ledger.
2. **The dimension survives.** The vehicle named on a journal item is preserved through every copy the ledger makes of that item within the accrual assistant, on the items that keep the same account.
3. **One service per qualifying item.** Posting a vendor bill creates exactly one Vehicle Service per product item that names a vehicle and does not already have one, and posting the same bill again creates none.
4. **The cost equals the debit.** The cost of a service that carries a journal item always equals that item's debit in the company currency, at every moment, and no path lets a user set it otherwise.
5. **Deleting the ledger record deletes the fleet record.** Clearing the vehicle on an item, or deleting the item, deletes the service. The guard that otherwise forbids deleting such a service is bypassed on exactly those two paths and on no other.
6. **The fleet analysis is not the ledger.** The Fleet Analysis Report includes contract costs that were never posted, dates services on their posting day rather than their accounting date, and multiplies its contract amounts when several kinds of contract overlap. It is an operational report and must never be presented as an accounting one. A rebuild should say so on the screen.
