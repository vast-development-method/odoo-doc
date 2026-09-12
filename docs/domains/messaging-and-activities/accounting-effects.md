# Messaging and Activities — Accounting Effects

## This domain produces no journal entries

No operation described in this folder creates, modifies, posts, reverses or deletes a Journal Entry (`account.move`, table `account_move`) or a Journal Item (`account.move.line`, table `account_move_line`). No operation of this folder reads a chart of accounts, selects an account, computes a tax, reconciles, or touches a currency rate.

That is deliberate and structural. The domain is a *transport and record-keeping* layer: it moves text, files, notifications and to-do items between parties, and it records who was told what and when. Nothing it does changes the economic position of the company.

The three things that might look like exceptions are not:

| Apparent exception | Why it is not an accounting effect |
|---|---|
| Postal mail consumes credits with an external printing service | The consumption happens in the external service's account. The platform records only the success or failure of the request on the Postal Letter and its Notification. Whatever billing follows is a separate purchase document created by the party that sells the credits, in the purchasing and payable domains, and is not produced here. |
| Text messages consume credits with an external messaging service | Identical reasoning. The Text Message record holds a state and a failure type; it holds no amount, no currency and no account. |
| A digest indicator may display a monetary amount | The digest **reads** an already-computed figure and formats it for display. It never writes anything, and the figures it reads belong to the domains that own them. |

Consequently there is no journal table, no account-selection rule, no debit-and-credit table and no reconciliation behavior to specify in this folder.

## What this domain does to other domains instead

Although it posts nothing to the ledger, this domain is a hard dependency of almost every accounting and operational flow. The dependency runs in one direction: other domains *call into* this one. The contract they rely on is listed here so that an implementation knows exactly what those callers expect to exist.

### 1. The audit trail of every financial document

Every financial document that carries a conversation — a customer invoice, a vendor bill, a payment, a bank statement, a journal entry — adopts the Thread behavior. Its posting, its validation, its cancellation and its reversal are recorded as Messages, and every change of a tracked field is recorded as a Tracking Value attached to a logged Message.

What the accounting domains expect:

| Expectation | Specified in |
|---|---|
| A change of a field marked as tracked always produces a Tracking Value, in the same transaction, with the old and the new value stored in the column pair matching the field type | [calculations.md](calculations.md), field change tracking |
| The change is attached to a Message that is either a visible message with a subtype, when the document model returns one, or an internal note when it does not | [calculations.md](calculations.md), field change tracking |
| The author of that Message is the acting user, unless the caller explicitly registered another author for the transaction | [calculations.md](calculations.md), field change tracking |
| Tracking Values are readable only by users who may read the field, so a salary field tracked on a document does not leak through the conversation | [business-rules.md](business-rules.md) |
| Tracking is suppressed during a duplication, so copying a document does not fabricate a change history | [entities.md](entities.md), Thread behavior |

The practical consequence for an accounting implementation: the "who changed the due date of this invoice, and when" question is answered entirely by records of this domain, not by anything in the ledger.

### 2. Sending a document to a counterpart

Every "send by email" flow of the receivable and payable domains is a call into the composer of this domain, in its single-record mode, with a Template that renders the document and attaches the generated report.

What those domains expect:

| Expectation | Specified in |
|---|---|
| A Template renders its subject, body, sender, recipients, reply address and scheduled date against the document | [calculations.md](calculations.md), template rendering |
| The reports listed on the Template are generated per document and attached | [entities.md](entities.md), Template |
| The resulting Message is attached to the document and therefore appears in its audit trail | [workflows.md](workflows.md) |
| The recipients are computed by the default-recipient rule, which prefers a contact holding a usable address over a bare address | [calculations.md](calculations.md), default recipients |
| A failure to deliver is visible on the document through the delivery-error counter | [calculations.md](calculations.md), miscellaneous derived values |

The **postal** sending method of the receivable domain is a call into the Postal Letter of this domain: the document is rendered, a Postal Letter is created, and the resulting notification is what the document displays.

### 3. Receiving a document from a counterpart

The vendor-bill intake of the payable domain and the incoming-document flows of other domains are calls into the incoming routing of this domain: an Alias points at the target model, the router creates or updates the record, and the attachment of the received message becomes the source document.

What those domains expect:

| Expectation | Specified in |
|---|---|
| An Alias may name default values applied at creation | [entities.md](entities.md), Alias |
| The record is created **on behalf of** the user matching the sender address, when one exists | [calculations.md](calculations.md), incoming routing, phase H |
| The attachments of the received message are attached to the created record | [calculations.md](calculations.md), incoming routing, phase C |
| A sender that is not allowed is bounced rather than silently dropped | [calculations.md](calculations.md), incoming routing, phase G |
| A flood of incoming messages cannot create an unbounded number of documents | [calculations.md](calculations.md), loop detection |

### 4. Follow-up and dunning

The receivable domain's follow-up process schedules Activities of this domain on overdue documents and on the counterparts themselves, and reads the derived activity indicator to decide what to show in its work list.

What it expects:

