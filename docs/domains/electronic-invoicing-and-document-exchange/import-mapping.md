# Import mapping

How an inbound structured file becomes a draft accounting document: how the file is recognised and routed, the exact order of the import steps, the element by element extraction of every document level and line level value, every matching strategy with its fallbacks, the correction passes that align the produced document with the totals stated in the file, the complete catalogue of the messages logged in the discussion thread of the document, and the differences of the cross industry invoice family and of the order profiles. The arithmetic of the reconstruction of quantity, unit price and discount and of the matching strategies is in sections 12 to 20 of [calculations.md](calculations.md); the recognition table is in section 12.10 of [entities.md](entities.md); the export direction is in [universal-business-language-mapping.md](universal-business-language-mapping.md) and [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md).

## Notation

The conventions of [universal-business-language-mapping.md](universal-business-language-mapping.md) apply: element names are quoted verbatim as data values and their meaning is given in full words, and the identifier element, which the standard spells with the two letters that abbreviate that word, is written `Identifier` in every path of this file. A path written with a leading `./` is relative to the root element of the file; a path written with a leading `.//` matches at any depth.

---

# 1. Entry points

A structured file reaches the import in one of four ways.

| Entry point | Origin | Target document |
|---|---|---|
| Inbox polling | a message pulled from the exchange network proxy | a new document created in the reception journal, or in a self billing sale journal for a self billed document; see section 13 of [workflows.md](workflows.md) |
| Attachment upload | a user drops a file on a draft document or on the document list | the document the user was working on, or a new one |
| Electronic mail alias | a message sent to the alias of a journal | a new document created by the alias |
| Manual decoding | a user asks for a file already attached to a document to be decoded | that document |

In every case the same three stage procedure runs: recognise the file type, select the decoder, run the decoder.

## 1.1 Guard before decoding

A decoder refuses to run on an accounting document that already has lines, and returns the standard reason of the accounts receivable domain saying that the document cannot be decoded because it already has lines. This prevents a second file from overwriting work a user has already done.

## 1.2 Deciding the document direction

The direction is read from the file, not from the journal, and then combined with the journal.

**Universal business language family.**

| Condition | Result |
|---|---|
| the root element is `Invoice` and the tax inclusive amount of the monetary total is negative | a credit note, and every quantity is negated |
| the root element is `Invoice` otherwise | an invoice, quantities kept as they are |
| the root element is `CreditNote` | a credit note, quantities kept as they are |
| any other root element | the file is not decoded |

**Cross industry invoice family.**

| Condition | Result |
|---|---|
| the document type code is `381` or `261` | a credit note, quantities kept as they are |
| the document type code is `380`, `389` or `527` and the grand total amount is negative | a credit note, and every quantity is negated |
| the document type code is `380`, `389` or `527` otherwise | an invoice, quantities kept as they are |
| the document type code is missing or is anything else | the file is not decoded |

The factor applied to the quantities is called the file document sign below; it is `1` or `-1`, and every amount read from the file is multiplied by it.

The direction is then combined with the journal: a sale journal produces a customer invoice or a customer credit note, a purchase journal produces a vendor bill or a vendor credit note, and a journal of any other type stops the import. When the document already existed with the other direction of the same pair, its direction is changed and the message `The invoice has been converted into a credit note and the quantities have been reverted.` is logged. When the existing direction belongs to the other pair entirely, the import stops without changing anything.

---

# 2. Order of the import steps

The steps run in this exact order. Later steps read values collected by earlier steps, so the order is part of the specification.

1. Initialise the working set: the target document, its company, whether the journal is a sale or a purchase journal, the parsed file, an empty message list and an empty set of values to write.
2. Read the document direction and the file document sign (section 1.2).
3. Update the direction of the target document (section 1.2).
4. Collect the payee account numbers (section 3.6), because the partner matching may use them.
5. Collect the partner values (section 3.1) and match the partner (section 4.1).
6. Create the partner when it could not be matched (section 4.2).
7. Read the issue date and the due date (section 3.2).
8. Read the currency code and resolve the currency and its rate (section 4.3).
9. Match or create the bank account (section 4.4).
10. Read the reference, the origin, the note, the payment reference and the delivery date (section 3.2).
11. Read the user defined extension fields at document level (section 3.7).
12. Read the delivery terms code and resolve the delivery terms (section 4.5).
13. Read the prepaid amount (section 3.3).
14. Read the tax totals of the document (section 3.4).
15. Read the document level allowances and charges (section 3.5).
16. Read every document line (section 5).
17. Match the products of every line (section 4.6).
18. Match the units of measure of every line (section 4.7).
19. Predict the accounts of every line (section 4.8).
20. Match the taxes of every line and of every document level allowance and charge (section 4.9).
21. Build the base lines (section 6).
22. Write the collected values onto the document (section 7).
23. Correct the tax amounts against the tax totals stated in the file (section 8).
24. Correct the untaxed amount against the tax exclusive amount stated in the file (section 9).
25. Post process: store the file, extract the embedded documents, and log the result (section 10).

