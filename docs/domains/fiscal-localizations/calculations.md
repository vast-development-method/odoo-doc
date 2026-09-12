# Fiscal Localizations: Calculations

Every formula and algorithm the domain executes: account code padding and code search, template composition and merging, the six report computation engines, carryover, withholding base and withheld amounts with their currency conversions and proration, progressive withholding scales, visual code payloads, and the country-specific arithmetic that country packages add. Each algorithm states its inputs, outputs, precision, order of operations and at least one worked example with real numbers. Rounding is always applied at the end of a chain, never to an intermediate ratio, unless the algorithm explicitly says otherwise.

Cross-domain: the tax amount arithmetic itself (percentage, fixed amount, division, group, base affected by previous taxes, price included) belongs to [Taxes](../taxes/calculations.md). The currency conversion primitive belongs to [Multi-Currency](../multi-currency/calculations.md). This file uses both and does not restate them.

---

## 1. Account code padding

**Inputs.** The account row's `code` as written in the template (text), and the template's `code_digits` (integer, default 6).

**Output.** The stored account code.

**Precision.** Exact text operation; no numeric rounding.

```
padded_code(code, code_digits) =
    code                                     if length(code) ≥ code_digits
    code + "0" repeated (code_digits − length(code))   otherwise
```

The padding character is the digit zero and it is appended on the **right**, so a code is extended, never re-based. Codes containing dots, letters or other separators are padded the same way, by total character count.

**Worked examples.**

| Template code | `code_digits` | Stored code |
|---|---|---|
| `1010` | 6 | `101000` |
| `40` | 6 | `400000` |
| `1234567` | 6 | `1234567` |
| `101.01.01` | 9 | `101.01.01` |
| `102.01` | 9 | `102.01000` |
| `4000` | 8 | `40000000` |
| `70` | 4 | `7000` |

**Why right padding.** A chart whose codes are hierarchical by prefix keeps its hierarchy under right padding: `40` and `401` become `400000` and `401000`, and `400000` still sorts before `401000`. Left padding would destroy the prefix relationship.

---

## 2. Searching for a free account code

**Inputs.** A starting code (text), the set of codes already known to be taken (optional), the active company.

**Output.** A code not used by any account belonging to a parent company or a child company of the active company.

**Algorithm.**

1. If the starting code is available, return it.
2. Split the starting code into three parts with the pattern "leading part, trailing run of digits, trailing part", where the trailing run of digits is the **last** run of digits in the code and the trailing part is whatever follows it.
3. If the trailing run of digits is non-empty, let `d` be its length and `n` its numeric value. For every integer `num` from `n + 1` up to `10^d − 1`, form `leading part + num rendered in exactly d digits with leading zeros + trailing part` and return the first available one.
4. If step 3 exhausts without success (or there was no digit run), for `num` from 0 to 98, form `starting code + ".copy"` when `num` is 0, and `starting code + ".copy" + (num + 1)` otherwise, and return the first available one.
5. If nothing is available, abort:

```
Cannot generate an unused account code.
```

**Availability rule.** A code is available when it is not in the known-taken set and no account bearing exactly that code belongs to a company that is an ancestor or a descendant of the active company (the active company included). Archived accounts count as taking their code.

**Worked examples.**

| Starting code | Codes tried, in order |
|---|---|
| `102100` | `102100`, `102101`, `102102`, `102103`, … , `102199` |
| `1598` | `1598`, `1599`, `1600`, `1601`, … , `9999` |
| `10.01.08` | `10.01.08`, `10.01.09`, `10.01.10`, `10.01.11`, … , `10.01.99` |
| `10.01.97` | `10.01.97`, `10.01.98`, `10.01.99`, then `10.01.97.copy`, `10.01.97.copy2`, … |
| `1021A` | `1021A`, `1022A`, `1023A`, … , `9999A` |
| `hello` | `hello`, `hello.copy`, `hello.copy2`, … , `hello.copy99` |
| `9998` | `9998`, `9999`, then `9998.copy`, `9998.copy2`, … |

---

## 3. Starting code for a prefixed utility account

**Inputs.** A prefix (text), `code_digits` (integer).

**Output.** The code from which the free-code search of section 2 starts.

```
start_code(prefix, code_digits) =
    prefix left-justified to (code_digits − 1) characters with "0", then "1"     if length(prefix) < code_digits
    prefix                                                                        otherwise
```

**Worked examples.**

| Prefix | `code_digits` | Starting code | First account created |
|---|---|---|---|
| `5710` | 6 | `571001` | `571001` if free, else `571002`, … |
| `999` | 6 | `999001` | `999001` |
| `102.01.0` | 9 | `102.01.01` | `102.01.01` |
| `1014` | 4 | `1014` | `1014` |

Note that the trailing `1` guarantees the first utility account never collides with a template account ending in zeros, because template accounts are right-padded with zeros.

---

## 4. Renumbering accounts when a liquidity prefix changes

**Inputs.** The old prefix, the new prefix, the accounts of the company whose code starts with the old prefix and whose account type is Bank and Cash or Credit Card, processed in ascending code order.

