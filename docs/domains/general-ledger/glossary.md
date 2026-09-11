# General Ledger — Glossary

Every term used in this domain, defined in full. Terms are listed alphabetically. Where a term has a storage name that other domains depend on, the name is given in code font.

---

**Account** (`account.account`) — One line of the chart of accounts: a code (different per company), a name, a classification called the account type, and behavioral flags. Journal Items are booked on accounts; every report is an aggregation of Journal Items grouped by account.

**Account code** (`code`) — The identifier of an account inside one company. It is not a plain column: the physical storage is a map from company to code, looked up through the root company, so that one account record can carry a different code in a parent company and in a subsidiary. Only letters, digits and dots are accepted, at most sixty-four characters.

**Account Group** (`account.group`) — A named range of account-code prefixes used to build the hierarchy shown in the chart of accounts and in the hierarchical mode of the financial reports. Two groups of the same company with prefixes of the same length may not overlap.

**Account Root** (`account.root`) — The first two characters of an account code, exposed as a facet so that the chart of accounts and the journal-item list can be filtered by broad category. It is computed, never stored.

**Account Tag** (`account.account.tag`) — A free label attachable to accounts, to taxes (where it acts as a tax grid) or to products. Only the first applicability belongs to this domain.

**Account type** (`account_type`) — The classification of an account into one of nineteen values. It decides the internal group, whether the balance is carried forward across fiscal years, whether reconciliation is allowed, and the behavior at closing.

**Accountable item** — A Journal Item whose display type is not a section, a subsection or a note. Only accountable items carry an account and an amount.

**Accounting date** (`date`) — The date at which an entry enters the ledger. It is the date used by every report and by every lock check. It is distinct from the document date of an invoice or a bill, which is the date printed on the paper.

**Accounting date rule** — The algorithm that moves a candidate date forward so that it leaves no closed period and so that a document recorded in the past still gets an increasing number. Fully specified in `calculations.md`.

**Adjusting entry** — One of the two entries produced by the change-of-period action of the automatic transfer wizard: the destination entry recognises an amount in the new period and the cancelling entry removes it from the old one.

**Amount in currency** (`amount_currency`) — The amount of a Journal Item expressed in the currency of that item. When the item currency is the company currency the value equals the balance.

**Audit trail** — The message thread attached to a Journal Entry, recording every tracked field change, every creation, modification and deletion of an item after the entry has been posted once, and the operational messages of the domain. When the company keeps a *restrictive* audit trail, those messages may not be deleted and a posted entry may never be deleted.

**Automatic posting** (`auto_post`) — The mode that makes a scheduled job post an entry on its accounting date. Five values: no, at date, monthly, quarterly, yearly. The three periodic values additionally create the next occurrence at posting time.

**Automatic sequence** (`sequence.mixin`) — The shared numbering behavior adopted by Journal Entries: a prefix, a period, a counter, a locking discipline that guarantees uniqueness under concurrency, and a gap detection.

**Balance** (`balance`) — The signed amount of a Journal Item in the company currency: positive for a debit, negative for a credit. It is the field actually stored and written; the debit and the credit columns are derived from it.

**Balance invariant** — The rule that the sum of the balances of the items of an entry, rounded to the company currency, is zero. Checked around every write, every deletion and every reconciliation.

**Balancing item** — The item on the current-year-earnings account that the opening-entry mechanism adjusts so that the opening entry always balances. Labelled "Automatic Balancing Line".

**Cancelling reversal** — A reversal that is posted immediately and reconciled with the original, so that the pair nets to zero. Used when undoing an entry that may not be deleted.

**Cash-basis entry** — A Journal Entry created at reconciliation time to recognise the taxes of a document whose exigibility is deferred to the payment. Hosted by this domain, specified in `../taxes/`.

**Chain** — see *Numbering chain*.

**Chart of accounts** — The whole set of accounts of a company, ordered by code.