---

# 3. Document level extraction

## 3.1 Partner values

The party group read is the customer party when the journal is a sale journal, and the supplier party when it is a purchase journal. When the party group is absent, no partner value is collected at all.

| Collected value | Path, relative to the party group; the first path that yields a value wins |
|---|---|
| tax identification number | `.//CompanyID` |
| telephone number | `.//Telephone` |
| name | `.//RegistrationName`, then `.//Name` |
| electronic mail address | `.//ElectronicMail` |
| country code | `.//Country//IdentificationCode` |
| street | `.//StreetName` |
| second street line | `.//AdditionalStreetName` |
| city | `.//CityName` |
| postal code | `.//PostalZone` |
| participant endpoint | the text of `.//EndpointID`, trimmed |
| electronic address scheme | the attribute `schemeID` of `.//EndpointID` |

**Fallback for the tax identification number.** When no tax identification number was found and a country code was, the electronic address scheme catalogue of that country is consulted; for each scheme of that country whose meaning is the tax identification number, the party identification whose scheme attribute equals that scheme code is read, and the first one found becomes the tax identification number.

**Country resolution.** The two letter country code is resolved to a country record. The code `GB` is resolved through the alternative key `UK`, because the country record of the United Kingdom is keyed that way.

## 3.2 Dates, references and notes

| Field written on the document | Path | Rule |
|---|---|---|
| invoice date | `./IssueDate` | parsed as a date |
| due date | `./DueDate`, falling back to `./PaymentDueDate` | parsed as a date |
| document number | `./Identifier` | only when the journal is a sale journal and the document is in the quick creation mode; the value becomes the name of the document |
| customer reference | `./Identifier` | in every other case |
| origin | `./OrderReference/Identifier` | see the fallback below |
| terms and conditions | every `./Note` followed by every `./PaymentTerms/Note` | each non empty text is escaped and wrapped in its own paragraph, and the paragraphs are concatenated |
| payment reference | every `./PaymentMeans/PaymentID` | the distinct non empty texts joined by commas, preserving their order |
| delivery date | `.//Delivery/ActualDeliveryDate` | parsed as a date |

**Origin fallback.** When the order reference identifier is empty and the purchasing capability package is installed, the sequence that numbers purchase orders for this company is read, its prefix, its suffix and its number padding are turned into a search pattern, and every item description of the file is searched for text matching that pattern. The matches, joined by spaces, become the origin. This recovers the purchase order number from a vendor who writes it inside the line descriptions instead of in the order reference.

## 3.3 Prepaid amount

The prepaid amount is read from `./LegalMonetaryTotal/PrepaidAmount` and multiplied by the file document sign. When it is zero in the document currency nothing happens. Otherwise it is remembered and the message `A payment of <amount> was detected.` is logged, with the amount formatted in the document currency.

## 3.4 Tax totals of the document

One entry per tax subtotal of `./TaxTotal/TaxSubtotal`, keyed by the pair of the tax category code and the percentage.

| Read from | Into |
|---|---|
| `.//TaxCategory/Identifier` | the tax category code of the key; a subtotal without one is skipped |
| `.//TaxCategory/Percent`, falling back to `.//Percent` | the percentage of the key; a subtotal without one is skipped |
| `.//TaxAmount` | the stated tax amount, multiplied by the file document sign and accumulated into the entry; a subtotal without one is skipped |

A subtotal whose tax amount carries a currency attribute that is not the document currency is skipped entirely, so that a multi currency file contributes only its document currency figures.

Each entry also remembers the intended usage of the tax, which is sale for a sale journal and purchase for a purchase journal, that the tax is a percentage tax, and the list of line level tax values that will be linked to it.

## 3.5 Document level allowances and charges

One entry per `./AllowanceCharge`.

| Read from | Into |
|---|---|
| `./ChargeIndicator` | whether the entry is a charge, when the text is `true` in any letter case, or an allowance otherwise |
| `./Amount` | the amount, zero when the element is absent |
| `./BaseAmount` | the base amount, empty when absent |
| `./AllowanceChargeReason` | the reason text, which becomes the label of the produced line |
| `./AllowanceChargeReasonCode` | the reason code |
| `./MultiplierFactorNumeric` | the multiplier factor |
| `./TaxCategory/Percent` | the percentage; an allowance or charge without a percentage is skipped entirely |
| `./TaxCategory/Identifier` | the tax category code |

Each entry also produces a tax value, built from the tax category group exactly as a line tax value is built (section 5.4), and that tax value is linked to the document tax total entry with the same key. A second, independent tax value is recorded for the attempt to resolve a percentage tax from the category code and the percentage.

## 3.6 Payee account numbers

Every `./PaymentMeans/PayeeFinancialAccount/Identifier` is collected into a set of account numbers.

