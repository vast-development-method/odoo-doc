# Calculations

Every formula and algorithm of the Electronic Invoicing and Document Interchange domain: the amounts written into an exported file, the grouping and aggregation of taxes, the presentation rounding, the reconstruction of quantity, unit price and discount when a file is imported, the correction of tax and untaxed totals, the prediction of tax category and exemption reason codes, and the code mappings. Each section states its inputs, its outputs, its precision and its order of operations, and closes with a worked example using real numbers.

Every monetary amount is produced by the tax engine of the [taxes](../taxes/calculations.md) domain. This domain supplies the grouping keys, the sign conventions and the presentation precision. The terms used for the amounts produced by that engine are:

| Term | Meaning |
|---|---|
| `raw_total_excluded` | The total of the line excluding tax, before any currency rounding, after the discount. |
| `raw_gross_total_excluded` | The total of the line excluding tax, before any currency rounding, before the discount. |
| `raw_discount_amount` | `raw_gross_total_excluded` minus `raw_total_excluded`. |
| `total_excluded` | `raw_total_excluded` rounded to the currency and adjusted so that the sum of the lines matches the document total. |
| `delta_total_excluded` | The adjustment applied to reach `total_excluded` from the rounded raw value. |
| `gross_total_excluded`, `discount_amount` | The rounded counterparts of the two raw values. |
| `raw_gross_price_unit` | The unit price excluding tax, before the discount, before rounding. |
| `tax_amount` | The amount of one tax on one line. |
| `base_amount` | The amount on which one tax applies. |

A suffix `_currency` means the amount is expressed in the document currency; the same name without the suffix means the company currency.

---

# 1. Presentation rounding

**Inputs:** an amount, a minimum number of decimal places, an optional maximum number of decimal places.
**Output:** the text written into the file.
**Order of operations:**

1. When no maximum number of decimal places is given, round the amount half away from zero to the minimum number of decimal places, write it with exactly that many decimal places and stop.
2. Otherwise round the amount half away from zero to the maximum number of decimal places.
3. Write the rounded amount with exactly the maximum number of decimal places and count the trailing zero characters of the text produced.
4. Compute the number of decimal places actually used.

```formula
decimal places used = the larger of ( minimum number of decimal places , maximum number of decimal places − count of trailing zero characters )
```

5. Write the rounded amount with exactly that number of decimal places.

Writing a value with a given number of decimal places always pads with zeros when the value has fewer significant decimals. Rounding to the maximum before writing is essential: with a maximum of two, the amount 0.499 must be written 0.50, not 0.49.

**Precision used per element family:**

| Element family | Minimum | Maximum |
|---|---|---|
| Every monetary amount of the universal business language family, except the unit price | the decimal places of the currency | none |
| The unit price of the universal business language family | 1 | 10 |
| Every monetary amount of the cross industry invoice family, except the allowance and charge amounts | 2 | 2 |
| The allowance and charge amounts of the cross industry invoice family | 2 | the decimal places of the currency |
| The unit price of the plain 2.0 builder | the decimal places of the product price precision | the same |

**Worked example.** Currency with two decimal places. The amount `1000.0` written as a unit price with minimum one and maximum ten: rounding to ten places gives `1000.0000000000`; that text has ten trailing zeros; ten minus ten is zero, which is below the minimum of one, so one decimal place is used and the text is `1000.0`. The amount `12.3456789` written the same way: rounding to ten places gives `12.3456789000`; three trailing zeros; ten minus three is seven; the text is `12.3456789`. The amount `199.5` written as a tax amount with minimum two and no maximum: the text is `199.50`.

---

# 2. Preparation of the base lines

**Inputs:** the accounting document.
**Output:** the list of base lines and tax lines used by every node builder.

## 2.1 Universal business language family

```
1. base_lines, tax_lines = the rounded base and tax lines of the accounting document
2. turn every emptying tax into a separate base line (section 2.3)
3. attach an empty value holder to the document and to every base line
4. round the raw totals excluding tax to 6 digits, in the document currency
5. round the raw totals excluding tax to 6 digits, in the company currency
6. add and round the raw gross totals and the raw discounts to 6 digits, in the document currency
7. add and round the raw gross totals and the raw discounts to 6 digits, in the company currency
8. round the raw gross totals and the raw discounts to 6 digits, in the document currency
9. round the raw gross totals and the raw discounts to 6 digits, in the company currency
10. (European semantic standard layer) turn every negative unit price into a positive unit price
    with a negated quantity
```

The six digit rounding matters because an element such as the unit price is written with up to ten decimal places; without it the text would carry the noise of binary floating point arithmetic.

## 2.2 Cross industry invoice family

1. Collect the document currency, the company currency, the supplier, the customer, the delivery address and the company. For a purchase document the supplier and the customer are swapped, and the delivery address becomes the first delivery child contact of the customer, or the customer itself when it has none.
2. Set the delivery date to the delivery date of the document, falling back to its invoice date.
3. Read the rounded base lines and tax lines of the accounting document.
4. Turn every negative unit price into a positive unit price with a negated quantity.
5. Move the cash rounding lines out of the base lines into their own list.
6. Move the early payment discount lines out of the base lines into their own list.
7. Round the raw totals excluding tax to six decimal places, in the document currency.
8. Add the raw gross totals and the raw discounts and round them to six decimal places, in the document currency.
9. Round the raw gross totals and the raw discounts to six decimal places, in the document currency.

## 2.3 Turning emptying taxes into extra lines

An emptying tax is a deposit charged on a returnable container. It must appear as a separate document line rather than as a tax, because it is not a tax at all.

```
1. split every base line into a part carrying the emptying taxes and a part carrying the rest
2. on every part that carries a removed tax, set the discount to zero, because a deposit is
   never discounted
3. group the removed tax parts by the tax itself, summing the quantities
4. for each group, create one extra base line whose quantity is the summed quantity and whose
   unit price is the price of the group divided by that quantity, and whose product is empty
5. the final base lines are the remaining parts followed by the extra lines
```

The item name of such an extra line is the name of the tax.

---

# 3. Line level amounts of the universal business language family

## 3.1 Quantity and unit code

The quantity written is the quantity of the base line, with the unit code of its unit of measure (section 11). The element differs per document type: invoiced quantity, credited quantity, debited quantity or plain quantity for an order.

## 3.2 Line allowances and charges

Three kinds of node are produced on a line, in this order.

**Discount.**

1. Take the discount amount of the line in the document currency, already rounded to the currency in step 6 of section 2.1.
2. When that amount is zero in the document currency, produce no allowance or charge node for the line and stop.
3. Otherwise produce one node whose parts are given by the table below.

| Part of the node | Value |
|---|---|
| Charge indicator | the reproduced value `true` when the discount amount is negative, otherwise the reproduced value `false` |
| Multiplier factor | the absolute value of the discount percentage of the line |
| Reason code | the reproduced code `ADK` when the discount amount is negative, otherwise the reproduced code `95` |
| Reason | the reproduced text "Discount" |
| Amount | the absolute value of the discount amount |
| Base amount | the gross total excluding tax of the line, in the document currency |