**Chart template** (`account.chart.template`) — A named, country-specific bundle of accounts, groups, taxes, tax groups, fiscal positions, journals, reconciliation models and company settings, loaded into a company in one operation.

**Commercial entity** (`commercial_partner_id`) — The top-most company in the hierarchy of a counterpart. Receivable and payable items are booked on the commercial entity rather than on the individual contact.

**Company currency** — The currency of the company of an entry. The balance invariant, the debit, the credit and every "signed" total are expressed in it.

**Counter** (`sequence_number`) — The trailing digit block of a document number, read as an integer. Zero when the number has no digit block.

**Counterpart** — Used in two senses. In reconciliation, the item on the other side of a match. In a document, the partner (customer, supplier, employee) the document is addressed to. This folder always says "the counterpart item" for the first sense.

**Credit** (`credit`) — The derived positive representation of a negative balance. Under storno accounting it is instead the derived negative representation of a positive balance.

**Current year earnings** (`equity_unaffected`) — The account type of the account that absorbs the profit or loss of the current fiscal year. Its balance is **not** carried forward at the fiscal year boundary; the reports reset it each year.

**Cut-off** — A colloquial name for the change-of-period transfer: moving a share of an amount from one accounting period to another through a pair of mirrored entries and an accrual account.

**Debit** (`debit`) — The derived positive representation of a positive balance. Under storno accounting it is instead the derived negative representation of a negative balance.

**Direction sign** (`direction_sign`) — The multiplier that converts a price into a balance and back: +1 for a plain entry and for outbound documents, −1 for inbound documents.

**Display type** (`display_type`) — The classification of a Journal Item: product, cost of goods sold, tax, discount, rounding, payment term, early payment discount, the three non-deductible kinds, section, subsection or note. It decides whether the item is accountable and where it sorts inside the entry.

**Document date** (`invoice_date`) — The date printed on an invoice or read from a received bill, distinct from the accounting date.

**Document type** (`move_type`) — The classification of a Journal Entry into one of seven values: plain entry, customer invoice, customer credit note, vendor bill, vendor credit note, sales receipt, purchase receipt.

**Exchange difference** — The gain or loss produced when two matched items, expressed in different currencies or booked at different rates, do not agree in the company currency after the match. It is recognised by a dedicated entry in the exchange journal.

**Exchange-line mode** — The special case of the reconciliation algorithm in which both items share a currency but at least one of them has nothing left in it; no rate is applied, because the correction must touch only the company-currency amount.

**Fiscal lock date** — The maximum of the effective Global Lock Date and the effective Hard Lock Date, raised to the effective Sales or Purchase Lock Date when the journal is of that type. Used by the deletion and copy rules.

**Fiscal year** — The twelve-month period of a company, defined by the day and the month on which it ends. Computed by the rule of `calculations.md`.

**Full Reconciliation** (`account.full.reconcile`) — The marker created when a matched group nets exactly to zero. It carries no amount; its identifier becomes the matching number of every item of the group.

**Gap** — A missing counter in a numbering chain. Detected by comparing each entry with its immediate neighbours of the same journal, prefix and suffix, and flagged on the entry that opens the hole.

**Global Lock Date** (`fiscalyear_lock_date`) — The lock date that applies to every entry of a company.

**Hard Lock Date** (`hard_lock_date`) — The lock date that applies to every entry, can never be removed, can never be moved backwards, and can never be relaxed by an exception.

**Hash** (`inalterable_hash`) — The value that binds an entry to its predecessor in a chain, making any later modification detectable. Stored as a dollar sign, the hash version, a dollar sign and the hexadecimal digest.

**Hash chain** — The ordered set of hashed entries of one journal and one numbering prefix, each hash computed from the digest of the previous one and the exact content of the entry.

**Inalterability** — The property guaranteed by the hash chain: a posted entry, once hashed, cannot be modified, reset to draft or deleted without the verification detecting it.