## 3.7 User defined extension fields

When the target document is a purchase document, the extension table that corresponds to its direction is consulted; for every declared field the declared path is evaluated against the file, and the first non empty value found is written onto the document, provided the field exists and its data type is one of the types the declaration allows. The same is done at line level, in section 5.5.

---

# 4. Matching strategies

Every strategy returns either exactly one record, which wins, or nothing, in which case the next strategy is tried. A strategy that finds several candidates counts as finding nothing.

## 4.1 Partner

The strategies are tried in this order:

1. by tax identification number;
2. by the pair of electronic address scheme and participant endpoint;
3. by payee account number, using the account numbers collected in section 3.6;
4. by electronic mail address;
5. by telephone number;
6. by name.

Strategies 1, 2, 4, 5 and 6 belong to the accounts payable domain; strategy 3 is inserted by this domain, which is why the payee account numbers are collected before the partner is matched.

## 4.2 Creating a missing partner

A partner is created only when the file carries both a name and a tax identification number.

| Situation | Outcome |
|---|---|
| a partner was matched and it already carries the same tax identification number, compared after normalisation | nothing happens |
| a partner was matched and it carries no tax identification number | the number from the file is written onto it, after validation; a number that fails validation is discarded and the field is left empty |
| a partner was matched and it carries a different tax identification number | a new partner is created and the message `Could not retrieve a partner corresponding to '<name>' with the same tax identification number. A new partner was created.` is logged, with the acronym expanded to "tax identification number" |
| no partner was matched | a new partner is created and the message `Could not retrieve a partner corresponding to '<name>'. A new partner was created.` is logged |

**Normalised comparison of tax identification numbers.** Both numbers are compared with every space removed and in upper case; the number from the file additionally has every full stop removed. When the resolved country is Switzerland, both numbers additionally have every full stop and every hyphen removed and a trailing tax register suffix removed, so that the three national spellings of the same number compare equal.

**Values written on a created partner:** it is marked as a company; the telephone number, the name, the electronic mail address, the street, the second street line, the postal code and the city are copied when present; the country is the resolved country; the tax identification number is written after validation, and is left empty when validation fails. When the file carried both an electronic address scheme and a participant endpoint, those two are copied as well.

## 4.3 Currency

The three letter code is read from `.//DocumentCurrencyCode`. When it is absent the company currency is used. When it is present the currency with that code is searched, including archived currencies.

| Situation | Outcome |
|---|---|
| a currency is found and it is active | it is used |
| a currency is found but it is archived | it is used, and the message `The currency '<code>' is not active.` is logged |
| no currency is found | the company currency is used, and the message `Could not retrieve currency: <code>. Did you enable the multicurrency option and activate the currency?` is logged |

The conversion rate from the company currency to the document currency is then read for the invoice date of the document, or for today when no invoice date was found.

## 4.4 Bank account

The account numbers collected in section 3.6 are attached to the party that owns them: the partner of the document for a customer credit note and for a vendor bill, the company itself for a customer invoice and for a vendor credit note. For any other direction nothing happens.

Each account number is either found on that party or created on it. A failure logs `The bank account couldn't be fetched: <the failure message>`. The first resulting account becomes the recipient bank account of the document.

## 4.5 Delivery terms

Every `./TransportExecutionTerms/DeliveryTerms/Identifier` is collected into a set of codes. When the set holds exactly one code, the delivery terms record with that code is searched and, when found, written onto the document. When the set holds nothing or more than one code, no delivery terms are written.

## 4.6 Product

The collected values per line and the strategy order are in section 16 of [calculations.md](calculations.md). The strategies are ranked, and this domain contributes two of them: matching a variant by its extended seller item identifier at rank 12 and matching a variant by its extended standard item identifier at rank 14, both only when the sales capability package is installed. The prediction strategy is appended last.

When no product is found the line is written with no product; this is not an error and produces no message on an accounting document. On an order it does produce a message, see section 12.

## 4.7 Unit of measure

The unit code is read from the attribute `unitCode` of the quantity element, which is `.//InvoicedQuantity` or, when that is absent, `.//CreditedQuantity`. The code is looked up in the unit code table of section 11 of [calculations.md](calculations.md), read backwards.

| Situation | Outcome |
|---|---|
| there is no unit code, or the code is not in the table, or the table entry names a unit that does not exist | the unit of the line is left empty |
| a unit is found and the line has no product | the unit is written on the line |
| a unit is found, the line has a product, and the unit shares a reference unit with the unit of that product | the unit is written on the line |
| a unit is found, the line has a product, and the unit does not share a reference unit with the unit of that product | the unit is left empty, the line is marked so that the empty unit survives the recomputation, and the message `The Unit of Measure '<unit>' (from unit code '<code>') was ignored on the line for product '<product>' because it is not compatible with the product's Unit of Measure '<product unit>'. The unit of measure was left empty.` is logged |

