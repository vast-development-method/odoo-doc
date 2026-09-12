# Calculations

Every formula and algorithm of the Payment Providers domain, with its inputs, its outputs, its precision, its order of operations and at least one worked example with real numbers.

---

## 1. Reference generation

### 1.1 The generic algorithm

**Inputs**: the provider code, an optional custom prefix, a separator (default `-`), and the create values of the transaction being built.
**Output**: a reference that is unique across the whole database.

1. When a custom prefix is given, transliterate it: decompose every character into its base letter plus its marks, drop everything that is not a plain unmarked character, and keep the result. `é` becomes `e`, `ä` becomes `a`, and a character with no plain equivalent disappears.
2. When the prefix is now empty, compute it from the create values (1.2).
3. When it is still empty, build a time-based prefix (1.3).
4. Set the candidate reference to the prefix alone.
5. Count the existing transactions whose reference equals the prefix exactly. When there is none, the candidate is the answer: **the first reference of a sequence carries no number**.
6. Otherwise, read the references of every transaction whose reference starts with the prefix followed by the separator.
7. Among them, keep those that match exactly "prefix, separator, one or more digits, end of string" and take the largest of those digit groups as `max_sequence_number`; when none matches, `max_sequence_number` is 0.
8. The answer is `prefix + separator + (max_sequence_number + 1)`.

The second search cannot rely on alphabetical ordering, because both the prefix and the separator are arbitrary: with the prefix `example` the references `example`, `example-1` and `example-ref` all match the "starts with" search, and `example-ref` sorts after `example-1`.

**Worked example A.** No transaction exists. Custom prefix `S00042`. Step 5 finds no exact match, therefore the reference is `S00042`.

**Worked example B.** `S00042` already exists. The "starts with `S00042-`" search returns nothing, therefore `max_sequence_number` is 0 and the reference is `S00042-1`.

**Worked example C.** `S00042`, `S00042-1` and `S00042-ref` exist. The regular expression matches only `S00042-1`, giving `max_sequence_number = 1`, and the reference is `S00042-2`.

**Worked example D.** A capture child of `S00043`. The prefix is `P-S00043`. `P-S00043` does not exist, therefore the reference is `P-S00043`. A second capture child then gets `P-S00043-1`.

### 1.2 The prefix computed from the document

**Inputs**: the separator and the create values.
**Output**: a prefix, or the empty text.

| Package | Key looked for in the create values | Prefix produced |
|---|---|---|
| base | none | empty |
| Accounting Payments | `invoice_ids` | The names of the referenced invoices, joined by the separator, skipping invoices with no name. When the create values also carry a next-installment name, that name replaces the whole prefix. If any referenced identifier does not resolve to an existing invoice, the prefix is empty. |
| Sales | `sale_order_ids` | The names of the referenced orders, joined by the separator. If any referenced identifier does not resolve, control passes to the next rule. |
| Point of Sale Online Payment | `pos_order_id` | The order's own point of sale reference. |

The rules are chained: the point of sale rule runs first, then the accounting rule, then the sales rule, then the empty base rule; the first one that produces a non-empty prefix wins.

**Worked example.** Create values referencing the invoices `INV/2026/00017` and `INV/2026/00018` with the separator `-` give the prefix `INV/2026/00017-INV/2026/00018`.

### 1.3 The time-based prefix

**Inputs**: an optional base prefix (default the two letters `tx`), a separator (default `-`) and an optional maximum length.
**Output**: `base prefix + separator + the current moment formatted as year, month, day, hour, minute, second, each zero-padded, with no separators` (fourteen digits).

When a maximum length is given, the base prefix is first truncated to `maximum length − length of the separator − 14`. The caller must ensure the maximum length is at least `1 + length of separator + 14`.

**Worked example.** At 2026-09-11 14:05:09 coordinated universal time, the default call gives `tx-20260911140509` (17 characters). With the base prefix `INV/2026/00017` and a maximum length of 35, the base is truncated to `35 − 1 − 14 = 20` characters, which leaves it unchanged, and the result is `INV/2026/00017-20260911140509` (29 characters).

These prefixes are **not** guaranteed unique; they only make a collision unlikely, and the sequence number of 1.1 resolves any collision that still happens.

### 1.4 Per-connector reference rules

| Provider code | Rule | Consequence |
|---|---|---|
| `aps` | The prefix is always replaced by the default time-based prefix. | The reference contains only letters, digits, `-` and `_`, as the provider requires, whatever the document is named. |
| `asiapay` | The document prefix is computed first, then the time-based prefix is built from it with a maximum length of 35. | The reference is at most 35 characters and is unique per merchant account. |
| `ecpay` | The time-based prefix is built with an empty separator and a maximum length of 20, and the reference is then computed with an empty separator. | The reference is at most 20 characters and purely alphanumeric. |
| `flutterwave` | The document prefix is computed first, then the time-based prefix is built from it with the default separator. | The reference is unique per merchant account. |
| `paymob` | Same as Flutterwave. | The reference is unique per merchant account. |
| `redsys` | The prefix is the last 10 digits of the current moment expressed as a number of seconds since the epoch, and the separator is the letter `S`. | The reference is between 9 and 12 characters and purely alphanumeric. |
| `toss_payments` | The prefix is always replaced by the default time-based prefix. | The reference is between 6 and 64 characters and contains only letters, digits, `-` and `_`. |
| `worldline` | The generic reference is computed first; when it is longer than 30 characters it is recomputed from the time-based prefix built on the two letters `WL`. | The reference is at most 30 characters. |