**Output.** New codes for those accounts.

```
remainder      = current_code with the first occurrence of old_prefix removed
trimmed        = remainder with leading "0" characters removed
target_width   = length(current_code) − length(new_prefix)
new_code       = new_prefix + trimmed right-justified to target_width with "0"
```

**Worked example.** Old prefix `1014`, new prefix `5100`, current code `101401`, current code length 6.

```
remainder     = "01"
trimmed       = "1"
target_width  = 6 − 4 = 2
padded        = "01"
new_code      = "5100" + "01" = "510001"
```

**Second worked example.** Old prefix `512`, new prefix `55`, current code `512004`, current code length 6.

```
remainder     = "004"
trimmed       = "4"
target_width  = 6 − 2 = 4
padded        = "0004"
new_code      = "55" + "0004" = "550004"
```

Accounts are processed in ascending current code order so that a renumbering never collides with a code that has not been moved yet.

---

## 5. Template composition

### 5.1 Parent chain

The parent chain of a template code is built by this procedure.

1. Start with an empty ordered list.
2. While the current code is a known template code, append the current code to the list and replace the current code by the parent code recorded in that template's descriptor.
3. When the current code is no longer a known template code, the list is the parent chain.

The chain is **specific first**. For the Spanish small and medium enterprise chart: `["es_pymes", "es_common"]`. For the Argentine registered taxpayer chart: `["ar_ri", "ar_base"]`. For a West African country such as Senegal: `["sn", "syscohada"]`.

### 5.2 Merging template functions

The composed result is a mapping from entity to a mapping from row identifier to field values. It is built by this procedure.

1. Start with an empty composed mapping.
2. Walk the list formed by the unqualified scope followed by the parent chain of the requested template code, in that order. The unqualified scope holds the shipped functions that carry no template code and therefore apply to every template.
3. For each element of that list, take the entities registered for it in the fixed entity order given in section 5.5, and for each entity take every registered function in registration order.
4. Invoke the function with the requested template code and take the mapping it returns.
5. When the entity is the company-level bucket, merge the returned values into the company-level mapping; the last value written for a key wins.
6. When the entity is any other entity, merge each returned row into the composed mapping under its row identifier; the last value written for a field of that row wins.

Because the loop walks the chain specific-first, a value written by the child is **overwritten** by the parent when both set the same key on the same row. Country packages therefore place a value that must win in the **parent-most** function whose scope still matches.

### 5.3 Merging delimited data files

The merged result is a mapping from row identifier to row. It is built by this procedure.

1. Start with an empty merged mapping.
2. Take the parent chain of the requested template code and reverse it, so that the most generic template comes first and the requested template comes last. When the parent chain is empty, use a single unnamed element instead.
3. For each element in that order, locate the shipped delimited data file of the target entity for that template code. When no such file is shipped, skip the element.
4. Read the file row by row in file order.
5. When the row's identifier column is non-empty, remember it as the current row identifier and merge every non-empty recognised column of the row into the merged mapping under that identifier.
6. For every path column of the row that carries a non-empty value, append a new nested creation instruction to the current row, or extend the nested creation instruction already opened for that path in the same physical row.

This loop walks the chain **generic first**, so for delimited data the **child wins**.

**Worked example.** The Spanish package ships `account.account-es_common.csv` with 598 account rows and `account.account-es_pymes.csv` with 9 rows. Loading `es_pymes` reads the common file first, then the small and medium enterprise file. The 9 overlapping row identifiers have their non-empty columns replaced. Result: 598 accounts, 9 of which carry the small and medium enterprise name or code.

**Worked example of asymmetry.** If the parent template function sets the company-level key `code_digits` to 6 and the child template function sets it to 8, the composed value is **6**, because the parent writes last. If the parent's delimited account file gives account `1000` the name "Capital" and the child's file gives it the name "Share capital", the composed name is **"Share capital"**, because the child's file is read last.

### 5.4 Row identifier continuation and nested instructions

Within one delimited file:

1. A row whose identifier column is non-empty opens a new record and becomes the current record.
2. A row whose identifier column is empty continues the current record and contributes only its path columns.
3. A path column `relation/field` creates, on the **first** use of the path `relation` within that physical row, one new nested creation instruction appended to `relation`, and then writes `field` into it. Every path column of the same row that shares the prefix `relation` writes into that same nested instruction.
4. A path column `relation/sub_relation/field` walks two levels the same way.

**Worked example.** Four consecutive rows of a tax file:

```
row 1: id = sale_tax_template, name = 15%, amount = 15, type_tax_use = sale,
       repartition_line_ids/document_type = invoice,
       repartition_line_ids/factor_percent = 100,
       repartition_line_ids/repartition_type = base
row 2: id = (empty), repartition_line_ids/document_type = invoice,
       repartition_line_ids/factor_percent = 100,
       repartition_line_ids/repartition_type = tax,
       repartition_line_ids/account_id = tax_received
row 3: id = (empty), repartition_line_ids/document_type = refund,
       repartition_line_ids/factor_percent = 100,
       repartition_line_ids/repartition_type = base
row 4: id = (empty), repartition_line_ids/document_type = refund,
       repartition_line_ids/factor_percent = 100,
       repartition_line_ids/repartition_type = tax,
       repartition_line_ids/account_id = tax_received
```