## 4.8 Account

The account is predicted, and only when the advanced accounting capability package is installed. The prediction takes the document, the label of the line and the partner, and returns the account most often used on similar lines. Identical triples are predicted once and reused, so that a document with many lines of the same label runs one prediction.

## 4.9 Tax

The strategy order and the construction of the tax values are in section 17 of [calculations.md](calculations.md). Three further rules apply here.

1. When an account was predicted for the line, that account is attached to every tax value of the line, so that the first strategy can use the default tax of the account.
2. When a partner was matched, a draft document is evaluated with that partner to derive the fiscal position, and that fiscal position is attached to every tax value of the document, of every line and of every allowance and charge.
3. Every tax value that was resolved is written into the tax list of its line. Every tax value that was not resolved produces a message: `Could not retrieve the tax: <rate> % for line '<the label>'.` when the tax value carries a name, and `Could not retrieve the tax: <rate> for the document level allowance/charge.` when it does not.

---

# 5. Line level extraction

One line is produced per `./InvoiceLine`, then per `./CreditNoteLine`, then per `./DebitNoteLine`. The steps run in this order for each line: allowances and charges, label, quantity and unit price and discount, product values, unit of measure values, account values, tax values.

## 5.1 Line allowances and charges

Two separate collections are made.

**Line level.** One entry per `./AllowanceCharge` of the line, holding whether it is a charge, read from `.//ChargeIndicator`; the amount, read from `.//Amount`, an entry without an amount being skipped; the base amount, read from `.//BaseAmount`; the reason, read from `.//AllowanceChargeReason`; and the reason code, read from `.//AllowanceChargeReasonCode`.

**Price level.** At most one entry, read from `./Price/AllowanceCharge`, holding a sign that is `+1` when its charge indicator says `true` and `-1` otherwise, defaulting to an allowance when the indicator is absent; the amount; the base amount; the reason; and the reason code.

## 5.2 Label

1. Read the item reference from the path `./Item/SellersItemIdentification/Identifier` and the name from the path `./Item/Name`.
2. When both an item reference and a name were read, and the name does not already contain the item reference enclosed in square brackets, prefix the name with the item reference enclosed in square brackets followed by one space.
3. Read the description as the texts of every `./Item/Description` element, each followed by a line break, with surrounding whitespace trimmed from the result.
4. Assemble the label with the first row of the table below whose condition holds.

| Order | Condition | Label |
|---|---|---|
| 1 | a name and a description were read | the name, a line break, then the description |
| 2 | only a name was read | the name |
| 3 | only a description was read | the description |
| 4 | neither was read | empty |

The bracketed reference is prefixed so that the label of the imported line has the same shape as a label produced from a product, which makes the later prediction strategies match.

## 5.3 Quantity, unit price and discount

The elements read are `.//LineExtensionAmount`, `.//Price/PriceAmount`, `.//InvoicedQuantity` or `.//CreditedQuantity`, and `./Price/BaseQuantity`. The line extension amount and the quantity are multiplied by the file document sign. The complete reconstruction, with all of its branches and a worked example per branch, is in section 12 of [calculations.md](calculations.md).

## 5.4 Product, unit of measure, account and tax values

**Product values** collected per line:

| Collected value | Path |
|---|---|
| barcode | `.//Item/StandardItemIdentification/Identifier` whose scheme attribute is `0160` |
| internal reference | `.//Item/SellersItemIdentification/Identifier`, falling back to `.//Item/BuyersItemIdentification/Identifier` |
| name | `.//Item/Name` |
| seller item identifier | `.//Item/SellersItemIdentification/Identifier` |
| buyer item identifier | `.//Item/BuyersItemIdentification/Identifier` |
| standard item identifier | `.//Item/StandardItemIdentification/Identifier` whose scheme attribute is `0160` |
| vendor partner | the commercial partner of the matched partner |
| prediction input | the document, the label of the line and the matched partner |
| variant internal reference | `./Item/SellersItemIdentification/ExtendedID`, when the sales capability package is installed |
| variant barcode | `./Item/StandardItemIdentification/ExtendedID`, when the sales capability package is installed |

**Commodity classification.** Every `./Item/CommodityClassification/ItemClassificationCode` that has both a text and a list attribute contributes a further collected value, according to its list attribute: `HS` gives the intra-community trade statistics code, `TST` gives the standard products and services classification code, `STI` gives the common procurement vocabulary code, and `CG` gives the national product classification code. These values are available to a country profile that matches a product on them; the base matching does not use them.

**Tax values of the line.** One per `.//Item/ClassifiedTaxCategory`: the percentage from `./Percent` and the category code from `./Identifier`. A category without both is skipped. A category whose pair does not appear among the document tax totals of section 3.4 is skipped as well, which is the rule that prevents a line from claiming a tax the document does not declare. A surviving tax value copies the intended usage, the rate and the category code from the matching document tax total, and, when a partner and a label are both known, carries the prediction input.