A negative discount is therefore reported as a charge, which is how an upsell is represented.

**Recycling contribution taxes.** One node per recycling contribution tax of the line.

```
node:
    charge indicator = "true" when tax_amount > 0, otherwise "false"
    reason code      = "CAV" when the lower case tax name contains "bebat",
                       otherwise "AEO"; but "100" when the indicator is "false"
    reason           = the tax name
    amount           = absolute value of tax_amount_currency
```

**Excise taxes.** One node per excise tax of the line.

```
node:
    charge indicator = "true" when tax_amount > 0, otherwise "false"
    reason           = the tax name
    amount           = absolute value of tax_amount_currency
```

No reason code is written for an excise tax.

## 3.3 Line net amount

1. Start from the gross total excluding tax of the line in the document currency, rounded to the document currency.
2. Take each allowance or charge node already produced on that line in turn. Add its amount when its charge indicator carries the reproduced value `true`; subtract its amount when the indicator carries the reproduced value `false`.

```formula
line net amount (document currency) = round( raw gross total excluding tax (document currency) , decimal places of the document currency )
                                      + sum over charge nodes of the line of ( amount of the node )
                                      − sum over allowance nodes of the line of ( amount of the node )
```

The order matters: the allowance and charge nodes are produced before the line net amount, and the line net amount reads the amounts that were actually written, not the amounts before presentation rounding. This is what makes the file internally consistent.

## 3.4 Unit price

```
gross_total_without_tax = the gross total of the line excluding every tax, in the chosen currency
unit_price             = the unit price of the line excluding every tax, derived from
                         gross_total_without_tax and the quantity
write unit_price with minimum 1 and maximum 10 decimal places
```

The unit price is the gross price, before the discount, because the discount is reported separately as a line allowance.

## 3.5 Worked example

Currency: euro, two decimal places. One line: product with internal reference `FURN_7777`, quantity 1, unit price 1000.00, discount 5 percent, one tax at 21 percent.

| Quantity | Value |
|---|---|
| `raw_gross_total_excluded_currency` | 1000.000000 |
| `raw_discount_amount_currency` | 50.000000 |
| `raw_total_excluded_currency` | 950.000000 |
| `gross_total_excluded_currency` | 1000.00 |

Nodes produced:

| Element | Value written |
|---|---|
| invoiced quantity | `1.0` with the unit code `C62` |
| allowance: charge indicator | `false` |
| allowance: multiplier factor | `5.0` |
| allowance: reason code | `95` |
| allowance: reason | `Discount` |
| allowance: amount | `50.00` |
| allowance: base amount | `1000.00` |
| line net amount | `1000.00 - 50.00 = 950.00` |
| unit price | `1000.0` |

---

# 4. Line level amounts of the plain 2.0 builder

The plain 2.0 and 2.1 syntaxes and the Belgian profile use a different line calculation, because they predate the six digit rounding.

1. Compute the total of the fixed taxes of the line: the sum of the tax amounts of the fixed taxes of the line when the profile reports fixed taxes as allowances and charges, and zero otherwise.
2. Add the rounding adjustment of the line and the total of the fixed taxes to the total excluding tax of the line.

```formula
total excluding tax (company currency) = total excluding tax (company currency)
                                         + rounding adjustment of the total excluding tax (company currency)
                                         + total of the fixed taxes (company currency)
```

3. Take the raw gross amount as the raw total excluding tax of the line in the document currency, and the rounded gross amount as the rounded total excluding tax of the line in the document currency. Subtract from the raw gross amount the raw tax amount of every recycling contribution tax of the line, and from the rounded gross amount the rounded tax amount of every recycling contribution tax of the line.

```formula
raw gross amount (document currency)     = raw total excluding tax (document currency)
                                           − sum over recycling contribution taxes of the line of ( raw tax amount )
rounded gross amount (document currency) = total excluding tax (document currency)
                                           − sum over recycling contribution taxes of the line of ( tax amount )
```

4. Compute the discount factor from the discount percentage of the line.

```formula
discount factor = 1 − discount percentage of the line ÷ 100
```

5. Compute the gross subtotal, rounded to the decimal places of the document currency, half away from zero.

| Condition | Gross subtotal |
|---|---|
| the discount factor differs from zero | round( raw gross amount ÷ discount factor ) to the document currency |
| the discount factor equals zero | round( unit price × quantity ) to the document currency |

6. Compute the gross unit price.

| Condition | Gross unit price |
|---|---|
| the quantity equals zero, or the discount factor equals zero | the unit price of the line, unchanged |
| otherwise | gross subtotal ÷ quantity |

7. Compute the discount amount.

```formula
discount amount (document currency) = gross subtotal (document currency) − rounded gross amount (document currency)
```

The line net amount written into the file is the total excluding tax of step 2; the unit price written is the gross unit price of step 6, rounded to the number of decimal places of the product price precision; the discount node carries the discount amount of step 7 with the reproduced reason code `95`, or, when that amount is negative, with the charge indicator carrying the reproduced value `true`.

**Worked example.** Quantity 5 units, unit price 100.00 in a currency with two decimal places, discount 20 percent, no fixed tax, no recycling contribution tax.

```formula
raw gross amount  = 400.000000
discount factor   = 1 − 20 ÷ 100 = 0.8
gross subtotal    = round( 400.000000 ÷ 0.8 ) = round( 500.000000 ) = 500.00
gross unit price  = 500.00 ÷ 5 = 100.00
discount amount   = 500.00 − 400.00 = 100.00
line net amount   = 400.00
```

---

# 5. Tax grouping keys

Every tax figure in a file is produced by aggregating the tax details of the base lines under a grouping key. Four keys exist and they are nested.

## 5.1 The tax category key

**Inputs:** one base line, one tax detail of that line or no tax detail at all, the export values collected in section 2, and a currency.
**Output:** the tax category key of that tax detail, or no key at all when the tax detail must not be reported as a tax.

1. When a tax detail is given and its tax is not a percentage tax, or its tax is a recycling contribution tax, or its tax is an excise tax, produce no key and stop: such a tax is reported as an allowance or a charge, not as a tax.
2. Under the European semantic standard layer only, when the base line is a cash rounding line, or the tax detail is a recycling contribution tax, or the tax detail is an excise tax, produce no key and stop.
3. Determine the tax scheme identifier: the reproduced value `GST` when the supplier country is one of the goods and services tax countries, and the reproduced value `VAT`, the identifier of the value-added tax scheme, in every other case.
4. When a tax detail is given and its tax is an intra-community reverse charge purchase tax, recognised as a percentage tax carrying a negative repartition factor, produce the key of the first row of the table below and stop.
5. When a tax detail is given and step 4 did not apply, produce the key of the second row of the table below and stop.
6. When no tax detail is given, produce the key of the third row of the table below.