| Expectation | Specified in |
|---|---|
| An Activity can be scheduled from business code with a type named by external identifier, a due date, a summary and a note | [entities.md](entities.md), activity behavior |
| The automated flag distinguishes activities the system created from activities a person typed | [entities.md](entities.md), Activity |
| The search, reschedule, complete and remove helpers act only on automated activities by default | [entities.md](entities.md), activity behavior |
| Completing an activity posts a message on the document, so the follow-up leaves a trace in the audit trail | [state-machines.md](state-machines.md), the completion sequence |
| The derived record indicator is searchable and groupable in the database, so a work list of thousands of overdue documents can be sorted by urgency | [calculations.md](calculations.md), the activity state |

### 5. Suppression of unwanted communication

The receivable domain must not send a reminder to an address that has opted out. It relies on the Blacklist Entry and the blacklist behavior of this domain, and on the bounce counter that rises when delivery fails.

| Expectation | Specified in |
|---|---|
| The normalized address of a counterpart is stored and indexed | [entities.md](entities.md), blacklist behavior |
| A suppressed address is detectable in one lookup | [entities.md](entities.md), Blacklist Entry |
| The bounce counter rises on every returned message and is reset when the address proves to work | [calculations.md](calculations.md), bounce detection and the bounce counter |
| Every change of the suppression state is itself an audit-trail entry, because the Blacklist Entry is a thread with two tracked fields | [entities.md](entities.md), Blacklist Entry |

### 6. Multi-company and multi-currency behavior of this domain

| Question | Answer |
|---|---|
| Does any record of this domain hold an amount? | No. No monetary field exists anywhere in the domain. The single currency link that exists, on a Tracking Value, is a *label*: it says which currency to use when displaying an old and a new monetary value that some other domain's field carried. It participates in no arithmetic. |
| Does any record of this domain hold a company? | Yes, but only for routing and branding: the alias domain of a company, the company stamped on a Message at creation, the company of a Digest, the company of a Postal Letter, the company of a Text Message. None of them scopes an amount. |
| Does any record of this domain need a company-currency conversion? | No. |
| Does a company change affect any stored value? | No. A Message keeps the company and the alias domain that were in force when it was created, precisely so that a later reorganization does not rewrite the return path of an already-sent message. |

## Where the boundary lies exactly

An implementation should treat the following as the complete list of points at which an accounting domain and this domain touch. Everything on the left belongs to the accounting domains; everything on the right is specified here.

| Accounting side | Messaging side |
|---|---|
| declares which of its fields are tracked, and with which order | records the change and renders it |
| decides which subtype a given change should raise | selects the followers of that subtype and notifies them |
| supplies a Template and a report | renders, attaches and sends |
| supplies an Alias and default values | routes, creates, checks the sender and bounces |
| schedules and completes Activities | stores them, derives their state, chains them and logs their completion |
| reads the delivery-error counter to warn the user | maintains it from the Notification statuses |
| reads the suppression list before sending | maintains the suppression list and the bounce counters |

No value crosses this boundary in the other direction. In particular, **nothing in this folder may be implemented as a hook that writes to a ledger**, and an implementation that finds itself needing one has mislocated a rule.

### 7. Postal mail as a sending method of an accounting document

One capability of this domain exists only to bridge into the receivable ledger's documents: it adds "by post" as a sending method of a customer invoice and of a follow-up report. What it adds is entirely on this side of the boundary.

| What the bridge adds | Where it is specified |
|---|---|
| The sending method "by Post" on a Contact, alongside the electronic-mail method | [entities.md](entities.md), section 46 |
| A Postal Letter created from the invoice or the follow-up report, with the report to render | [workflows.md](workflows.md), section 21 |
| The Notification of the postal channel that the document then displays | [state-machines.md](state-machines.md), section 8 |
| The estimate call that shows how many stamps a batch will consume before anything is sent | [workflows.md](workflows.md), section 21 |

The bridge creates **no** Journal Entry and **no** Journal Item. The stamps it consumes are a balance held by the external printing service; whatever purchase document eventually records the purchase of those stamps is created by the party that sells them, in the purchasing and payable folders, and never by this one.

### 8. What a reader must check in a rebuild

An implementation of this folder is correct with respect to the ledger when all of the following hold.

1. No operation of the folder opens, writes, posts, reverses or deletes a Journal Entry or a Journal Item.
2. No record of the folder carries a monetary amount. The single currency link, on a Tracking Value, is a label and participates in no arithmetic.
3. Every figure a Digest displays is read from the domain that owns it and is formatted for display only.
4. Every credit consumption — text messages, postal mail, contact enrichment — is recorded as a state and a failure type on the record that requested it, and never as an amount.
5. Every accounting flow that calls into this folder does so through one of the seven contact points of the table above, and no value crosses the boundary in the other direction.

---

## Reconciliation notes

1. **Scope of the statement.** Both source versions agree that the domain posts nothing to the ledger. One of them stopped at that statement; the other listed the contact points with the accounting folders. This document keeps the statement, the contact points, the boundary table and the postal bridge, so a reader can verify the claim rather than take it on trust.
2. **The currency on a Tracking Value.** One version listed the currency link among the domain's fields without saying what it is for. It is a rendering label for a monetary value that some other domain's field carried; section 6 says so explicitly.