**Fixed tax values from charges.** Every line level charge whose reason code is `AEO` produces one further tax value: a fixed amount tax whose name is the reason of the charge and whose amount is the charge amount divided by the quantity of the line.

## 5.5 User defined extension fields at line level

As in section 3.7, restricted to a purchase document, using the line extension table that corresponds to the direction of the document, and evaluated against the line rather than against the whole file.

---

# 6. Building the base lines

The lines are assembled in this order: first one base line per document level charge, then one per document level allowance, then one per document line.

## 6.1 A document level allowance or charge line

The sign is +1 for a charge and −1 for an allowance.

| Condition | Unit price | Quantity |
|---|---|---|
| a base amount was read on the node | base amount × sign × document sign of the file | multiplier factor ÷ 100 |
| no base amount was read on the node | amount of the node × sign × document sign of the file | 1 |

The label of the produced base line is the reason of the allowance or charge, and its tax is the one resolved for the node, when one was resolved.

Expressing a percentage based allowance as a base amount with a quantity equal to the percentage reproduces the amount exactly and keeps the percentage visible on the produced line.

## 6.2 A document line

The base line carries the quantity, the unit price and the discount reconstructed in section 5.3, the taxes resolved in section 4.9, the product, the unit of measure and the account when they were resolved, and the label. Every base line also carries the direction sign of the document, whether the document is a credit note, the document currency, the conversion rate, the matched partner, and the instruction that the amounts given are totals excluding tax.

## 6.3 Folding a charge into the line

Before the base line is built, every line level charge whose fixed tax was actually resolved is folded into the line, because its amount is already included in the line extension amount and must not be counted twice. The formula and a worked example are in section 13 of [calculations.md](calculations.md).

## 6.4 Correcting a price included tax

After the tax details have been computed and rounded, the unit price of every base line is corrected when at least one of its taxes is a tax whose price is included, because the file states amounts excluding tax while such a tax expects a unit price that includes it.

**When the discount of the line is not exactly 100 percent**, take each tax of the line whose price is included in turn and raise the unit price.

```formula
grossed raw tax amount = raw tax amount of the tax (document currency) ÷ ( 1 − discount percentage ÷ 100 )
unit price             = unit price + grossed raw tax amount ÷ quantity
```

The quantity used in that division is taken as 1 when the quantity of the line is zero.

**When the discount of the line is exactly 100 percent**, recompute the tax details of the same line with a discount of zero, then take each tax of that recomputation whose price is included in turn.

```formula
unit price = unit price + raw tax amount of the tax (document currency) ÷ quantity
```

The quantity used in that division is again taken as 1 when the quantity of the line is zero.

The special treatment of a full discount exists because dividing by one minus one is impossible; recomputing without the discount gives the amount the tax would have had.

## 6.5 Removing empty lines

A base line whose total including tax is zero in the document currency is dropped, unless it carries a discount. A line with a discount is kept even at zero, because a hundred percent discount line is meaningful information.

---

# 7. Writing the document

Every base line becomes one line of the document carrying its label, its quantity, its unit price, its discount and its tax list, plus its product, its unit of measure and its account when those were resolved and only then, so that a value that was not resolved is left to the ordinary recomputation rules of the accounting document. The whole set of collected document level values and the line list are written in one operation, with the balance check, the discount precision limit and the dynamic line synchronisation suspended for the duration of the write, so that intermediate states are never validated.

---

# 8. Correcting the tax amounts

The purpose is to make the tax amounts of the produced document equal the tax amounts stated in the file, because the issuer may have applied a different rounding. The algorithm is in section 14 of [calculations.md](calculations.md). Two conditions gate it:

1. **Completeness.** Every document tax total entry must have had at least one line level tax value whose tax was actually resolved. When any entry is incomplete, no correction is made and the document is marked as having incomplete taxes, which also disables the untaxed amount correction of section 9.
2. **Tolerance.** The absolute difference between the tax total of the produced document and the sum of the stated tax amounts must not exceed three hundredths of the document currency unit. Beyond that the discrepancy is assumed to come from somewhere else and nothing is corrected.

When both hold, the tax amount of each group is redistributed over the individual tax details of that group in proportion to their raw amounts, and the tax lines of the document are updated with the result.

---

# 9. Correcting the untaxed amount

Run only when the taxes were complete.

```
expected = tax exclusive amount stated in ./LegalMonetaryTotal/TaxExclusiveAmount
         + payable rounding amount stated in ./LegalMonetaryTotal/PayableRoundingAmount
expected = expected × file document sign
difference = round(expected - the untaxed amount of the produced document, currency)
for every line level charge whose fixed tax was resolved:
    difference = difference - the amount of that charge
if the difference is zero in the currency: stop
```