| Case | Category code | Exemption reason and code | Percentage | Scheme | Withholding flag | Currency |
|---|---|---|---|---|---|---|
| Intra-community reverse charge purchase tax | the predicted category code of section 8 | the predicted exemption reason and code of section 9 | 0.0 | the scheme of step 3 | false | the given currency |
| Any other tax detail | the predicted category code of section 8 | the predicted exemption reason and code of section 9 | see the percentage table below | the scheme of step 3 | true when the tax rate is negative, otherwise false | the given currency |
| No tax detail at all | the predicted category code of section 8 for an absent tax | the predicted exemption reason of section 9 for an absent tax | 0.0 | the scheme of step 3 | false | the given currency |

The percentage of the second row is chosen as follows.

| Condition | Percentage |
|---|---|
| the predicted category code is `O`, that is "Services outside scope of tax" | no percentage at all; the element is omitted |
| the tax carries a negative repartition factor | 0.0 |
| in every other case | the rate of the tax |

The Netherlands profile clears the exemption reason code from this key. The Singapore profile clears both the exemption reason and its code and replaces the category by `ZR` when there is no tax or the rate is zero and by `SR` otherwise.

## 5.2 The tax subtotal key

The subset of the tax category key made of the currency, the withholding flag, the category code, the scheme and the percentage. Two taxes with the same category and the same rate but different exemption reasons therefore land in the same subtotal, which is what the standard requires: taxes are grouped by category and rate, never by exemption reason.

## 5.3 The tax total key

The pair made of the withholding flag and the currency. There is therefore at most one tax total per currency for the ordinary taxes and at most one withholding tax total per currency.

## 5.4 Profile adjustments to the keys

| Profile | Adjustment |
|---|---|
| European semantic standard layer | A category `E` with no exemption reason receives the reason text `Exempt from tax`. |
| Peppol international billing layer | The withholding tax total key is suppressed, because a withholding tax total is not allowed; the withholding amounts are reported as a prepaid amount instead. When the document currency differs from the company currency, the subtotal key of the company currency total is suppressed, so the second tax total carries only its total amount. |
| German, Netherlands, Singapore, Australia and New Zealand profiles | The tax total key of the company currency is suppressed entirely, so no second tax total is written. |

## 5.5 Aggregation

For each of the three keys, the tax details of every base line are aggregated: the base amounts are summed into `base_amount`, the tax amounts into `tax_amount`, in both currencies. A withholding group is written with the opposite sign, so that the amounts are positive in the file.

---

# 6. Document level allowances and charges

## 6.1 Early payment discount

A base line marked as an early payment discount is not written as a document line. It becomes one allowance or charge per tax category key.

Take each tax category key produced by the early payment discount lines in turn. The amount of the group is the aggregated total excluding tax of that group. One node is produced per group, with the parts below.

| Part of the node | Value |
|---|---|
| Charge indicator | the reproduced value `true` when the amount of the group is greater than zero, otherwise the reproduced value `false` |
| Reason code | the reproduced code `ZZZ` when the charge indicator is `true`, otherwise the reproduced code `64` |
| Reason | the reproduced text "Conditional cash/payment discount" |
| Amount | the absolute value of the amount of the group, rounded to the decimal places of the currency |
| Tax categories | one tax category node per key, carrying the category code, the percentage and the scheme of that key |

The grouping key of an early payment allowance additionally clears the exemption reason when the category is `E` and a tax is present, so that the positive and the negative parts of a mixed early payment are not merged.

## 6.2 Global discount

A base line marked as a global discount becomes one allowance or charge per tax category key.

```
node:
    charge indicator = "true" when the aggregated total excluding tax > 0, otherwise "false"
    reason code      = "ADK" when the indicator is "true", otherwise "95"
    reason           = "General upsell" when the indicator is "true", otherwise "General discount"
    amount           = absolute value of that total
    tax categories   = one node per key, carrying the category code, the percentage and the scheme
```

## 6.3 The plain 2.0 builder

The plain builder treats only the early payment lines as document level allowances and charges, with a fixed reason code: `66` when the amount is negative and `ZZZ` otherwise, and the reason `Conditional cash/payment discount`.

---

# 7. Document monetary totals of the universal business language family

The six elements are computed in this exact order, each reading the ones before it.

1. The line extension amount is the sum of the line net amounts of every document line node.

```formula
line extension amount = sum over document line nodes of ( line net amount of the node )
```

2. The tax exclusive amount starts from the line extension amount; every document level charge node adds its amount and every document level allowance node subtracts its amount.

```formula
tax exclusive amount = line extension amount
                       + sum over document level charge nodes of ( amount of the node )
                       − sum over document level allowance nodes of ( amount of the node )
```

3. The tax amount is the sum of the tax total amounts of the tax total nodes expressed in the chosen currency, less the sum of the withholding tax total amounts of the same currency. The tax inclusive amount adds it to the tax exclusive amount.

```formula
tax amount           = sum over ordinary tax total nodes in the chosen currency of ( tax total amount )
                       − sum over withholding tax total nodes in the chosen currency of ( tax total amount )
tax inclusive amount = tax exclusive amount + tax amount
```

4. The allowance total amount is the sum of the amounts of the document level nodes whose charge indicator carries the reproduced value `false`, and the charge total amount is the sum of the amounts of the document level nodes whose charge indicator carries the reproduced value `true`. Each of the two elements is omitted from the file when its sum is zero.

```formula
allowance total amount = sum over document level allowance nodes of ( amount of the node )
charge total amount    = sum over document level charge nodes of ( amount of the node )
```

5. The payable rounding amount is the difference between the expected total of the document and the tax inclusive amount. The element is omitted from the file when that difference is zero in the currency. Under the Peppol international billing layer the withholding total is then subtracted from it, because the withholding amounts are reported as a prepaid amount instead.

```formula
expected total          = sum over base lines of ( total excluding tax of the line (document currency)
                                                   + tax amount of the line (document currency) )
payable rounding amount = expected total − tax inclusive amount
payable rounding amount (Peppol international billing layer)
                        = expected total − tax inclusive amount − withholding total
```

6. The prepaid amount is the total of the accounting document less its residual amount, and the payable amount is the residual amount of the accounting document. Under the Peppol international billing layer the prepaid amount is increased by the withholding total.

```formula
prepaid amount = total of the accounting document − residual amount of the accounting document
payable amount = residual amount of the accounting document
prepaid amount (Peppol international billing layer)
               = total of the accounting document − residual amount of the accounting document + withholding total
```

When the amounts are expressed in the company currency rather than the document currency, the prepaid amount and the payable amount are read from the signed company currency totals of the accounting document instead.

The payable rounding amount therefore absorbs, in one element, both a cash rounding applied by the accounting document and any residual discrepancy between the sum of the presented line amounts and the true document total.

## 7.1 Worked example: two lines, a line discount and a document charge

**Setup.** Company in Belgium, currency euro with two decimal places, customer in Belgium, profile Peppol billing 3.0. The invoice is unpaid.

| Line | Product | Quantity | Unit price | Discount | Tax |
|---|---|---|---|---|---|
| 1 | product with internal reference `FURN_7777` | 1 | 1000.00 | 5 percent | 21 percent |
| 2 | product with internal reference `FURN_8888` | 1 | 500.00 | none | 6 percent |
| global discount line | none | 1 | 50.00 | none | 21 percent |