produces one Tax row named `15%` with four repartition creation instructions, in that order.

### 5.5 Value conversion

| Column shape | Conversion |
|---|---|
| empty value | column ignored entirely |
| column name contains `@` | value kept verbatim as a translation |
| column name contains `/` | value used only to fill the nested instruction; the top-level key is initialised as an empty list |
| target field is boolean, integer or decimal | value parsed from its text form; `1`/`0` and `True`/`False` are accepted for boolean |
| target field is single-line text | leading and trailing whitespace removed |
| any other field | value kept as text and resolved later |

---

## 6. Withholding tax amounts

### 6.1 Source amounts

When the payment registration wizard first opens, it computes, for every aggregated withholding key, four source values:

| Value | Meaning |
|---|---|
| `source_base_amount_currency` | The base on which the withholding tax applies, in the **invoice** currency. |
| `source_base_amount` | The same base, in the **company** currency. |
| `source_tax_amount_currency` | The withheld amount, in the invoice currency, sign-flipped to positive. |
| `source_tax_amount` | The same withheld amount, in the company currency, sign-flipped to positive. |
| `source_currency` | The invoice currency. |

The sign flip is needed because a withholding tax carries a negative rate; the aggregation returns a negative tax amount and the line stores the absolute value.

### 6.2 Converting source amounts into line amounts

Let:

- `source_currency` be the currency in which the source amounts were captured,
- `company_currency` be the company currency,
- `line_currency` be the currency of the payment (or the company currency when the payment has none),
- `date` be the payment date.

Four mutually exclusive cases produce `original_base_amount` and `original_tax_amount`, both expressed in `line_currency`:

| Case | Condition | Rate used | Base taken from | Withheld taken from |
|---|---|---|---|---|
| A | there is no source currency (the line was typed by hand) | 1 | the line's own base amount | recomputed by running the tax engine on the typed base with withholding enabled, then negated |
| B | `source_currency = line_currency` | 1 | `source_base_amount_currency` | `source_tax_amount_currency` |
| C | `source_currency ≠ company_currency` and `line_currency = company_currency` | rate from `source_currency` to `company_currency` at `date` | `source_base_amount_currency` | `source_tax_amount_currency` |
| D | every other combination | rate from `company_currency` to `line_currency` at `date` | `source_base_amount` | `source_tax_amount` |

```
original_base_amount = round(base_taken × rate, line_currency_decimal_places)
original_tax_amount  = round(withheld_taken × rate, line_currency_decimal_places)
```

**Worked example, case B.** Invoice of 10,000.00 in the company currency with a 10 percent withholding tax. The wizard captures `source_base_amount_currency = 10,000.00` and `source_tax_amount_currency = 1,000.00`. The payment is in the same currency. `rate = 1`. `original_base_amount = 10,000.00`, `original_tax_amount = 1,000.00`.

**Worked example, case C.** Invoice of 8,000.00 in a foreign currency, company currency different, payment in the company currency, rate at the payment date 1.25 company units per foreign unit. `original_base_amount = round(8,000.00 × 1.25, 2) = 10,000.00`; with a 3 percent withholding, `source_tax_amount_currency = 240.00` and `original_tax_amount = round(240.00 × 1.25, 2) = 300.00`.

**Worked example, case D.** Company currency amounts captured as `source_base_amount = 10,000.00` and `source_tax_amount = 300.00`; the payment is made in a foreign currency whose rate from the company currency is 0.80. `original_base_amount = round(10,000.00 × 0.80, 2) = 8,000.00` and `original_tax_amount = round(300.00 × 0.80, 2) = 240.00`.

### 6.3 Proration factor for partial payments

Only the wizard prorates; a stored line on a payment uses a factor of 1.

```
moves_total_amount = Σ over the payment-term journal items of the invoices in the first batch,
                     each converted into the wizard currency at the payment date
full_amount        = the amount currently due for the batch, in the wizard currency
split_factor       = | full_amount ÷ moves_total_amount |
percentage_paid    = | wizard_amount ÷ full_amount | × split_factor
```

When `full_amount` is zero, `percentage_paid = 0`.

```
base_amount = round(original_base_amount × percentage_paid, line_currency_decimal_places)
```

**Worked example, full payment.** Invoice total 10,000.00, nothing paid yet, so `moves_total_amount = 10,000.00`, `full_amount = 10,000.00`, `split_factor = 1`. The user pays 10,000.00: `percentage_paid = 1`, `base_amount = 10,000.00`.

**Worked example, half payment.** Same invoice, the user pays 5,000.00: `percentage_paid = |5,000 ÷ 10,000| × 1 = 0.5`, `base_amount = round(10,000.00 × 0.5, 2) = 5,000.00`.