**Inbound document** — A document that brings money in or reduces what is owed: a customer invoice, a sales receipt or a vendor credit note. Its direction sign is −1.

**Internal group** (`internal_group`) — The first part of the account type before the underscore: equity, asset, liability, income, expense or off-balance.

**Item** — short for *Journal Item*.

**Journal** (`account.journal`) — A book of entries with its own numbering prefix, default accounts, payment methods and settings. Six types exist: sales, purchase, cash, bank, credit card and miscellaneous.

**Journal Entry** (`account.move`) — One accounting document: a header, a state, a number, a date and a set of Journal Items summing to zero in the company currency.

**Journal Group** (`account.journal.group`) — A named selection of journals expressed by exclusion, offered as a report filter under the label "Ledger".

**Journal Item** (`account.move.line`) — One debit or credit posting of a Journal Entry on one account, optionally in a foreign currency, optionally attached to a counterpart, taxes, tax grids and an analytic distribution.

**Ledger group** — see *Journal Group*.

**Liquidity account** — An account of type Bank and Cash or Credit Card. It is the account that holds the money of a bank, cash or credit-card journal, and it may be matched even when its reconcilable flag is off.

**Lock date** — A date on or before which nothing may be added or modified. Five exist: Global, Sales, Purchase, Tax Return and Hard.

**Lock Exception** (`account.lock_exception`) — A time-limited and optionally user-limited relaxation of one soft lock date, recorded with the company value it relaxed so that the trace is auditable.

**Matching number** (`matching_number`) — The label that identifies a matched group on each of its items: the decimal identifier of the Full Reconciliation when the group is closed, the letter `P` followed by the smallest match identifier of the component while it is only partially matched, or the letter `I` followed by anything for a label imported from another system and not yet resolved.

**Miscellaneous journal** — A journal of type "general", used for entries that are neither sales, nor purchases, nor money movements.

**Non-accountable item** — A Journal Item whose display type is a section, a subsection or a note. It carries no account and no amount.

**Non-trade** (`non_trade`) — A flag on a receivable or payable account that moves it out of the trade receivables or trade payables of the reports.

**Number** (`name`) — The document number of a Journal Entry. The placeholder value is a single slash, which means "no number consumed yet".

**Numbering chain** — The set of entries that share a journal, a numbering prefix and, where the journal splits them, a document family (credit notes versus other documents), a payment nature (payment entries versus other entries) or a counterpart (self-billing). A counter is unique within a chain.

**Numbering grammar** — The five shapes by which a document number is read: year-range monthly, monthly, year range, yearly and fixed. Fully specified in `calculations.md`.

**Off-balance** (`off_balance`) — An account type whose items are kept outside the balance sheet. An entry may not mix off-balance accounts with any other type, and off-balance items may carry no tax and may not be reconciled.

**Onboarding step** — One item of the guided checklist shown on the accounting dashboard.

**Opening entry** — The Journal Entry holding the initial balances of every account of a company. Dated the day before the opening date, kept balanced by an automatic item on the current-year-earnings account.

**Outbound document** — A document that sends money out or increases what is owed: a plain entry, a vendor bill, a purchase receipt or a customer credit note. Its direction sign is +1.

**Outstanding account** — The intermediate account on which a payment sits between its recording and its appearance on a bank transaction. Configured per payment method line of a liquidity journal. Specified in `../payments-and-bank-reconciliation/`.

**Partial Reconciliation** (`account.partial.reconcile`) — One matched amount between exactly one debit item and exactly one credit item. It carries three positive amounts: one in the company currency, one in the currency of the debit item and one in the currency of the credit item.

**Payment status** (`payment_state`) — The derived status of an invoice-like document: not paid, partially paid, in payment, paid, reversed, blocked or the frozen legacy value.

**Period** — In this domain, the span over which a numbering counter runs: a month, a year, a fiscal year or a month of a fiscal year, depending on the deduced periodicity. It is not a stored entity; there is no period table.