**Worked example (Redsys).** At the moment whose epoch second count is 1 789 123 456, the prefix is the last ten digits, `1789123456`. The first transaction gets `1789123456` (10 characters); a collision gives `1789123456S1` (12 characters).

---

## 2. Minor unit conversion

Providers exchange amounts either in the major unit of the currency (for instance 120.00 euro) or in its minor unit (12 000 cents). The conversion uses a **payment precision**, which is the number of decimal places the payment industry uses for that currency, not the accounting precision stored on the currency record.

### 2.1 The payment precision table

The table below is complete: every currency not listed uses 2 decimal places.

| Decimal places | Currency codes |
|---|---|
| 0 | `ADP`, `AOK`, `AON`, `AOR`, `AYM`, `BIF` (Burundian franc), `BYR`, `CLP` (Chilean peso), `DJF` (Djiboutian franc), `ECS`, `ESP`, `GEK`, `GNF` (Guinean franc), `ISK` (Icelandic króna), `ITL`, `JPY` (Japanese yen), `KMF` (Comorian franc), `KRW` (South Korean won), `MGF`, `PTE`, `PYG` (Paraguayan guaraní), `ROL`, `RWF` (Rwandan franc), `SML`, `TJR`, `TPE`, `TRL`, `UGX` (Ugandan shilling), `UYI`, `VAL`, `VND` (Vietnamese dong), `VUV` (Vanuatu vatu), `XAF` (Central African franc), `XEU`, `XOF` (West African franc), `XPF` (Pacific franc) |
| 3 | `BHD` (Bahraini dinar), `IQD` (Iraqi dinar), `JOD` (Jordanian dinar), `KWD` (Kuwaiti dinar), `LYD` (Libyan dinar), `OMR` (Omani rial), `TND` (Tunisian dinar) |
| 4 | `CLF` (Chilean unit of account), `UYW` (Uruguayan indexed unit) |

When a currency code is absent from the table altogether, the accounting precision of the currency record is used instead.

### 2.2 Major to minor

```formula
decimals = arbitrary_precision, when the caller supplies one
         = payment_precision(currency_code), when the code is in the table
         = currency.decimal_places, otherwise

minor_amount = integer( round_down( major_amount × 10^decimals ) )
```

The rounding is always towards zero (down), therefore an amount that carries more decimals than the provider accepts is never rounded up into a charge larger than the document.

**Worked examples.**