**Worked example, second installment after a first payment.** Invoice total 10,000.00, 4,000.00 already paid, so `full_amount = 6,000.00` and `moves_total_amount = 10,000.00`, hence `split_factor = 0.6`. The user now pays the remaining 6,000.00: `percentage_paid = |6,000 ÷ 6,000| × 0.6 = 0.6`, `base_amount = round(10,000.00 × 0.6, 2) = 6,000.00`. The base therefore reflects the share of the **original invoice** being settled, not the share of the residual.

### 6.4 Withheld amount from the base

```
amount = round(original_tax_amount × base_amount ÷ original_base_amount, line_currency_decimal_places)   when original_base_amount ≠ 0
amount = 0                                                                                               when original_base_amount = 0
```

The ratio is never rounded on its own; the multiplication and the division are performed first and the single rounding is applied to the result.

**Worked example.** `original_base_amount = 10,000.00`, `original_tax_amount = 1,000.00`, the user overrides `base_amount` to 3,333.33.

```
amount = round(1,000.00 × 3,333.33 ÷ 10,000.00, 2) = round(333.333, 2) = 333.33
```

If the ratio had been rounded first (0.33) the result would have been 330.00, which is wrong.

**Worked example with a fractional rate.** A 10.666666666667 percent withholding on a base of 7,500.00. The engine computes `original_tax_amount = round(7,500.00 × 0.10666666666667, 2) = 800.00`. Overriding the base to 7,000.00 gives `amount = round(800.00 × 7,000.00 ÷ 7,500.00, 2) = round(746.666…, 2) = 746.67`.

### 6.5 Net amount of the payment

```
net_amount = payment_amount − Σ over withholding lines of line.amount
```

The net amount is expressed in the wizard currency. A negative net amount is refused.

**Worked example.** Payment amount 10,000.00, two withholding lines of 1,000.00 and 200.00: `net_amount = 8,800.00`. The customer transfers 8,800.00 and the two withheld amounts are remitted to the tax administration by the customer.

### 6.6 Conversion of the line into the payment entry

When the journal items are prepared, the line is converted back:

```
conversion_rate = rate from company_currency to line_currency at the payment date
base_amount_in_company_currency = round(base_amount ÷ conversion_rate, company_currency_decimal_places)   when conversion_rate ≠ 0
tax_amount_in_company_currency  = round(−amount ÷ conversion_rate, company_currency_decimal_places)       when conversion_rate ≠ 0
```

When the conversion rate is zero, both company-currency amounts are zero.

The sign applied to the whole base line is `+1` for an inbound payment and `−1` for an outbound payment.

**Refund detection.** The line is treated as concerning a refund when

```
(tax scope = "sale" AND payment direction = "outbound") OR (tax scope = "purchase" AND payment direction = "inbound")
```

which selects the refund half of the tax's repartition instead of the invoice half.

### 6.7 Placeholder numbering

Lines are sorted by their natural order and grouped by the numbering series they would consume. Within a group of `k` lines drawing on a series whose next value is `v`:

```
placeholder(line at position i, zero based) = the text the series produces for value (v + i)
```

Lines with no series show no placeholder.

**Worked example.** A series with prefix `WH/` and padding 5 whose next value is 42. Three lines draw on it. Their placeholders are `WH/00042`, `WH/00043`, `WH/00044`. No series value is consumed until the payment is posted.

---

## 7. Report expression engines

Every engine returns, for one expression, one figure per report column period. All engines run against the journal items in the selected period, company scope and, where applicable, foreign registration scope.

### 7.1 Domain engine

**Formula.** A condition expression over journal items. **Subformula.** `sum` or `-sum`.

```
value = Σ over matching journal items of item.balance        when subformula = "sum"
value = − Σ over matching journal items of item.balance      when subformula = "-sum"
```

The formula is validated at save time by parsing it and running a search with it; failure raises the invalid-formula message.

**Worked example.** Formula `[("account_id.account_type", "=", "liability_payable")]`, subformula `-sum`. The payable accounts carry a credit balance of −45,000.00. The expression returns 45,000.00.

### 7.2 Tax tags engine

**Formula.** A report tag name, optionally prefixed with a hyphen.

Every tag exists as a **positive variant** and a **negative variant**. A journal item that carries the positive variant contributes `+balance`; one that carries the negative variant contributes `−balance`. A leading hyphen on the formula swaps the two roles for this expression.

```
value = Σ over items carrying positive variant of balance − Σ over items carrying negative variant of balance
```

Tag resolution is restricted to tags whose applicability is "taxes" and whose country is the report's country.

**Worked example.** Tag `05` in a value-added tax return. Sales invoices tagged `05` on their base lines total a credit balance of −120,000.00 which, with the sign convention of a positive tag on a credit, contributes 120,000.00. One credit note tagged with the negative variant of `05` carries a debit balance of 5,000.00, contributing −5,000.00. The expression returns 115,000.00.

### 7.3 Aggregation engine

**Formula.** Either the reserved word `sum_children`, or an arithmetic expression built from:

- numbers, matching `[+-]?(digits[.digits]|.digits)([eE][+-]?digits)?`,
- terms of the form `line_code.expression_label`,
- the operators `+`, `−`, `*`, `/`,
- parentheses and whitespace.