**Placeholder number** — The value `/`, meaning that the entry has not yet consumed a number.

**Posted** (`posted`) — The state of an entry that is part of the ledger: numbered, frozen except for a few non-legal fields, included in every report, reconcilable.

**Posted before** (`posted_before`) — The flag set the first time an entry is posted. It governs the numbering-reset rule and the audit-trail deletion rule.

**Prefix** (`sequence_prefix`) — Everything in a document number before the trailing digit block.

**Purchase Lock Date** (`purchase_lock_date`) — The lock date that applies to entries of purchase journals.

**Quick encoding** — A mode in which a document is captured from its total rather than line by line. It relaxes the chain-end guard on deletion and switches off the date-alignment check on the number.

**Reconciliation** — The operation that matches debit items against credit items on the same account, producing Partial Reconciliations and, when the group nets to zero, a Full Reconciliation.

**Reconciliation currency** — The currency in which one match is computed: the debit currency when both sides publish a residual in it, otherwise the credit currency under the same condition, otherwise the company currency.

**Reconciliation plan** — The structured request handed to the reconciliation routine: a list whose members are either sets of items or nested plans. Nested plans are processed first, then the union.

**Reset periodicity** — The frequency at which a numbering counter restarts: never, monthly, yearly, per fiscal year, or per month of a fiscal year. Deduced from the previous number.

**Residual** (`amount_residual`, `amount_residual_currency`) — What is left to match on an item, in the company currency and in the item currency. Zero for an item on an account that neither allows matching nor is a liquidity account.

**Restrictive audit trail** — see *Audit trail*.

**Reversal** — A Journal Entry of the opposite document type, linked to an original, whose amounts undo the original. It may be a plain correction, or a cancelling reversal that is immediately reconciled with the original.

**Sales Lock Date** (`sale_lock_date`) — The lock date that applies to entries of sale journals.

**Section, subsection, note** — Three display types that structure a printed document and carry no accounting value.

**Self-billing journal** — A journal in which the counterpart issues the document on behalf of the company, so that each counterpart gets its own numbering chain.

**Sequence override pattern** (`sequence_override_regex`) — A pattern stored on a journal that replaces the built-in numbering grammar for that journal, defining the named parts first prefix, year, second prefix, month, third prefix, counter and suffix.

**Signed amount** — An amount multiplied by the direction sign of its document, so that inbound and outbound documents accumulate coherently in a report.

**Soft lock date** — Any of the four lock dates that an exception can relax: Global, Sales, Purchase, Tax Return.

**Storno accounting** (`account_storno`) — A convention in which a reversal is booked as a negative amount on the same side as the original rather than as an amount on the opposite side, so that the gross turnover of the account is not inflated. Mandatory in eleven countries and available in four more.

**Suspense account** (`suspense_account_id`) — The account on which a bank transaction is parked until it is matched with a business document.

**Tax grid** (`tax_tag_ids`) — An Account Tag of the tax applicability carried by a Journal Item, telling the tax report which box the item feeds. Specified in `../taxes/`.

**Tax Return Lock Date** (`tax_lock_date`) — The lock date that applies to entries affecting the tax report.

**Transfer entry** — The entry produced by the change-of-account action of the automatic transfer wizard, moving the balance of a set of items to another account and reconciling the source items with their mirrors.

**Trial balance** — The report showing, per account, the total debit, the total credit and the net balance over a period. Specified in `../financial-reporting/`; obtainable from the journal-item pivot grouped by account.

**Unmerge** — The operation that splits an account shared by several companies into one account per company, moving every per-company reference to the matching copy and keeping the codes unchanged.

**Unreconciliation** — The operation that deletes matches, reverses or deletes the entries they produced, and recomputes the residuals and the matching numbers.

**Year range** — A numbering shape whose period is the fiscal year and whose number therefore carries two years, for example `2015-2016` or `15-16`. The two years must be consecutive after truncation to the width of the second one.
