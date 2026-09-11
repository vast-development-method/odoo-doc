# Acceptance scenario index

Every numbered scenario across the specification: 1,324 scenarios in 7 domains, of which 393 assert concrete monetary amounts.

A scenario is referenced as the domain name, a full stop, and its number, for example `sales.A1`. That reference never changes and never gets reused, so a failing test in a rebuild can always be traced back to what it asserts. The rules are in [traceability rules](../reimplementation/traceability-rules.md).

## Scenarios by domain

| Domain | Scenarios | With amounts |
|---|---|---|
| [accounts-payable](../domains/accounts-payable/acceptance-criteria.md) | 134 | 39 |
| [accounts-receivable](../domains/accounts-receivable/acceptance-criteria.md) | 164 | 61 |
| [general-ledger](../domains/general-ledger/acceptance-criteria.md) | 251 | 45 |
| [inventory-valuation-and-costing](../domains/inventory-valuation-and-costing/acceptance-criteria.md) | 206 | 111 |
| [products-and-catalog](../domains/products-and-catalog/acceptance-criteria.md) | 216 | 29 |
| [purchasing](../domains/purchasing/acceptance-criteria.md) | 170 | 47 |
| [sales](../domains/sales/acceptance-criteria.md) | 183 | 61 |

## accounts-payable

| Reference | Group | Scenario | Exercises |
|---|---|---|---|
| `accounts-payable.A1` | A — Capturing a bill | A bill of two expense lines with a purchase tax | workflows, calculations, accounting effects, state machines |
| `accounts-payable.A2` | A — Capturing a bill | Posting refuses a bill without a bill date | workflows, business rules |
| `accounts-payable.A3` | A — Capturing a bill | Posting refuses a bill without a vendor | workflows, business rules |
| `accounts-payable.A4` | A — Capturing a bill | Posting refuses a negative total | workflows, calculations, accounting effects, business rules, state machines |
| `accounts-payable.A5` | A — Capturing a bill | Posting refuses an empty document | workflows, business rules, state machines |
| `accounts-payable.A6` | A — Capturing a bill | A purchase document cannot live in a sale journal | accounting effects, business rules, state machines |
| `accounts-payable.A7` | A — Capturing a bill | A receivable account cannot be used on a bill | workflows, accounting effects, business rules, state machines |
| `accounts-payable.A8` | A — Capturing a bill | A payable account must carry a maturity date | accounting effects, business rules, state machines |
| `accounts-payable.B1` | B — Dates | A late bill lands in its own month | workflows, accounting effects |
| `accounts-payable.B2` | B — Dates | A bill of the current month keeps today | accounting effects |
| `accounts-payable.B3` | B — Dates | A future-dated bill keeps its own date | accounting effects |
| `accounts-payable.B4` | B — Dates | A locked period pushes the accounting date forward | workflows, accounting effects, state machines |
| `accounts-payable.B5` | B — Dates | A yearly series pushes to 31 December | accounting effects |
| `accounts-payable.C1` | C — Payment terms | Sixty days from end of month | calculations, accounting effects |
| `accounts-payable.C2` | C — Payment terms | Two instalments, thirty per cent then the balance | workflows, calculations |
| `accounts-payable.C3` | C — Payment terms | The last line absorbs the rounding drift | calculations |
| `accounts-payable.C4` | C — Payment terms | A fixed line followed by a balance line | workflows, calculations |
| `accounts-payable.C5` | C — Payment terms | A term whose percentages do not sum to 100 is rejected | calculations, business rules |
| `accounts-payable.C6` | C — Payment terms | The fourth delay type | calculations |
| `accounts-payable.C7` | C — Payment terms | Early payment discount, mode *On early payment* | calculations |
| `accounts-payable.C8` | C — Payment terms | Early payment discount, mode *Never* | calculations |
| `accounts-payable.C9` | C — Payment terms | The early discount refuses a multi-line term | calculations, business rules |
| `accounts-payable.D1` | D — Duplicate detection | An exact duplicate, red warning | workflows, calculations, state machines |
| `accounts-payable.D2` | D — Duplicate detection | A probable duplicate, amber warning | calculations, state machines |
| `accounts-payable.D3` | D — Duplicate detection | A different calendar year is not a duplicate |  |
| `accounts-payable.D4` | D — Duplicate detection | A different currency is not a duplicate | state machines |
| `accounts-payable.D5` | D — Duplicate detection | A cancelled document is not a duplicate | workflows, state machines |
| `accounts-payable.D6` | D — Duplicate detection | Three copies detect each other | state machines |
| `accounts-payable.D7` | D — Duplicate detection | Deleting the duplicates | state machines |
| `accounts-payable.D8` | D — Duplicate detection | A document being typed is compared before it is saved | workflows, calculations, state machines |
| `accounts-payable.E1` | E — The reviewed flag and mass posting | Posting marks a bill reviewed | workflows, accounting effects, state machines |
| `accounts-payable.E2` | E — The reviewed flag and mass posting | The to-review queue | workflows, calculations, accounting effects, state machines |
| `accounts-payable.E3` | E — The reviewed flag and mass posting | Mass posting schedules future-dated documents | workflows, accounting effects, state machines |
| `accounts-payable.E4` | E — The reviewed flag and mass posting | Mass posting with Force | workflows, state machines |
| `accounts-payable.E5` | E — The reviewed flag and mass posting | Mass posting skips hashing journals | workflows, accounting effects, state machines |
| `accounts-payable.E6` | E — The reviewed flag and mass posting | Mass posting with nothing to do | workflows, accounting effects, business rules, state machines |
| `accounts-payable.E7` | E — The reviewed flag and mass posting | Silencing abnormal warnings from the dialogue | workflows, calculations, state machines |
| `accounts-payable.F1` | F — Automatic posting | A vendor set to Always is posted automatically | workflows, calculations, accounting effects, state machines |
| `accounts-payable.F2` | F — Automatic posting | A duplicate blocks automatic posting | workflows, state machines |
| `accounts-payable.F3` | F — Automatic posting | An abnormal amount blocks automatic posting | workflows, calculations, state machines |
| `accounts-payable.F4` | F — Automatic posting | The learning wizard opens after three untouched bills | workflows, state machines |
| `accounts-payable.F5` | F — Automatic posting | The learning wizard does not open below the threshold | workflows, state machines |
| `accounts-payable.F6` | F — Automatic posting | The learning wizard does not open for a manually typed bill | workflows, state machines |
| `accounts-payable.F7` | F — Automatic posting | The scheduled job posts due scheduled bills | workflows, accounting effects, business rules, state machines |
| `accounts-payable.G1` | G — Reversal and refund | A refund of a partly paid bill | workflows, calculations, accounting effects, state machines |
| `accounts-payable.G2` | G — Reversal and refund | A cancelling reversal of a partly paid bill unwinds the payment | workflows, accounting effects, state machines |
| `accounts-payable.G3` | G — Reversal and refund | A full reversal of an unpaid bill | workflows, accounting effects, state machines |
| `accounts-payable.G4` | G — Reversal and refund | The reversal journal must match | workflows, accounting effects, business rules, state machines |
| `accounts-payable.G5` | G — Reversal and refund | A future-dated reversal is scheduled | workflows, state machines |
| `accounts-payable.G6` | G — Reversal and refund | The payment term is cleared except in the mixed discount mode | workflows, calculations, accounting effects, state machines |
| `accounts-payable.H1` | H — Cheque printing | Amount in words for one thousand two hundred thirty-four point five six | calculations, state machines |
| `accounts-payable.H2` | H — Cheque printing | Amount in words with no cents | calculations |
| `accounts-payable.H3` | H — Cheque printing | The document-level words strip commas | calculations |
| `accounts-payable.H4` | H — Cheque printing | Manual numbering consumes the sequence at posting | workflows, accounting effects, state machines |
| `accounts-payable.H5` | H — Cheque printing | Ten bills paid by one grouped cheque | workflows, calculations, accounting effects, state machines |
| `accounts-payable.H6` | H — Cheque printing | Ten bills paid by ten separate cheques | calculations, accounting effects |
| `accounts-payable.H7` | H — Cheque printing | Pre-printed stationery renumbers on printing | workflows, accounting effects, state machines |
| `accounts-payable.H8` | H — Cheque printing | A non-numeric cheque number is refused | business rules |
| `accounts-payable.H9` | H — Cheque printing | A duplicate cheque number is refused | workflows, accounting effects, business rules, state machines |
| `accounts-payable.H10` | H — Cheque printing | Printing refuses a mixed selection | accounting effects, business rules |
| `accounts-payable.H11` | H — Cheque printing | Printing refuses when no layout is configured | accounting effects, business rules |
| `accounts-payable.H12` | H — Cheque printing | Unmark sent puts the cheque back in the queue | accounting effects, state machines |
| `accounts-payable.H13` | H — Cheque printing | Void cancels the payment and releases the bill | workflows, accounting effects, state machines |
| `accounts-payable.H14` | H — Cheque printing | A very large cheque number | workflows, accounting effects, state machines |
| `accounts-payable.H15` | H — Cheque printing | Stub lines in a foreign currency | workflows, calculations, state machines |
| `accounts-payable.H16` | H — Cheque printing | The cheque number reaches the journal items | workflows, calculations, accounting effects, state machines |
| `accounts-payable.I1` | I — Line defaulting and deductibility | The purchase unit comes from the supplier record | workflows |
| `accounts-payable.I2` | I — Line defaulting and deductibility | The label uses the purchase description | workflows |
| `accounts-payable.I3` | I — Line defaulting and deductibility | The taxes come from the supplier taxes | calculations |
| `accounts-payable.I4` | I — Line defaulting and deductibility | An account with taxes seeds a line with no product | calculations, accounting effects |
| `accounts-payable.I5` | I — Line defaulting and deductibility | An imported line is never overwritten | calculations |
| `accounts-payable.I6` | I — Line defaulting and deductibility | The most frequent account of a vendor | accounting effects |
| `accounts-payable.I7` | I — Line defaulting and deductibility | The neighbours' account | accounting effects |
| `accounts-payable.I8` | I — Line defaulting and deductibility | The journal default is the last resort | accounting effects |
| `accounts-payable.I9` | I — Line defaulting and deductibility | Partial deductibility | workflows, calculations, accounting effects, state machines |
| `accounts-payable.I10` | I — Line defaulting and deductibility | Deductibility is refused outside purchase documents | workflows, business rules |
| `accounts-payable.I11` | I — Line defaulting and deductibility | Deductibility out of range | business rules |
| `accounts-payable.J1` | J — Upload, mail and decoding | Five separate files make five bills | calculations, accounting effects, business rules, state machines |
| `accounts-payable.J2` | J — Upload, mail and decoding | A container and its embedded structured file make one bill |  |
| `accounts-payable.J3` | J — Upload, mail and decoding | A mail with three representations makes one bill | accounting effects, business rules |
| `accounts-payable.J4` | J — Upload, mail and decoding | A mail with three Portable Document Format files makes three bills | business rules |
| `accounts-payable.J5` | J — Upload, mail and decoding | A mail with no attachment bounces | workflows, accounting effects, business rules |
| `accounts-payable.J6` | J — Upload, mail and decoding | A mail forwarded by an internal user resolves the real sender | workflows, accounting effects, business rules |
| `accounts-payable.J7` | J — Upload, mail and decoding | A decoder failure is reported without losing the bill | business rules, state machines |
| `accounts-payable.J8` | J — Upload, mail and decoding | A decoder refusal is reported as a reason | business rules |
| `accounts-payable.J9` | J — Upload, mail and decoding | No decoder means nothing is written |  |
| `accounts-payable.J10` | J — Upload, mail and decoding | A supplier posting a message does not trigger decoding | workflows, business rules, state machines |
| `accounts-payable.J11` | J — Upload, mail and decoding | Uploading without a journal | workflows, accounting effects, business rules |
| `accounts-payable.K1` | K — Reset, cancel and delete | Reset to draft keeps the number | workflows, state machines |
| `accounts-payable.K2` | K — Reset, cancel and delete | Reset refuses a hashed document | workflows, accounting effects, business rules, state machines |
| `accounts-payable.K3` | K — Reset, cancel and delete | Cancel removes reconciliations | workflows, state machines |
| `accounts-payable.K4` | K — Reset, cancel and delete | Cancel refuses an unresettable document | workflows, calculations, accounting effects, business rules, state machines |
| `accounts-payable.K5` | K — Reset, cancel and delete | Deleting a numbered bill in the middle of a chain | workflows, accounting effects, business rules, state machines |
| `accounts-payable.K6` | K — Reset, cancel and delete | The restrictive audit trail blocks deletion | workflows, accounting effects, business rules, state machines |
| `accounts-payable.K7` | K — Reset, cancel and delete | Bulk removal classifies each document | workflows, state machines |
| `accounts-payable.K8` | K — Reset, cancel and delete | Switching the type of a numbered document | workflows, business rules, state machines |
| `accounts-payable.L1` | L — Payment status | Partial payment | workflows, state machines |
| `accounts-payable.L2` | L — Payment status | Full payment | workflows, accounting effects, state machines |
| `accounts-payable.L3` | L — Payment status | Offset entirely by a credit note | workflows, accounting effects, state machines |
| `accounts-payable.L4` | L — Payment status | A zero-total bill | workflows, calculations, state machines |
| `accounts-payable.L5` | L — Payment status | Blocking | workflows, business rules, state machines |
| `accounts-payable.L6` | L — Payment status | Blocking a paid bill is refused | workflows, business rules |
| `accounts-payable.L7` | L — Payment status | Unreconciling reverts the status | workflows, state machines |
| `accounts-payable.M1` | M — The invoice analysis report | A bill line appears with negative amounts | workflows, calculations, accounting effects, state machines |
| `accounts-payable.M2` | M — The invoice analysis report | Quantities are restated in the template unit | calculations, state machines |
| `accounts-payable.M3` | M — The invoice analysis report | The weighted average price | calculations |
| `accounts-payable.M4` | M — The invoice analysis report | Zero quantity does not break the average | calculations |
| `accounts-payable.M5` | M — The invoice analysis report | Inventory value on a purchase | calculations |
| `accounts-payable.M6` | M — The invoice analysis report | Margin on a customer invoice and on a credit note | workflows, calculations, accounting effects |
| `accounts-payable.M7` | M — The invoice analysis report | Multi-currency conversion | calculations, state machines |
| `accounts-payable.M8` | M — The invoice analysis report | Only product lines are rows | calculations |
| `accounts-payable.M9` | M — The invoice analysis report | Multi-company visibility |  |
| `accounts-payable.N1` | N — Debit notes | A debit note from a bill | workflows, accounting effects, state machines |
| `accounts-payable.N2` | N — Debit notes | Numbering with a dedicated debit note sequence | workflows, accounting effects, state machines |
| `accounts-payable.N3` | N — Debit notes | A debit note from a vendor credit note | workflows, accounting effects, state machines |
| `accounts-payable.N4` | N — Debit notes | A second debit note is refused | workflows, accounting effects, business rules |
| `accounts-payable.O1` | O — Quick encoding | One line generated from a typed total | workflows, calculations, accounting effects, state machines |
| `accounts-payable.O2` | O — Quick encoding | The mixed early-discount adjustment | calculations |
| `accounts-payable.O3` | O — Quick encoding | A surviving mismatch blocks posting | workflows, calculations, business rules |
| `accounts-payable.O4` | O — Quick encoding | The suggested bill date | workflows, accounting effects, state machines |
| `accounts-payable.P1` | P — Abnormal detection | An unusually large bill warns | workflows, calculations, state machines |
| `accounts-payable.P2` | P — Abnormal detection | Too little history means no warning | workflows, state machines |
| `accounts-payable.P3` | P — Abnormal detection | An early bill warns | workflows, calculations, state machines |
| `accounts-payable.P4` | P — Abnormal detection | Silencing | workflows |
| `accounts-payable.Q1` | Q — Intercompany clearing | A bill of company A paid by company B | workflows, calculations, accounting effects, state machines |
| `accounts-payable.Q2` | Q — Intercompany clearing | Nothing happens without the configuration | accounting effects |
| `accounts-payable.R1` | R — Multi-currency | A bill in a foreign currency | workflows, calculations, accounting effects, state machines |
| `accounts-payable.R2` | R — Multi-currency | A zero or negative rate is refused | calculations, business rules |
| `accounts-payable.R3` | R — Multi-currency | An archived currency blocks posting | workflows, business rules |
| `accounts-payable.S1` | S — Access and multi-company | A billing clerk may post but not review in a narrowed installation | workflows, accounting effects, business rules |
| `accounts-payable.S2` | S — Access and multi-company | A user without the invoicing group cannot post | workflows, accounting effects, business rules |
| `accounts-payable.S3` | S — Access and multi-company | A portal supplier sees only their own bills | workflows, state machines |
| `accounts-payable.S4` | S — Access and multi-company | Cross-company accounts are refused at posting | workflows, accounting effects, business rules |
| `accounts-payable.S5` | S — Access and multi-company | Documents are scoped by company | accounting effects |

## accounts-receivable