```
sum_children  →  value = Σ over child lines of (the child expression carrying the same label)
otherwise     →  value = the arithmetic expression with each term replaced by that expression's value
```

**Dependency expansion.** Before computing, the engine expands the expression into its transitive dependency set: for each term, it finds the expression whose line code and label match, within the same report, or within the report named by a `cross_report(...)` subformula. Any dependency that is itself an aggregation is expanded in turn, until no new expression is added.

**Conditional subformula.** `if_other_expr_above(line_code.label, threshold)` yields the computed value only when the named expression's value is strictly above the threshold, and zero otherwise. `if_other_expr_below(line_code.label, threshold)` is the mirror image.

**Worked example.** Line `NET` has the expression `balance` with the aggregation formula `OUTPUT.balance - INPUT.balance`. `OUTPUT.balance` evaluates to 21,000.00 and `INPUT.balance` to 13,400.00. `NET.balance = 7,600.00`.

**Worked example of `sum_children`.** Line `SALES` has children `SALES_DOMESTIC` (100,000.00), `SALES_UNION` (25,000.00) and `SALES_EXPORT` (12,500.00), all with a `balance` expression. `SALES.balance = 137,500.00`.

**Worked example of a conditional.** Line `PENALTY` has the formula `NET.balance * 0.02` with the subformula `if_other_expr_above(NET.balance, 0)`. With `NET.balance = 7,600.00` the penalty is `152.00`. With `NET.balance = −500.00` the penalty is `0.00`.

### 7.4 Prefix of account codes engine

**Formula.** A signed sum of prefix terms, written without needing spaces.

Parsing: all spaces are removed, then the text is split immediately **before** every `+` and `−`. Each non-empty token must match:

```
[+|-]  prefix  [ \( excluded, excluded, ... \) ]  [D|C]
```

where `prefix` is a run of letters, digits and dots, or the form `tag(external identifier)`; the parenthesised list names prefixes to subtract from the term's own prefix; and the trailing letter restricts the term to accounts whose balance is a debit (`D`) or a credit (`C`).

```
term_value(prefix, excluded, balance_character) =
    Σ over accounts whose code starts with prefix
      and whose code does not start with any excluded prefix
      and (no restriction, or balance > 0 for D, or balance < 0 for C)
    of account.balance
value = Σ over terms of (term sign × term_value)
```

**Worked example.** Formula `21\(210,215\)+22D-70C`.

- Term 1: `+21\(210,215\)` sums every account whose code starts with `21` except those starting with `210` or `215`. Accounts `211000` (−4,000.00), `212000` (−1,500.00) and `215000` (−900.00, excluded) give `−5,500.00`.
- Term 2: `+22D` sums accounts starting with `22` that carry a debit balance: `220000` (3,000.00) counts, `221000` (−200.00) does not. Gives `3,000.00`.
- Term 3: `−70C` subtracts the sum of accounts starting with `70` that carry a credit balance: `700000` (−60,000.00) gives `−60,000.00`, negated to `+60,000.00`.

```
value = −5,500.00 + 3,000.00 + 60,000.00 = 57,500.00
```

### 7.5 External value engine

**Formula.** `sum` or `most_recent`. **Subformula.** `editable`, optionally followed by `;rounding=<digits>`.

```
sum         → value = Σ over external values of this expression, this company, dated inside the period, of value
most_recent → value = the value of the external value with the greatest date inside the period; when several share that
              date, the one with the greatest identifier
```

`editable` makes the figure typeable in the report screen; `rounding=<digits>` fixes the number of decimal places of the typed figure (0 for a whole percentage).

**Shortcut forms.** When a line declares an external formula shortcut:

| Shortcut value | Stored formula | Stored subformula | Stored figure type |
|---|---|---|---|
| `percentage` | `most_recent` | `editable;rounding=0` | Percentage |
| `monetary` | `sum` | `editable` | Monetary |
| anything else | `most_recent` | `editable` | the shortcut value |

**Worked example.** A line holds a manually declared prorated deduction rate for the year. In January the accountant types 78; the external value `78` dated 1 January is stored. Every month of the year, `most_recent` returns 78. In July the rate is revised to 81 dated 1 July; from July onwards the expression returns 81 and the periods before July still return 78.

### 7.6 Custom function engine

**Formula.** The name of a computation contributed by a country package. **Subformula.** Free text handed to that computation.

A custom expression is **not auditable**: its figure cannot be expanded into journal items. All five other engines are auditable.

---

## 8. Carryover

**Inputs.** A `_carryover_<name>` expression's value for the closing period, the target expression, the company, the period end date.

**Output.** A Financial Report External Value.

**Target resolution.**

```
target = the expression named by carryover_target, written "line_code.expression_label"
target = the expression of the same line whose label is "_applied_carryover_" + <name>   when carryover_target is empty
```

**Stored record.**

| Field | Value |
|---|---|
| Target expression | the resolved target |
| Value | the carryover expression's computed amount |
| Date | the last day of the period that produced the amount |
| Company | the company the return was run for |
| Origin line | the line that produced the amount |
| Origin expression label | the carryover expression's label |