Otherwise one extra line is appended to the document with the label `Rounding`, a quantity of one, a unit price equal to the difference and no tax. That single line absorbs both a cash rounding stated by the issuer and any residual presentation discrepancy. The full derivation and a worked example are in section 15 of [calculations.md](calculations.md).

---

# 10. Post processing

1. **Store the file.** When the file came from an attachment and the document is a purchase document or a receipt, that attachment is turned into the stored structured file of the document.
2. **Restore the printed document.** When the file was extracted from a printed document, that printed document becomes the main attachment of the accounting document again.
3. **Extract the embedded documents.** Every additional document reference of the file that holds an embedded binary object whose declared media type is one of the supported media types is turned into an attachment of the accounting document. The name is taken from the identifier of the reference, reduced to its last path segment and given the extension of its media type, and defaults to `invoice` when the identifier is empty. The content is decoded from base sixty four, with the padding repaired when the encoded text is not a multiple of the block length. When the main attachment of the document is a markup file and the extracted document is a printed document, the extracted document becomes the main attachment instead. Nothing is extracted when the main attachment of the document is already a printed document, because the document then looks as though it has already been imported.
4. **Produce a substitute printed document.** When nothing was extracted, the document has no main attachment, the document is a purchase document, and the configuration parameter that disables this behaviour is not set, a printed document is rendered from the imported values using the dedicated preview template and attached to the document as its main attachment. The template renders the ordinary invoice layout with the trading partner shown as the issuer, and stamps it with the word `Preview` and the sentences `This is a visual representation of the markup content.` and `It is not an official document.` A rendering failure is logged and leaves the document without a printed representation.
5. **Log the result.** One message is posted on the discussion thread of the document whose bold title is `Format used to import the invoice: <the display name of the profile>` and whose body is an unordered list of the collected messages, with duplicates removed while preserving their order. The source attachment and every extracted attachment are linked to that message.

When the document arrived from the exchange network, the title of the message becomes `Peppol invoice received` instead and the first item of the list is `Peppol document UUID: <the message identifier>`, with the acronym expanded to "universally unique identifier".

---

# 11. The cross industry invoice family

The procedure is the same as in section 2, with these element differences.

## 11.1 Document level

| Value | Path |
|---|---|
| customer reference | `./ExchangedDocument/Identifier` |
| origin | `.//BuyerOrderReferencedDocument/IssuerAssignedID` |
| invoice date | `./ExchangedDocument/IssueDateTime/DateTimeString`, parsed with the compact date form |
| due date | `.//SpecifiedTradePaymentTerms/DueDateDateTime/DateTimeString`, parsed with the compact date form |
| delivery date | `.//ActualDeliverySupplyChainEvent/OccurrenceDateTime/DateTimeString`, parsed with the compact date form |
| terms and conditions | every `./ExchangedDocument/IncludedNote`, each rendered as the subject code followed by a colon and a space when the note has a subject code, then the content |
| currency code | `.//InvoiceCurrencyCode` |
| payment reference | `./SupplyChainTradeTransaction/ApplicableHeaderTradeSettlement/PaymentReference` |
| prepaid amount | `.//ApplicableHeaderTradeSettlement/SpecifiedTradeSettlementHeaderMonetarySummation/TotalPrepaidAmount` |
| payee account numbers | `.//SpecifiedTradeSettlementPaymentMeans/PayeePartyCreditorFinancialAccount/IBANID`, falling back per group to `.../ProprietaryID` |

The party group read is the seller trade party when the journal is a purchase journal and the buyer trade party when it is a sale journal. Its values are read as follows: the tax identification number from `.//SpecifiedTaxRegistration/Identifier` restricted to values longer than five characters, the name from `.//Name`, the telephone number from `.//DefinedTradeContact/TelephoneUniversalCommunication/CompleteNumber`, the electronic mail address from `.//EmailURIUniversalCommunication/URIID`, and the postal address from `.//PostalTradeAddress`, whose country is `.//CountryID`, whose street is `.//LineOne`, whose second street line is `.//LineTwo`, whose city is `.//CityName` and whose postal code is `.//PostcodeCode`.

## 11.2 Line level