The global discount line has a positive amount, so it is a charge.

**Step 1: line amounts.**

| Quantity | Line 1 | Line 2 |
|---|---|---|
| `raw_gross_total_excluded_currency` | 1000.000000 | 500.000000 |
| `raw_discount_amount_currency` | 50.000000 | 0.000000 |
| `raw_total_excluded_currency` | 950.000000 | 500.000000 |
| tax amount | 950.00 × 0.21 = 199.50 | 500.00 × 0.06 = 30.00 |

**Step 2: the two document line nodes.**

| Element | Line 1 | Line 2 |
|---|---|---|
| identifier | `1` | `2` |
| invoiced quantity | `1.0` unit code `C62` | `1.0` unit code `C62` |
| allowance: indicator, reason code, multiplier, amount, base | `false`, `95`, `5.0`, `50.00`, `1000.00` | none |
| line net amount | `1000.00 - 50.00 = 950.00` | `500.00` |
| unit price | `1000.0` | `500.0` |
| classified tax category | identifier `S`, percentage `21.0`, scheme `VAT` | identifier `S`, percentage `6.0`, scheme `VAT` |

**Step 3: the document charge node.**

| Element | Value |
|---|---|
| charge indicator | `true` |
| reason code | `ADK` |
| reason | `General upsell` |
| amount | `50.00` |
| tax category | identifier `S`, percentage `21.0`, scheme `VAT` |

The tax of the charge is 50.00 × 0.21 = 10.50.

**Step 4: the tax total.** Two subtotal keys exist: standard rate at 21 percent and standard rate at 6 percent.

| Subtotal | Taxable amount | Tax amount | Percentage |
|---|---|---|---|
| standard rate 21 percent | 950.00 + 50.00 = `1000.00` | 199.50 + 10.50 = `210.00` | not written at subtotal level in this profile; written inside the tax category node as `21.0` |
| standard rate 6 percent | `500.00` | `30.00` | written inside the tax category node as `6.0` |
| total tax amount | | `240.00` | |

The taxable amount of each subtotal is recomputed by the Peppol international billing layer from the line net amounts and the document allowances and charges whose tax category matches, which is exactly what produced `1000.00` for the 21 percent group. Section 8 explains why.

**Step 5: the monetary total.**

```
line extension amount  = 950.00 + 500.00                      = 1450.00
tax exclusive amount   = 1450.00 + 50.00                      = 1500.00
tax amount             = 210.00 + 30.00                       =  240.00
tax inclusive amount   = 1500.00 + 240.00                     = 1740.00
allowance total amount = 0.00                                 → element omitted
charge total amount    = 50.00                                =   50.00
expected total         = (950.00 + 199.50) + (500.00 + 30.00) + (50.00 + 10.50) = 1740.00
payable rounding       = 1740.00 - 1740.00 = 0.00             → element omitted
prepaid amount         = 1740.00 - 1740.00                    =    0.00
payable amount         = 1740.00                              = 1740.00
```

**The complete monetary total node:**

| Element | Value |
|---|---|
| line extension amount | `1450.00` |
| tax exclusive amount | `1500.00` |
| tax inclusive amount | `1740.00` |
| allowance total amount | omitted |
| charge total amount | `50.00` |
| payable rounding amount | omitted |
| prepaid amount | `0.00` |
| payable amount | `1740.00` |

Every one of the semantic rules of the European standard holds: the line extension amount equals the sum of the line net amounts; the tax exclusive amount equals the line extension amount plus the charges minus the allowances; each taxable amount equals the sum of the line net amounts plus the document charges minus the document allowances of the same category and rate; the tax inclusive amount equals the tax exclusive amount plus the tax amount; and the payable amount equals the tax inclusive amount plus the payable rounding amount minus the prepaid amount.

---

# 8. Recomputation of the taxable amount

The Peppol international billing layer replaces the aggregated taxable amount of a tax subtotal by a value recomputed from the nodes already written.

Two tax category nodes **match** when they carry the same category identifier, the same percentage and the same currency. The recomputation collects a list of matching amounts, initially empty, and then, for each tax category node of the subtotal being written:

1. Take each document line node in turn, and each classified tax category node of its item in turn. When that node matches, add the line net amount of the document line to the list of matching amounts.
2. Take each document level allowance node, that is each node whose charge indicator carries the reproduced value `false`, in turn, and each tax category node of that allowance in turn. When that node matches, add the negated amount of the allowance to the list of matching amounts.
3. Take each document level charge node, that is each node whose charge indicator carries the reproduced value `true`, in turn, and each tax category node of that charge in turn. When that node matches, add the amount of the charge to the list of matching amounts.

When the list of matching amounts is not empty, the taxable amount of the subtotal becomes the sum of that list. When the list is empty, the aggregated taxable amount is kept unchanged.

```formula
recomputed taxable amount = sum over matching document line nodes of ( line net amount )
                            + sum over matching document level charge nodes of ( amount of the node )
                            − sum over matching document level allowance nodes of ( amount of the node )
```

The reason is arithmetical. The tax engine computes the taxable amount by summing the base amounts of the individual tax details, each of which was rounded on its own; the semantic rule of the standard requires the taxable amount to equal the sum of the **presented** line amounts. Those two values can differ by a cent when several lines share a rate. Recomputing from the presented amounts makes the file satisfy the rule.

The same layer also clears the percentage element at subtotal level, because the percentage belongs inside the tax category node in this profile.

**Worked example.** Two lines of 33.33 and 33.33 at 21 percent, and a document allowance of 6.66 at 21 percent. The aggregated base amounts might give 59.99 after per line rounding while the presented line amounts give 33.33 + 33.33 − 6.66 = 60.00. The recomputation writes 60.00, so the semantic rule holds.

---

# 9. The cross industry invoice monetary summation

The cross industry invoice family computes the seven summation elements in this order:

```formula
line total amount      = sum over tax detail groups of ( base amount of the group (document currency) )
tax basis total amount = sum over tax detail groups of ( base amount of the group (document currency) )
tax total amount       = sum over tax detail groups of ( tax amount of the group (document currency) )
rounding amount        = sum over cash rounding lines of ( total excluding tax of the line (document currency) )
grand total amount     = line total amount + tax total amount + rounding amount
total prepaid amount   = grand total amount − residual amount of the accounting document
due payable amount     = grand total amount − total prepaid amount
```

The seven elements are written in that exact order, each reading the ones before it. The tax total amount carries the currency code as an attribute. The rounding amount element is omitted from the file when its sum is zero.

The line level summation of a document line is:

```formula
line total = round( raw total excluding tax of the line (document currency) , decimal places of the document currency )
             + sum over line level charge nodes of the line of ( amount of the node )
             − sum over line level allowance nodes of the line of ( amount of the node )
```

A line level node counts as a charge when its charge indicator carries the reproduced value `true` and as an allowance when it carries the reproduced value `false`.