**Reading back.** The `_applied_carryover_<name>` expression of the following period reads the external values whose date falls in or before that period, according to its own engine (usually `sum` with a date scope reaching back).

**Worked example.** A value-added tax regime that allows a credit to be carried forward.

| Period | Output tax | Input tax | Applied carryover in | Net | Carryover out |
|---|---|---|---|---|---|
| March | 4,000.00 | 5,250.00 | 0.00 | −1,250.00 | 1,250.00 |
| April | 6,000.00 | 4,000.00 | 1,250.00 | 750.00 | 0.00 |

March: `NET = 4,000.00 − 5,250.00 − 0.00 = −1,250.00`. Because the net is negative, `_carryover_balance` evaluates to 1,250.00 and an external value of 1,250.00 dated 31 March is written against `NET._applied_carryover_balance`.

April: `NET._applied_carryover_balance` reads 1,250.00. `NET = 6,000.00 − 4,000.00 − 1,250.00 = 750.00`. The net is positive, `_carryover_balance` evaluates to 0.00 and nothing is carried further. The company pays 750.00.

---

## 9. Report line level

```
level(line) = 1                                  when the line has no parent
level(line) = level(parent) + 3                  when level(parent) = 0
level(line) = level(parent) + 2                  otherwise
```

**Worked example.** A root line at level 1 has a child at level 3, whose child is at level 5. A line explicitly set to level 0 (used by headers that must not be indented) has children at level 3.

---

## 10. Report copy renaming

```
copied_report_name = name + " (copy)", repeated until no report carries that name
copied_line_code   = code + "_COPY", repeated until no line carries that code   (a line with no code keeps none)
```

Every aggregation formula and subformula of the copied report has each old line code replaced by its new code, matching only on whole tokens (the replacement requires a non-word character on each side).

**Worked example.** Report "Tax Return" with lines coded `OUT`, `IN`, `NET`, where `NET.balance = OUT.balance - IN.balance`. The copy is named "Tax Return (copy)" with lines `OUT_COPY`, `IN_COPY`, `NET_COPY` and `NET_COPY.balance = OUT_COPY.balance - IN_COPY.balance`. Copying that copy produces "Tax Return (copy) (copy)" with codes `OUT_COPY_COPY` and so on.

---

## 11. Account group assignment by prefix

An account is assigned to the account group whose prefix range contains its code and whose starting prefix is the longest.

```
candidate(group, account) =
     group.code_prefix_start ≤ left(account.code, length(group.code_prefix_start))
 AND group.code_prefix_end   ≥ left(account.code, length(group.code_prefix_end))
```

Among candidates, order by the **length of the starting prefix descending**, then by the group identifier ascending, and take the first.

**Worked example.** Groups `4` (Third parties, prefix range `4` to `4`), `40` (Suppliers, `40` to `40`) and `401` (Suppliers, ordinary accounts, `401` to `401`). Account `401000` matches all three. The longest starting prefix is `401`, so the account belongs to "Suppliers, ordinary accounts". Account `408000` matches `4` and `40`; the longest is `40`, so it belongs to "Suppliers".

**Overlap guard.** Two groups of the same company whose starting prefixes have the same length may not have overlapping ranges:

```
Account Groups with the same granularity can't overlap
```

**Prefix length guard.** The starting and ending prefixes must be the same length:

```
The length of the starting and the ending code prefix must be the same
```

When one prefix is empty it defaults to the other; when the ending prefix sorts before the starting prefix the ending prefix is replaced by the starting prefix.

---

## 12. Postal code range normalisation for fiscal position detection

**Inputs.** A lower bound and an upper bound, both text.

Normalisation runs as follows.

1. When either bound is empty, or either bound contains a character that is not a digit, both bounds are stored unchanged.
2. Otherwise the padding width is the greater of the two bound lengths.
3. The lower bound is right-justified to the padding width by prefixing the digit zero as many times as needed.
4. The upper bound is right-justified to the padding width the same way.

Comparison at detection time is a plain text comparison of the contact's postal code against the two padded bounds, inclusive at both ends.

**Worked example.** Bounds `100` and `9500` become `0100` and `9500`. A contact whose postal code is `0500` matches. A contact whose postal code is `9600` does not. A contact whose postal code is `500` does **not** match textually, which is why country packages that use ranges also normalise contact postal codes.

**Validation.**

```
Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From".
```

---

## 13. Progressive scale withholding

Several countries compute a withholding on a progressive scale. The framework stores the scale as a set of brackets, each with a lower bound of accumulated base, a fixed amount, a percentage and the bound from which the percentage applies.

```
bracket = the bracket with the greatest from_amount that does not exceed accumulated_base
withheld_total = bracket.fixed_amount + (accumulated_base − bracket.percentage_from) × bracket.percentage ÷ 100
withheld_now   = round(withheld_total − already_withheld_this_period, currency_decimal_places)
```

`accumulated_base` is the sum of the bases of every payment to the same counterpart within the applicable period, including the current one. `already_withheld_this_period` is the sum of the amounts already withheld against that counterpart in the same period.

