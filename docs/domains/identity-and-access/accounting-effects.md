# Identity and Access — Accounting Effects

**This domain produces no journal entries.**

No entity of the identity-and-access domain is a financial document, none of them carries an
amount, a currency, an account, a tax or an analytic distribution, and no operation described in
[workflows.md](workflows.md) posts, reverses or reconciles anything. Signing in, switching company,
granting a group, creating a record rule, issuing an application key, registering a passkey,
granting customer-facing access, running a privacy search and recycling stale records all leave the
general ledger untouched.

What the domain does instead is decide, for every financial operation performed anywhere else in the
system, **who may perform it** and **on which records**. Those decisions have consequences for the
ledger that are entirely indirect but entirely real, and they are enumerated below so that a
re-implementation understands what it is responsible for.

---

## 1. How this domain reaches the ledger

### 1.1 Through the company on a document

Every financial document carries a company, and that company is what determines the chart of
accounts, the journals, the sequences, the currency and the tax regime that apply to it. The company
of a new document is, by default, the **current company** of the environment that creates it — that
is, the first element of the active-company list validated by section 8.2 of
[business-rules.md](business-rules.md).

Consequences:

| Mechanism specified here | Consequence in the ledger |
|---|---|
| The active-company list and its validation | A document created while company *Beta* is current is posted in *Beta*'s journals, numbered from *Beta*'s sequences, and denominated in *Beta*'s currency. |
| The company-consistency check (section 7 of [business-rules.md](business-rules.md)) | A journal item cannot point at an account of another company; a payment cannot point at a journal of another company. The refusal is *Uh-oh! You've got some company inconsistencies here:* followed by the offending lines. This is the guard that prevents a chart of accounts from being silently mixed. |
| The company tree and the root-delegated currency | Every branch of a company tree carries the same currency as its root, so consolidation across a tree never has to convert. |
| The archival guard on a company | A company that is still the default company of an active account cannot be archived, which prevents orphaning the documents that would otherwise be created against it. |

### 1.2 Through record rules on financial entities

Every financial domain ships record rules written in the language specified in section 3 of
[business-rules.md](business-rules.md). The canonical one, present on essentially every financial
entity, is the multi-company rule: *the record's company is one of the active companies* (often with
"or the record has no company"). Because it is a **global** rule, it is combined by intersection and
therefore cannot be widened by any group.

The practical consequence for the ledger is that the trial balance, the general ledger and every
statutory report are naturally scoped: the report reads the journal items the acting user may read,
and that set is exactly the set the multi-company rule leaves visible.

### 1.3 Through access rights on financial entities

The accounting groups — the ones that distinguish an invoicing clerk from an accountant from a
financial administrator — are instances of the group model specified here, and the permissions they
carry are access-right rows of the shape specified in section 8 of
[entities.md](entities.md). Posting a journal entry, reversing it, changing a lock date, running a
tax closing: each is gated by a group, and the gate is the same four-flag row.

### 1.4 Through field-level restriction on financial fields

Several financial fields are restricted to a group — for example the fields that only appear for
holders of the multi-currency group or the analytic group. The restriction is the field-level
mechanism of section 6 of [business-rules.md](business-rules.md), and the refusal is the message
given there.

### 1.5 Through privilege elevation in financial code

Financial code elevates in well-defined places: to read a company's settings while acting for
another company, to create the counterpart entry of an inter-company operation, to post an entry
that the triggering user could not have posted by hand. Because elevation does **not** suspend
constraints, an elevated posting is still balanced, still numbered and still consistent; what it
suspends is only the permission question.

---

## 2. What the recycling and privacy mechanisms must not do

Two mechanisms of this domain delete or archive arbitrary records with elevation, and both can in
principle be pointed at financial entities. The rules that protect the ledger are not in this
domain — they are the ordinary deletion guards of the financial entities themselves — but the
interaction must be understood:

| Mechanism | Interaction |
|---|---|
| A recycling rule with the action *Delete* pointed at a posted journal entry | The deletion is attempted with elevation, so no permission stops it; the financial domain's own guard (a posted entry cannot be deleted) raises, the candidate stays and the batch continues. |
| A recycling rule with the action *Archive* pointed at an entity with no archive flag | Refused at configuration time: *This model doesn't manage archived records. Only deletion is possible.* |
| A personal-data search deleting a contact referenced by an invoice | The deletion raises because of the referential constraint; the line keeps its state and the operator sees the failure. |
| A personal-data search archiving a contact | Permitted; the contact disappears from pickers but every posted document keeps its reference. |
| The account deletion queue deleting a contact referenced by a document | Expected to fail; the failure is logged as a warning and the contact survives while the account is gone. |

---

## 3. Where to look instead

| For | See |
|---|---|
| The journal entries themselves | [../general-ledger/accounting-effects.md](../general-ledger/accounting-effects.md) |
| The multi-company behaviour of financial documents | [../general-ledger/entities.md](../general-ledger/entities.md) and [../multi-currency/calculations.md](../multi-currency/calculations.md) |
| The accounting security groups and their rights | [../general-ledger/configuration.md](../general-ledger/configuration.md) |
| The customer-facing invoice pages that use the token mechanism specified here | [../accounts-receivable/interfaces.md](../accounts-receivable/interfaces.md) |