| Value | Path |
|---|---|
| the line collection | `./SupplyChainTradeTransaction/IncludedSupplyChainTradeLineItem` |
| gross unit price | `./SpecifiedLineTradeAgreement/GrossPriceProductTradePrice/ChargeAmount` |
| price level allowance amount | `./SpecifiedLineTradeAgreement/GrossPriceProductTradePrice/AppliedTradeAllowanceCharge/ActualAmount` |
| net unit price | `./SpecifiedLineTradeAgreement/NetPriceProductTradePrice/ChargeAmount` |
| base quantity | `./SpecifiedLineTradeAgreement/GrossPriceProductTradePrice/BasisQuantity`, falling back to `./SpecifiedLineTradeAgreement/NetPriceProductTradePrice/BasisQuantity` |
| delivered quantity | `./SpecifiedLineTradeDelivery/BilledQuantity` |
| line allowances and charges | `.//SpecifiedLineTradeSettlement/SpecifiedTradeAllowanceCharge`, whose indicator is `./ChargeIndicator/Indicator`, whose amount is `./ActualAmount`, whose reason is `./Reason` and whose reason code is `./ReasonCode` |
| line net amount | `./SpecifiedLineTradeSettlement/SpecifiedTradeSettlementLineMonetarySummation/LineTotalAmount` |
| label | `./SpecifiedTradeProduct/Description`, falling back to `./SpecifiedTradeProduct/Name` |
| internal reference | `./SpecifiedTradeProduct/SellerAssignedID` |
| product name | `./SpecifiedTradeProduct/Name` |
| barcode | `./SpecifiedTradeProduct/GlobalID` |
| deferred start date | `./SpecifiedLineTradeSettlement/BillingSpecifiedPeriod/StartDateTime/DateTimeString` |
| deferred end date | `./SpecifiedLineTradeSettlement/BillingSpecifiedPeriod/EndDateTime/DateTimeString` |

## 11.3 Document level allowances and charges and tax rates

The document level allowance and charge collection is `./SupplyChainTradeTransaction/ApplicableHeaderTradeSettlement/SpecifiedTradeAllowanceCharge`, whose indicator is `./ChargeIndicator/Indicator`, whose base amount is `./BasisAmount`, whose amount is `./ActualAmount`, whose reason is `./Reason`, whose percentage is `./CalculationPercent` and whose tax percentage is `./CategoryTradeTax/RateApplicablePercent`.

The tax rates of the document are read from every `.//ApplicableTradeTax/RateApplicablePercent`.

---

# 12. Importing an order

A file whose customization identifier is the ordering transaction identifier is routed to the ordering profile of the sales domain or of the purchasing domain, according to the model of the target record.

## 12.1 Shared extraction

| Value written on the order | Path |
|---|---|
| order date | `.//EndDate`, falling back to `.//IssueDate` |
| note | `./Note` |
| payment terms | the payment terms whose name equals the text of `.//PaymentTerms/Note`, searched within the companies of the order; nothing when no name is given or no record matches |
| currency | `.//DocumentCurrencyCode`, resolved as in section 4.3 |

## 12.2 Importing into a sales order

| Value written | Path |
|---|---|
| partner | matched from the buyer customer party, using the partner values of section 3.1 |
| customer reference | `./Identifier` |
| origin | `./QuotationDocumentReference/Identifier` |
| shipping address | matched from the delivery party |
| lines | one per `./OrderLine/LineItem` |

The note of the file is discarded, because the terms and conditions of the sales order take precedence over the ones of the purchase order that produced the file. Every line whose product could not be matched logs `Could not retrieve the product named: <the label>`. The deferred date fields are removed from every line, and the discount is removed from every line, so that the pricing rules of the sales domain decide it. After the lines have been written, the unit price and the discount of every line that did match a product are recomputed from the pricing rules.

## 12.3 Importing into a purchase order

| Value written | Path |
|---|---|
| partner | matched from the seller supplier party |
| vendor reference | `./Identifier` |
| origin | `./OriginatorDocumentReference/Identifier` |
| destination address | matched from the delivery party |
| lines | one per `./OrderLine/LineItem` |

Every line whose product could not be matched logs the same message as above. The deferred date fields are removed from every line; the discount is kept.

## 12.4 Reporting the result of an order import

A message is posted on the order whose body is the bold sentence `Format used to import the document: <the display name of the profile>`. When any message was collected, an activity of the "to do" kind is created on the order, assigned to the current user, whose note is `Some information could not be imported:` followed by an unordered list of the collected messages.

---

# 13. Worked example: importing a vendor bill

**The file.** A billing profile invoice in euro, with one line.

```
./Identifier                                            = "F-2024-77"
./IssueDate                                             = 2024-03-04
./DueDate                                               = 2024-04-03
./DocumentCurrencyCode                                  = EUR
./AccountingSupplierParty/.../CompanyID                 = "BE0477472701"
./AccountingSupplierParty/.../Name                      = "Test Vendor"
./AccountingSupplierParty/.../IdentificationCode        = "BE"
./PaymentMeans/PayeeFinancialAccount/Identifier         = "BE15001559627230"
./TaxTotal/TaxAmount                                    = 199.50
./TaxTotal/TaxSubtotal/TaxableAmount                    = 950.00
./TaxTotal/TaxSubtotal/TaxAmount                        = 199.50
./TaxTotal/TaxSubtotal/TaxCategory/Identifier           = "S"
./TaxTotal/TaxSubtotal/TaxCategory/Percent              = 21.0
./LegalMonetaryTotal/TaxExclusiveAmount                 = 950.00
./LegalMonetaryTotal/PayableAmount                      = 1149.50
./InvoiceLine/Identifier                                = "1"
./InvoiceLine/InvoicedQuantity                          = 1.0, unit code "C62"
./InvoiceLine/LineExtensionAmount                       = 950.00
./InvoiceLine/AllowanceCharge/ChargeIndicator           = "false"
./InvoiceLine/AllowanceCharge/AllowanceChargeReasonCode = "95"
./InvoiceLine/AllowanceCharge/Amount                    = 50.00
./InvoiceLine/AllowanceCharge/BaseAmount                = 1000.00
./InvoiceLine/Item/Name                                 = "Office Chair"
./InvoiceLine/Item/SellersItemIdentification/Identifier = "FURN_7777"
./InvoiceLine/Item/ClassifiedTaxCategory/Identifier     = "S"
./InvoiceLine/Item/ClassifiedTaxCategory/Percent        = 21.0
./InvoiceLine/Price/PriceAmount                         = 1000.0
```