**Worked example.** A scale with the brackets below, all amounts in the local currency:

| From accumulated base | Fixed amount | Percentage | Percentage applies from |
|---|---|---|---|
| 0.00 | 0.00 | 5 | 0.00 |
| 100,000.00 | 5,000.00 | 10 | 100,000.00 |
| 300,000.00 | 25,000.00 | 15 | 300,000.00 |

A counterpart has already received 250,000.00 this period, on which 20,000.00 was withheld. A new payment of 100,000.00 is registered.

```
accumulated_base = 250,000.00 + 100,000.00 = 350,000.00
bracket          = the third one (from 300,000.00)
withheld_total   = 25,000.00 + (350,000.00 − 300,000.00) × 15 ÷ 100 = 25,000.00 + 7,500.00 = 32,500.00
withheld_now     = round(32,500.00 − 20,000.00, 2) = 12,500.00
```

The payment therefore withholds 12,500.00 and the counterpart receives 87,500.00.

---

## 14. Rounding rules of the domain

1. **Every monetary result is rounded to the decimal places of the currency it is expressed in.** The rounding is half-up away from zero: a value exactly on the half rounds to the larger absolute value. **Industry-standard completion** for the tie-breaking direction.
2. **A ratio is never rounded before it is used.** Multiplication and division are performed at full precision and the single rounding is applied to the final product.
3. **A report column typed as an integer applies the report's integer rounding setting**: Nearest (half-up), Up (towards positive infinity in absolute terms, that is away from zero) or Down (towards zero).
4. **The tax computation rounding method** is a company setting written by the template: "Round per Tax" computes each tax's amount on the whole document and rounds once, "Round per Line" rounds each tax on each line. The platform default is "Round per Tax".
5. **Percentages typed into report external values** obey the `rounding=<digits>` part of the subformula, `0` meaning whole percentage points.

**Worked example of the two tax rounding methods.** Three lines of 33.33 each with a 21 percent tax, currency with two decimal places.

```
Round per Tax : tax = round(99.99 × 0.21, 2) = round(20.9979, 2) = 21.00
Round per Line: tax = round(33.33 × 0.21, 2) × 3 = round(6.9993, 2) × 3 = 7.00 × 3 = 21.00
```

With four lines of 33.33:

```
Round per Tax : tax = round(133.32 × 0.21, 2) = round(27.9972, 2) = 28.00
Round per Line: tax = 7.00 × 4 = 28.00
```

With three lines of 10.05:

```
Round per Tax : tax = round(30.15 × 0.21, 2) = round(6.3315, 2) = 6.33
Round per Line: tax = round(10.05 × 0.21, 2) × 3 = round(2.1105, 2) × 3 = 2.11 × 3 = 6.33
```

With three lines of 10.03:

```
Round per Tax : tax = round(30.09 × 0.21, 2) = round(6.3189, 2) = 6.32
Round per Line: tax = round(10.03 × 0.21, 2) × 3 = round(2.1063, 2) × 3 = 2.11 × 3 = 6.33
```

The one-cent divergence is the reason the setting exists and the reason a country package that mandates one method writes it into the template.

---

## 15. Country arithmetic

### 15.1 Mexico: the payment complement exchange rate

A payment in a currency other than the local one carries an exchange rate on the electronic document:

```
exchange_rate = round(payment_amount_in_local_currency ÷ payment_amount_in_foreign_currency, 6)
```

When the payment currency equals the local currency, no rate is transmitted.

**Worked example.** A payment of 1,000.00 in a foreign currency settles 17,432.50 local units. `exchange_rate = round(17.4325, 6) = 17.432500`.

### 15.2 Mexico: the total in words

The invoice total is rendered in words in the local language, with the cents expressed as a fraction of one hundred, and the currency code appended.

**Worked example.** 1,234.56 renders as "MIL DOSCIENTOS TREINTA Y CUATRO PESOS 56/100 M.N.".

### 15.3 India: splitting a combined goods and services tax

A single rate is split into a central part and a state part when the place of supply is inside the supplier's state, and applied as one integrated tax when it is not.

```
intra-state:  central_part = round(base × rate ÷ 2 ÷ 100, 2)
              state_part   = round(base × rate ÷ 2 ÷ 100, 2)
inter-state:  integrated   = round(base × rate ÷ 100, 2)
```

**Worked example.** Base 10,000.00, rate 18 percent, intra-state: central part 900.00, state part 900.00, total 1,800.00. Inter-state: integrated 1,800.00. The total is identical; the accounts, the tags and the return lines differ.

### 15.4 India: cess on top of a tax

A cess is expressed either as a percentage of the base or as a fixed amount per unit, and is computed on the base, not on the tax:

```
cess_percentage = round(base × cess_rate ÷ 100, 2)
cess_per_unit   = round(quantity × cess_amount_per_unit, 2)
```

### 15.5 Brazil: tax base reductions

Brazilian taxes are declared with a reduction of the base and, for some of them, with the tax included in its own base.