Note the difference with the universal business language family: the cross industry invoice line total starts from the total **after** the discount, because the discount of a line is reported inside the price node rather than as a line allowance.

**Worked example.** The same invoice as section 7.1 but exported in the cross industry invoice profile, without the global discount line (the profile moves early payment lines out but keeps global discount lines as ordinary lines).

```
line 1 total = 950.00 ; line 2 total = 500.00
tax groups: standard rate 21 percent → base 950.00, tax 199.50
            standard rate  6 percent → base 500.00, tax  30.00
line total amount      = 950.00 + 500.00 = 1450.00
tax basis total amount = 1450.00
tax total amount       = 199.50 + 30.00  =  229.50
rounding amount        = omitted
grand total amount     = 1450.00 + 229.50 = 1679.50
total prepaid amount   = 1679.50 - 1679.50 = 0.00
due payable amount     = 1679.50 - 0.00   = 1679.50
```

---

# 10. Tax category code prediction

**Inputs:** the commercial partner of the customer, the supplier, one tax (possibly empty).
**Output:** a tax category code.

The six steps are tried in order and the first one that produces a code stops the prediction.

1. When there is no tax at all on the line, the code is `E`.
2. When the tax carries a configured category code, that code is used unchanged.
3. When the customer country is Spain and the customer address carries a postal code, the first two characters of that postal code decide: `35` or `38` gives the code `L`, the general indirect tax of the Canary Islands; `51` or `52` gives the code `M`, the production, services and importation tax of Ceuta and Melilla. Any other postal code falls through to step 4.
4. When the supplier country equals the customer country, the code is `E` when the tax rate is zero, `AE` when the tax carries a negative repartition factor, and `S` in every other case.
5. When the supplier country or the customer country lies in the European economic area **and** the supplier carries a tax identification number, the code is `S` when the rate is not zero and the tax carries no negative repartition factor, `K` when both parties lie in the area, and `G` otherwise.
6. When no earlier step produced a code, the code is `S` when the rate is not zero and `E` when it is zero.

| Step | Condition | Code produced |
|---|---|---|
| 1 | no tax at all | `E` |
| 2 | the tax carries a configured category code | that code |
| 3 | customer country Spain, postal code starting `35` or `38` | `L` |
| 3 | customer country Spain, postal code starting `51` or `52` | `M` |
| 4 | supplier country equals customer country, rate zero | `E` |
| 4 | supplier country equals customer country, negative repartition factor | `AE` |
| 4 | supplier country equals customer country, any other case | `S` |
| 5 | either party in the European economic area, supplier has a tax identification number, rate not zero, no negative repartition factor | `S` |
| 5 | either party in the European economic area, supplier has a tax identification number, both parties in the area | `K` |
| 5 | either party in the European economic area, supplier has a tax identification number, any other case | `G` |
| 6 | rate not zero | `S` |
| 6 | rate zero | `E` |

Step 4 with a negative repartition factor is the self billing case: a purchase reverse charge tax has two repartition lines that cancel each other out, so from the point of view of the buyer it is an ordinary tax with a rate, while from the point of view of the seller it is a zero rate tax whose liability has been shifted. The code written is the one the seller would have used.

## 10.1 Worked example: an intra community supply

**Setup.** Supplier established in Belgium with the tax identification number `BE0477472701`. Customer established in France with the tax identification number `FR23334175221`. One line of 1000.00 with a tax at zero percent used for intra community supplies, carrying no configured category code.

| Step | Evaluation | Outcome |
|---|---|---|
| 1 | a tax exists | continue |
| 2 | no configured category code | continue |
| 3 | the customer country is France, not Spain | continue |
| 4 | Belgium is not France | continue |
| 5 | the supplier country lies in the European economic area, the customer country lies in the European economic area, and the supplier carries a tax identification number | the step applies |
| 5 | the rate is zero, so the first case of the step does not apply | continue inside the step |
| 5 | both parties lie in the area | the code is `K` |

The category code written is `K`. Section 10.2 then produces the exemption reason.

## 10.2 Exemption reason prediction

**Inputs:** the commercial partner of the customer, the supplier, one tax.
**Output:** an exemption reason code and an exemption reason text.

The three steps are tried in order and the first one that produces a result stops the prediction.

1. When a tax exists, its rate is zero and the Belgian co-contractor note applies, the code is `VATEX-EU-AE` and the reason is that note.
2. When the tax carries a configured exemption reason code, that code is used, and the reason is the sentence carried by that code in the exemption reason code list; when the code is not in that list and the predicted category requires a reason, the reason is the reproduced text "Exempt from tax"; when the code is not in that list and the predicted category does not require a reason, no reason is written.
3. Otherwise the predicted category code of section 10 decides.

| Predicted category | Exemption reason written | Exemption reason code written |
|---|---|---|
| no tax at all, or category `E` | the reproduced text "Exempt from tax" | none |
| category `G` | the reproduced text "Export outside the EU", in which the abbreviation expands to European Union | `VATEX-EU-G` |
| category `K` | the reproduced text "Intra-Community supply" | `VATEX-EU-IC` |
| any other category | none | none |

The Belgian co-contractor note applies when an accounting document is in context, the customer country is Belgium, the supplier country equals the customer country, and the fiscal position of the document is the Belgian co-contractor fiscal position of the chart of accounts. The note is the note of that fiscal position when it has one, otherwise the default sentence: `Reverse charge: In the absence of a written objection within one month of receipt of the invoice, the customer is deemed to acknowledge that they are a taxable person required to file periodic returns. If this condition is not met, the customer will be liable for the payment of the tax, interest, and penalties due in relation to this condition.`

**Continuing the worked example.** The category is `K`, no configured code exists, the Belgian note does not apply because the countries differ. The result is the code `VATEX-EU-IC` and the reason `Intra-Community supply`. The tax subtotal of the exported file therefore carries the category identifier `K`, the exemption reason code `VATEX-EU-IC`, the exemption reason `Intra-Community supply`, the percentage `0.0` and the scheme `VAT`.

Because the customer and the supplier are in different European countries, the European semantic standard layer additionally requires the delivery location and either the actual delivery date or the invoicing period, and the cross industry invoice family additionally requires both tax identification numbers. See [business-rules.md](business-rules.md) rules 099, 100 and 156.

---

# 11. Unit of measure code mapping

**Input:** the unit of measure of a line. **Output:** the unit code written as an attribute of the quantity element.

The mapping is by the stable external name of the unit. A unit that is not in the table, or that has no stable external name, maps to `C62`, the code for "one".

| Unit | Code |
|---|---|
| unit | `C62` |
| dozen | `DZN` |
| kilogram | `KGM` |
| gram | `GRM` |
| day | `DAY` |
| hour | `HUR` |
| minute | `MIN` |
| tonne | `TNE` |
| metre | `MTR` |
| kilometre | `KMT` |
| centimetre | `CMT` |
| millimetre | `MMT` |
| litre | `LTR` |
| cubic metre | `MTQ` |
| pound | `LBR` |
| ounce | `ONZ` |
| inch | `INH` |
| foot | `FOT` |
| mile | `SMI` |
| fluid ounce | `OZA` |
| quart | `QTL` |
| gallon | `GLL` |
| cubic inch | `INQ` |
| cubic foot | `FTQ` |
| square metre | `MTK` |
| square foot | `FTK` |
| yard | `YRD` |
| kilowatt hour | `KWH` |