**The import, step by step.**

1. The root element is `Invoice` and the tax inclusive amount is positive, so the direction is an invoice and the file document sign is `1`. The journal is a purchase journal, so the document becomes a vendor bill.
2. The payee account number `BE15001559627230` is collected.
3. The supplier party values are collected: the tax identification number `BE0477472701`, the name `Test Vendor`, the country code `BE`. The partner strategies run: the first one, matching by tax identification number, finds the vendor. No partner is created.
4. The invoice date becomes the fourth of March 2024 and the due date the third of April 2024.
5. The currency code `EUR` resolves to the euro, which is active, so no message is logged. The conversion rate for the fourth of March 2024 is read.
6. The account number is found on the vendor, or created on it, and becomes the recipient bank account of the bill.
7. The customer reference becomes `F-2024-77`; there is no order reference, so the origin stays empty unless the purchasing capability package finds a purchase order number inside the item description, which it does not here.
8. The prepaid amount is absent, so nothing is logged.
9. One document tax total entry is created with the key made of the category code `S` and the percentage `21.0`, holding the stated tax amount `199.50`.
10. There is no document level allowance or charge.
11. The line is read. Its line level allowance holds the amount `50.00` and the base amount `1000.00`. The label becomes `[FURN_7777] Office Chair`. The reconstruction of section 12 of [calculations.md](calculations.md) takes the branch in which both a line net amount and an invoiced quantity are present: the quantity is `1.0`, the subtotal is `950.00 + 50.00 = 1000.00`, the price level contributes a subtotal of `1000.00` over a quantity of one with no price level discount, so the unit price is `1000.00` and the discount amount is `50.00`, which becomes a discount of `50.00 × 100 ÷ 1000.00 = 5.0` percent.
12. The product values are collected: the internal reference `FURN_7777`, the name `Office Chair`, no barcode. The strategies run in rank order; the vendor catalogue strategy at rank 5 finds nothing, the barcode strategy at rank 10 finds nothing, the internal reference strategy at rank 15 finds exactly one product, the office chair, and wins.
13. The unit code `C62` resolves to the unit "unit", which shares a reference with the unit of the matched product, so it is written on the line.
14. The account is predicted from the label and the vendor.
15. The line tax value is built from the classified tax category: the category code `S`, the percentage `21.0`. The pair appears in the document tax totals, so the value survives. The tax strategies run: the default tax of the predicted account, then the prediction, then the matching by rate; one of them resolves the twenty one percent purchase tax.
16. The base line is built with quantity `1.0`, unit price `1000.00`, discount `5.0` percent and that tax. No price included tax is involved, so no unit price correction happens. The total including tax is not zero, so the line is kept.
17. The document is written: one line, the vendor, the dates, the currency, the reference and the bank account.
18. The tax correction runs. The document tax total entry is complete, because its line tax value resolved a tax. The produced tax amount is `950.00 × 0.21 = 199.50`, and the stated amount is `199.50`, so the difference is zero and nothing is redistributed.
19. The untaxed amount correction runs. The expected untaxed amount is `950.00 + 0.00 = 950.00`, the produced untaxed amount is `950.00`, the difference is zero, so no rounding line is added.
20. The file is stored as the structured file of the bill, no embedded document is found, a substitute printed document is rendered, and one message is posted whose title names the profile and whose list is empty.

**The produced vendor bill.**

| Field | Value |
|---|---|
| direction | vendor bill |
| partner | Test Vendor |
| customer reference | `F-2024-77` |
| invoice date | 2024-03-04 |
| due date | 2024-04-03 |
| currency | euro |
| recipient bank account | `BE15001559627230` |
| line label | `[FURN_7777] Office Chair` |
| line product | the office chair |
| line quantity | `1.0` |
| line unit of measure | unit |
| line unit price | `1000.00` |
| line discount | `5.0` percent |
| line tax | the twenty one percent purchase tax |
| untaxed amount | `950.00` |
| tax amount | `199.50` |
| total | `1149.50` |