```
reduced_base   = round(base × (1 − reduction_percentage ÷ 100), 2)
tax_on_reduced = round(reduced_base × rate ÷ 100, 2)
```

For a tax included in its own base (the "gross up" form):

```
grossed_base = round(base ÷ (1 − rate ÷ 100), 2)
tax          = round(grossed_base × rate ÷ 100, 2)
```

**Worked example.** Base 1,000.00, reduction 26.57 percent, rate 18 percent.

```
reduced_base   = round(1,000.00 × 0.7343, 2) = 734.30
tax_on_reduced = round(734.30 × 0.18, 2)     = 132.17
```

**Worked example of the gross up.** Base 1,000.00, rate 18 percent included.

```
grossed_base = round(1,000.00 ÷ 0.82, 2) = 1,219.51
tax          = round(1,219.51 × 0.18, 2) = 219.51
```

### 15.6 Chile and Peru: rounding the document total to whole units

Where a country requires the printed total to carry no fractional units, the document total is rounded to zero decimal places and the difference is posted to a rounding account.

```
rounded_total = round(total, 0)
difference    = rounded_total − total
```

**Worked example.** Total 118,456.42 becomes 118,456.00 and a difference of −0.42 is posted to the rounding account. Total 118,456.62 becomes 118,457.00 and a difference of +0.38 is posted.

### 15.7 Italy: the stamp duty threshold

A stamp duty is added to an invoice whose exempt base exceeds a threshold.

```
stamp_duty_applies = (Σ base of lines carrying an exempt tax) > threshold
stamp_duty_amount  = the fixed amount configured for the company
```

**Worked example.** Threshold 77.47, fixed amount 2.00. An invoice with 150.00 of exempt base carries a stamp duty of 2.00. An invoice with 60.00 of exempt base carries none.

### 15.8 Saudi Arabia and the Gulf: the receipt visual code payload

The payload encodes five fields, each written as a type byte, a length byte and the value in the encoding of the field:

| Field number | Content |
|---|---|
| 1 | Seller name |
| 2 | Seller tax identification number |
| 3 | Invoice timestamp in the extended date and time format |
| 4 | Invoice total including tax |
| 5 | Tax total |

The concatenated bytes are encoded in base 64 and rendered as a two-dimensional visual code.

**Worked example of one field.** Seller name "Al Noor Trading" is 15 characters, so the field is the byte `1`, the byte `15`, then the 15 characters.

### 15.9 European Union one-stop shop: choosing the destination rate

When the one-stop shop regime is active for a company, a sale of goods or services to a private consumer in another member state is taxed at the **destination** rate. The system creates one fiscal position per destination country and, in it, a mapping from each domestic tax to the destination tax of the closest matching rate.

Generation runs as follows, once per company.

1. Take every member state of the union other than the company's own fiscal country.
2. For each such destination country, take every rate published for that country in the shipped rate table.
3. For each published rate, create a sales-scoped tax whose name is the rate percentage, a space, the percent sign, a space and the destination country code, or reuse the tax of that name when it already exists for the company.
4. Create one fiscal position for the destination country, or reuse the one that already exists.
5. For each domestic sales tax of the company, add to that fiscal position a mapping from the domestic tax to the destination tax whose rate is closest to the domestic rate. An exact match is preferred; when no rate matches exactly, the next higher published rate is used; when no higher rate exists, the next lower published rate is used.

**Worked example.** A company in a member state whose standard rate is 21 percent sells to a consumer in a member state whose standard rate is 23 percent. The one-stop shop fiscal position for that destination maps the 21 percent domestic tax to a 23 percent destination tax. The invoice shows 23 percent and the amount is reported in the one-stop shop return rather than in the domestic return.

### 15.10 Argentina: the value-added tax perception base

A perception is a tax collected in advance on behalf of the administration, computed on the invoice base after the ordinary value-added tax:

```
perception = round(base × perception_rate ÷ 100, 2)
```

and is added to the amount due without changing the taxable base of the ordinary tax.

**Worked example.** Base 100,000.00, ordinary rate 21 percent (21,000.00), perception rate 3 percent (3,000.00). The invoice total is 124,000.00; the return reports 21,000.00 of ordinary tax and 3,000.00 of perception.

---

## 16. Order of operations for a template load

The following order is normative because later steps read values written by earlier ones.

```
 1. resolve template descriptor and parent chain
 2. compose template functions           (specific first, parent overwrites)
 3. compose delimited files              (generic first, child overwrites)
 4. purge existing configuration         (only when no accounting exists)
 5. write company-level values           (currency, prefixes, fiscal country, defaults)
 6. pad account codes                    (uses code_digits written in step 5's source data)
 7. reorder models                       (fiscal positions and reconciliation models last)
 8. drop unknown columns
 9. translate the journal code
10. write records, deferring forward references
11. create utility accounts              (uses the prefixes written in step 5)
12. wire journals and company defaults   (uses the records written in step 10)
13. write property defaults
14. load translations
15. re-parent account groups             (uses the accounts written in step 10)
```

Reversing steps 5 and 11 would create utility accounts under the previous company's prefixes; reversing steps 10 and 15 would leave every account without a group.