The same table is inverted on import: a unit code that appears in it resolves to that unit; a code that does not appear leaves the unit empty.

---

# 12. Import: reconstruction of quantity, unit price and discount

This is the most intricate calculation of the domain, because the standard expresses a line with five figures that are related by one equation, and this platform stores four figures related by a different equation.

The standard states:

```
line_net_amount = (gross_unit_price - rebate) × (delivered_quantity ÷ base_quantity)
                  - allowance_charge_amount
```

where the rebate is the item price discount written inside the price node, and the allowance and charge amounts are the ones written at line level.

This platform stores a quantity, a unit price, a discount percentage and a subtotal related by:

```
subtotal = quantity × unit_price × (1 - discount ÷ 100)
```

## 12.1 The current import path

**Inputs, all read from the line and multiplied by the document sign where the standard allows a negative document:**

| Name | Source |
|---|---|
| `line_net_amount` | the line net amount element |
| `price_amount` | the price amount element |
| `invoiced_quantity` | the invoiced quantity, or the credited quantity |
| `base_quantity` | the base quantity of the price node |
| `total_allowances` | the sum of the line level allowance amounts |
| `total_charges` | the sum of the line level charge amounts |
| `price_allowance_base_amount`, `price_allowance_amount` | the base amount and the amount of the allowance inside the price node, the amount signed by its own charge indicator |

**Step 1: the price level.** This describes what one purchase of `base_quantity` units costs and how much is discounted on it.

```
if price_amount is present:
    price_quantity = base_quantity or 1
    if price_allowance_base_amount is present:
        price_discount = price_allowance_base_amount - price_amount
        price_subtotal = price_allowance_base_amount
    else if price_allowance_amount is present:
        price_discount = - price_allowance_amount
        price_subtotal = price_amount - price_allowance_amount
    else:
        price_discount = 0 ; price_subtotal = price_amount
else if price_allowance_base_amount is present:
    price_subtotal = price_allowance_base_amount
    price_quantity = base_quantity or 1
    price_discount = - (price_allowance_amount or 0)
else:
    price_subtotal = 0 ; price_quantity = 0 ; price_discount = 0
```

**Step 2: the line level.** `subtotal = line_net_amount + total_allowances - total_charges`.

```
case A: a line net amount is present and the quantity is absent or zero
    quantity        = 1
    unit_price      = subtotal
    discount_amount = total_allowances
    if price_subtotal is not zero:
        quantity        = subtotal × price_quantity ÷ (price_subtotal - price_discount)
        unit_price      = subtotal ÷ quantity + price_discount ÷ price_quantity
                          when the quantity is not zero, otherwise price_amount
        discount_amount = discount_amount + price_discount × quantity ÷ price_quantity

case B: a line net amount is present and the quantity is non zero
    quantity        = invoiced_quantity
    unit_price      = subtotal ÷ quantity
    discount_amount = total_allowances
    if price_subtotal is not zero:
        unit_price      = price_subtotal ÷ price_quantity
        discount_amount = discount_amount + price_discount × quantity ÷ price_quantity

case C: no line net amount
    quantity = 0 ; unit_price = 0 ; discount_amount = total_allowances
    if price_subtotal is not zero:
        unit_price      = price_subtotal ÷ price_quantity
        quantity        = price_quantity
        discount_amount = discount_amount + price_discount
```

**Step 3: the charges.** `unit_price = unit_price + total_charges ÷ (quantity or 1)`.

**Step 4: the discount percentage.**

```
gross_subtotal = unit_price × quantity
discount = discount_amount × 100 ÷ gross_subtotal   when gross_subtotal is not zero
           0                                        otherwise
```

**Worked example A.** A line with a line net amount of 1000.00, a price amount of 250.00, a base quantity of 5, a price allowance base amount of 1250.00, and no line level allowance or charge.

```
price_quantity = 5
price_discount = 1250.00 - 250.00 = 1000.00     ← the price node says 1250 becomes 250 for 5 units
price_subtotal = 1250.00
subtotal       = 1000.00
case A (no invoiced quantity):
    quantity        = 1000.00 × 5 ÷ (1250.00 - 1000.00) = 5000 ÷ 250 = 20
    unit_price      = 1000.00 ÷ 20 + 1000.00 ÷ 5 = 50.00 + 200.00 = 250.00
    discount_amount = 0 + 1000.00 × 20 ÷ 5 = 4000.00
gross_subtotal = 250.00 × 20 = 5000.00
discount       = 4000.00 × 100 ÷ 5000.00 = 80 percent
check: 20 × 250.00 × (1 - 0.80) = 1000.00 ✔
```

**Worked example B.** A line with an invoiced quantity of 6, a line net amount of 1200.00, a price amount of 250.00, a base quantity of 5, a price allowance base amount of 1250.00 and a price allowance amount of 50.00.

```
price_quantity = 5
price_discount = 1250.00 - 250.00 = 1000.00
price_subtotal = 1250.00
subtotal       = 1200.00
case B:
    quantity        = 6
    unit_price      = 1250.00 ÷ 5 = 250.00
    discount_amount = 0 + 1000.00 × 6 ÷ 5 = 1200.00
gross_subtotal = 250.00 × 6 = 1500.00
discount       = 1200.00 × 100 ÷ 1500.00 = 80 percent
check: 6 × 250.00 × (1 - 0.80) = 300.00
```

The check does not reproduce the stated line net amount, because the price node and the line net amount of this file contradict each other. The platform believes the price node. That is a deliberate choice: the price node is the semantic description of the item, and the tax correction of section 14 repairs the totals afterwards.

**Worked example C.** A line with an invoiced quantity of zero, a line net amount of 0.00 and a price amount of 100.00.

```
price_quantity = 1 ; price_discount = 0 ; price_subtotal = 100.00
subtotal = 0.00
case A (the quantity is zero):
    quantity        = 0.00 × 1 ÷ (100.00 - 0) = 0
    the quantity is zero, so unit_price = price_amount = 100.00
    discount_amount = 0
gross_subtotal = 0 ; discount = 0
```

This branch exists because a line with a zero quantity, a zero net amount and a non zero price used to produce a division by zero.

## 12.2 The plain import path

Used by the order import and by the import of the plain 2.0 and 2.1 syntaxes.