| Reference | Group | Scenario | Exercises |
|---|---|---|---|
| `accounts-receivable.A1` | A — Creating and posting a customer invo | Happy path: a three-line invoice with two taxes | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.A2` | A — Creating and posting a customer invo | Product defaulting | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.A3` | A — Creating and posting a customer invo | Product defaulting through a fiscal position | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.A4` | A — Creating and posting a customer invo | Posting without a customer | workflows, business rules, state machines |
| `accounts-receivable.A5` | A — Creating and posting a customer invo | Posting an empty document | workflows, state machines |
| `accounts-receivable.A6` | A — Creating and posting a customer invo | Posting a negative invoice | workflows, calculations, state machines |
| `accounts-receivable.A7` | A — Creating and posting a customer invo | Several failures are reported together | workflows, accounting effects, business rules, state machines |
| `accounts-receivable.A8` | A — Creating and posting a customer invo | Posting fills in a missing document date | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.A9` | A — Creating and posting a customer invo | Posting a future-dated document softly | workflows, state machines |
| `accounts-receivable.A10` | A — Creating and posting a customer invo | Posting a zero-total invoice | workflows, calculations, state machines |
| `accounts-receivable.A11` | A — Creating and posting a customer invo | The customer rank increases | workflows, state machines |
| `accounts-receivable.B1` | B — Payment terms and instalments | Thirty percent immediately, the balance in forty-five days | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.B2` | B — Payment terms and instalments | The balance rule absorbs the rounding | workflows, calculations |
| `accounts-receivable.B3` | B — Payment terms and instalments | A fixed line plus a balance line | workflows, calculations |
| `accounts-receivable.B4` | B — Payment terms and instalments | A fixed line larger than the total | workflows, calculations |
| `accounts-receivable.B5` | B — Payment terms and instalments | Two lines on the same date merge | workflows, calculations |
| `accounts-receivable.B6` | B — Payment terms and instalments | Every delay type | workflows |
| `accounts-receivable.B7` | B — Payment terms and instalments | No payment term at all | workflows, calculations |
| `accounts-receivable.B8` | B — Payment terms and instalments | Changing the payment term rebuilds the instalments | workflows, state machines |
| `accounts-receivable.B9` | B — Payment terms and instalments | Deleting a payment term line by hand | state machines |
| `accounts-receivable.B10` | B — Payment terms and instalments | Payment term validation | business rules |
| `accounts-receivable.B11` | B — Payment terms and instalments | Deleting a referenced payment term |  |
| `accounts-receivable.B12` | B — Payment terms and instalments | The preview | calculations |
| `accounts-receivable.C1` | C — Early payment discount | Mode "On early payment" — the invoice | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.C2` | C — Early payment discount | Mode "On early payment" — paying early | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.C3` | C — Early payment discount | Mode "On early payment" — paying late | workflows, calculations, state machines |
| `accounts-receivable.C4` | C — Early payment discount | Mode "Never" | workflows, calculations, accounting effects |
| `accounts-receivable.C5` | C — Early payment discount | Mode "Always (upon invoice)" — the invoice | workflows, calculations, accounting effects |
| `accounts-receivable.C6` | C — Early payment discount | Mode "Always (upon invoice)" — paying early and late | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.C7` | C — Early payment discount | Eligibility | workflows, calculations, state machines |
| `accounts-receivable.C8` | C — Early payment discount | Non-discountable taxes are excluded | workflows, calculations |
| `accounts-receivable.C9` | C — Early payment discount | Distribution over several lines | workflows, calculations |
| `accounts-receivable.D1` | D — Cash rounding | Add a rounding line, nearest, 0.05 | workflows, calculations, accounting effects |
| `accounts-receivable.D2` | D — Cash rounding | Modify the biggest tax amount, nearest, 0.05 | workflows, calculations, accounting effects |
| `accounts-receivable.D3` | D — Cash rounding | Rounding down | calculations, accounting effects |
| `accounts-receivable.D4` | D — Cash rounding | Rounding up | calculations |
| `accounts-receivable.D5` | D — Cash rounding | An already-rounded total | calculations |
| `accounts-receivable.D6` | D — Cash rounding | Switching strategies | calculations |
| `accounts-receivable.D7` | D — Cash rounding | Removing the cash rounding method | workflows, calculations |
| `accounts-receivable.D8` | D — Cash rounding | Biggest-tax strategy with no tax | workflows, calculations |
| `accounts-receivable.D9` | D — Cash rounding | Cash rounding and instalments | workflows, calculations |
| `accounts-receivable.D10` | D — Cash rounding | Validation | calculations |
| `accounts-receivable.E1` | E — Discount allocation | Two lines, merged keys, weighted analytic distribution | workflows, calculations, accounting effects |
| `accounts-receivable.E2` | E — Discount allocation | No discount allocation account | workflows, calculations, accounting effects |
| `accounts-receivable.E3` | E — Discount allocation | A line whose account is already the discount account | calculations, accounting effects |
| `accounts-receivable.E4` | E — Discount allocation | A discount that rounds to zero | calculations |
| `accounts-receivable.F1` | F — Numbering | The first document of the year | workflows, accounting effects, state machines |
| `accounts-receivable.F2` | F — Numbering | The next document | workflows, state machines |
| `accounts-receivable.F3` | F — Numbering | A new year restarts the counter | workflows, state machines |
| `accounts-receivable.F4` | F — Numbering | A dedicated credit note sequence | workflows, accounting effects, state machines |
| `accounts-receivable.F5` | F — Numbering | A dedicated debit note sequence | workflows, accounting effects, state machines |
| `accounts-receivable.F6` | F — Numbering | A staggered fiscal year | workflows, state machines |
| `accounts-receivable.F7` | F — Numbering | A draft has no number but shows a placeholder | workflows, state machines |
| `accounts-receivable.F8` | F — Numbering | Changing the date of a never-posted draft clears a stale number | workflows, state machines |
| `accounts-receivable.F9` | F — Numbering | Duplicate numbers are impossible | workflows, accounting effects, state machines |
| `accounts-receivable.F10` | F — Numbering | Changing the journal of a numbered document | workflows, accounting effects, state machines |
| `accounts-receivable.F11` | F — Numbering | Deleting a document in the middle of a chain | workflows, accounting effects, state machines |
| `accounts-receivable.F12` | F — Numbering | The accounting date is pushed past a lock date | workflows, accounting effects, state machines |
| `accounts-receivable.G1` | G — The payment reference | Full reference, based on invoice | workflows, accounting effects, state machines |
| `accounts-receivable.G2` | G — The payment reference | Full reference, based on customer | workflows, state machines |
| `accounts-receivable.G3` | G — The payment reference | Numbers only, based on invoice | workflows, state machines |
| `accounts-receivable.G4` | G — The payment reference | Numbers only, based on customer |  |
| `accounts-receivable.G5` | G — The payment reference | European, based on invoice | workflows, accounting effects |
| `accounts-receivable.G6` | G — The payment reference | European, based on customer | accounting effects |
| `accounts-receivable.G7` | G — The payment reference | An unknown combination | workflows, accounting effects, state machines |
| `accounts-receivable.G8` | G — The payment reference | The reference reaches the line labels | workflows, state machines |
| `accounts-receivable.G9` | G — The payment reference | The sanitised form | state machines |
| `accounts-receivable.H1` | H — Payment status | A partial payment | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.H2` | H — Payment status | Full payment | workflows, state machines |
| `accounts-receivable.H3` | H — Payment status | In payment becomes paid | workflows, state machines |
| `accounts-receivable.H4` | H — Payment status | Undoing a reconciliation | workflows, calculations, state machines |
| `accounts-receivable.H5` | H — Payment status | Reversed rather than paid | workflows, accounting effects, state machines |
| `accounts-receivable.H6` | H — Payment status | Blocking | workflows, state machines |
| `accounts-receivable.H7` | H — Payment status | A draft invoice with a non-zero total qualifies | workflows, calculations, state machines |
| `accounts-receivable.H8` | H — Payment status | The imported-balance status is never overwritten | workflows, calculations, state machines |
| `accounts-receivable.I1` | I — Credit notes and reversal | Credit note against a paid invoice | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.I2` | I — Credit notes and reversal | Credit note against an unpaid invoice, fully reconciled | workflows, accounting effects, state machines |
| `accounts-receivable.I3` | I — Credit notes and reversal | Reverse and create invoice | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.I4` | I — Credit notes and reversal | A future-dated reversal is not cancelling | workflows, accounting effects, state machines |
| `accounts-receivable.I5` | I — Credit notes and reversal | The early-discount payment term survives | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.I6` | I — Credit notes and reversal | Reversal validation | business rules |
| `accounts-receivable.J1` | J — Debit notes | A debit note copying the lines | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.J2` | J — Debit notes | A debit note without lines | accounting effects |
| `accounts-receivable.J3` | J — Debit notes | A debit note from a credit note | workflows, accounting effects, state machines |
| `accounts-receivable.J4` | J — Debit notes | Debit note validation | accounting effects |
| `accounts-receivable.K1` | K — Foreign currency | A foreign-currency invoice | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.K2` | K — Foreign currency | An exchange difference on settlement | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.K3` | K — Foreign currency | Changing the rate on a draft | workflows, calculations, state machines |
| `accounts-receivable.K4` | K — Foreign currency | Changing the currency on a draft | calculations, state machines |
| `accounts-receivable.K5` | K — Foreign currency | A non-positive rate is refused | workflows, calculations, business rules, state machines |
| `accounts-receivable.K6` | K — Foreign currency | An archived currency cannot be posted | workflows, business rules, state machines |
| `accounts-receivable.L1` | L — The dynamic line synchronisation | Adding a line rebuilds the taxes and the instalments | workflows, calculations |
| `accounts-receivable.L2` | L — The dynamic line synchronisation | Removing every taxed line removes the tax lines | workflows, calculations |
| `accounts-receivable.L3` | L — The dynamic line synchronisation | A manual tax correction is preserved | workflows, calculations |
| `accounts-receivable.L4` | L — The dynamic line synchronisation | A user-created derived line is respected | calculations |
| `accounts-receivable.L5` | L — The dynamic line synchronisation | A deleted derived line is not resurrected |  |
| `accounts-receivable.L6` | L — The dynamic line synchronisation | Recycling preserves identifiers | workflows |
| `accounts-receivable.L7` | L — The dynamic line synchronisation | The order of reconciliation | workflows, calculations, accounting effects |
| `accounts-receivable.L8` | L — The dynamic line synchronisation | A posted document is never synchronised | workflows, state machines |
| `accounts-receivable.L9` | L — The dynamic line synchronisation | The balance invariant holds after every save | calculations |
| `accounts-receivable.M1` | M — Sending | Sending one invoice by e-mail | workflows, accounting effects, business rules, state machines |
| `accounts-receivable.M2` | M — Sending | Sending a credit note picks the credit note template | workflows, accounting effects, state machines |
| `accounts-receivable.M3` | M — Sending | A recipient without an e-mail address blocks a single send | workflows, state machines |
| `accounts-receivable.M4` | M — Sending | Batch sending | workflows, calculations, state machines |
| `accounts-receivable.M5` | M — Sending | Batch sending with the job archived | workflows |
| `accounts-receivable.M6` | M — Sending | Sending an unposted document | workflows, state machines |
| `accounts-receivable.M7` | M — Sending | Sending a vendor bill | workflows, state machines |
| `accounts-receivable.M8` | M — Sending | Resetting to draft detaches the file | workflows, state machines |
| `accounts-receivable.M9` | M — Sending | The job retries a retryable error | workflows, business rules |
| `accounts-receivable.N1` | N — The customer portal | The list page | workflows, accounting effects, state machines |
| `accounts-receivable.N2` | N — The customer portal | Receipts are invisible in the portal | workflows, state machines |
| `accounts-receivable.N3` | N — The customer portal | The overdue count | workflows, state machines |
| `accounts-receivable.N4` | N — The customer portal | Downloading | workflows, calculations, state machines |
| `accounts-receivable.N5` | N — The customer portal | Paying online | workflows, calculations, state machines |
| `accounts-receivable.N6` | N — The customer portal | Paying is refused | workflows, business rules |
| `accounts-receivable.N7` | N — The customer portal | A pending transaction blocks a second payment | workflows, state machines |
| `accounts-receivable.N8` | N — The customer portal | A tampered custom amount | calculations |
| `accounts-receivable.N9` | N — The customer portal | Batch payment of overdue invoices | workflows, calculations |
| `accounts-receivable.N10` | N — The customer portal | Instalments in the portal | workflows, calculations, state machines |
| `accounts-receivable.N11` | N — The customer portal | The early payment discount in the portal | workflows, calculations, business rules, state machines |
| `accounts-receivable.O1` | O — The credit limit | Below the limit | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.O2` | O — The credit limit | Above the limit | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.O3` | O — The credit limit | With orders awaiting invoicing | workflows, calculations, accounting effects, state machines |
| `accounts-receivable.O4` | O — The credit limit | On a posted document | workflows, accounting effects, state machines |
| `accounts-receivable.O5` | O — The credit limit | The partner limit toggle | accounting effects |
| `accounts-receivable.P1` | P — Duplicate detection | Two identical customer invoices | workflows, calculations, state machines |
| `accounts-receivable.P2` | P — Duplicate detection | A different total is not a duplicate | calculations |
| `accounts-receivable.P3` | P — Duplicate detection | A different currency is not a duplicate | calculations |
| `accounts-receivable.P4` | P — Duplicate detection | A cancelled document is not a duplicate | workflows, state machines |
| `accounts-receivable.P5` | P — Duplicate detection | Deleting duplicates |  |
| `accounts-receivable.Q1` | Q — Locking, hashing and the audit trail | A lock date blocks a change of date | workflows, accounting effects, state machines |
| `accounts-receivable.Q2` | Q — Locking, hashing and the audit trail | A hashed document cannot be reset | workflows, accounting effects, business rules, state machines |
| `accounts-receivable.Q3` | Q — Locking, hashing and the audit trail | A hashed document cannot be edited | business rules |
| `accounts-receivable.Q4` | Q — Locking, hashing and the audit trail | The restrictive audit trail forbids deletion | workflows, business rules, state machines |
| `accounts-receivable.Q5` | Q — Locking, hashing and the audit trail | The get-rid-of routine picks the right path | workflows, state machines |
| `accounts-receivable.R1` | R — Line-level rules | A receivable account requires a due date | workflows, accounting effects, state machines |
| `accounts-receivable.R2` | R — Line-level rules | A payable account on a sale document | workflows, accounting effects, state machines |
| `accounts-receivable.R3` | R — Line-level rules | An archived account | workflows, accounting effects, state machines |
| `accounts-receivable.R4` | R — Line-level rules | An account from another company | workflows, accounting effects, state machines |
| `accounts-receivable.R5` | R — Line-level rules | Off-balance accounts | calculations, accounting effects, state machines |
| `accounts-receivable.R6` | R — Line-level rules | Deleting a line of a posted document | workflows, state machines |
| `accounts-receivable.R7` | R — Line-level rules | Changing the taxes of a posted line | workflows, calculations, state machines |
| `accounts-receivable.R8` | R — Line-level rules | A section line with an amount | calculations, accounting effects, business rules |
| `accounts-receivable.R9` | R — Line-level rules | Deductibility on a customer invoice | workflows, state machines |
| `accounts-receivable.R10` | R — Line-level rules | A reconciliation-breaking edit | workflows, accounting effects, state machines |
| `accounts-receivable.S1` | S — Scheduled jobs | The auto-post job posts due documents | workflows, state machines |
| `accounts-receivable.S2` | S — Scheduled jobs | The auto-post job isolates a failure | workflows, state machines |
| `accounts-receivable.S3` | S — Scheduled jobs | A recurring invoice | workflows, state machines |
| `accounts-receivable.S4` | S — Scheduled jobs | The sending job processes ten at a time | workflows, accounting effects |
| `accounts-receivable.T1` | T — Multi-company and access | Company scoping of documents |  |
| `accounts-receivable.T2` | T — Multi-company and access | A branch company may use a parent's accounts | workflows, accounting effects |
| `accounts-receivable.T3` | T — Multi-company and access | Posting without the invoicing group | workflows, accounting effects |
| `accounts-receivable.T4` | T — Multi-company and access | The portal user sees only their own documents | workflows, state machines |
| `accounts-receivable.T5` | T — Multi-company and access | Reading the credit figures | workflows, calculations, accounting effects |
| `accounts-receivable.U1` | U — Numeric edge cases | The rounding delta on price-included taxes | calculations |
| `accounts-receivable.U2` | U — Numeric edge cases | A very small amount | workflows, calculations |
| `accounts-receivable.U3` | U — Numeric edge cases | A three-decimal currency | calculations |
| `accounts-receivable.U4` | U — Numeric edge cases | A zero-decimal currency | calculations |
| `accounts-receivable.U5` | U — Numeric edge cases | An instalment on a leap day | calculations |
| `accounts-receivable.U6` | U — Numeric edge cases | "Days end of month on the" with a short target month | calculations |
| `accounts-receivable.V1` | V — End-to-end scenarios | Quote to cash with instalments and an early discount | calculations, state machines |
| `accounts-receivable.V2` | V — End-to-end scenarios | Invoice, partial payment, credit note, refund | workflows, accounting effects, state machines |
| `accounts-receivable.V3` | V — End-to-end scenarios | Foreign currency with instalments and cash rounding | calculations |
| `accounts-receivable.V4` | V — End-to-end scenarios | A fully worked cash rounding and instalment interaction | calculations |

## general-ledger

| Reference | Group | Scenario | Exercises |
|---|---|---|---|
| `general-ledger.1.1` |  | Creating an account inherits the type of the preceding code. | accounting effects |
| `general-ledger.1.2` |  | Creating an account before every existing code falls back to current assets. | accounting effects |
| `general-ledger.1.3` |  | A receivable account must be reconcilable. | workflows, accounting effects, business rules |
| `general-ledger.1.4` |  | Changing the type to Receivable forces the reconcilable flag on. | workflows, accounting effects |
| `general-ledger.1.5` |  | Changing the type between two neutral asset types keeps the flag. | workflows, accounting effects |
| `general-ledger.1.6` |  | An off-balance account cannot be reconcilable. | workflows, calculations, accounting effects, business rules |
| `general-ledger.1.7` |  | Duplicate codes in a company hierarchy are refused. | accounting effects, business rules |
| `general-ledger.1.8` |  | The same code may exist in two unrelated companies. | accounting effects |
| `general-ledger.1.9` |  | One account, two codes. | accounting effects |
| `general-ledger.1.10` |  | The free-code walk increments the digits. | workflows, accounting effects |
| `general-ledger.1.11` |  | The free-code walk falls back to a copy suffix. | workflows |
| `general-ledger.1.12` |  | The free-code walk on a code with no digits. | workflows |
| `general-ledger.1.13` |  | An invalid code is refused. | accounting effects, business rules |
| `general-ledger.1.14` |  | An account with items cannot be deleted. | accounting effects, business rules |
| `general-ledger.1.15` |  | Typing a code inside the name splits it. | accounting effects |
| `general-ledger.1.16` |  | Switching reconciliation off with pending matches is refused. | workflows, accounting effects, business rules |
| `general-ledger.1.17` |  | Switching reconciliation on rewrites the residuals. | workflows, calculations, accounting effects |
| `general-ledger.1.18` |  | A bank-and-cash account cannot be shared. | accounting effects, business rules |
| `general-ledger.1.19` |  | Unmerging preserves the codes. | accounting effects, business rules |
| `general-ledger.2.1` |  | Prefix lengths must match. | business rules |
| `general-ledger.2.2` |  | A single prefix fills both ends. |  |
| `general-ledger.2.3` |  | Overlapping groups of the same length are refused. | accounting effects, business rules |
| `general-ledger.2.4` |  | Parenting is derived from the prefixes. |  |
| `general-ledger.2.5` |  | The most specific group wins. | accounting effects |
| `general-ledger.2.6` |  | The display name of a group. |  |
| `general-ledger.2.7` |  | Deleting a group re-parents its children. |  |
| `general-ledger.2.8` |  | The root of an account. | accounting effects |
| `general-ledger.3.1` |  | The default code of a new sale journal. | accounting effects |
| `general-ledger.3.2` |  | The name placeholder. | accounting effects |
| `general-ledger.3.3` |  | The code is unique per company. | accounting effects, business rules |
| `general-ledger.3.4` |  | Duplicating a journal invents a code. | accounting effects |
| `general-ledger.3.5` |  | Creating a bank journal creates a liquidity account. | accounting effects |
| `general-ledger.3.6` |  | Archiving a journal with draft entries is refused. | accounting effects, business rules, state machines |
| `general-ledger.3.7` |  | Switching hashing off after hashing is refused. | workflows, accounting effects, business rules, state machines |
| `general-ledger.3.8` |  | Changing the company of a journal with entries is refused. | accounting effects, business rules |
| `general-ledger.3.9` |  | The display name carries a foreign currency. | accounting effects |
| `general-ledger.3.10` |  | The foreign currency propagates to the liquidity account. | accounting effects |
| `general-ledger.4.1` |  | A balanced entry saves. | calculations, accounting effects |
| `general-ledger.4.2` |  | An unbalanced entry is refused. | calculations, accounting effects, business rules |
| `general-ledger.4.3` |  | Several unbalanced entries are reported together. | calculations, business rules |
| `general-ledger.4.4` |  | An imbalance in the foreign currency is tolerated. | calculations, accounting effects |
| `general-ledger.4.5` |  | A rounding difference below the step does not break the balance. | calculations, accounting effects |
| `general-ledger.4.6` |  | The default balance of a new item balances the entry. | calculations, accounting effects, state machines |
| `general-ledger.5.1` |  | A positive balance is a debit. | calculations, accounting effects |
| `general-ledger.5.2` |  | A negative balance is a credit. | calculations, accounting effects |
| `general-ledger.5.3` |  | Typing into the credit box. | calculations, accounting effects |
| `general-ledger.5.4` |  | A negative debit marks the item as storno. | calculations, accounting effects |
| `general-ledger.5.5` |  | The sign coherence check. | calculations, accounting effects, business rules |
| `general-ledger.5.6` |  | A section carries no amount. | calculations, accounting effects, business rules |
| `general-ledger.5.7` |  | An accountable item needs an account. | accounting effects, business rules |
| `general-ledger.6.1` |  | The happy path. | workflows, calculations, accounting effects, state machines |
| `general-ledger.6.2` |  | An entry with no accountable item. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.3` |  | An archived account blocks posting. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.4` |  | An archived journal blocks posting. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.5` |  | Several failures are reported together. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.6` |  | Posting a posted entry. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.7` |  | A future entry in soft mode is scheduled. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.8` |  | A future entry in hard mode is posted. | workflows, accounting effects, state machines |
| `general-ledger.6.9` |  | A future entry already scheduled cannot be forced by the button. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.10` |  | Posting an entry of another company's account. | workflows, accounting effects, business rules, state machines |
| `general-ledger.6.11` |  | Posting without the permission. | workflows, accounting effects, business rules |
| `general-ledger.7.1` |  | The starting number of a miscellaneous journal. | workflows, accounting effects, state machines |
| `general-ledger.7.2` |  | The starting number of a sale journal. | accounting effects |
| `general-ledger.7.3` |  | The starting number with a staggered fiscal year. | workflows, accounting effects, state machines |
| `general-ledger.7.4` |  | The same month on the other side of the fiscal boundary. | workflows, accounting effects, state machines |
| `general-ledger.7.5` |  | The counter increments within a chain. | workflows, accounting effects, state machines |
| `general-ledger.7.6` |  | A new month restarts a monthly chain. | workflows, accounting effects, state machines |
| `general-ledger.7.7` |  | A new year restarts a yearly chain. | workflows, state machines |
| `general-ledger.7.8` |  | Format inference, twelve cases. | workflows, state machines |
| `general-ledger.7.9` |  | A short counter grows instead of overflowing. | workflows, state machines |
| `general-ledger.7.10` |  | The prefix of the newest entry wins, not the alphabetically greatest. | workflows, accounting effects, state machines |
| `general-ledger.7.11` |  | Ordering when posting several entries at once. | workflows, accounting effects, state machines |
| `general-ledger.7.12` |  | Two entries cannot share a number. | workflows, accounting effects, business rules, state machines |
| `general-ledger.7.13` |  | A journal may switch from yearly to monthly. | accounting effects |
| `general-ledger.7.14` |  | Introducing a fiscal-year prefix changes the following chain. | accounting effects |
| `general-ledger.7.15` |  | A monthly shape wins over a two-digit year range. | workflows, accounting effects, state machines |
| `general-ledger.7.16` |  | A journal pattern override forces the year-range reading. | workflows, accounting effects, state machines |
| `general-ledger.7.17` |  | A non-numeric number gets a counter appended. | workflows, accounting effects, state machines |
| `general-ledger.7.18` |  | Credit notes have their own chain. | workflows, accounting effects, state machines |
| `general-ledger.7.19` |  | A date that no longer matches the number clears it. | workflows, accounting effects, state machines |
| `general-ledger.7.20` |  | A date that no longer matches on a posted entry is refused. | workflows, accounting effects, business rules, state machines |
| `general-ledger.7.21` |  | The journal of a numbered entry cannot change. | workflows, accounting effects, business rules, state machines |
| `general-ledger.7.22` |  | Clearing the number allows the journal change. | workflows, accounting effects |
| `general-ledger.7.23` |  | Concurrency. | workflows, accounting effects |
| `general-ledger.7.24` |  | Several numbers in one transaction. | workflows, state machines |
| `general-ledger.8.1` |  | A deleted middle entry flags its successor. | accounting effects |
| `general-ledger.8.2` |  | Filling the gap clears the flag. | accounting effects |
| `general-ledger.8.3` |  | A draft entry between two posted entries is flagged. | workflows, accounting effects, state machines |
| `general-ledger.8.4` |  | Suffixes make independent chains. | accounting effects |
| `general-ledger.8.5` |  | An entry numbered through the locked increment is never flagged. | workflows, accounting effects, state machines |
| `general-ledger.9.1` |  | Keeping the current order. | accounting effects |
| `general-ledger.9.2` |  | Reordering by accounting date. | accounting effects |
| `general-ledger.9.3` |  | Reformatting a year-range prefix. | workflows, state machines |
| `general-ledger.9.4` |  | The counter of the first number applies to the last period only. |  |
| `general-ledger.9.5` |  | Renumbering by date in a hash-secured journal is refused. | workflows, accounting effects, business rules, state machines |
| `general-ledger.9.6` |  | A selection spanning journals is refused. | accounting effects, business rules |
| `general-ledger.9.7` |  | Mixing credit notes with invoices is refused. | workflows, accounting effects, business rules |
| `general-ledger.10.1` |  | A late vendor bill of a past month. | workflows, accounting effects, state machines |
| `general-ledger.10.2` |  | A vendor bill of the current month. | workflows, accounting effects, state machines |
| `general-ledger.10.3` |  | A vendor bill dated in the future. | workflows, accounting effects, state machines |
| `general-ledger.10.4` |  | A customer invoice in the past with nothing locked. | workflows, accounting effects, state machines |
| `general-ledger.10.5` |  | A customer invoice in a locked period. | workflows, state machines |
| `general-ledger.10.6` |  | A customer invoice in a locked period with a monthly sale chain. | workflows, accounting effects |
| `general-ledger.10.7` |  | A vendor bill in a locked period. | workflows, accounting effects, state machines |
| `general-ledger.10.8` |  | A journal with no earlier number. | accounting effects |
| `general-ledger.10.9` |  | A yearly purchase chain. | accounting effects |
| `general-ledger.11.1` |  | Posting into a locked period shifts the date. | workflows, accounting effects, state machines |
| `general-ledger.11.2` |  | Modifying a posted entry in a locked period is refused. | workflows, accounting effects, business rules, state machines |
| `general-ledger.11.3` |  | A date exactly equal to the lock date is locked. | accounting effects |
| `general-ledger.11.4` |  | The sales lock date applies only to sale journals. | workflows, accounting effects, state machines |
| `general-ledger.11.5` |  | The tax lock date applies only to tax-bearing items. | workflows, calculations, accounting effects, business rules, state machines |
| `general-ledger.11.6` |  | Several violated lock dates are listed chronologically. | workflows, business rules, state machines |
| `general-ledger.11.7` |  | The hard lock date cannot be removed. | business rules |
| `general-ledger.11.8` |  | The hard lock date cannot move backwards. | workflows, business rules |
| `general-ledger.11.9` |  | Draft entries block a hard lock date. | workflows, accounting effects, business rules, state machines |
| `general-ledger.11.10` |  | Unreconciled bank transactions block a lock date. | workflows, business rules, state machines |
| `general-ledger.11.11` |  | The lock dates of the ancestors apply. | accounting effects |
| `general-ledger.11.12` |  | The later of the two wins. | accounting effects |
| `general-ledger.12.1` |  | An exception relaxes the lock date for its beneficiary. | workflows, accounting effects, state machines |
| `general-ledger.12.2` |  | An exception for everybody. | accounting effects |
| `general-ledger.12.3` |  | An exception that removes the lock date. |  |
| `general-ledger.12.4` |  | An exception may not relax the hard lock date. |  |
| `general-ledger.12.5` |  | An expired exception has no effect. | state machines |
| `general-ledger.12.6` |  | Exactly one lock date per exception. | business rules |
| `general-ledger.12.7` |  | Revocation requires the administrator group. | business rules |
| `general-ledger.12.8` |  | Changing the company lock date recreates the exception. |  |
| `general-ledger.12.9` |  | An exception may not be duplicated. | business rules |
| `general-ledger.12.10` |  | The audit query of an exception. | accounting effects, business rules |
| `general-ledger.13.1` |  | Posting in a securing journal hashes the entry. | workflows, accounting effects, business rules, state machines |
| `general-ledger.13.2` |  | The chain binds the entries. |  |
| `general-ledger.13.3` |  | The serialised content. | accounting effects |
| `general-ledger.13.4` |  | A hashed entry cannot change its date. | accounting effects, business rules |
| `general-ledger.13.5` |  | An item of a hashed entry cannot change its amount. | calculations, accounting effects, business rules |
| `general-ledger.13.6` |  | A hashed entry cannot be reset to draft. | accounting effects, business rules, state machines |
| `general-ledger.13.7` |  | An item of a hashed entry cannot be deleted. | accounting effects, business rules |
| `general-ledger.13.8` |  | A gap blocks the hashing. | workflows, accounting effects, business rules, state machines |
| `general-ledger.13.9` |  | The bulk wizard tolerates the gap. | calculations |
| `general-ledger.13.10` |  | An unreconciled bank transaction blocks the chain. | workflows, accounting effects, business rules, state machines |
| `general-ledger.13.11` |  | Hashing a chain hashes its predecessors. | workflows, accounting effects, state machines |
| `general-ledger.13.12` |  | The verification detects a modification. | accounting effects |
| `general-ledger.13.13` |  | The verification of an untouched chain. | accounting effects |
| `general-ledger.13.14` |  | Printing without the permission. | accounting effects, business rules |
| `general-ledger.14.1` |  | A posted entry returns to draft and keeps its number. | workflows, accounting effects, state machines |
| `general-ledger.14.2` |  | Posting it again reuses the number. | workflows |
| `general-ledger.14.3` |  | An exchange-difference entry cannot return to draft. | workflows, accounting effects, business rules, state machines |
| `general-ledger.14.4` |  | A cash-basis entry cannot return to draft. | workflows, calculations, accounting effects, business rules, state machines |
| `general-ledger.14.5` |  | Cancelling a posted entry. | workflows, accounting effects, state machines |
| `general-ledger.14.6` |  | A cancelled entry is excluded from the ledger. | workflows, accounting effects, state machines |
| `general-ledger.14.7` |  | Deleting the last entry of a chain. | accounting effects, state machines |
| `general-ledger.14.8` |  | Deleting a middle entry is refused for a clerk. | accounting effects, business rules, state machines |
| `general-ledger.14.9` |  | An accountant may delete it and leave a gap. | accounting effects |
| `general-ledger.14.10` |  | The restrictive audit trail forbids deleting a posted entry. | workflows, accounting effects, business rules, state machines |
| `general-ledger.14.11` |  | Deleting a posted item is refused. | workflows, accounting effects, business rules, state machines |
| `general-ledger.14.12` |  | The automatic choice. | workflows, state machines |
| `general-ledger.15.1` |  | Reversing a plain entry. | workflows, accounting effects, business rules, state machines |
| `general-ledger.15.2` |  | The reference of the reversal. |  |
| `general-ledger.15.3` |  | Under storno accounting. | accounting effects |
| `general-ledger.15.4` |  | A reversal dated in the future is scheduled. | workflows |
| `general-ledger.15.5` |  | Reversing an invoice produces a credit note. | workflows, calculations, accounting effects, state machines |
| `general-ledger.15.6` |  | Reverse and modify. | workflows, accounting effects, state machines |
| `general-ledger.15.7` |  | A selection spanning companies is refused. | business rules |
| `general-ledger.15.8` |  | A draft entry cannot be reversed. | workflows, accounting effects, business rules, state machines |
| `general-ledger.15.9` |  | The journal type must match. | workflows, accounting effects, business rules |
| `general-ledger.16.1` |  | The daily job posts a scheduled entry. | workflows, accounting effects, state machines |
| `general-ledger.16.2` |  | A failure switches the mode off. | workflows, accounting effects, business rules, state machines |
| `general-ledger.16.3` |  | A monthly recurrence produces the next occurrence. | workflows, accounting effects, state machines |
| `general-ledger.16.4` |  | The day of the month is preserved. | workflows, accounting effects, state machines |
| `general-ledger.16.5` |  | The end date stops the recurrence. | workflows, state machines |
| `general-ledger.16.6` |  | No duplicate occurrence. | workflows, state machines |
| `general-ledger.16.7` |  | Resetting to draft deletes the next draft occurrence. | workflows, state machines |
| `general-ledger.16.8` |  | A vendor document scheduled without a document date. | workflows, accounting effects, business rules, state machines |
| `general-ledger.17.1` |  | A full match of two items. | workflows, accounting effects |
| `general-ledger.17.2` |  | A partial match. | workflows, accounting effects |
| `general-ledger.17.3` |  | Several credits against one debit. | workflows, accounting effects |
| `general-ledger.17.4` |  | The pairing order follows the due date. | workflows, accounting effects |
| `general-ledger.17.5` |  | Items of different accounts are refused. | workflows, accounting effects, business rules |
| `general-ledger.17.6` |  | An account that does not allow matching is refused. | workflows, accounting effects, business rules |
| `general-ledger.17.7` |  | A liquidity account may be reconciled even without the flag. | workflows, accounting effects |
| `general-ledger.17.8` |  | Cancelled entries are refused. | workflows, accounting effects, business rules, state machines |
| `general-ledger.17.9` |  | Already reconciled items are refused. | workflows, business rules |
| `general-ledger.17.10` |  | Extending a partially matched group is allowed. | workflows |
| `general-ledger.18.1` |  | Both items in the same foreign currency. | workflows, calculations, accounting effects |
| `general-ledger.18.2` |  | Two different foreign currencies. | workflows, accounting effects |
| `general-ledger.18.3` |  | The reconciliation currency is the debit currency when both publish it. | workflows, accounting effects |
| `general-ledger.18.4` |  | An invoice in the company currency matched with a payment in a foreign currency. | workflows, calculations, accounting effects |
| `general-ledger.18.5` |  | An exchange-difference item is matched without a rate. | calculations, accounting effects |
| `general-ledger.18.6` |  | A zero-balance item with a foreign amount still participates. | calculations, accounting effects |
| `general-ledger.19.1` |  | A loss. | workflows, calculations, accounting effects |
| `general-ledger.19.2` |  | A gain. | workflows, accounting effects |
| `general-ledger.19.3` |  | The date of the exchange entry. | workflows, accounting effects |
| `general-ledger.19.4` |  | The exchange entry is posted only when both sides are posted. | workflows, accounting effects, state machines |
| `general-ledger.19.5` |  | Rounding noise is suppressed. | workflows, calculations, accounting effects |
| `general-ledger.19.6` |  | A missing exchange journal. | calculations, accounting effects |
| `general-ledger.19.7` |  | A missing loss account. | calculations, accounting effects |
| `general-ledger.20.1` |  | A partial match. | workflows |
| `general-ledger.20.2` |  | A closed group. | workflows |
| `general-ledger.20.3` |  | Merging two groups. |  |
| `general-ledger.20.4` |  | Removing the Full Reconciliation. | workflows |
| `general-ledger.20.5` |  | Removing every match. |  |
| `general-ledger.20.6` |  | An imported label. | accounting effects |
| `general-ledger.20.7` |  | Resolving imported labels. | workflows, accounting effects, state machines |
| `general-ledger.20.8` |  | An imported label on a matched item is refused. | business rules |
| `general-ledger.21.1` |  | Undoing a match restores the residuals. | workflows, accounting effects |
| `general-ledger.21.2` |  | Undoing reverses the exchange entry. | workflows, accounting effects, state machines |
| `general-ledger.21.3` |  | Undoing deletes a draft exchange entry. | workflows, accounting effects, state machines |
| `general-ledger.21.4` |  | The reversal date respects the lock dates. | workflows, accounting effects |
| `general-ledger.21.5` |  | Writing a protected field breaks the reconciliation. | workflows, calculations |
| `general-ledger.21.6` |  | Changing the account of a whole group keeps the matches. | accounting effects |
| `general-ledger.21.7` |  | Changing the account of one item of the group breaks them. | accounting effects |
| `general-ledger.21.8` |  | Undoing an entire matched group. | workflows |
| `general-ledger.22.1` |  | The date of the opening entry. | accounting effects |
| `general-ledger.22.2` |  | With no opening date. | accounting effects |
| `general-ledger.22.3` |  | The balancing item. | calculations, accounting effects |
| `general-ledger.22.4` |  | A balanced set produces no balancing item. | workflows, calculations, accounting effects |
| `general-ledger.22.5` |  | Updating an amount adjusts the balancing item. | calculations, accounting effects |
| `general-ledger.22.6` |  | Setting an amount to zero deletes its item. | calculations, accounting effects |
| `general-ledger.22.7` |  | The earnings account is created when missing. | calculations, accounting effects |
| `general-ledger.22.8` |  | Modifying a posted opening entry is refused. | workflows, calculations, accounting effects, business rules, state machines |
| `general-ledger.23.1` |  | Changing the account. | workflows, accounting effects, state machines |
| `general-ledger.23.2` |  | Two different counterparts produce two counterpart items. | workflows |
| `general-ledger.23.3` |  | A change of account never uses the "Transfer from" label with several source accounts. | workflows, accounting effects |
| `general-ledger.23.4` |  | Changing the period, full amount. | workflows, calculations, accounting effects, state machines |
| `general-ledger.23.5` |  | Changing the period, forty per cent. | calculations |
| `general-ledger.23.6` |  | The nature is deduced from the sign. | calculations, accounting effects |
| `general-ledger.23.7` |  | Mixing account types is refused. | accounting effects, business rules |
| `general-ledger.23.8` |  | A reconciled item is refused. | workflows, accounting effects, business rules |
| `general-ledger.23.9` |  | A locked target date is refused. | business rules |
| `general-ledger.23.10` |  | The percentage is bounded. | calculations |
| `general-ledger.24.1` |  | A journal of a parent company is visible from a subsidiary. | accounting effects |
| `general-ledger.24.2` |  | An entry of a parent company is not visible from a subsidiary. | accounting effects |
| `general-ledger.24.3` |  | The company of an entry follows the journal. | accounting effects |
| `general-ledger.24.4` |  | An account shared by two companies keeps one balance per company view. | calculations, accounting effects |
| `general-ledger.24.5` |  | Reconciling across companies is refused. | workflows, business rules |
| `general-ledger.24.6` |  | The fiscal year is delegated to the root company. |  |
| `general-ledger.24.7` |  | Loading a chart template cascades. |  |
| `general-ledger.25.1` |  | A read-only user cannot create an entry. | accounting effects, business rules |
| `general-ledger.25.2` |  | A billing clerk can create and post. | workflows |
| `general-ledger.25.3` |  | A billing clerk cannot create an account. | accounting effects, business rules |
| `general-ledger.25.4` |  | A portal user sees only its own documents. | workflows, state machines |
| `general-ledger.25.5` |  | The account code is hidden from a user without the read group. | accounting effects |
| `general-ledger.25.6` |  | Revoking an exception requires the administrator group. |  |
| `general-ledger.25.7` |  | Resequencing requires the administrator group. | business rules |
| `general-ledger.25.8` |  | Typing a non-conforming number. | accounting effects, business rules |
| `general-ledger.26.1` |  | A currency with a five-cent step. | calculations |
| `general-ledger.26.2` |  | Half away from zero. | calculations |
| `general-ledger.26.3` |  | A currency with no subdivision. | calculations |
| `general-ledger.26.4` |  | The balance invariant uses the company rounding step. | calculations, accounting effects |
| `general-ledger.26.5` |  | A conversion at a rate that does not divide evenly. | calculations |
| `general-ledger.26.6` |  | The residual reaches exactly zero. | workflows, accounting effects |
| `general-ledger.26.7` |  | A residual below the rounding step counts as zero. | calculations, accounting effects |
| `general-ledger.26.8` |  | The tolerance range uses half the step of the source currency. | calculations |