| Major amount | Currency | Precision used | Minor amount |
|---|---|---|---|
| 120.00 | `EUR` (euro) | 2 | 12000 |
| 1111.11 | `EUR` | 2 | 111111 |
| 1200 | `JPY` | 0 | 1200 |
| 12.345 | `KWD` | 3 | 12345 |
| 99.999 | `EUR` | 2 | 9999 (rounded down from 9999.9) |
| 120.00 | `IDR` (Indonesian rupiah), Adyen | 0 (the connector's own deviation) | 120 |

### 2.3 Minor to major

```formula
decimals = same choice as above
major_amount = round( minor_amount, 0 ) ÷ 10^decimals
```

**Worked example.** 12000 minor units of the euro give `12000 ÷ 10^2 = 120.00`.

### 2.4 Connector deviations from the payment precision table

| Connector | Currency | Precision the connector uses |
|---|---|---|
| Adyen | `CLP` | 2 |
| Adyen | `CVE` (Cape Verdean escudo) | 0 |
| Adyen | `IDR` | 0 |
| Adyen | `ISK` | 2 |
| Stripe | `ISK` | 2 |
| Stripe | `UGX` | 2 |
| Stripe | `MGA` (Malagasy ariary) | 0 |
| Mercado Pago | `COP` (Colombian peso), `HNL` (Honduran lempira), `NIO` (Nicaraguan córdoba) | 0 |
| Xendit | `IDR`, `MYR`, `PHP`, `SGD`, `THB`, `USD`, `VND` | 0 |

---

## 3. Amount validation

**Inputs**: the transaction and the payment data.
**Output**: nothing, or the transaction moved to `error`.

1. When the transaction's operation is `validation`, stop. (PAY-RULE-041)
2. Ask the connector for the amount data. When it returns nothing at all, stop. (PAY-RULE-042)
3. Read `amount`, `currency_code` and the optional `precision_digits` from the amount data.
4. When `amount` or `currency_code` is empty, set the transaction to `error` with `The amount or currency is missing from the payment data.` and stop.
5. When the operation is `refund`, negate `amount`, because the platform stores a refund as a negative amount while every provider reports it as positive.
6. When `precision_digits` is empty, take the payment precision of the transaction's currency, falling back to the currency's accounting precision.
7. Compute `transaction_amount = round_down(transaction.amount, precision_digits)`.
8. Compare `amount` with `transaction_amount` using the currency's comparison. When they differ, set the transaction to `error` with `The amount from the payment data doesn't match the one from the transaction.` and stop.
9. When `currency_code` differs from the transaction currency's code, set the transaction to `error` with `The currency from the payment data doesn't match the one from the transaction.`

**Worked example A, a match.** Transaction of 120.00 euro; the provider reports 120.0 and `EUR`. `precision_digits` is 2, `transaction_amount` is 120.00, the comparison succeeds, the codes match, nothing happens.

**Worked example B, a currency with more accounting decimals than payment decimals.** The euro is configured in the database with a rounding step of 0.001, and the transaction amount is 123.452. The provider reports 123.45. `precision_digits` is 2 (the payment precision of the euro), therefore `transaction_amount = round_down(123.452, 2) = 123.45`. The comparison succeeds and the transaction is not set to `error`. Without step 7 the comparison would have failed.

**Worked example C, a refund.** Transaction of −30.00 euro with operation `refund`; the provider reports 30.0 and `EUR`. Step 5 turns the reported amount into −30.0; `transaction_amount = round_down(−30.00, 2) = −30.00`; the comparison succeeds.

**Worked example D, a mismatch.** Transaction of 120.00 euro; the provider reports 100.0. The comparison fails and the transaction moves to `error` with the amount-mismatch message. The connector's update step is then skipped entirely (PAY-RULE-043), therefore the transaction does not become confirmed.

---

## 4. Capture allocation

### 4.1 The four amounts of the capture wizard

**Inputs**: the set of source transactions `T`.

```formula
authorized_amount = Σ over t in T of t.amount

captured_amount   = Σ over t in T of t.amount        where t.state = "done" and t has no children
                  + Σ over c in children(T) of c.amount  where c.state = "done"

voided_amount     = Σ over c in children(T) of c.amount  where c.state = "cancel"

available_amount  = authorized_amount − captured_amount − voided_amount
```

The first term of `captured_amount` covers a transaction that was captured in one step and therefore has no child; the second covers every partial capture.

`amount_to_capture` defaults to `available_amount` and may be lowered.
`is_amount_to_capture_valid` is `0 < amount_to_capture ≤ available_amount`.
`has_remaining_amount` is `amount_to_capture < available_amount`; when it becomes false, `void_remaining_amount` is forced to false.

**Worked example.** Two authorized transactions of 120.00 and 200.00. The first already has a confirmed capture child of 80.00 and a cancelled void child of 10.00.

```formula
authorized_amount = 120.00 + 200.00 = 320.00
captured_amount   = 0.00 (no childless done source) + 80.00 = 80.00
voided_amount     = 10.00
available_amount  = 320.00 − 80.00 − 10.00 = 230.00
```

### 4.2 The allocation loop

**Inputs**: `amount_to_capture`, `void_remaining_amount`, the source transactions in their own order.

The wizard walks the source transactions in the order in which it received them and performs the following procedure.

1. Set the remaining amount to capture to the amount to capture entered in the wizard.
2. Take the next source transaction of the selection whose state is `authorized`. A source transaction in any other state is skipped without effect.
3. Compute the remaining amount of that source transaction: subtract from its amount the sum of the amounts of its child transactions whose state is `done`, and round the difference to the number of decimal places of the transaction currency.
4. When the remaining amount to capture is not zero, capture on that source transaction the smaller of the remaining amount of the source transaction and the remaining amount to capture. The capture creates one capture child transaction for that amount. Subtract the captured amount from the remaining amount to capture and from the remaining amount of the source transaction.
5. When the remaining amount of the source transaction is still not zero and the void-remaining flag is set, void that remaining amount on the source transaction. The void creates one void child transaction for that amount.
6. Otherwise, when the remaining amount to capture has reached zero and the void-remaining flag is not set, stop the walk immediately; no later source transaction of the selection is touched.
7. When the walk was not stopped, return to step 2 until the source transactions are exhausted.

**Worked example A, a partial capture that leaves the rest authorized.** One authorized transaction of 120.00, no children, `amount_to_capture = 80.00`, void checkbox unticked.

```formula
remaining_to_capture = 80.00
source_remaining     = 120.00 − 0.00 = 120.00
a                    = min(120.00, 80.00) = 80.00      → capture child of 80.00
remaining_to_capture = 0.00
source_remaining     = 40.00
void checkbox unticked and remaining_to_capture = 0 → stop
```

The source transaction stays `authorized` with 40.00 still authorized at the provider. A later Void Transaction produces a void child of `120.00 − 80.00 = 40.00`.

**Worked example B, capture and void in one step.** Same transaction, `amount_to_capture = 80.00`, void checkbox ticked.

```formula
a = 80.00                → capture child of 80.00
source_remaining = 40.00 → void child of 40.00
```

**Worked example C, several sources.** Sources of 120.00 and 200.00, both authorized and childless, `amount_to_capture = 150.00`, void checkbox unticked.

```formula
source 1: source_remaining = 120.00 ; a = min(120.00, 150.00) = 120.00 → capture child of 120.00
          remaining_to_capture = 30.00 ; source_remaining = 0.00
          source_remaining = 0 and not voiding and remaining ≠ 0 → continue
source 2: source_remaining = 200.00 ; a = min(200.00, 30.00) = 30.00  → capture child of 30.00
          remaining_to_capture = 0.00 ; source_remaining = 170.00
          source_remaining ≠ 0 but the void checkbox is unticked, and remaining_to_capture = 0 → stop
```

Source 1 closes as `done` (its children cover it exactly); source 2 stays `authorized` with 170.00 still authorized.

### 4.3 The amount voided by the Void Transaction operation

```formula
already_captured = Σ amounts of children(t) with state "done" and operation = t.operation
amount_to_void   = t.amount − already_captured
```

**Worked example.** Source of 120.00 with one confirmed capture child of 80.00: `amount_to_void = 120.00 − 80.00 = 40.00`.

---

## 5. Source transaction closure

**Inputs**: a child transaction `c` that just reached `done` or `cancel`.

1. Collect the sibling set: every child transaction of the source transaction of the settled child whose state is `done` or `cancel` and whose operation equals the operation of the settled child.
2. Sum the amounts of the sibling set and round the sum to the number of decimal places of the currency of the settled child. Call the result the processed amount.
3. When the processed amount differs from the amount of the source transaction, stop. The source transaction is left in its current state.
4. When the processed amount equals the amount of the source transaction, test whether every member of the sibling set is in state `cancel`. When it is, the target state is `cancel`; otherwise the target state is `done`.
5. Move the source transaction from `authorized` to the target state with an empty state message. The move is performed by the plain state-update step rather than by the transition helpers, in order that reaching `done` or `cancel` on the source transaction does not start a second closure evaluation. A source transaction that is not in `authorized` is refused by that step and stays as it is.
6. Log the received message of the source transaction on every document linked to it.

**Worked example A.** Source 120.00; children: capture 80.00 `done`, void 40.00 `cancel`. `processed = 120.00`, equal to the source amount; not every child is `cancel`, therefore the source becomes `done`.

**Worked example B.** Source 120.00; children: void 120.00 `cancel`. `processed = 120.00`; every child is `cancel`, therefore the source becomes `cancel`.

**Worked example C.** Source 120.00; children: capture 80.00 `done`. `processed = 80.00 ≠ 120.00`, therefore the source stays `authorized`.

**Worked example D.** Source 120.00 already `done`; a refund child of −30.00 reaches `done`. Its operation is `refund`, which differs from the source's operation, therefore `siblings` is empty for that operation and the source is untouched.

---

## 6. Refundable amount of a Payment

```formula
refund_payments  = every Payment whose source_payment is this payment
refunded_amount  = | Σ over p in refund_payments of p.amount |
amount_available_for_refund =
      payment.amount − refunded_amount, when all of:
          the payment came from a Payment Transaction,
          the transaction's provider supports refunds (support_refund ≠ "none"),
          the primary payment method supports refunds (support_refund ≠ "none"),
          the transaction's operation is not "refund"
      0, otherwise
```

Only Payments are counted, never refund transactions, therefore a refund transaction that never reached `done` does not lock the amount.

**Worked example.** Payment of 120.00 with one posted refund payment of 30.00 (stored as an outbound payment of amount 30.00).

```formula
refunded_amount = |30.00| = 30.00
amount_available_for_refund = 120.00 − 30.00 = 90.00
refunded_amount shown on the wizard = 120.00 − 90.00 = 30.00
```

The effective refund capability of the wizard is:

```formula
support_refund = "none"       when provider.support_refund = "none" or method.support_refund = "none"
               = "full_only"  when either of them is "full_only"
               = "partial"    when both are "partial"
```

---

## 7. Token display name

**Inputs**: `payment_details`, `create_date`, a maximum length (default 34, which fits the longest international bank account numbers) and a padding flag (default true).

1. When the creation moment is empty, the display name is the empty text and the procedure ends.
2. Compute the padding length as the maximum length minus the number of characters of the payment details, counting zero characters when the payment details are empty.
3. When the payment details are empty, the display name is the fixed text "Payment details saved on " followed by the creation date rendered as four-digit year, oblique stroke, two-digit month, oblique stroke, two-digit day; the procedure ends.
4. Otherwise, when the padding length is two or more, build the padding as the bullet character repeated the smaller of the padding length minus one and four, followed by one space; when the padding flag is false the padding is the empty text. The display name is the padding followed by the payment details; the procedure ends.
5. Otherwise, when the padding length is greater than zero, the display name is the payment details unchanged; the procedure ends.
6. Otherwise, when the maximum length is greater than zero, the display name is the last maximum-length characters of the payment details; the procedure ends.
7. Otherwise the display name is the empty text.

**Worked examples.**

| `payment_details` | Maximum length | Padding flag | Result |
|---|---|---|---|
| `1234` | 34 | true | `•••• 1234` (`padding_length = 30`, hence 4 bullets and a space) |
| `1234` | 6 | true | `• 1234` (`padding_length = 2`, hence 1 bullet and a space) |
| `1234` | 34 | false | `1234` |
| `1234` | 5 | true | `1234` (`padding_length = 1`, which is greater than 0 but lower than 2) |
| `1234` | 3 | true | `234` (`padding_length = −1`) |
| empty | 34 | true | `Payment details saved on 2024/01/31` |

The demo connector always builds the name without padding.

---

## 8. Provider availability

**Inputs**: the company, the paying contact, the amount, the currency, and the flags "tokenization forced", "express checkout" and "validation".
**Output**: the compatible providers and an availability report.

```formula
P0 = providers with company parent_of given company and state in {"enabled","test"}
P1 = P0                       when the user is internal
   = { p in P0 : p.is_published }      otherwise
P2 = P1                       when the contact has no country
   = { p in P1 : p.available_countries is empty or contains contact.country }   otherwise
P3 = P2                       for a validation operation or when no currency is known
   = { p in P2 : p.maximum_amount is 0 or p.maximum_amount ≥ converted_amount } otherwise
       where converted_amount = convert(amount, from currency, to company.currency, at today's rate)
P4 = P3                       when no currency is known
   = { p in P3 : p.available_currencies is empty or contains currency }
P5 = P4                       when tokenization is neither forced nor required
   = { p in P4 : p.allow_tokenization }
P6 = P5                       when the payment is not an express checkout
   = { p in P5 : p.allow_express_checkout }
```

Capability packages add three further filters, applied after the six above: the website filter, the cash-on-delivery filter and the validation-not-supported filter (PAY-RULE-083 to PAY-RULE-085).

**Worked example, the maximum amount.** Company currency euro; a provider with `maximum_amount` 500.00; a payment of 600.00 United States dollars on a day when 1 dollar is worth 0.90 euro.

```formula
converted_amount = 600.00 × 0.90 = 540.00 euro
comparison: 500.00 ≥ 540.00 is false → the provider is removed
Reason recorded: "maximum amount exceeded"
```

With a payment of 500.00 dollars: `converted_amount = 450.00` and `500.00 ≥ 450.00` is true, therefore the provider stays. With `maximum_amount` 0, the provider stays whatever the amount.

The comparison uses the currency's own comparison, therefore a converted amount of exactly 500.00 keeps the provider.

---

## 9. Payment method availability

**Inputs**: the compatible providers, the paying contact, the currency, and the flags "tokenization forced" and "express checkout".

```formula
M0 = every active primary payment method
M1 = { m in M0 : m.providers intersects the compatible providers }
M2 = M1                       when the contact has no country
   = { m in M1 : m.supported_countries is empty or contains contact.country }
M3 = M2                       when no currency is known
   = { m in M2 : m.supported_currencies is empty or contains currency }
M4 = M3                       when tokenization is not forced
   = { m in M3 : m.support_tokenization }
M5 = M4                       when the payment is not an express checkout
   = { m in M4 : m.support_express_checkout }
```

**Worked example.** A payment form is opened for a contact whose country is Belgium, for 1111.11 euro, with tokenization not forced and no express checkout. The catalogue holds five active methods and the compatible providers are the single provider "Buckaroo".

| Method | Primary | Providers | Supported countries | Supported currencies | Tokenization |
|---|---|---|---|---|---|
| `card` | yes | Buckaroo, Stripe | (empty) | (empty) | yes |
| `ideal` | yes | Buckaroo | Netherlands | euro | no |
| `bancontact` | yes | Buckaroo | Belgium | euro | no |
| `paypal` | yes | PayPal | (empty) | (empty) | no |
| `visa` | no, brand of `card` | Buckaroo | (empty) | (empty) | yes |

```formula
M0 = { card, ideal, bancontact, paypal }          the brand visa is not primary and never enters
M1 = { card, ideal, bancontact }                  paypal removed: "no supported provider available"
M2 = { card, bancontact }                         ideal removed: "incompatible country" (Belgium ∉ {Netherlands})
M3 = { card, bancontact }                         both pass: card has no currency list, bancontact lists the euro
M4 = M3                                           tokenization is not forced, the filter is skipped
M5 = M4                                           not an express checkout, the filter is skipped
result = { card, bancontact }
```

The payment form therefore offers two methods. Both are active and both carry the shipped display sequence 1000, so the default ordering falls through to the name ascending and the form lists `Bancontact` before `Card`. The brand `visa` is never offered as a choice of its own; its logo is drawn next to `card`.

Same inputs, but with tokenization forced (the customer asked to save the method):

```formula
M4 = { card }                                     bancontact removed: "tokenization not supported"
M5 = { card }
result = { card }
```

### 9.1 The availability report

The report is a two-part structure:

| Part | Keyed by | Value carried for each key |
|---|---|---|
| Providers | One entry per provider that entered the computation | An availability flag, true or false, and a reason text |
| Payment methods | One entry per payment method that entered the computation | An availability flag, true or false, a reason text, and the list of the method's providers, each paired with that provider's own availability flag |

Every filter adds the records it removed with `available` false and its own reason; the records that survive every filter were added with `available` true by the very first step. For a payment method, the report also lists, for each of its providers that appears in the provider part of the report, whether that provider is available. The nine reasons are:

| Key | Text |
|---|---|
| maximum amount exceeded | `maximum amount exceeded` |
| express checkout not supported | `express checkout not supported` |
| incompatible country | `incompatible country` |
| incompatible currency | `incompatible currency` |
| incompatible website | `incompatible website` |
| manual capture not supported | `manual capture not supported` |
| no supported provider available | `no supported provider available` |
| tokenization not supported | `tokenization not supported` |
| tokenization without payment not supported | `tokenization without payment no supported` |

---

## 10. Validation amount and currency

**Validation amount**: the amount charged to prove that a payment method works. The base value is 0. Authorize uses 0.01; Razorpay uses 1.

**Validation currency**, computed for a provider and, when known, a payment method:

1. Read the provider currency set from the provider's available currencies. An empty set means the provider accepts every currency.
2. Read the method currency set from the payment method's supported currencies. An empty set means the method accepts every currency, and the set is empty as well when no payment method is known at this point.
3. When both sets are non-empty, the candidate set is their intersection.
4. When only the provider currency set is non-empty, the candidate set is the provider currency set.
5. When only the method currency set is non-empty, the candidate set is the method currency set.
6. When the candidate set is non-empty, the validation currency is its first currency in the ordering of the currency records.
7. When the candidate set is empty, because both sets were empty or because the intersection of two non-empty sets is empty, the validation currency is the accounting currency of the provider's company.

**Worked example.** Provider restricted to the euro and the United States dollar; card method restricted to the United States dollar and the pound sterling. The intersection is the United States dollar, therefore the validation transaction is created for 0.01 United States dollar when the provider is Authorize.

---

## 11. Access token

**Inputs**: an ordered list of values.
**Output**: a text token.

```formula
token_text   = the values, each rendered as text, joined by the character "|"
access_token = keyed_hash( database secret, scope "generate_access_token", token_text )
```

Verification recomputes the token from the same values in the same order and compares it with a constant-time comparison, which prevents the comparison time from leaking how many leading characters matched. An empty received token always fails.

The values used, per flow:

| Flow | Values, in order |
|---|---|
| Pay page and transaction service | contact identifier, amount, currency identifier |
| Payment method management page | contact identifier, nothing, nothing |
| Generic confirmation page | contact of the transaction, amount of the transaction, currency of the transaction |
| Payment link built by the wizard (base) | contact identifier, amount, currency identifier |
| Payment link built by the wizard (invoice) | amount |
| Adyen inline payment endpoint | reference, converted amount, currency identifier, contact identifier |
| Authorize inline payment endpoint | reference, contact identifier |
| Nuvei abandonment return | reference |
| Toss Payments failure return | reference |
| Xendit inline payment endpoint | reference |
| Xendit success return | reference, amount |

Rendering rules for the values, which a replacement must reproduce exactly, because they are part of the hashed input:

1. An integer identifier is rendered in decimal with no padding and no thousands separator: the identifier 14 becomes `14`.
2. A decimal amount is rendered with the digits it actually carries, with a full stop as the decimal separator and no thousands separator and no trailing zeros beyond those the value carries: 1111.11 becomes `1111.11`, 120.00 becomes `120.0`, and 120 becomes `120`.
3. A value that is absent is rendered as the fixed four-character text `None`. The management page, which signs only a contact, therefore signs three values of which the last two are that fixed text.
4. The separator is the single vertical-bar character. It is inserted between values only, never before the first or after the last, and it is inserted even when the neighbouring value renders to the empty text.

**Worked example A, the pay page.** Contact identifier 14, amount 1111.11, currency identifier 1.

```formula
values     = 14, 1111.11, 1
token_text = "14" + "|" + "1111.11" + "|" + "1"
           = "14|1111.11|1"
access_token = keyed_hash( database secret, "generate_access_token", "14|1111.11|1" )
```

The resulting token is a hexadecimal text whose value depends on the database secret, therefore it differs from database to database; what a replacement must reproduce is the exact text that is hashed, since a difference of one character there makes every link generated by one system unusable on the other. Verification of the same address recomputes `"14|1111.11|1"` from the address parameters and compares the two texts character by character in constant time.

**Worked example B, the payment method management page.** Contact identifier 14, no amount, no currency.

```formula
values     = 14, absent, absent
token_text = "14|None|None"
```

**Worked example C, a link built by the wizard for an invoice.** The invoice route signs the amount alone, 120.00.

```formula
values     = 120.00
token_text = "120.0"
```

There is no separator at all, because a single value has no neighbour.

**Worked example D, the Adyen inline payment endpoint.** Reference `S00042-1`, converted amount 111111 (minor units), currency identifier 1, contact identifier 14.

```formula
token_text = "S00042-1|111111|1|14"
```

---

## 12. Idempotency key

```formula
idempotency_key = hexadecimal secure hash, 160-bit variant, of
                  ( database identity + transaction reference + scope, or the empty text when no scope )
```

The same logical request therefore always produces the same key, while a different endpoint (a different scope), a different transaction or a different database produces a different one. The scopes in use are: `payment_request_controller`, `payment_details_request_controller`, `payment_request_token`, `payment_request_order`, `direct_payment`, `token_payment`, `payment_intents`.

**Worked example.** Two consecutive clicks on the same payment button for the transaction `S00042` produce the same key, therefore the provider rejects the second charge instead of charging the customer twice.

---

## 13. Signature computation

Connectors use one of four shapes. The per-connector details, including the exact field order, are in `provider-connector-contracts.md`.

**Shape 1, a keyed hash over a concatenation.** Concatenate the agreed values in the agreed order, with the agreed separator, then compute a keyed hash with the shared secret and the agreed hash function, then encode the result as hexadecimal or base-64.

**Shape 2, a plain hash over a concatenation that embeds the secret.** Concatenate the secret, the agreed values and possibly the secret again, then hash the whole string.

**Shape 3, a shared token.** The provider simply sends the agreed secret in a header; the connector compares it with the stored one using a constant-time comparison.

**Shape 4, a verification call.** The connector sends the received event and its transport headers back to the provider and accepts the event only when the provider answers that the verification succeeded.

**Worked example, shape 2 (the Amazon Payment Services connector).** An incoming notification carries the three keys `merchant_reference` with the value `tx-1`, `amount` with the value `12000`, and `signature` with the value the provider computed. The response phrase stored on the provider is the text `PHRASE`.

```formula
keys kept, sorted ascending  = amount, merchant_reference          (signature is always excluded)
sign_data                    = "amount=12000" + "merchant_reference=tx-1"
                             = "amount=12000merchant_reference=tx-1"
signing_string               = "PHRASE" + sign_data + "PHRASE"
                             = "PHRASEamount=12000merchant_reference=tx-1PHRASE"
signature                    = hexadecimal secure hash, 256-bit variant, of signing_string
                             = ed25ee102bf796ce586b1edd9d5ecd9cfc98ff9814eafd5c62f65a96291538b0
```

The notification is accepted only when the value it carries under the key `signature` is exactly that 64-character text. The outgoing direction uses the same steps with the request phrase instead of the response phrase.

**Worked example, shape 1 (the AsiaPay connector, incoming).** Values in the fixed order `src`, `prc`, `successcode`, `Ref`, `PayRef`, `Cur`, `Amt`, `payerAuth`, followed by the shared secret, joined by `|`, then hashed with the configured function. With `src=""`, `prc="0"`, `successcode="0"`, `Ref="tx-1"`, `PayRef="123"`, `Cur="344"`, `Amt="120.00"`, `payerAuth=""` and the secret `S`:

```formula
signing_string = "|0|0|tx-1|123|344|120.00||S"
signature      = hexadecimal hash of signing_string with the configured function
               = 6c5676ecfe1589eeea8b3080d6a4c10d5e15bd98         when the function is the 160-bit variant
               = fe4f85783a26e353a96124f9cea4897f70f7e17d464b8a8464c5bb414a49f595
                                                                  when the function is the 256-bit variant
```

The first two empty values produce the two leading separators and the empty field before the secret; a replacement that drops an empty value instead of joining it produces a different text and every notification is then refused.

---

## 14. Sales order communication for a custom provider

**Inputs**: the transaction and one linked sales order.

```formula
order_reference = order.name                                   when provider.sales_order_reference_type = "so_name"
                = "CUST/" + two digits of ( contact identifier modulo 97 ), zero-padded on the left
                                                               when provider.sales_order_reference_type = "partner"
                = nothing                                      when the field is empty
then: the sale journal of the transaction's company, when one exists, formats the reference
      according to its own structured-reference rules.
```

**Worked example.** Contact identifier 1234, reference type "Based on Customer Identifier": `1234 modulo 97 = 68`, therefore the reference is `CUST/68`. Contact identifier 97 gives `97 modulo 97 = 0`, padded to `00`, therefore the reference is `CUST/00`.

The communication actually displayed to the customer is chosen separately:

```formula
communication = the payment reference of the first linked invoice, when there is one
              = the reference of the first linked sales order, when there is one
              = the transaction reference, otherwise
```

---

## 15. Provider card colour

```formula
colour = 4 (blue)   when the provider has a package and that package is not installed
       = 3 (yellow) when state = "disabled"
       = 2 (orange) when state = "test"
       = 7 (green)  when state = "enabled"
```

The four branches are evaluated in that order and the first that matches wins, therefore the package test is made before the state is looked at. The value is an integer index into the palette of the card view; it is recomputed whenever the package state or the provider state changes and it is not stored.

**Worked example.** Six cards of one provider list, in the order the list produces them:

| Provider | Package | Package state | `state` | Branch that matches | `color` |
|---|---|---|---|---|---|
| Stripe | Payment Provider: Stripe | installed | `enabled` | fourth | 7 |
| Adyen | Payment Provider: Adyen | installed | `test` | third | 2 |
| Mollie | Payment Provider: Mollie | installed | `disabled` | second | 3 |
| Worldline | Payment Provider: Worldline | not installed | `disabled` | first | 4 |
| Worldline, after installing the package and leaving it untouched | Payment Provider: Worldline | installed | `disabled` | second | 3 |
| Single Euro Payments Area direct debit | the euro-area direct debit package | not installed | `disabled` | first | 4 |

The fourth and fifth rows are the same record before and after the package is installed: installing the package changes nothing on the record itself, yet the card turns from 4 to 3, because the first branch stops matching. A provider record that names no package at all never matches the first branch and is coloured by its state alone.

---

## 16. Post-processing retry window

```formula
retry_limit_date = now − 4 days
candidates       = transactions where is_post_processed = false
                   AND last_state_change ≥ retry_limit_date
```

The job runs every 10 minutes. A transaction whose last state change is older than four days is never picked up again by the job; the Post-process operation on its form is then the only way to process it.

**Worked example.** A transaction confirmed on the 1st of the month at 09:00 and never processed is retried at 09:10, 09:20, and at each following interval, up to the 5th at 09:00, which is 576 attempts, after which it is abandoned by the job.

---

## 17. Language code resolution

**Inputs**: a language tag such as `fr_BE`, a mapping from tags and bare language codes to provider codes, and a fallback key (default the two letters `en`).

```formula
code = mapping[tag]                      when the tag is a key of the mapping
     = mapping[ tag up to the first "_" ] when that shorter key is in the mapping
     = mapping[fallback]                  otherwise
```

**Worked example (AsiaPay).** The tag `zh_TW` is a key of the mapping and gives `C`. The tag `fr_BE` is not a key, but `fr` is, giving `F`. The tag `pt_BR` matches neither, therefore the fallback `en` gives `E`.

---

## 18. Amounts for methods that refuse decimals

Some payment methods refuse a fractional amount even in a currency that has decimals. The connectors that implement this round the amount **down** to zero decimals for those methods.

```formula
rounding = 0                          when the payment method code is in the connector's integer-only list
         = currency.decimal_places    otherwise
rounded_amount = round_down(amount, rounding)
```

**Worked example (the Nuvei connector, the Webpay method, United States dollar).** An amount of 120.75 becomes 120 in the request and the amount validation is then run with a precision of 0, therefore the provider's report of 120 matches.

The Xendit connector uses a fixed table instead: every currency it supports is treated as having 0 decimals, therefore 120.75 Indonesian rupiah, Malaysian ringgit, Philippine peso, Singapore dollar, Thai baht, United States dollar or Vietnamese dong all become 120.

The Mercado Pago connector rounds down to its own precision table before building the request: 120.75 Colombian pesos become 120.

---

## 19. Electronic mandate maximum amount (the Razorpay connector)

**Inputs**: the transaction, the primary payment method code and the mandate values supplied by the document being paid.

```formula
base_maximum_in_rupees = 1 000 000   when the method code is "card"
                       = 100 000     when the method code is "upi"
                       = 100 000     for any other method
base_maximum = convert(base_maximum_in_rupees, from the Indian rupee, to the transaction currency)

maximum = min( base_maximum , max( document.amount × 1.5 , document.monthly_recurring_revenue × 5 ) )
                     when the document supplies both an amount and a monthly recurring revenue
        = base_maximum
                     otherwise
```

**Worked example.** A card payment in Indian rupees for a document of 2 000 with a monthly recurring revenue of 500.

```formula
base_maximum = 1 000 000
max(2 000 × 1.5, 500 × 5) = max(3 000, 2 500) = 3 000
maximum = min(1 000 000, 3 000) = 3 000
```

The warning shown when a later charge exceeds the maximum is `You can not pay amounts greater than %(currency_symbol)s %(max_amount)s with this payment method`, with the maximum rounded to zero decimals.

---

## 20. Electronic mandate options (the Stripe connector)

For the currencies in the connector's mandate list, a mandate is attached to the intent with:

```formula
reference   = the transaction reference
amount type = "maximum"
amount      = to_minor_units( document.amount , currency )   when the document supplies one
            = to_minor_units( 15 000 , currency )            otherwise
start date  = the document's start moment, or now
interval    = the document's recurrence unit, when it supplies one, otherwise "sporadic"
interval count = the document's recurrence duration, when it supplies one
end date    = the document's end moment, when it supplies one
supported types = India
```

**Worked example.** No document values, currency Indian rupee: the mandate amount is `15 000 × 10^2 = 1 500 000` minor units, the interval is "sporadic" and there is no end date.