```
base_quantity      = the base quantity element, or 1, and 1 again when it reads as zero
gross_unit_price   = the gross price element when present
net_unit_price     = the net price element when present
delivered_quantity = the delivered quantity element, or 1
line_net_amount    = the line total element when present
quantity           = delivered_quantity × document_sign

rebate = the rebate element when present,
         else gross_unit_price - net_unit_price when both are present,
         else 0

discount_amount, charges = the line level allowances summed, and the line level charges collected
charge_amount            = the sum of the collected charge amounts
allowance_charge_amount  = discount_amount - charge_amount

unit_price = gross_unit_price ÷ base_quantity                            when a gross price exists
           = (net_unit_price + rebate) ÷ base_quantity                   else when a net price exists
           = (line_net_amount + allowance_charge_amount) ÷ (delivered_quantity or 1)
                                                                         else when a net amount exists
           = 0                                                           otherwise

discount = 0
if delivered_quantity × unit_price is not zero in the company currency and a net amount exists:
    inferred = 100 × (1 - (line_net_amount - charge_amount)
                          ÷ round(delivered_quantity × unit_price, company currency))
    discount = inferred unless it is zero in the company currency

# repair of contradictory files
if a net price and a net amount both exist and
   line_net_amount ≠ net_unit_price × (delivered_quantity ÷ base_quantity) - allowance_charge_amount:
       net price zero and delivered quantity zero: quantity = 1 ; unit_price = line_net_amount
       net price zero:                             unit_price = line_net_amount ÷ delivered_quantity
       delivered quantity zero:                    quantity   = line_net_amount ÷ unit_price
```

A line whose reconstructed subtotal is missing is dropped entirely.

**Worked example.** A line with a gross price of 30.00, a base quantity of 3, a delivered quantity of 3, a rebate of 2.00 and a line net amount of 28.00.

```
unit_price = 30.00 ÷ 3 = 10.00
quantity   = 3
inferred   = 100 × (1 - 28.00 ÷ round(3 × 10.00)) = 100 × (1 - 28 ÷ 30) = 6.666666...
discount   = 6.666666... percent
check: 3 × 10.00 × (1 - 0.0666666) = 28.00 ✔
```

---

# 13. Import: folding a charge into the line

A line level charge that could not be matched to a fixed tax must not create a second line, because the standard already counted it inside the line net amount. It is folded into the unit price and the discount so that the line still totals the same amount.

```
subtotal_before = unit_price × quantity × (1 - discount ÷ 100)
subtotal_after  = subtotal_before - charge_amount
unit_price      = unit_price - charge_amount ÷ quantity
gross_after     = unit_price × quantity
discount        = (1 - subtotal_after ÷ gross_after) × 100
```

**Worked example.** A line with a unit price of 19.00, a quantity of 10 and a discount of 10 percent carries a charge of 25.00 that was matched to a fixed tax of 50.00 over the whole line, that is 5.00 per unit.

```
subtotal_before = 19.00 × 10 × 0.90        = 171.00
subtotal_after  = 171.00 - 50.00           = 121.00
unit_price      = 19.00 - 50.00 ÷ 10       =  14.00
gross_after     = 14.00 × 10               = 140.00
discount        = (1 - 121.00 ÷ 140.00) × 100 = 13.5714286 percent
check: 14.00 × 10 × (1 - 0.135714286) = 121.00, and 121.00 + 50.00 = 171.00 ✔
```

The fixed tax then adds its 50.00 back, so the line total in the platform equals the line total in the file.

In the plain path the same folding is expressed with the opposite sign, because there the charge has not yet been removed from the unit price:

```
subtotal_before = unit_price × line_quantity × (1 - discount ÷ 100)
subtotal_after  = subtotal_before + charge_amount
unit_price      = unit_price + charge_amount ÷ line_quantity
discount        = (1 - subtotal_after ÷ (unit_price × line_quantity)) × 100
```

---

# 14. Import: correction of the tax amounts

**Inputs:** the tax totals stated in the file, grouped by the pair of category code and percentage; the taxes that were matched for each of those groups; the accounting document after its lines have been written.
**Output:** adjusted tax lines.

```
tolerance     = 0.03 in the document currency
stated_total  = the sum of the tax amounts of every stated group
complete      = every stated group resolved every one of its related taxes

if not complete or |document tax total - stated_total| - tolerance > 0:
    stop, correct nothing

build the map from each matched tax to the set of taxes of its group
build the map from each set of taxes to the tax amount stated for that group

read the rounded base and tax lines of the document
aggregate the tax details under the grouping function "the set of taxes of this tax"
for each set of taxes and its aggregated values:
    target  = the stated tax amount of that set
    factors = one entry per tax detail of that set, weighted by its raw tax amount
    distribute target over those factors, smoothing the rounding difference
    write the distributed amount back into each tax detail
recompute the accounting data of the base lines and rebuild the tax lines
update the amount and the balance of every tax line that changed
```

The distribution is the smooth distribution helper of the taxes domain: it gives each factor its proportional share rounded to the currency, then spreads the remaining cents one by one over the factors with the largest remainders, so that the sum of the parts equals the target exactly.

**Worked example.** A file states one tax group, standard rate at 21 percent, with a tax amount of 210.01. The platform computed 210.00 from two lines whose raw tax amounts are 199.50 and 10.50.

```
complete  = true
|210.00 - 210.01| - 0.03 = -0.02, which is not greater than zero → correct
target    = 210.01
factors   = [199.50, 10.50] ; total 210.00
shares    = [210.01 × 199.50 ÷ 210.00, 210.01 × 10.50 ÷ 210.00] = [199.5095, 10.5005]
rounded   = [199.51, 10.50] ; sum 210.01 ✔
```

The two tax details receive 199.51 and 10.50, and the tax line of the document is written with 210.01.

**Counter example.** The same file states 213.00 instead. Then `|210.00 - 213.00| - 0.03 = 2.97 > 0`, and nothing is corrected: the discrepancy is too large to be a rounding artefact, so the platform keeps its own computation and leaves the difference visible.

---

# 15. Import: correction of the untaxed amount

**Runs only when every stated tax group resolved every one of its taxes.**

```
stated_exclusive = the tax exclusive amount of the file × document sign
stated_rounding  = the payable rounding amount of the file × document sign
expected         = stated_exclusive + stated_rounding
difference       = round(expected - the untaxed total of the document, currency)
for every line charge that was matched to a fixed tax:
    difference = difference - the charge amount
if difference is zero in the currency: stop
add one line: label "Rounding", quantity 1, unit price = difference, no tax
```

The fixed tax charges are subtracted because their amounts are already carried by the taxes, not by the untaxed total.

**Worked example.** A file states a tax exclusive amount of 1500.00 and a payable rounding amount of 0.03. The lines that were written total 1500.00 excluding tax, and no line charge matched a fixed tax.

```
expected   = 1500.00 + 0.03 = 1500.03
difference = 1500.03 - 1500.00 = 0.03
a line "Rounding" with quantity 1, unit price 0.03 and no tax is added
```

---

# 16. Import: matching a product

**Inputs:** the item identifiers and the label of the line, the partner of the document.
**Output:** the product of the line, or nothing.

The values collected per line are:

| Name | Source element |
|---|---|
| `barcode` | the standard item identifier whose scheme is `0160` |
| `default_code` | the seller item identifier, falling back to the buyer item identifier |
| `name` | the item name |
| `sellers_item_id` | the seller item identifier |
| `buyers_item_id` | the buyer item identifier |
| `standard_item_id` | the standard item identifier whose scheme is `0160` |
| `vendor_partner` | the commercial partner of the matched partner |
| `variant_default_code` | the extended seller item identifier, when the sales package is installed |
| `variant_barcode` | the extended standard item identifier, when the sales package is installed |

The strategies are tried in the order of their rank. The shipped ranks come from the product matching plan of the general ledger domain; this domain inserts the variant strategies at ranks 12 and 14 and appends the prediction strategy last. A strategy that finds exactly one product wins; a strategy that finds several or none moves on.

**Worked example: matching by internal reference.** The file line carries:

```
item name                 : "Office Chair"
seller item identifier    : "FURN_7777"
standard item identifier  : absent
```

The collected values are `default_code = "FURN_7777"`, `sellers_item_id = "FURN_7777"`, `name = "Office Chair"`, `barcode` empty.

1. The barcode strategy finds nothing, because no barcode was collected.
2. The internal reference strategy searches for a product whose internal reference equals `FURN_7777` within the companies of the document. Exactly one product matches, the office chair. It wins.
3. The remaining strategies, including the vendor catalogue strategy, the variant strategies and the prediction strategy, are not run.

The line is written with that product. Its unit of measure is then resolved from the unit code of the quantity element: when the code is `C62` the unit is "unit"; if that unit had no common reference with the unit of the matched product, the unit would be left empty and the message of [business-rules.md](business-rules.md) rule 192 would be added.

**Variant example.** The same file additionally carries the extended seller item identifier `FURN_7777_BLACK`. Ranks 12 and 14 are tried before the vendor catalogue and the prediction strategies, so a product variant whose internal reference is `FURN_7777_BLACK` wins over the template level match. This is how a colour variant is resolved.

---

# 17. Import: matching a tax

**Inputs:** the classified tax category of a line, the tax totals of the document, the account predicted for the line, the fiscal position derived from the partner.

The tax values of a line are built from the classified tax category nodes:

```
percentage    = the percentage element of the category
category_code = the identifier element of the category
if either is missing: the category produces no tax value
key = (category_code, percentage)
if the document has no stated tax total for that key: the category produces no tax value
tax value = { amount type: percentage,
              usage:       "sale" for a sale journal, "purchase" for a purchase journal,
              rate:        the stated rate of that group,
              category code: the stated category code of that group,
              key:         key,
              prediction:  the document, the line label and the partner, when both are known }
```

The strategies are tried in this order and the first that resolves a tax wins:

1. the default tax of the predicted account of the line;
2. the prediction from previous documents of the same partner with a similar line label;
3. the matching by rate, preferring a tax whose price is excluded over one whose price is included.

A line level charge whose reason code is `AEO` additionally produces a fixed tax value whose rate is the charge amount divided by the quantity of the line and whose name is the reason of the charge.

Document level allowances and charges produce their own tax value from the tax category written inside them, following the same rules.

---

# 18. Plain import: matching a tax by rate

Used by the order import and the plain 2.0 and 2.1 syntaxes.

```
for every tax node collected on the line:
    rate   = the value of the node
    domain = same company family, percentage amount type, the usage of the journal,
             the rate, and the tax country of the document
    tax    = the specific prediction for the line label, filtered by that domain, when available
    when a tax exigibility is requested, four searches are tried in order, each adding the
    fiscal position filter first and then dropping it:
        price excluded and the requested exigibility
        price included and the requested exigibility
        price excluded and the requested exigibility, without the fiscal position filter
        price included and the requested exigibility, without the fiscal position filter
    and, when none matched, the message of rule 193 is added for the exigibility
    four more searches are then tried in the same shape without the exigibility filter
    when a tax is found and its price is included:
        unit_price = unit_price × (1 + rate ÷ 100)
```

The fiscal position filter keeps only taxes attached to the fiscal position of the document, plus, for a domestic fiscal position, the taxes attached to no fiscal position.

---

# 19. Plain import: document level allowances and charges

```
for every document level allowance or charge node:
    name      = its reason, or an empty text
    indicator = -1 when its charge indicator reads "false", otherwise +1
    amount    = its amount, or 0
    base      = its base amount, or 0
    if base is not zero:
        unit_price = base × indicator × document_sign
        percentage = its multiplier factor, or 100
        quantity   = percentage ÷ 100
    else:
        unit_price = amount × indicator × document_sign
        quantity   = 1
    taxes = every tax of the company whose rate equals the percentage of the tax category of
            the node, whose amount type is a percentage and whose usage matches the journal
    produce one extra line with that name, quantity, unit price and taxes, at sequence 0
```

**Worked example.** A document level allowance with a base amount of 1000.00, a multiplier factor of 10 and a tax category at 21 percent, on a purchase document with a document sign of one.

```
indicator  = -1
unit_price = 1000.00 × -1 × 1 = -1000.00
quantity   = 10 ÷ 100 = 0.1
subtotal   = -100.00
```

The resulting line is a discount of one hundred, taxed at 21 percent.

---

# 20. Import of a document level allowance or charge in the current path

```
charge_indicator_sign = +1 when the charge indicator reads "true", otherwise -1
if a base amount is present:
    unit_price = base_amount × charge_indicator_sign × document_sign
    quantity   = multiplier_factor ÷ 100
else:
    unit_price = amount × charge_indicator_sign × document_sign
    quantity   = 1
the line label is the reason of the node
the taxes are the ones resolved from the tax category of the node
```

An allowance or charge node that carries no tax category percentage is skipped entirely, because it cannot be attached to a tax and would unbalance the document.

---

# 21. Point of sale receipt totals

```
document type      = an invoice when the receipt total is not negative, a credit note otherwise
the line, allowance, charge and tax totals follow the plain 2.0 rules of sections 4 and 6.3
prepaid amount = tax exclusive amount - the amount paid on the receipt
payable amount = the amount paid on the receipt
```

**Worked example.** A receipt of 121.00 including a tax of 21.00, fully paid in cash.

```
tax exclusive amount = 100.00
prepaid amount       = 100.00 - 121.00 = -21.00
payable amount       = 121.00
```

The prepaid amount is negative because the profile subtracts the paid amount from the amount excluding tax rather than from the amount including tax. A replacement must reproduce this arithmetic to stay behaviourally equivalent.

---

# 22. The sending batch size and the retrigger

```
job_count  = the parameter of the sending scheduled action, 20 by default
all_jobs   = the jobs prepared from the pending documents
taken      = the first job_count jobs
remaining  = count(all_jobs) - count(taken)
if remaining > 0: schedule the same scheduled action to run again as soon as possible
```

The network polling scheduled actions use a different rule: they collect the batch size plus one record, process the first batch size, and, when the collection exceeded the batch size, schedule themselves again in five minutes. The default batch size is fifty.

**Worked example.** Ninety pending documents for a format with no batching key produce ninety jobs. The first run takes twenty, leaves seventy, and triggers itself. Five runs in total drain the queue: twenty, twenty, twenty, twenty, ten.