## inventory-valuation-and-costing

| Reference | Group | Scenario | Exercises |
|---|---|---|---|
| `inventory-valuation-and-costing.A1` | A — Average cost | Average cost after receipts of ten at ten and ten at twelve, then a delivery of five | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.A2` | A — Average cost | The same sequence under perpetual valuation with no location valuation account | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.A3` | A — Average cost | The closing entry after scenario A1 | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.A4` | A — Average cost | A second closing after a further delivery | workflows, accounting effects, state machines |
| `inventory-valuation-and-costing.A5` | A — Average cost | Average cost is not disturbed by an outgoing movement | workflows, calculations |
| `inventory-valuation-and-costing.A6` | A — Average cost | Average cost with a receipt while nothing is on hand | workflows, calculations |
| `inventory-valuation-and-costing.A7` | A — Average cost | Two receipts validated in one batch | workflows, calculations |
| `inventory-valuation-and-costing.A8` | A — Average cost | A receipt whose quantity is later reduced | workflows, calculations |
| `inventory-valuation-and-costing.A9` | A — Average cost | Average cost with a consigned receipt | workflows, calculations |
| `inventory-valuation-and-costing.A10` | A — Average cost | The full average worked sequence | calculations |
| `inventory-valuation-and-costing.B1` | B — First in first out | First in first out with receipts of ten at ten and ten at twelve, then a delivery of fifteen | workflows, calculations |
| `inventory-valuation-and-costing.B2` | B — First in first out | The closing entry after scenario B1 | accounting effects, state machines |
| `inventory-valuation-and-costing.B3` | B — First in first out | The textbook first in first out sequence | workflows |
| `inventory-valuation-and-costing.B4` | B — First in first out | Non-integer quantities under first in first out | workflows, calculations |
| `inventory-valuation-and-costing.B5` | B — First in first out | Increasing the quantity of an already-completed receipt | workflows, calculations |
| `inventory-valuation-and-costing.B6` | B — First in first out | First in first out remaining value with a corrected movement value | workflows, calculations |
| `inventory-valuation-and-costing.C1` | C — Standard price | A standard price moved from ten to twelve with thirty on hand | calculations, accounting effects |
| `inventory-valuation-and-costing.C2` | C — Standard price | The same change viewed as of a past date | calculations, state machines |
| `inventory-valuation-and-costing.C3` | C — Standard price | A standard price change on a product with nothing on hand | calculations |
| `inventory-valuation-and-costing.C4` | C — Standard price | Writing the same unit cost again | calculations |
| `inventory-valuation-and-costing.C5` | C — Standard price | Writing a unit cost on a first in first out product | calculations |
| `inventory-valuation-and-costing.C6` | C — Standard price | A product created with a unit cost | calculations |
| `inventory-valuation-and-costing.C7` | C — Standard price | Standard price at a date for a non-standard product | calculations |
| `inventory-valuation-and-costing.C8` | C — Standard price | A standard-price product valued at a date | workflows, calculations |
| `inventory-valuation-and-costing.D1` | D — Negative stock and its correction | A delivery of five before any receipt, then a receipt at eleven, under average cost | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.D2` | D — Negative stock and its correction | The same sequence under first in first out | workflows, calculations |
| `inventory-valuation-and-costing.D3` | D — Negative stock and its correction | Negative stock resolved in two receipts under first in first out | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.D4` | D — Negative stock and its correction | Delivering more than is on hand under first in first out | workflows, calculations |
| `inventory-valuation-and-costing.D5` | D — Negative stock and its correction | Negative stock under average cost recovered by editing an earlier receipt | workflows, calculations |
| `inventory-valuation-and-costing.13.333` | D — Negative stock and its correction | 3333… |  |
| `inventory-valuation-and-costing.D6` | D — Negative stock and its correction | Setting the quantity of a receipt to zero while stock is negative | calculations, state machines |
| `inventory-valuation-and-costing.E1` | E — Inventory adjustments | An adjustment of minus three under average cost | calculations, accounting effects |
| `inventory-valuation-and-costing.E2` | E — Inventory adjustments | The same adjustment under periodic valuation | workflows, accounting effects |
| `inventory-valuation-and-costing.E3` | E — Inventory adjustments | A positive adjustment | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.E4` | E — Inventory adjustments | Two products adjusted at once | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.E5` | E — Inventory adjustments | An adjustment with an accounting date | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.E6` | E — Inventory adjustments | The accounting date field is hidden for periodic products | accounting effects |
| `inventory-valuation-and-costing.F1` | F — Returns | A customer return | workflows, calculations |
| `inventory-valuation-and-costing.F2` | F — Returns | A customer return after the cost has moved | workflows, calculations |
| `inventory-valuation-and-costing.F3` | F — Returns | A return of everything | workflows |
| `inventory-valuation-and-costing.F4` | F — Returns | A return with no originating movement | workflows, calculations |
| `inventory-valuation-and-costing.F5` | F — Returns | A vendor return | workflows, calculations |
| `inventory-valuation-and-costing.F6` | F — Returns | The "update quantities on the order" switch | workflows |
| `inventory-valuation-and-costing.G1` | G — Landed costs | A landed cost of one hundred split by quantity across two products | workflows, calculations, accounting effects, business rules, state machines |
| `inventory-valuation-and-costing.G2` | G — Landed costs | The same landed cost split equally | workflows, calculations |
| `inventory-valuation-and-costing.G3` | G — Landed costs | The same landed cost split by weight | workflows, calculations |
| `inventory-valuation-and-costing.G4` | G — Landed costs | The same landed cost split by volume | workflows, calculations |
| `inventory-valuation-and-costing.G5` | G — Landed costs | The same landed cost split by current cost | workflows, calculations |
| `inventory-valuation-and-costing.G6` | G — Landed costs | A landed cost with a rounding residue | workflows, calculations |
| `inventory-valuation-and-costing.G7` | G — Landed costs | A landed cost whose goods are partly consumed | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.G8` | G — Landed costs | A landed cost on a product using standard price | calculations |
| `inventory-valuation-and-costing.G9` | G — Landed costs | A landed cost on a product using periodic valuation | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.G10` | G — Landed costs | A negative landed cost reversing an earlier one | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.G11` | G — Landed costs | A posted landed cost cannot be cancelled | workflows, calculations, business rules, state machines |
| `inventory-valuation-and-costing.G12` | G — Landed costs | A landed cost with no target | workflows, calculations, state machines |
| `inventory-valuation-and-costing.G13` | G — Landed costs | A landed cost whose split was edited so that it no longer sums | workflows, calculations, state machines |
| `inventory-valuation-and-costing.G14` | G — Landed costs | A landed cost whose cost line has no account and whose product has no expense account | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.G15` | G — Landed costs | A landed cost created from a vendor bill | workflows, calculations, state machines |
| `inventory-valuation-and-costing.G16` | G — Landed costs | A landed cost created from a vendor credit note | calculations, accounting effects |
| `inventory-valuation-and-costing.G17` | G — Landed costs | A landed cost created from a foreign-currency bill | calculations |
| `inventory-valuation-and-costing.G18` | G — Landed costs | A landed cost on a manufacturing order | calculations |
| `inventory-valuation-and-costing.G19` | G — Landed costs | A landed cost on a subcontracted receipt | workflows, calculations |
| `inventory-valuation-and-costing.G20` | G — Landed costs | A landed-cost flag on a non-service line | calculations |
| `inventory-valuation-and-costing.G21` | G — Landed costs | Turning a used landed-cost service product into a storable product | calculations, business rules |
| `inventory-valuation-and-costing.H1` | H — Cost of goods sold and deferred reco | An invoice for a partly delivered order under deferred cost recognition | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.H2` | H — Cost of goods sold and deferred reco | Invoicing in full before delivering | workflows, calculations, state machines |
| `inventory-valuation-and-costing.H3` | H — Cost of goods sold and deferred reco | A credit note reversing an invoice, first in first out | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.H4` | H — Cost of goods sold and deferred reco | A credit note reversing an invoice, standard price | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.H5` | H — Cost of goods sold and deferred reco | A credit note that is not a reversal | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.H6` | H — Cost of goods sold and deferred reco | A credit note under negative-number bookkeeping | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.H7` | H — Cost of goods sold and deferred reco | A resettable invoice | workflows, calculations, state machines |
| `inventory-valuation-and-costing.H8` | H — Cost of goods sold and deferred reco | Copying an invoice | workflows, state machines |
| `inventory-valuation-and-costing.H9` | H — Cost of goods sold and deferred reco | An invoice line with no label | workflows, state machines |
| `inventory-valuation-and-costing.H10` | H — Cost of goods sold and deferred reco | A product with no expense account | workflows, accounting effects, state machines |
| `inventory-valuation-and-costing.H11` | H — Cost of goods sold and deferred reco | A drop-shipped line | workflows, accounting effects, state machines |
| `inventory-valuation-and-costing.H12` | H — Cost of goods sold and deferred reco | A cost-of-goods-sold unit price with consignment | workflows, calculations |
| `inventory-valuation-and-costing.H13` | H — Cost of goods sold and deferred reco | A cost-of-goods-sold unit price over a delivery and a return | workflows, calculations |
| `inventory-valuation-and-costing.H14` | H — Cost of goods sold and deferred reco | A cost-of-goods-sold unit price with several products | calculations |
| `inventory-valuation-and-costing.I1` | I — Vendor bills | A bill priced differently from its receipt, first in first out | workflows, calculations, state machines |
| `inventory-valuation-and-costing.I2` | I — Vendor bills | The same under standard price with anglo-saxon accounting | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.I3` | I — Vendor bills | The same under standard price without anglo-saxon accounting | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.I4` | I — Vendor bills | A bill under periodic valuation | workflows, accounting effects, state machines |
| `inventory-valuation-and-costing.I5` | I — Vendor bills | A bill with a discount | calculations |
| `inventory-valuation-and-costing.I6` | I — Vendor bills | A bill posted before the receipt | workflows, state machines |
| `inventory-valuation-and-costing.120.0` | I — Vendor bills | 0 |  |
| `inventory-valuation-and-costing.I7` | I — Vendor bills | A bill covering more than one receipt | workflows, calculations, state machines |
| `inventory-valuation-and-costing.I8` | I — Vendor bills | A partial bill | workflows, calculations, state machines |
| `inventory-valuation-and-costing.I9` | I — Vendor bills | A vendor credit note | workflows, accounting effects, state machines |
| `inventory-valuation-and-costing.I10` | I — Vendor bills | A bill in a foreign currency | workflows, calculations, state machines |
| `inventory-valuation-and-costing.I11` | I — Vendor bills | A purchase order line whose price changes after the receipt | workflows, calculations |
| `inventory-valuation-and-costing.I12` | I — Vendor bills | A purchase order line in a different unit of measure | workflows |
| `inventory-valuation-and-costing.I13` | I — Vendor bills | A receipt in a different unit of measure with no purchase order | workflows, calculations |
| `inventory-valuation-and-costing.I14` | I — Vendor bills | A receipt whose reference unit is coarser than the movement unit | workflows, calculations |
| `inventory-valuation-and-costing.J1` | J — Manual value adjustment | Adjusting the value of one movement | calculations, accounting effects |
| `inventory-valuation-and-costing.J2` | J — Manual value adjustment | The adjustment suppresses landed costs | calculations |
| `inventory-valuation-and-costing.J3` | J — Manual value adjustment | Several adjustments |  |
| `inventory-valuation-and-costing.J4` | J — Manual value adjustment | Adjusting more than one movement at once |  |
| `inventory-valuation-and-costing.J5` | J — Manual value adjustment | The adjustment dialog's details line | calculations |
| `inventory-valuation-and-costing.J6` | J — Manual value adjustment | Adjusting a movement of a lot-valuated product | calculations |
| `inventory-valuation-and-costing.J7` | J — Manual value adjustment | An adjustment as of a past date |  |
| `inventory-valuation-and-costing.K1` | K — Valuation by lot or serial number | Enabling lot valuation while untracked stock exists | business rules |
| `inventory-valuation-and-costing.K2` | K — Valuation by lot or serial number | Enabling lot valuation cleanly | calculations |
| `inventory-valuation-and-costing.K3` | K — Valuation by lot or serial number | A receipt without a lot on a lot-valuated product | workflows |
| `inventory-valuation-and-costing.K4` | K — Valuation by lot or serial number | An outgoing movement of a lot-valuated product | workflows, calculations |
| `inventory-valuation-and-costing.K5` | K — Valuation by lot or serial number | A line with no lot on an outgoing movement of a lot-valuated product | workflows, calculations |
| `inventory-valuation-and-costing.K6` | K — Valuation by lot or serial number | The total value of a lot-valuated product | calculations |
| `inventory-valuation-and-costing.K7` | K — Valuation by lot or serial number | Writing a unit cost on a lot-valuated product | calculations |
| `inventory-valuation-and-costing.K8` | K — Valuation by lot or serial number | Writing a cost on one lot | calculations |
| `inventory-valuation-and-costing.K9` | K — Valuation by lot or serial number | The unit cost of a lot-valuated product is an aggregate | calculations |
| `inventory-valuation-and-costing.K10` | K — Valuation by lot or serial number | A lot created on a lot-valuated product | workflows, calculations |
| `inventory-valuation-and-costing.L1` | L — Perimeter and classification | An internal transfer | workflows |
| `inventory-valuation-and-costing.L2` | L — Perimeter and classification | A transfer to a company transit location | workflows |
| `inventory-valuation-and-costing.L3` | L — Perimeter and classification | A transfer to a company-less transit location | workflows |
| `inventory-valuation-and-costing.L4` | L — Perimeter and classification | A transfer through an archived internal location | workflows |
| `inventory-valuation-and-costing.L5` | L — Perimeter and classification | A movement with an unpicked line | workflows, calculations |
| `inventory-valuation-and-costing.L6` | L — Perimeter and classification | A movement whose lines are all consigned | workflows |
| `inventory-valuation-and-costing.L7` | L — Perimeter and classification | A movement with a mix of owned and consigned lines | workflows, calculations |
| `inventory-valuation-and-costing.L8` | L — Perimeter and classification | A drop shipment | workflows |
| `inventory-valuation-and-costing.L9` | L — Perimeter and classification | A stock quantity record's value | calculations |
| `inventory-valuation-and-costing.L10` | L — Perimeter and classification | A consigned stock quantity record's value | calculations |
| `inventory-valuation-and-costing.L11` | L — Perimeter and classification | A stock quantity record outside the valued perimeter | calculations |
| `inventory-valuation-and-costing.L12` | L — Perimeter and classification | Summing values in a grouped read | calculations |
| `inventory-valuation-and-costing.M1` | M — Multi-company, multi-currency and br | Two companies with different costing methods | calculations |
| `inventory-valuation-and-costing.M2` | M — Multi-company, multi-currency and br | The total value across companies | calculations |
| `inventory-valuation-and-costing.M3` | M — Multi-company, multi-currency and br | A closing in one company does not touch the other | accounting effects |
| `inventory-valuation-and-costing.M4` | M — Multi-company, multi-currency and br | The scheduled job across companies |  |
| `inventory-valuation-and-costing.M5` | M — Multi-company, multi-currency and br | A company whose closing fails during the job | accounting effects |
| `inventory-valuation-and-costing.M6` | M — Multi-company, multi-currency and br | A branch company overriding the valuation account | accounting effects |
| `inventory-valuation-and-costing.M7` | M — Multi-company, multi-currency and br | The scope-company rule | calculations |
| `inventory-valuation-and-costing.M8` | M — Multi-company, multi-currency and br | A bill in a foreign currency and the price difference | calculations |
| `inventory-valuation-and-costing.N1` | N — Manufacturing | The cost of a finished good | calculations |
| `inventory-valuation-and-costing.N2` | N — Manufacturing | The same with a by-product | calculations |
| `inventory-valuation-and-costing.N3` | N — Manufacturing | A by-product using standard price | calculations |
| `inventory-valuation-and-costing.N4` | N — Manufacturing | A by-product whose cost share rounds to zero | calculations |
| `inventory-valuation-and-costing.N5` | N — Manufacturing | The extra unit cost | calculations |
| `inventory-valuation-and-costing.N6` | N — Manufacturing | Component consumption and finished goods under perpetual valuation | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.N7` | N — Manufacturing | The labour entry | workflows, accounting effects |
| `inventory-valuation-and-costing.N8` | N — Manufacturing | The labour entry is skipped | workflows, accounting effects |
| `inventory-valuation-and-costing.N9` | N — Manufacturing | The work-in-progress entry | workflows, accounting effects, state machines |
| `inventory-valuation-and-costing.N10` | N — Manufacturing | Work-in-progress guards | workflows, calculations, accounting effects, business rules |
| `inventory-valuation-and-costing.N11` | N — Manufacturing | Work-in-progress order selection | accounting effects, state machines |
| `inventory-valuation-and-costing.N12` | N — Manufacturing | A kit product is not valued | calculations, accounting effects |
| `inventory-valuation-and-costing.N13` | N — Manufacturing | Cost of production on the inventory valuation report | calculations, accounting effects |
| `inventory-valuation-and-costing.O1` | O — Reporting | The unit cost history of an average-cost product | workflows, calculations |
| `inventory-valuation-and-costing.O2` | O — Reporting | The unit cost history excludes standard-price products' movements | calculations |
| `inventory-valuation-and-costing.O3` | O — Reporting | The unit cost history uses the reference unit | calculations |
| `inventory-valuation-and-costing.O4` | O — Reporting | The unit cost history after a costing method change | calculations |
| `inventory-valuation-and-costing.O5` | O — Reporting | The report row identifiers do not collide |  |
| `inventory-valuation-and-costing.O6` | O — Reporting | The inventory valuation report with no inventory-loss account | accounting effects |
| `inventory-valuation-and-costing.O7` | O — Reporting | The inventory valuation report at a past date | calculations, accounting effects |
| `inventory-valuation-and-costing.O8` | O — Reporting | Generate Entry passes the date only when it differs | calculations, accounting effects |
| `inventory-valuation-and-costing.O9` | O — Reporting | The forecast report's value | calculations |
| `inventory-valuation-and-costing.O10` | O — Reporting | The valuation list hides the right columns | calculations |
| `inventory-valuation-and-costing.O11` | O — Reporting | The lot column of the valuation list |  |
| `inventory-valuation-and-costing.O12` | O — Reporting | The remaining-quantity filter | calculations |
| `inventory-valuation-and-costing.P1` | P — Closing behaviour | Nothing to close | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.P2` | P — Closing behaviour | A missing journal | workflows, accounting effects |
| `inventory-valuation-and-costing.P3` | P — Closing behaviour | A missing valuation account | workflows, accounting effects |
| `inventory-valuation-and-costing.P4` | P — Closing behaviour | Closing before the last closing | workflows, state machines |
| `inventory-valuation-and-costing.P5` | P — Closing behaviour | An account with no variation account and no company expense account | workflows, accounting effects |
| `inventory-valuation-and-costing.P6` | P — Closing behaviour | An account with no variation account but a company expense account | workflows, accounting effects |
| `inventory-valuation-and-costing.P7` | P — Closing behaviour | Two closings on the same day | workflows, state machines |
| `inventory-valuation-and-costing.P8` | P — Closing behaviour | A draft closing is not an anchor | workflows, accounting effects, state machines |
| `inventory-valuation-and-costing.P9` | P — Closing behaviour | The closing register is capped | workflows |
| `inventory-valuation-and-costing.P10` | P — Closing behaviour | A location reclassification with movements in both directions | workflows, accounting effects |
| `inventory-valuation-and-costing.P11` | P — Closing behaviour | A location reclassification whose balance nets to zero | workflows, calculations |
| `inventory-valuation-and-costing.P12` | P — Closing behaviour | The continental perpetual period variation | workflows, accounting effects |
| `inventory-valuation-and-costing.P13` | P — Closing behaviour | Part three is skipped | accounting effects |
| `inventory-valuation-and-costing.Q1` | Q — Locking and dates | Back-dating a completed transfer into a locked fiscal year | workflows, business rules |
| `inventory-valuation-and-costing.Q2` | Q — Locking and dates | The sale, purchase and tax locks do not apply | workflows, calculations |
| `inventory-valuation-and-costing.Q3` | Q — Locking and dates | The hard lock applies |  |
| `inventory-valuation-and-costing.Q4` | Q — Locking and dates | A non-completed transfer's planned date | workflows |
| `inventory-valuation-and-costing.Q5` | Q — Locking and dates | The bypass parameter | workflows |
| `inventory-valuation-and-costing.Q6` | Q — Locking and dates | Editability of a completed transfer's date | workflows |
| `inventory-valuation-and-costing.R1` | R — Costing method and configuration cha | Changing a category's costing method to average | calculations |
| `inventory-valuation-and-costing.R2` | R — Costing method and configuration cha | Changing a category's costing method to first in first out | calculations |
| `inventory-valuation-and-costing.R3` | R — Costing method and configuration cha | Changing a category's costing method to standard price | calculations |
| `inventory-valuation-and-costing.R4` | R — Costing method and configuration cha | Moving a product to a category with a different costing method | calculations |
| `inventory-valuation-and-costing.R5` | R — Costing method and configuration cha | Moving a product to no category | calculations |
| `inventory-valuation-and-costing.R6` | R — Costing method and configuration cha | Switching a category from periodic to perpetual | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.R7` | R — Costing method and configuration cha | Filtering products by valuation mode |  |
| `inventory-valuation-and-costing.S1` | S — Edge cases and robustness | Deleting the valuation history of a product | calculations |
| `inventory-valuation-and-costing.S2` | S — Edge cases and robustness | A batch where only some products have an anchor |  |
| `inventory-valuation-and-costing.S3` | S — Edge cases and robustness | Adding a line to an already-completed incoming movement |  |
| `inventory-valuation-and-costing.S4` | S — Edge cases and robustness | Editing a line quantity on an already-completed outgoing movement | workflows, calculations |
| `inventory-valuation-and-costing.S5` | S — Edge cases and robustness | A movement created directly in the completed state | workflows, state machines |
| `inventory-valuation-and-costing.S6` | S — Edge cases and robustness | A movement whose product has no cost and no source of value | workflows, calculations |
| `inventory-valuation-and-costing.S7` | S — Edge cases and robustness | Reverting a movement out of the completed state | state machines |
| `inventory-valuation-and-costing.S8` | S — Edge cases and robustness | A movement with lines crossing in both directions | calculations |
| `inventory-valuation-and-costing.S9` | S — Edge cases and robustness | A zero-quantity movement | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.S10` | S — Edge cases and robustness | The landed-cost residue with several cost lines | calculations |
| `inventory-valuation-and-costing.S11` | S — Edge cases and robustness | A landed cost whose movement went negative | calculations |
| `inventory-valuation-and-costing.S12` | S — Edge cases and robustness | Adding a product to an invoice line whose display type is the injected one | workflows |
| `inventory-valuation-and-costing.S13` | S — Edge cases and robustness | Changing the currency of an invoice carrying injected items | workflows |
| `inventory-valuation-and-costing.S14` | S — Edge cases and robustness | A reversal posted to cancel another entry | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.S15` | S — Edge cases and robustness | An inventory user validating a delivery of an average-cost product | workflows, calculations, accounting effects |
| `inventory-valuation-and-costing.S16` | S — Edge cases and robustness | A user with limited access writing a unit cost | calculations, accounting effects |
| `inventory-valuation-and-costing.S17` | S — Edge cases and robustness | The value of a stock quantity record when the reference quantity is zero | calculations |
| `inventory-valuation-and-costing.S18` | S — Edge cases and robustness | Two receipts at the same instant under first in first out |  |
| `inventory-valuation-and-costing.S19` | S — Edge cases and robustness | Two valuation history records at the same instant |  |
| `inventory-valuation-and-costing.S20` | S — Edge cases and robustness | A movement of the same purchase order line dated the same instant |  |
| `inventory-valuation-and-costing.T1` | T — End-to-end conformance runs | Perpetual anglo-saxon, first in first out, purchase to sale | calculations, accounting effects |
| `inventory-valuation-and-costing.T2` | T — End-to-end conformance runs | Periodic, average cost, purchase to sale with a closing | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.T3` | T — End-to-end conformance runs | Perpetual, standard price, with a price difference and a landed cost | calculations, accounting effects |
| `inventory-valuation-and-costing.T4` | T — End-to-end conformance runs | The complete negative-stock round trip under periodic valuation | workflows, calculations, accounting effects, state machines |
| `inventory-valuation-and-costing.T5` | T — End-to-end conformance runs | A full landed cost round trip | workflows, calculations, accounting effects, state machines |

## products-and-catalog

| Reference | Group | Scenario | Exercises |
|---|---|---|---|
| `products-and-catalog.A-1` | A. Creating products and variants | Creating a product with no attributes gives exactly one variant | calculations |
| `products-and-catalog.A-2` | A. Creating products and variants | Values supplied at creation reach the variant | calculations |
| `products-and-catalog.A-3` | A. Creating products and variants | Adding two attributes generates six variants | calculations |
| `products-and-catalog.A-4` | A. Creating products and variants | Extra prices change variant prices without changing the variant set | calculations, state machines |
| `products-and-catalog.A-5` | A. Creating products and variants | An exclusion removes one variant | state machines |
| `products-and-catalog.A-6` | A. Creating products and variants | Removing the exclusion brings the variant back | state machines |
| `products-and-catalog.A-7` | A. Creating products and variants | An exclusion whose variant is referenced archives instead of deleting | workflows, state machines |
| `products-and-catalog.A-8` | A. Creating products and variants | Adding a single-value attribute does not recreate the variants | state machines |
| `products-and-catalog.A-9` | A. Creating products and variants | Making the single-value attribute multi-value creates the new variants | state machines |
| `products-and-catalog.A-10` | A. Creating products and variants | The generation ceiling is enforced | business rules |
| `products-and-catalog.A-11` | A. Creating products and variants | Exactly the limit is allowed | business rules |
| `products-and-catalog.A-12` | A. Creating products and variants | A configuration that leaves no possible variant is refused | business rules |
| `products-and-catalog.B-1` | B. Dynamic variants | A dynamic template starts with no variants | calculations |
| `products-and-catalog.B-2` | B. Dynamic variants | Ordering a combination creates exactly one variant | calculations, state machines |
| `products-and-catalog.B-3` | B. Dynamic variants | Ordering the same combination again reuses the variant | workflows, state machines |
| `products-and-catalog.B-4` | B. Dynamic variants | A second combination creates a second variant | calculations, state machines |
| `products-and-catalog.B-5` | B. Dynamic variants | An archived dynamic variant is not silently revived | workflows, state machines |
| `products-and-catalog.B-6` | B. Dynamic variants | A non-dynamic template refuses on-demand creation | workflows, business rules |
| `products-and-catalog.B-7` | B. Dynamic variants | Deleting the last variant of a dynamic template does not delete the template | state machines |
| `products-and-catalog.B-8` | B. Dynamic variants | Deleting the last variant of a non-dynamic template deletes the template |  |
| `products-and-catalog.C-1` | C. Combination possibility | The empty combination on an attribute-less template is possible | workflows |
| `products-and-catalog.C-2` | C. Combination possibility | The empty combination on a template with attributes is impossible | workflows |
| `products-and-catalog.C-3` | C. Combination possibility | A combination missing one attribute is impossible | workflows |
| `products-and-catalog.C-4` | C. Combination possibility | A combination naming a foreign attribute is impossible | workflows |
| `products-and-catalog.C-5` | C. Combination possibility | A combination naming an archived value is impossible | workflows |
| `products-and-catalog.C-6` | C. Combination possibility | A multi-checkbox attribute may contribute zero values | workflows |
| `products-and-catalog.C-7` | C. Combination possibility | A parent exclusion makes a child value impossible | workflows |
| `products-and-catalog.C-8` | C. Combination possibility | A parent exclusion with no values excludes the whole product |  |
| `products-and-catalog.C-9` | C. Combination possibility | Exclusions are completed with their inverses in the payload | accounting effects |
| `products-and-catalog.C-10` | C. Combination possibility | The archived-combination list excludes combinations that also have an active variant |  |
| `products-and-catalog.C-11` | C. Combination possibility | The closest possible combination drops the last value first | workflows |
| `products-and-catalog.C-12` | C. Combination possibility | The first possible combination follows the sequences |  |
| `products-and-catalog.C-13` | C. Combination possibility | An archived template yields no combination | calculations |
| `products-and-catalog.D-1` | D. Extra prices | Variant price extra is the sum of its members | calculations |
| `products-and-catalog.D-2` | D. Extra prices | A never-create-variant value contributes through the context | calculations |
| `products-and-catalog.D-3` | D. Extra prices | A value already on the variant is not double-counted |  |
| `products-and-catalog.D-4` | D. Extra prices | Unit conversion on the variant sales price converts the base only | calculations |
| `products-and-catalog.D-5` | D. Extra prices | Writing the variant sales price inverts that computation | calculations |
| `products-and-catalog.D-6` | D. Extra prices | The general price computation converts extras too | calculations |
| `products-and-catalog.D-7` | D. Extra prices | The cost falls back to the first variant on a template | calculations |
| `products-and-catalog.E-1` | E. Combos | A choice group's base price is the minimum item price | calculations |
| `products-and-catalog.E-2` | E. Combos | An empty choice group has a base price of zero and is refused on save | calculations, business rules |
| `products-and-catalog.E-3` | E. Combos | Prorating a combo price | calculations |
| `products-and-catalog.E-4` | E. Combos | The rounding remainder goes to the last choice group | calculations |
| `products-and-catalog.E-5` | E. Combos | Zero base prices distribute evenly | calculations |
| `products-and-catalog.E-6` | E. Combos | A combo may not contain a combo | business rules |
| `products-and-catalog.E-7` | E. Combos | A choice group may not repeat a product | business rules |
| `products-and-catalog.E-8` | E. Combos | A combo product may not have attributes | business rules |
| `products-and-catalog.E-9` | E. Combos | A product inside a combo may not become a combo | business rules |
| `products-and-catalog.E-10` | E. Combos | A combo product with no choice group is refused | business rules |
| `products-and-catalog.E-11` | E. Combos | A sellable combo may not contain an unsellable product | business rules |
| `products-and-catalog.E-12` | E. Combos | Changing the type away from combo empties the choice groups |  |
| `products-and-catalog.E-13` | E. Combos | Setting the type to combo clears the purchasable flag |  |
| `products-and-catalog.E-14` | E. Combos | Deleting a variant deletes the combo items naming it |  |
| `products-and-catalog.F-1` | F. Barcodes — the classic nomenclature | The check digit of a twelve-digit prefix | workflows |
| `products-and-catalog.F-2` | F. Barcodes — the classic nomenclature | The encoding check |  |
| `products-and-catalog.F-3` | F. Barcodes — the classic nomenclature | Parsing a weight barcode |  |
| `products-and-catalog.F-4` | F. Barcodes — the classic nomenclature | The product must be stored under the zeroed base code | state machines |
| `products-and-catalog.F-5` | F. Barcodes — the classic nomenclature | No rule matching gives the error outcome | workflows, business rules |
| `products-and-catalog.F-6` | F. Barcodes — the classic nomenclature | A whole-barcode numeric rule |  |
| `products-and-catalog.F-7` | F. Barcodes — the classic nomenclature | Rule order decides between competing rules | business rules |
| `products-and-catalog.F-8` | F. Barcodes — the classic nomenclature | Sequence beats creation order |  |
| `products-and-catalog.F-9` | F. Barcodes — the classic nomenclature | A thirteen-digit rule with a partial numeric field |  |
| `products-and-catalog.F-10` | F. Barcodes — the classic nomenclature | Pattern validation |  |
| `products-and-catalog.F-11` | F. Barcodes — the classic nomenclature | A Global Standards One rule pattern needs two groups | business rules |
| `products-and-catalog.F-12` | F. Barcodes — the classic nomenclature | Decoding a lot-level trade item uniform resource identifier | workflows |
| `products-and-catalog.F-13` | F. Barcodes — the classic nomenclature | Decoding a serialised trade item uniform resource identifier |  |
| `products-and-catalog.F-14` | F. Barcodes — the classic nomenclature | Decoding a ninety-six-bit serialised trade item drops the filter |  |
| `products-and-catalog.F-15` | F. Barcodes — the classic nomenclature | Decoding a logistic unit uniform resource identifier |  |
| `products-and-catalog.F-16` | F. Barcodes — the classic nomenclature | The shipped default nomenclature may not be deleted | business rules |
| `products-and-catalog.G-1` | G. Barcodes — the Global Standards One n | A four-element label | workflows |
| `products-and-catalog.G-2` | G. Barcodes — the Global Standards One n | Product, batch and expiry with a separator | workflows |
| `products-and-catalog.G-3` | G. Barcodes — the Global Standards One n | The same data without the separator is misread | workflows |
| `products-and-catalog.G-4` | G. Barcodes — the Global Standards One n | Placing the fixed-length field first removes the need for a separator | workflows |
| `products-and-catalog.G-5` | G. Barcodes — the Global Standards One n | A bad check digit skips the rule rather than failing | workflows |
| `products-and-catalog.G-6` | G. Barcodes — the Global Standards One n | A partially decomposable string returns nothing | workflows |
| `products-and-catalog.G-7` | G. Barcodes — the Global Standards One n | The decimal position |  |
| `products-and-catalog.G-8` | G. Barcodes — the Global Standards One n | A decimal rule whose application identifier has no digit fails loudly |  |
| `products-and-catalog.G-9` | G. Barcodes — the Global Standards One n | A measure rule that matches non-digits fails loudly | business rules |
| `products-and-catalog.G-10` | G. Barcodes — the Global Standards One n | Date conversion |  |
| `products-and-catalog.G-11` | G. Barcodes — the Global Standards One n | A date field of the wrong length skips the rule | workflows |
| `products-and-catalog.G-12` | G. Barcodes — the Global Standards One n | Symbology identifiers are stripped |  |
| `products-and-catalog.G-13` | G. Barcodes — the Global Standards One n | Search unpadding |  |
| `products-and-catalog.G-14` | G. Barcodes — the Global Standards One n | Search unpadding of a lot keeps the original comparison |  |
| `products-and-catalog.G-15` | G. Barcodes — the Global Standards One n | Unpadding an unparseable value |  |
| `products-and-catalog.G-16` | G. Barcodes — the Global Standards One n | Membership conditions are expanded |  |
| `products-and-catalog.G-17` | G. Barcodes — the Global Standards One n | An empty membership list is untouched |  |
| `products-and-catalog.G-18` | G. Barcodes — the Global Standards One n | The uniqueness check suppresses the rewrite |  |
| `products-and-catalog.H-1` | H. Barcode uniqueness | Two products may not share a barcode in one company | business rules |
| `products-and-catalog.H-2` | H. Barcode uniqueness | Products of different companies may share a barcode |  |
| `products-and-catalog.H-3` | H. Barcode uniqueness | A shared product collides with every company | business rules |
| `products-and-catalog.H-4` | H. Barcode uniqueness | Inaccessible duplicates are noted | business rules |
| `products-and-catalog.H-5` | H. Barcode uniqueness | A product barcode may not collide with a packaging barcode | business rules |
| `products-and-catalog.H-6` | H. Barcode uniqueness | A packaging barcode may not collide with a product barcode | business rules |
| `products-and-catalog.H-7` | H. Barcode uniqueness | Two packaging barcodes may not share a value, even across companies | business rules |
| `products-and-catalog.H-8` | H. Barcode uniqueness | Duplicating a product does not duplicate its barcode | business rules |
| `products-and-catalog.I-1` | I. Attributes and values — guards | The variant-creation mode is frozen once used | business rules |
| `products-and-catalog.I-2` | I. Attributes and values — guards | The mode may be changed when only archived templates use it |  |
| `products-and-catalog.I-3` | I. Attributes and values — guards | An attribute in use may not be deleted | business rules |
| `products-and-catalog.I-4` | I. Attributes and values — guards | An attribute in use may not be archived | business rules |
| `products-and-catalog.I-5` | I. Attributes and values — guards | A multi-checkbox attribute may not create variants | business rules |
| `products-and-catalog.I-6` | I. Attributes and values — guards | Choosing multi-checkbox on an unused attribute sets the mode |  |
| `products-and-catalog.I-7` | I. Attributes and values — guards | A value's attribute is frozen once the value is used | business rules |
| `products-and-catalog.I-8` | I. Attributes and values — guards | A value used on active products may not be deleted | business rules |
| `products-and-catalog.I-9` | I. Attributes and values — guards | A value used only on archived variants is archived |  |
| `products-and-catalog.I-10` | I. Attributes and values — guards | A value's display name includes the attribute |  |
| `products-and-catalog.I-11` | I. Attributes and values — guards | Selecting a never-create-variant attribute pre-selects all its values |  |
| `products-and-catalog.I-12` | I. Attributes and values — guards | Selecting any other attribute filters the existing selection |  |
| `products-and-catalog.J-1` | J. Template attribute lines and values | An active line must offer a value | business rules |
| `products-and-catalog.J-2` | J. Template attribute lines and values | A value must belong to the line's attribute | business rules |
| `products-and-catalog.J-3` | J. Template attribute lines and values | A line may not be moved to another template | business rules |
| `products-and-catalog.J-4` | J. Template attribute lines and values | A line's attribute may not be changed | business rules |
| `products-and-catalog.J-5` | J. Template attribute lines and values | Removing an attribute and adding it back restores the variants | workflows |
| `products-and-catalog.J-6` | J. Template attribute lines and values | Archiving a line clears its values |  |
| `products-and-catalog.J-7` | J. Template attribute lines and values | A value may be moved between lines of the same attribute |  |
| `products-and-catalog.J-8` | J. Template attribute lines and values | Each value appears once per line | business rules |
| `products-and-catalog.J-9` | J. Template attribute lines and values | The variant link may not be written from the value side | business rules |
| `products-and-catalog.J-10` | J. Template attribute lines and values | The underlying value and the template are frozen | business rules |
| `products-and-catalog.J-11` | J. Template attribute lines and values | Deleting a single-value line's value does not delete the variants |  |
| `products-and-catalog.K-1` | K. Naming and display | The template display name with an internal reference |  |
| `products-and-catalog.K-2` | K. Naming and display | The variant display name with a multi-value combination |  |
| `products-and-catalog.K-3` | K. Naming and display | Single-value lines are hidden from the name | state machines |
| `products-and-catalog.K-4` | K. Naming and display | No-variant values are hidden from the name |  |
| `products-and-catalog.K-5` | K. Naming and display | An archived variant keeps its historical name |  |
| `products-and-catalog.K-6` | K. Naming and display | The vendor-specific display name |  |
| `products-and-catalog.K-7` | K. Naming and display | A variant-specific vendor line wins |  |
| `products-and-catalog.K-8` | K. Naming and display | Several vendor lines produce a joined name | workflows |
| `products-and-catalog.K-9` | K. Naming and display | The category complete name |  |
| `products-and-catalog.K-10` | K. Naming and display | Category cycles are refused | business rules |
| `products-and-catalog.K-11` | K. Naming and display | Staged name search | workflows |
| `products-and-catalog.K-12` | K. Naming and display | The template name search covers the variants | workflows |
| `products-and-catalog.L-1` | L. Images | A variant with no image falls back to the template |  |
| `products-and-catalog.L-2` | L. Images | A variant with its own image does not fall back |  |
| `products-and-catalog.L-3` | L. Images | Writing an image on the only variant writes it on the template |  |
| `products-and-catalog.L-4` | L. Images | Writing an image on one of several variants writes it on the variant |  |
| `products-and-catalog.L-5` | L. Images | Clearing an already-empty image clears the template |  |
| `products-and-catalog.L-6` | L. Images | Deleting the last variant moves its image up |  |
| `products-and-catalog.L-7` | L. Images | Changing the template image invalidates the variants' caches | workflows |
| `products-and-catalog.L-8` | L. Images | Zoomability |  |
| `products-and-catalog.M-1` | M. Documents | Uploading through the route | workflows, state machines |
| `products-and-catalog.M-2` | M. Documents | An invalid owner model is rejected | business rules |
| `products-and-catalog.M-3` | M. Documents | A caller without write access is rejected | business rules |
| `products-and-catalog.M-4` | M. Documents | One failing file does not stop the others |  |
| `products-and-catalog.M-5` | M. Documents | A file in the message thread becomes a document | business rules |
| `products-and-catalog.M-6` | M. Documents | A bad web address is refused | business rules |
| `products-and-catalog.M-7` | M. Documents | Deleting a document deletes its attachment |  |
| `products-and-catalog.M-8` | M. Documents | The template's document count includes the variants' |  |
| `products-and-catalog.N-1` | N. Labels | Printing labels for a service is refused | business rules |
| `products-and-catalog.N-2` | N. Labels | A non-positive copy count is refused | workflows, calculations, business rules |
| `products-and-catalog.N-3` | N. Labels | An empty selection is refused | workflows, business rules |
| `products-and-catalog.N-4` | N. Labels | Grid dimensions and page counts |  |
| `products-and-catalog.N-5` | N. Labels | The no-price document is chosen for the no-price format | calculations |
| `products-and-catalog.O-1` | O. Expiry | Receiving creates the four dates |  |
| `products-and-catalog.O-2` | O. Expiry | Postponing the expiration shifts the other three | workflows, calculations, state machines |
| `products-and-catalog.O-3` | O. Expiry | An empty derived date stays empty under a shift | workflows |
| `products-and-catalog.O-4` | O. Expiry | Clearing the product flag clears the derived dates |  |
| `products-and-catalog.O-5` | O. Expiry | Turning off tracking turns off expiry |  |
| `products-and-catalog.O-6` | O. Expiry | The expiry alert flag |  |
| `products-and-catalog.O-7` | O. Expiry | Expired stock is not available | calculations |
| `products-and-catalog.O-8` | O. Expiry | The reminder fires once |  |
| `products-and-catalog.O-9` | O. Expiry | A lot with no internal stock is marked reminded without an activity |  |
| `products-and-catalog.O-10` | O. Expiry | Delivering an expired lot opens the dialog | workflows |
| `products-and-catalog.O-11` | O. Expiry | Several expired lots give the generic message | business rules |
| `products-and-catalog.O-12` | O. Expiry | Confirming proceeds | workflows |
| `products-and-catalog.O-13` | O. Expiry | Discarding removes the expired lines | workflows |
| `products-and-catalog.O-14` | O. Expiry | A line whose removal date is exactly now triggers the dialog but is not discarded |  |
| `products-and-catalog.O-15` | O. Expiry | Move-line date defaults | workflows |
| `products-and-catalog.O-16` | O. Expiry | First expiry first out ordering | calculations, accounting effects |
| `products-and-catalog.O-17` | O. Expiry | The forecast "to remove" figure |  |
| `products-and-catalog.P-1` | P. Import and export | A combined import creates templates, attributes, values and variants | calculations |
| `products-and-catalog.P-2` | P. Import and export | Empty cells are filled from the template's first variant | calculations |
| `products-and-catalog.P-3` | P. Import and export | A row with product values but no name is refused | business rules |
| `products-and-catalog.P-4` | P. Import and export | A product value with no attribute part is refused | business rules |
| `products-and-catalog.P-5` | P. Import and export | The same attribute twice in one row is refused | business rules |
| `products-and-catalog.P-6` | P. Import and export | The same pair twice in one row is refused | business rules |
| `products-and-catalog.P-7` | P. Import and export | Re-importing an existing variant with a different combination is refused | business rules |
| `products-and-catalog.P-8` | P. Import and export | Re-importing the same combination asserts without blanking |  |
| `products-and-catalog.P-9` | P. Import and export | Exporting a combination round-trips | calculations |
| `products-and-catalog.P-10` | P. Import and export | A direct variant import with product values is redirected | workflows |
| `products-and-catalog.Q-1` | Q. The catalog contract | The default product filter |  |
| `products-and-catalog.Q-2` | Q. The catalog contract | Products already on the document report their line data | calculations |
| `products-and-catalog.Q-3` | Q. The catalog contract | Products not on the document report the defaults | calculations |
| `products-and-catalog.Q-4` | Q. The catalog contract | Typing a quantity calls the update route | calculations |
| `products-and-catalog.Q-5` | Q. The catalog contract | Both routes act as the document's company |  |
| `products-and-catalog.R-1` | R. The matrix | The grid shape |  |
| `products-and-catalog.R-2` | R. The matrix | The enumeration order |  |
| `products-and-catalog.R-3` | R. The matrix | A row header for several remaining lines | calculations |
| `products-and-catalog.R-4` | R. The matrix | A template with one attribute line |  |
| `products-and-catalog.R-5` | R. The matrix | Extras hidden | calculations |
| `products-and-catalog.S-1` | S. Multi-company and currency | A shared template uses the main company's currency | calculations, state machines |
| `products-and-catalog.S-2` | S. Multi-company and currency | A company-scoped template uses its own company's currency | calculations |
| `products-and-catalog.S-3` | S. Multi-company and currency | The cost is per company | calculations |
| `products-and-catalog.S-4` | S. Multi-company and currency | Writing the cost on a multi-variant template does nothing | calculations |
| `products-and-catalog.S-5` | S. Multi-company and currency | The company record rule |  |
| `products-and-catalog.S-6` | S. Multi-company and currency | A new company gets the default nomenclature |  |
| `products-and-catalog.S-7` | S. Multi-company and currency | Installing the barcode capability back-fills nomenclatures |  |
| `products-and-catalog.T-1` | T. Settings and parameters | The weight unit |  |
| `products-and-catalog.T-2` | T. Settings and parameters | One parameter governs volume and length |  |
| `products-and-catalog.T-3` | T. Settings and parameters | Turning off pricelists archives them | calculations |
| `products-and-catalog.T-4` | T. Settings and parameters | Turning on pricelists bootstraps a default per company | calculations |
| `products-and-catalog.T-5` | T. Settings and parameters | An existing rule-free default is revived rather than duplicated | calculations |
| `products-and-catalog.T-6` | T. Settings and parameters | Changing a company's currency creates the pricelist with the new currency | calculations |
| `products-and-catalog.T-7` | T. Settings and parameters | Archiving a currency archives its pricelists | calculations |
| `products-and-catalog.U-1` | U. Access and visibility | An internal user may read but not change |  |
| `products-and-catalog.U-2` | U. Access and visibility | An internal user may use the label dialog |  |
| `products-and-catalog.U-3` | U. Access and visibility | The cost is hidden from non-internal readers | calculations |
| `products-and-catalog.U-4` | U. Access and visibility | The bulk-update dialog cannot be deleted | business rules |
| `products-and-catalog.U-5` | U. Access and visibility | Nomenclatures are administered by the system administration group |  |
| `products-and-catalog.U-6` | U. Access and visibility | The variant group reveals the variant interface |  |
| `products-and-catalog.V-1` | V. Cross-cutting invariants | Every active combination has at most one active variant |  |
| `products-and-catalog.V-2` | V. Cross-cutting invariants | A template always has at least one variant unless it is dynamic or archived |  |
| `products-and-catalog.V-3` | V. Cross-cutting invariants | The sum of prorated combo shares equals the combo price | calculations |
| `products-and-catalog.V-4` | V. Cross-cutting invariants | A barcode identifies at most one thing per company |  |
| `products-and-catalog.V-5` | V. Cross-cutting invariants | A parse is total or nothing under the Global Standards One nomenclature | calculations |
| `products-and-catalog.V-6` | V. Cross-cutting invariants | Derived expiry dates keep their offsets under a postponement | workflows |
| `products-and-catalog.V-7` | V. Cross-cutting invariants | The catalog posts nothing | workflows |

## purchasing

| Reference | Group | Scenario | Exercises |
|---|---|---|---|
| `purchasing.1.1` | Common fixture | A new request for quotation gets a reference and defaults |  |
| `purchasing.1.2` | Common fixture | Choosing a vendor applies its defaults |  |
| `purchasing.1.3` | Common fixture | Adding a product suggests a quantity and a price | calculations |
| `purchasing.1.4` | Common fixture | A vendor price break at one hundred units | calculations |
| `purchasing.1.5` | Common fixture | The price falls back to the product cost | calculations |
| `purchasing.1.6` | Common fixture | A manual price survives when the vendor sells the product at no listed price | calculations |
| `purchasing.1.7` | Common fixture | Sections, subsections and notes |  |
| `purchasing.1.8` | Common fixture | A line that is neither a display line nor a down payment must be complete |  |
| `purchasing.1.9` | Common fixture | Header amounts | calculations |
| `purchasing.1.10` | Common fixture | The header expected arrival is the earliest line arrival |  |
| `purchasing.2.1` | Common fixture | Sending moves a draft order to sent | workflows, state machines |
| `purchasing.2.2` | Common fixture | Sending an already sent order does not change the status | workflows, state machines |
| `purchasing.2.3` | Common fixture | Printing the quotation also marks a draft order as sent | state machines |
| `purchasing.2.4` | Common fixture | Confirmation refuses a line with no product | workflows, business rules |
| `purchasing.2.5` | Common fixture | Confirmation under a one-step policy goes straight through | workflows |
| `purchasing.2.6` | Common fixture | An approval required above five thousand |  |
| `purchasing.2.7` | Common fixture | The threshold comparison is strict |  |
| `purchasing.2.8` | Common fixture | The threshold in a foreign currency |  |
| `purchasing.2.9` | Common fixture | Approval locks under the lock policy |  |
| `purchasing.2.10` | Common fixture | Vendor price learning | calculations |
| `purchasing.2.11` | Common fixture | Vendor price learning on a contact address | calculations |
| `purchasing.2.12` | Common fixture | The ten-entry cap | accounting effects |
| `purchasing.2.13` | Common fixture | Price restatement when learning from a different unit | calculations, state machines |
| `purchasing.3.1` | Common fixture | A request for quotation of two lines becoming an order and a receipt |  |
| `purchasing.3.2` | Common fixture | An order of services creates no receipt |  |
| `purchasing.3.3` | Common fixture | Adding a line to a confirmed order extends the receipt | workflows |
| `purchasing.3.4` | Common fixture | Raising a quantity on a confirmed order | workflows, calculations |
| `purchasing.3.5` | Common fixture | Lowering a quantity on a confirmed order raises an exception | workflows, calculations |
| `purchasing.3.6` | Common fixture | The vendor must have a supplier location |  |
| `purchasing.3.7` | Common fixture | A transfer created with no move is removed | workflows |
| `purchasing.3.8` | Common fixture | Changing a line's expected arrival moves the deadline |  |
| `purchasing.3.9` | Common fixture | Changing a line's unit price re-prices the move | calculations |
| `purchasing.4.1` | Common fixture | A partial receipt then a bill on received quantities | workflows |
| `purchasing.4.2` | Common fixture | A bill on ordered quantities before receipt |  |
| `purchasing.4.3` | Common fixture | A return then a refund | workflows |
| `purchasing.4.4` | Common fixture | A return not flagged to be refunded changes nothing commercially | workflows |
| `purchasing.4.5` | Common fixture | Grouping several orders into one bill |  |
| `purchasing.4.6` | Common fixture | Sections are only emitted when something follows them |  |
| `purchasing.4.7` | Common fixture | An order with only sections produces an empty bill | workflows |
| `purchasing.4.8` | Common fixture | Attachments are refused when several bills are produced | workflows, business rules |
| `purchasing.4.9` | Common fixture | Billing status falls back when a bill is deleted | state machines |
| `purchasing.4.10` | Common fixture | A cancelled bill does not count | workflows, state machines |
| `purchasing.4.11` | Common fixture | Down payments |  |
| `purchasing.5.1` | Common fixture | Cancelling an order with an open receipt | workflows |
| `purchasing.5.2` | Common fixture | Cancelling an order whose receipt is already done | workflows, state machines |
| `purchasing.5.3` | Common fixture | A posted bill blocks cancellation | workflows, state machines |
| `purchasing.5.4` | Common fixture | A draft bill does not block cancellation | workflows, state machines |
| `purchasing.5.5` | Common fixture | A locked order cannot be cancelled | workflows, business rules, state machines |
| `purchasing.5.6` | Common fixture | Deletion requires cancellation | workflows |
| `purchasing.5.7` | Common fixture | A line of a confirmed order cannot be deleted | workflows, business rules |
| `purchasing.5.8` | Common fixture | The display type cannot change | business rules |
| `purchasing.5.9` | Common fixture | Resetting a confirmed order to draft unwinds nothing | workflows, state machines |
| `purchasing.5.10` | Common fixture | Duplicating an order |  |
| `purchasing.6.1` | Common fixture | A blanket agreement with a fixed price used by two orders | calculations |
| `purchasing.6.2` | Common fixture | A purchase template copies quantities |  |
| `purchasing.6.3` | Common fixture | Confirming an empty agreement is refused | workflows, business rules |
| `purchasing.6.4` | Common fixture | Confirming a blanket order with a zero price is refused | workflows, calculations, business rules |
| `purchasing.6.5` | Common fixture | Confirming a blanket order with a zero quantity is refused | workflows, calculations, business rules |
| `purchasing.6.6` | Common fixture | A zero price cannot be written on a confirmed blanket order | workflows, calculations, business rules |
| `purchasing.6.7` | Common fixture | Changing the agreed price propagates | calculations |
| `purchasing.6.8` | Common fixture | The end date must not precede the start date | business rules |
| `purchasing.6.9` | Common fixture | The type cannot change once the agreement leaves draft | business rules, state machines |
| `purchasing.6.10` | Common fixture | Changing the type of a draft agreement renumbers it | state machines |
| `purchasing.6.11` | Common fixture | Cancelling an agreement cancels its draft requests | workflows, state machines |
| `purchasing.6.12` | Common fixture | Deleting an agreement |  |
| `purchasing.6.13` | Common fixture | The duplicate blanket order warning |  |
| `purchasing.6.14` | Common fixture | An agreement's own vendor prices are exclusive | calculations |
| `purchasing.7.1` | Common fixture | A call for tenders with three alternatives where one is chosen |  |
| `purchasing.7.2` | Common fixture | Choosing a winner when a loser is already confirmed | workflows |
| `purchasing.7.3` | Common fixture | Choosing when there is nothing to clear |  |
| `purchasing.7.4` | Common fixture | A group of one deletes itself |  |
| `purchasing.7.5` | Common fixture | Ties keep every tied line |  |
| `purchasing.8.1` | Common fixture | A reminder sent two days before the planned date | calculations |
| `purchasing.8.2` | Common fixture | Goods received before the reminder date | workflows |
| `purchasing.8.3` | Common fixture | An order of only services is never reminded |  |
| `purchasing.8.4` | Common fixture | An order mixing a service and a good is reminded |  |
| `purchasing.8.5` | Common fixture | The reminder preview |  |
| `purchasing.8.6` | Common fixture | The reminder privilege gates everything |  |
| `purchasing.8.7` | Common fixture | The vendor updates expected arrivals through the portal |  |
| `purchasing.8.8` | Common fixture | The vendor updates dates when the receipt is already validated | workflows |
| `purchasing.8.9` | Common fixture | A malformed date is skipped |  |
| `purchasing.8.10` | Common fixture | A line identifier from another order redirects |  |
| `purchasing.8.11` | Common fixture | The portal lists |  |
| `purchasing.8.12` | Common fixture | The portal serves the right document |  |
| `purchasing.9.1` | Common fixture | Two mergeable requests |  |
| `purchasing.9.2` | Common fixture | Arrivals more than a day apart do not merge |  |
| `purchasing.9.3` | Common fixture | Fewer than two mergeable records |  |
| `purchasing.9.4` | Common fixture | Records that do not share a merge key |  |
| `purchasing.9.5` | Common fixture | Sources and vendor references are concatenated |  |
| `purchasing.10.1` | Common fixture | Creating a bill from selected order lines |  |
| `purchasing.10.2` | Common fixture | Positional matching |  |
| `purchasing.10.3` | Common fixture | Unmatched rows on a single bill |  |
| `purchasing.10.4` | Common fixture | No order line selected |  |
| `purchasing.10.5` | Common fixture | Adding bill lines to a purchase order |  |
| `purchasing.10.6` | Common fixture | Adding bill lines with no product |  |
| `purchasing.10.7` | Common fixture | Two vendors selected |  |
| `purchasing.10.8` | Common fixture | Two orders involved |  |
| `purchasing.10.9` | Common fixture | Automatic matching, exact total | calculations |
| `purchasing.10.10` | Common fixture | Automatic matching, ambiguous subset |  |
| `purchasing.10.11` | Common fixture | Automatic matching, unique subset |  |
| `purchasing.10.12` | Common fixture | Automatic matching by vendor and amount | calculations |
| `purchasing.10.13` | Common fixture | Auto-completing a bill from an order |  |
| `purchasing.11.1` | Common fixture | A reordering rule creates a request for quotation |  |
| `purchasing.11.2` | Common fixture | No vendor found, from a reordering rule |  |
| `purchasing.11.3` | Common fixture | No vendor found, from a sales order |  |
| `purchasing.11.4` | Common fixture | Two needs merge onto one line |  |
| `purchasing.11.5` | Common fixture | A second need extends an existing line |  |
| `purchasing.11.6` | Common fixture | Daily grouping |  |
| `purchasing.11.7` | Common fixture | Weekly grouping with a target week day |  |
| `purchasing.11.8` | Common fixture | The order deadline is pulled back for a new line |  |
| `purchasing.11.9` | Common fixture | Cancelling a procurement-driven order | workflows |
| `purchasing.12.1` | Common fixture | A sold service generates a request for quotation | calculations |
| `purchasing.12.2` | Common fixture | A second sold service joins the same order |  |
| `purchasing.12.3` | Common fixture | Raising the sold quantity while the purchase is still a draft | calculations, state machines |
| `purchasing.12.4` | Common fixture | Raising the sold quantity after the purchase is confirmed | workflows, calculations |
| `purchasing.12.5` | Common fixture | Lowering the sold quantity | calculations |
| `purchasing.12.6` | Common fixture | No vendor for a subcontracted service |  |
| `purchasing.12.7` | Common fixture | The subcontract flag is guarded |  |
| `purchasing.12.8` | Common fixture | Cancelling the purchase warns the sales order | workflows |
| `purchasing.13.1` | Common fixture | A kit is received in components | workflows |
| `purchasing.13.2` | Common fixture | Kit cost shares must total 100 | calculations |
| `purchasing.13.3` | Common fixture | Cost shares split the billed value | calculations |
| `purchasing.14.1` | Common fixture | Order amounts in a foreign currency | calculations |
| `purchasing.14.2` | Common fixture | The value handed to the stock move |  |
| `purchasing.14.3` | Common fixture | Comparing offers in two currencies |  |
| `purchasing.14.4` | Common fixture | Global tax rounding | calculations |
| `purchasing.14.5` | Common fixture | Products of another company are refused | business rules |
| `purchasing.14.6` | Common fixture | Record rules hide other companies' orders |  |
| `purchasing.14.7` | Common fixture | A portal user sees only their own orders |  |
| `purchasing.15.1` | Common fixture | Purchase analysis quantities are in the product unit |  |
| `purchasing.15.2` | Common fixture | The average price is re-weighted when grouped | calculations |
| `purchasing.15.3` | Common fixture | Days to confirm and days to receive | workflows, calculations |
| `purchasing.15.4` | Common fixture | The on-time delivery rate | workflows, calculations |
| `purchasing.15.5` | Common fixture | The weighted vendor delay rate | calculations |
| `purchasing.15.6` | Common fixture | The dashboard days-to-order figure | calculations |
| `purchasing.15.7` | Common fixture | The dashboard on-time figure |  |
| `purchasing.16.1` | Common fixture | A suggestion from thirty-day demand |  |
| `purchasing.16.2` | Common fixture | A suggestion that rounds to zero | calculations |
| `purchasing.16.3` | Common fixture | A suggestion from actual demand |  |
| `purchasing.16.4` | Common fixture | Applying the suggestion collapses existing lines |  |
| `purchasing.16.5` | Common fixture | The suggestion parameters are remembered |  |
| `purchasing.17.1` | Common fixture | Goods received not billed | workflows |
| `purchasing.17.2` | Common fixture | Billed not received | workflows |
| `purchasing.17.3` | Common fixture | Guards |  |
| `purchasing.17.4` | Common fixture | The analytic blend on the balancing item |  |
| `purchasing.18.1` | Common fixture | A positive price difference | calculations |
| `purchasing.18.2` | Common fixture | A discount suppresses the adjustment | calculations |
| `purchasing.18.3` | Common fixture | A refund reverses the adjustment | workflows |
| `purchasing.19.1` | Common fixture | The vendor warning |  |
| `purchasing.19.2` | Common fixture | The parent company's warning |  |
| `purchasing.19.3` | Common fixture | A product warning |  |
| `purchasing.19.4` | Common fixture | Duplicates are removed and the privilege gates everything |  |
| `purchasing.19.5` | Common fixture | The duplicate order warning |  |
| `purchasing.20.1` | Common fixture | A grid creates one line per non-empty cell |  |
| `purchasing.20.2` | Common fixture | A grid refuses to change a product on two lines | business rules |
| `purchasing.20.3` | Common fixture | A grid cell set to zero |  |
| `purchasing.20.4` | Common fixture | Exporting a structured order |  |
| `purchasing.20.5` | Common fixture | Importing a structured order |  |
| `purchasing.20.6` | Common fixture | Grouping bill lines by tax is refused on a matched bill | calculations, business rules |
| `purchasing.21.1` | Common fixture | A quantity to bill that is not exactly zero | calculations |
| `purchasing.21.2` | Common fixture | A quantity to bill of one hundredth | calculations |
| `purchasing.21.3` | Common fixture | A generated document whose total is exactly zero | calculations |
| `purchasing.21.4` | Common fixture | A move unit price rounding | calculations |
| `purchasing.21.5` | Common fixture | A unit conversion rounding half up | calculations |
| `purchasing.22.1` | Common fixture | A purchase user may work with vendor bills only |  |
| `purchasing.22.2` | Common fixture | A purchase administrator may manage vendor pricelists | calculations |
| `purchasing.22.3` | Common fixture | An inventory user may read orders |  |
| `purchasing.22.4` | Common fixture | A portal user may not create an order |  |
| `purchasing.22.5` | Common fixture | The dashboard refuses a non-internal caller | business rules |
| `purchasing.22.6` | Common fixture | Agreements are readable but not writable by the administrator row alone |  |

## sales

| Reference | Group | Scenario | Exercises |
|---|---|---|---|
| `sales.A1` | A — Creating and pricing a quotation | Defaults on a new quotation |  |
| `sales.A2` | A — Creating and pricing a quotation | Company default validity of zero disables expiration |  |
| `sales.A3` | A — Creating and pricing a quotation | The expiration date can be overridden |  |
| `sales.A4` | A — Creating and pricing a quotation | A quotation template replaces the lines | calculations |
| `sales.A5` | A — Creating and pricing a quotation | Changing the customer re-applies an untouched template |  |
| `sales.A6` | A — Creating and pricing a quotation | Changing the customer does not re-apply a modified template | calculations |
| `sales.A7` | A — Creating and pricing a quotation | Line description of a configurable product |  |
| `sales.A8` | A — Creating and pricing a quotation | Price list rule with the discount feature enabled | calculations |
| `sales.A9` | A — Creating and pricing a quotation | The same rule with the discount feature disabled | calculations |
| `sales.A10` | A — Creating and pricing a quotation | A surcharge is folded into the price | calculations |
| `sales.A11` | A — Creating and pricing a quotation | A manually typed price is not recomputed | calculations |
| `sales.A12` | A — Creating and pricing a quotation | "Update Prices" overrides a manual price | workflows, calculations |
| `sales.A13` | A — Creating and pricing a quotation | A price is not recomputed once something is invoiced | workflows, calculations |
| `sales.A14` | A — Creating and pricing a quotation | Combo price proration | calculations |
| `sales.A15` | A — Creating and pricing a quotation | Combo price residue lands on the last choice | calculations |
| `sales.A16` | A — Creating and pricing a quotation | Combo choices with zero base prices are split evenly | calculations |
| `sales.A17` | A — Creating and pricing a quotation | A combo line carries no tax | calculations |
| `sales.B1` | B — Amounts and totals | Three lines with a ten percent discount (mandatory worked example, part one) | calculations |
| `sales.B2` | B — Amounts and totals | Tax included in the price | calculations |
| `sales.B3` | B — Amounts and totals | Early payment discount in the mixed mode | calculations |
| `sales.B4` | B — Amounts and totals | Per-line and global tax rounding | calculations |
| `sales.B5` | B — Amounts and totals | Amount before discount excludes special lines | calculations |
| `sales.B6` | B — Amounts and totals | Margin | calculations |
| `sales.B7` | B — Amounts and totals | Margin on a line delivered but never ordered | workflows, calculations |
| `sales.C1` | C — Confirmation | Three lines with a ten percent discount confirmed into a delivery (mandatory worked example, part two) | workflows, calculations |
| `sales.C2` | C — Confirmation | Confirmation with the shipping policy "when all products are ready" | workflows, calculations |
| `sales.C3` | C — Confirmation | Confirmation is refused when a line has no product | workflows, calculations, business rules |
| `sales.C4` | C — Confirmation | Confirmation is refused on a confirmed order | workflows, business rules, state machines |
| `sales.C5` | C — Confirmation | A confirmation with the lock feature enabled | workflows |
| `sales.C6` | C — Confirmation | A service line creating a task (mandatory worked example) | workflows |
| `sales.C7` | C — Confirmation | A second service line reuses the project | workflows, calculations |
| `sales.C8` | C — Confirmation | A service line with a global project but no project configured | workflows |
| `sales.C9` | C — Confirmation | A service line that must be bought | workflows |
| `sales.C10` | C — Confirmation | A service that must be bought but has no vendor | workflows |
| `sales.C11` | C — Confirmation | Re-confirmation does not duplicate downstream documents | workflows, state machines |
| `sales.C12` | C — Confirmation | Confirmation sends no message by default | workflows, business rules, state machines |
| `sales.C13` | C — Confirmation | A quotation template with its own confirmation message | workflows, business rules |
| `sales.C14` | C — Confirmation | Asynchronous message sending | workflows, business rules |
| `sales.D1` | D — Invoicing on ordered quantities | Full invoice of the three-line order | workflows, calculations, state machines |
| `sales.D2` | D — Invoicing on ordered quantities | Invoicing twice produces a second invoice | workflows, business rules |
| `sales.D3` | D — Invoicing on ordered quantities | Deleting the draft invoice restores the order | workflows, calculations, state machines |
| `sales.D4` | D — Invoicing on ordered quantities | Grouping two orders onto one invoice | workflows |
| `sales.D5` | D — Invoicing on ordered quantities | Consolidated billing off | workflows |
| `sales.D6` | D — Invoicing on ordered quantities | Orders with different currencies are never grouped | workflows |
| `sales.D7` | D — Invoicing on ordered quantities | A section with nothing invoiceable is not carried over | workflows |
| `sales.D8` | D — Invoicing on ordered quantities | A section with something invoiceable is carried over once | workflows |
| `sales.D9` | D — Invoicing on ordered quantities | An order made only of sections and notes is skipped | workflows |
| `sales.D10` | D — Invoicing on ordered quantities | A discount line cannot be invoiced alone | workflows, calculations, business rules, state machines |
| `sales.E1` | E — Advance invoices | An advance invoice of thirty percent then a final invoice deducting it (mandatory worked example) | workflows, calculations, accounting effects, state machines |
| `sales.E2` | E — Advance invoices | A fixed advance | workflows, calculations |
| `sales.E3` | E — Advance invoices | An advance amount that does not divide evenly | calculations |
| `sales.E4` | E — Advance invoices | Two successive advances | workflows, calculations, state machines |
| `sales.E5` | E — Advance invoices | An advance on an order with nothing invoiceable | workflows, calculations |
| `sales.E6` | E — Advance invoices | A cancelled advance invoice | workflows, calculations, state machines |
| `sales.E7` | E — Advance invoices | A deleted advance invoice removes the advance lines | workflows, state machines |
| `sales.E8` | E — Advance invoices | An advance that was reversed and re-issued | workflows, state machines |
| `sales.E9` | E — Advance invoices | Advance invoicing requires one order | business rules |
| `sales.E10` | E — Advance invoices | A non-positive advance amount | workflows, calculations |
| `sales.F1` | F — Invoicing on delivered quantities | Invoicing on delivered quantities after a partial delivery (mandatory worked example) | workflows, calculations, state machines |
| `sales.F2` | F — Invoicing on delivered quantities | Closing a delivery short of the ordered quantity | workflows, calculations |
| `sales.F3` | F — Invoicing on delivered quantities | Invoicing before any delivery | workflows, business rules |
| `sales.F4` | F — Invoicing on delivered quantities | Invoicing after a full payment forces the ordered policy | workflows, calculations |
| `sales.G1` | G — Upselling | An upsell case (mandatory worked example) | workflows, calculations |
| `sales.G2` | G — Upselling | An upsell the salesperson decides not to charge | state machines |
| `sales.G3` | G — Upselling | No upselling on a delivered-quantities product | workflows, calculations, state machines |
| `sales.G4` | G — Upselling | No upsell activity without a salesperson | state machines |
| `sales.G5` | G — Upselling | The upsell activity replaces the previous one | state machines |
| `sales.H1` | H — Returns and refunds | A return producing a refund (mandatory worked example) | workflows, state machines |
| `sales.H2` | H — Returns and refunds | A credit note created directly from the invoice | workflows, calculations, accounting effects |
| `sales.H3` | H — Returns and refunds | A return that is not flagged as refundable | workflows, calculations, accounting effects |
| `sales.H4` | H — Returns and refunds | A non-final run ignores a negative quantity | workflows, calculations, business rules |
| `sales.I1` | I — Portal acceptance | Acceptance by signature from the customer portal (mandatory worked example) |  |
| `sales.I2` | I — Portal acceptance | Signature when payment is also required | workflows, calculations, state machines |
| `sales.I3` | I — Portal acceptance | A second view on the same day posts no second note | workflows, state machines |
| `sales.I4` | I — Portal acceptance | A link preview posts no note | workflows, state machines |
| `sales.I5` | I — Portal acceptance | Signature refused on a confirmed order | workflows, business rules, state machines |
| `sales.I6` | I — Portal acceptance | Signature refused without an image | business rules |
| `sales.I7` | I — Portal acceptance | Declining with a reason |  |
| `sales.I8` | I — Portal acceptance | Declining without a reason | business rules |
| `sales.I9` | I — Portal acceptance | A product document exposed only on a confirmed order | workflows |
| `sales.J1` | J — Payment-driven confirmation | A payment of half the total confirming the order (mandatory worked example) | workflows, calculations, state machines |
| `sales.J2` | J — Payment-driven confirmation | A payment below the prepayment amount | workflows, calculations |
| `sales.J3` | J — Payment-driven confirmation | Two partial payments accumulate | workflows, calculations |
| `sales.J4` | J — Payment-driven confirmation | A grouped payment confirms nothing | workflows |
| `sales.J5` | J — Payment-driven confirmation | An authorized transaction confirms the order | workflows, calculations |
| `sales.J6` | J — Payment-driven confirmation | A pending transaction of a manual provider fills the payment reference | state machines |
| `sales.J7` | J — Payment-driven confirmation | Signature still required despite payment | workflows, calculations, business rules |
| `sales.J8` | J — Payment-driven confirmation | A zero-total order with payment required | calculations |
| `sales.J9` | J — Payment-driven confirmation | Posting an invoice after an online payment reconciles it | workflows, accounting effects, state machines |
| `sales.K1` | K — Expiration | An expired quotation (mandatory worked example) | workflows |
| `sales.K2` | K — Expiration | An expiration date equal to today |  |
| `sales.K3` | K — Expiration | A confirmed order is never expired | workflows, state machines |
| `sales.L1` | L — Locking | A locked order (mandatory worked example) | workflows |
| `sales.L2` | L — Locking | The advance-line description exception | workflows, calculations, business rules, state machines |
| `sales.L3` | L — Locking | Unlocking is possible even with the feature disabled |  |
| `sales.M1` | M — Cancellation and reset | Cancelling a confirmed order | workflows, state machines |
| `sales.M2` | M — Cancellation and reset | Cancelling with a generated purchase request | workflows, calculations, state machines |
| `sales.M3` | M — Cancellation and reset | Resetting to a quotation | workflows, state machines |
| `sales.M4` | M — Cancellation and reset | Re-confirming after a reset | workflows, state machines |
| `sales.M5` | M — Cancellation and reset | Mass cancellation | workflows, state machines |
| `sales.M6` | M — Cancellation and reset | Deleting a quotation | workflows, state machines |
| `sales.M7` | M — Cancellation and reset | A salesperson cannot delete | business rules, state machines |
| `sales.N1` | N — Line editing rules | Deleting a line after confirmation | workflows |
| `sales.N2` | N — Line editing rules | Deleting a section after confirmation | workflows |
| `sales.N3` | N — Line editing rules | Deleting an un-invoiced advance line | workflows |
| `sales.N4` | N — Line editing rules | Decreasing below the delivered quantity | workflows, calculations, business rules, state machines |
| `sales.N5` | N — Line editing rules | Changing the display type | business rules |
| `sales.N6` | N — Line editing rules | Changing the product after a delivery | workflows, business rules |
| `sales.N7` | N — Line editing rules | The unit is frozen after confirmation | workflows |
| `sales.N8` | N — Line editing rules | Adding a line to a confirmed order | workflows, state machines |
| `sales.N9` | N — Line editing rules | Section membership |  |
| `sales.N10` | N — Line editing rules | Collapsed composition on the document | calculations |
| `sales.O1` | O — Statuses and visibility | A quotation is invisible in the customer's quotation list until it is sent | state machines |
| `sales.O2` | O — Statuses and visibility | A confirmed order appears in the orders list | workflows |
| `sales.O3` | O — Statuses and visibility | Per-salesperson visibility | business rules |
| `sales.O4` | O — Statuses and visibility | All-documents visibility | workflows |
| `sales.O5` | O — Statuses and visibility | Multi-company visibility |  |
| `sales.O6` | O — Statuses and visibility | Portal visibility |  |
| `sales.P1` | P — Discounts | A per-line discount applied to the whole order | workflows, calculations |
| `sales.P2` | P — Discounts | A global discount | calculations |
| `sales.P3` | P — Discounts | A global discount with one tax only | calculations |
| `sales.P4` | P — Discounts | A fixed discount amount | calculations |
| `sales.P5` | P — Discounts | A percentage greater than one is refused | calculations, business rules |
| `sales.P6` | P — Discounts | No discount product and no rights | calculations |
| `sales.P7` | P — Discounts | The discount product is created on demand | workflows, calculations |
| `sales.Q1` | Q — Multi-currency | Order in a foreign currency | calculations |
| `sales.Q2` | Q — Multi-currency | The rate is frozen on the order | calculations |
| `sales.Q3` | Q — Multi-currency | The invoice uses its own rate | workflows, calculations, accounting effects, state machines |
| `sales.Q4` | Q — Multi-currency | The invoiced amount is converted at the invoice date | workflows, calculations |
| `sales.Q5` | Q — Multi-currency | A combo item's extra price is converted | calculations |
| `sales.R1` | R — Multi-company | Products of another company |  |
| `sales.R2` | R — Multi-company | Restricting a product already sold elsewhere | business rules |
| `sales.R3` | R — Multi-company | A shared quotation template with restricted products | business rules |
| `sales.R4` | R — Multi-company | A team member of the wrong company |  |
| `sales.R5` | R — Multi-company | Setting a company on a team with foreign members |  |
| `sales.S1` | S — Teams and membership | Team assignment from the salesperson |  |
| `sales.S2` | S — Teams and membership | Team assignment with several memberships | accounting effects |
| `sales.S3` | S — Teams and membership | A context default wins when it matches |  |
| `sales.S4` | S — Teams and membership | Single-membership mode archives the other membership | accounting effects |
| `sales.S5` | S — Teams and membership | A duplicate active membership is refused | business rules |
| `sales.S6` | S — Teams and membership | An archived duplicate is allowed |  |
| `sales.S7` | S — Teams and membership | The main team of a user |  |
| `sales.S8` | S — Teams and membership | Deleting a team in use | workflows, state machines |
| `sales.S9` | S — Teams and membership | Deleting a team with four orders | workflows, state machines |
| `sales.S10` | S — Teams and membership | Deleting a shipped default team | business rules |
| `sales.S11` | S — Teams and membership | The invoiced figure of a team | workflows, state machines |
| `sales.T1` | T — Analysis and aggregates | One row per line | workflows |
| `sales.T2` | T — Analysis and aggregates | Quantities are converted to the reference unit | calculations |
| `sales.T3` | T — Analysis and aggregates | The unit price measure is an average | calculations |
| `sales.T4` | T — Analysis and aggregates | The discount amount measure | calculations |
| `sales.T5` | T — Analysis and aggregates | The number of orders is a distinct count |  |
| `sales.T6` | T — Analysis and aggregates | Quantity sold on a product | workflows, calculations |
| `sales.T7` | T — Analysis and aggregates | Quantity sold for a non-salesperson | calculations |
| `sales.T8` | T — Analysis and aggregates | The customer's order count rolls up |  |
| `sales.U1` | U — Revenue accrual | Delivered but not invoiced | workflows, calculations, accounting effects |
| `sales.U2` | U — Revenue accrual | Invoiced but not delivered | workflows, calculations, state machines |
| `sales.U3` | U — Revenue accrual | Different companies refused | business rules |
| `sales.U4` | U — Revenue accrual | Different currencies refused | accounting effects, business rules |
| `sales.U5` | U — Revenue accrual | Reversal date not after the date | workflows |
| `sales.U6` | U — Revenue accrual | A manual amount | workflows, calculations, accounting effects |
| `sales.U7` | U — Revenue accrual | Weighted analytic distribution on the counterpart | calculations |
| `sales.V1` | V — Duplicate detection | Same customer reference | state machines |
| `sales.V2` | V — Duplicate detection | Source document matching a reference | state machines |
| `sales.V3` | V — Duplicate detection | A cancelled counterpart is ignored | workflows, state machines |
| `sales.V4` | V — Duplicate detection | Detection only for draft orders with a reference | workflows, state machines |
| `sales.W1` | W — Duplication of an order | Copy excludes advances and identity fields | workflows |
| `sales.W2` | W — Duplication of an order | Section flags are copied | calculations |
| `sales.X1` | X — Catalogue interaction | Adding a product | workflows, calculations, state machines |
| `sales.X2` | X — Catalogue interaction | Setting the quantity to zero on a quotation | workflows, calculations |
| `sales.X3` | X — Catalogue interaction | Setting the quantity to zero on a confirmed order | workflows, calculations, business rules |
| `sales.X4` | X — Catalogue interaction | Sections separate catalogue lines | calculations |
| `sales.X5` | X — Catalogue interaction | Catalogue clicks do not pollute the thread | workflows, state machines |
| `sales.X6` | X — Catalogue interaction | A read-only order in the catalogue | workflows, business rules, state machines |
| `sales.Y1` | Y — Expense re-invoicing | Re-invoicing at cost | workflows, calculations |
| `sales.Y2` | Y — Expense re-invoicing | Re-invoicing at the sales price | workflows, calculations |
| `sales.Y3` | Y — Expense re-invoicing | Re-invoicing onto a quotation | workflows |
| `sales.Y4` | Y — Expense re-invoicing | Re-invoicing onto a cancelled order | workflows, business rules, state machines |
| `sales.Y5` | Y — Expense re-invoicing | Delivered quantity from two analytic lines on one journal item | workflows, calculations, accounting effects |
| `sales.Z1` | Z — End-to-end regression | The full order-to-cash path |  |
| `sales.Z2` | Z — End-to-end regression | The advance-account invariant | workflows, calculations, accounting effects, state machines |
| `sales.Z3` | Z — End-to-end regression | The tax invariant | workflows, calculations |
| `sales.Z4` | Z — End-to-end regression | The quantity invariant | workflows, calculations |
| `sales.Z5` | Z — End-to-end regression | The cost invariant | workflows, calculations |

