# Cross industry invoice mapping

The complete element by element mapping between an accounting document and a structured file of the cross industry invoice family, which is the hybrid format whose structured file is carried inside the printed document: the document skeleton with its element order, every element of the document context, the document header, the line items, the trade agreement, the trade delivery and the trade settlement, the value written into each one, the condition under which it is omitted, the validations, and the rules for embedding the structured file inside the printed document. The arithmetic behind the amounts is in [calculations.md](calculations.md); the reverse direction is in [import-mapping.md](import-mapping.md).

## Notation used in this file

The conventions of section "Notation used in this file" of [universal-business-language-mapping.md](universal-business-language-mapping.md) apply here as well, including the spelling of the identifier element, which is written `Identifier` in every path of this file while the standard spells it with the two letters that abbreviate that word: element names and code list values are quoted verbatim as data values because a replacement must emit them character for character, a path reads from the root element downwards, an element with an empty value is not written, and the element order inside every parent is fixed by the standard.

**Namespace assignment rule.** The root element and the three elements directly below it belong to the cross industry invoice namespace, whose namespace identifier is `urn:un:unece:uncefact:data:standard:CrossIndustryInvoice:100`. Every other element belongs to the reusable aggregate business information entity namespace, whose namespace identifier is `urn:un:unece:uncefact:data:standard:ReusableAggregateBusinessInformationEntity:100`, except the date and time value element and the indicator value element, which belong to the unqualified data type namespace, whose namespace identifier is `urn:un:unece:uncefact:data:standard:UnqualifiedDataType:100`. Two further namespace identifiers are declared on the root element although no element uses them: the qualified data type namespace, `urn:un:unece:uncefact:data:standard:QualifiedDataType:100`, and the schema instance namespace, `http://www.w3.org/2001/XMLSchema-instance`.

**Dates.** Every date is written as a date and time value element whose text is the date in the compact form of four digits for the year, two for the month and two for the day, with no separator, and which carries an attribute named `format` holding the value `102`, the code of that date format in the published code list.

**Amounts.** Every monetary amount is written with at most two decimal places, except the amounts of the line level allowances and charges, which are written with at most as many decimal places as the currency has. Only the total tax amount carries a currency attribute; every other amount is understood to be in the invoice currency declared once in the settlement group. The presentation rounding rule is in section 1 of [calculations.md](calculations.md).

---

# 1. Profile identity

| Property | Value |
|---|---|
| Root element | `CrossIndustryInvoice` |
| Guideline context identifier written | `urn:cen.eu:en16931:2017#conformant#urn:factur-x.eu:1p0:extended` |
| Version of the syntax | 2.2.0 |
| File name of the produced file | the document name with every solidus replaced by an underscore, followed by `_zugferd` with the extension `xml` when the commercial contact of the customer is established in Germany, and by `_factur_x` with the extension `xml` otherwise |
| File name used when the file is embedded in the printed document | always `factur-x` with the extension `xml` |

The same builder serves the French hybrid invoice and the German hybrid invoice; only the produced file name differs. The business process context identifier element exists in the skeleton but is not written by this builder; a country profile that needs it writes it.

---

# 2. Document skeleton and element order

```
CrossIndustryInvoice
├── ExchangedDocumentContext
│   ├── BusinessProcessSpecifiedDocumentContextParameter / Identifier
│   └── GuidelineSpecifiedDocumentContextParameter / Identifier
├── ExchangedDocument
│   ├── Identifier
│   ├── TypeCode
│   ├── IssueDateTime / DateTimeString
│   └── IncludedNote (repeatable)
│       ├── Content
│       └── SubjectCode
└── SupplyChainTradeTransaction
    ├── IncludedSupplyChainTradeLineItem (repeatable)
    ├── ApplicableHeaderTradeAgreement
    ├── ApplicableHeaderTradeDelivery
    └── ApplicableHeaderTradeSettlement
```

## 2.1 Line item

```
IncludedSupplyChainTradeLineItem
├── AssociatedDocumentLineDocument / LineID
├── SpecifiedTradeProduct
│   ├── GlobalID
│   ├── SellerAssignedID
│   ├── Name
│   └── Description
├── SpecifiedLineTradeAgreement
│   ├── GrossPriceProductTradePrice
│   │   ├── ChargeAmount
│   │   └── AppliedTradeAllowanceCharge
│   │       ├── ChargeIndicator / Indicator
│   │       └── ActualAmount
│   └── NetPriceProductTradePrice / ChargeAmount
├── SpecifiedLineTradeDelivery / BilledQuantity
└── SpecifiedLineTradeSettlement
    ├── ApplicableTradeTax (repeatable)
    │   ├── TypeCode
    │   ├── CategoryCode
    │   └── RateApplicablePercent
    ├── BillingSpecifiedPeriod
    │   ├── StartDateTime / DateTimeString
    │   └── EndDateTime / DateTimeString
    ├── SpecifiedTradeAllowanceCharge (repeatable)
    │   ├── ChargeIndicator / Indicator
    │   ├── ActualAmount
    │   ├── ReasonCode
    │   └── Reason
    └── SpecifiedTradeSettlementLineMonetarySummation / LineTotalAmount
```

## 2.2 Trade agreement, delivery and settlement

```
ApplicableHeaderTradeAgreement
├── BuyerReference
├── SellerTradeParty
├── BuyerTradeParty
├── BuyerOrderReferencedDocument / IssuerAssignedID
└── ContractReferencedDocument / IssuerAssignedID

ApplicableHeaderTradeDelivery
├── ShipToTradeParty
└── ActualDeliverySupplyChainEvent / OccurrenceDateTime / DateTimeString

ApplicableHeaderTradeSettlement
├── PaymentReference
├── InvoiceCurrencyCode
├── SpecifiedTradeSettlementPaymentMeans
│   ├── TypeCode
│   └── PayeePartyCreditorFinancialAccount
│       ├── IBANID
│       └── ProprietaryID
├── ApplicableTradeTax (repeatable)
│   ├── CalculatedAmount
│   ├── TypeCode
│   ├── ExemptionReason
│   ├── BasisAmount
│   ├── CategoryCode
│   ├── ExemptionReasonCode
│   ├── DueDateTypeCode
│   └── RateApplicablePercent
├── BillingSpecifiedPeriod
│   ├── StartDateTime / DateTimeString
│   └── EndDateTime / DateTimeString
├── SpecifiedTradePaymentTerms
│   ├── Description
│   ├── DueDateDateTime / DateTimeString
│   └── ApplicableTradePaymentDiscountTerms
│       ├── BasisPeriodMeasure
│       └── CalculationPercent
└── SpecifiedTradeSettlementHeaderMonetarySummation
    ├── LineTotalAmount
    ├── TaxBasisTotalAmount
    ├── TaxTotalAmount
    ├── RoundingAmount
    ├── GrandTotalAmount
    ├── TotalPrepaidAmount
    └── DuePayableAmount
```

## 2.3 Trade party

```
<party>
├── Identifier
├── Name
├── SpecifiedLegalOrganization / Identifier
├── DefinedTradeContact
│   ├── PersonName
│   ├── TelephoneUniversalCommunication / CompleteNumber
│   └── EmailURIUniversalCommunication / URIID
├── PostalTradeAddress
│   ├── PostcodeCode
│   ├── LineOne
│   ├── LineTwo
│   ├── CityName
│   └── CountryID
├── URIUniversalCommunication / URIID
└── SpecifiedTaxRegistration / Identifier
```

The seller party and the buyer party carry all of the above. The ship to party carries only the identifier, the name, the contact and the postal address; its legal organisation, its universal communication and its tax registration are never written.

---

# 3. Preparation of the export values

The steps run in this exact order before any element is written.

1. **Validate the tax structure** of every tax used on a line. A failure refuses the export with `Tax '<tax name>' is invalid: <the repartition error message>`.
2. **Resolve the parties.** For a document the company issues: the supplier is the contact of the company, the customer is the partner of the document and the shipping party is the shipping address of the document, falling back to the partner. For a vendor document the supplier and the customer are swapped, and the shipping party becomes the first child contact of the customer whose address kind is the delivery address, falling back to the customer.
3. **Resolve the currencies**: the document currency and the company currency.
4. **Set the delivery date** to the delivery date of the accounting document, falling back to its invoice date.
5. **Build the base lines** of the document with their tax details.
6. **Turn every negative unit price positive** by negating both the quantity and the unit price.
7. **Split out the cash rounding lines** into their own list; they leave the line list and are reported only in the rounding amount.
8. **Split out the early payment discount lines** into their own list; they leave the line list and are not reported at all by this builder.
9. **Apply the three digit and six digit global rounding** to the raw amounts of every base line, in the document currency: round the raw total excluding tax; add and round the raw gross total excluding tax and the raw discount amount; round the raw gross total excluding tax and the raw discount amount.

Unlike the universal business language family, this builder does not turn emptying taxes into extra document lines, and does not produce document level allowances and charges. A recycling contribution tax and an excise tax are still reported as line level allowances and charges.

---

# 4. Document context and document header

| Path | Value written | Omitted when |
|---|---|---|
| `ExchangedDocumentContext / GuidelineSpecifiedDocumentContextParameter / Identifier` | `urn:cen.eu:en16931:2017#conformant#urn:factur-x.eu:1p0:extended`; this is the identifier of the guideline the file conforms to | never |
| `ExchangedDocument / Identifier` | the name of the accounting document; this is the identifier of the invoice | never |
| `ExchangedDocument / TypeCode` | `380` when the accounting document is a customer invoice, `381` in every other case | never |
| `ExchangedDocument / IssueDateTime / DateTimeString` | the invoice date of the accounting document, in the compact date form, with the attribute `format` holding `102` | the document has no invoice date |
| `ExchangedDocument / IncludedNote / Content` | the combined note of section 4.1 | there is no sentence to write |
| `ExchangedDocument / IncludedNote / SubjectCode` | not written on the combined note | always on the combined note |

## 4.1 The note

The first included note holds one single text made of these sentences joined by single spaces, and carries no subject code:

1. When the document carries withholding taxes, the sentence `The prepaid amount of <amount> corresponds to the withholding tax applied.` where the amount is the total withholding amount formatted in the document currency with its symbol, and with the non breaking space removed.
2. The terms and conditions of the accounting document, converted from rich text to plain text, when they are not empty.

A country profile may declare further default notes; each of those becomes an additional included note carrying both a content and a subject code, appended after the combined note in the order in which the profile declares them. The base builder declares none.

---

# 5. Line items

One line item is produced per base line, in the order of the base lines, numbered from one.

| Path | Value written | Omitted when |
|---|---|---|
| `AssociatedDocumentLineDocument / LineID` | the position of the line, starting at one; this is the identifier of the line | never |
| `SpecifiedTradeProduct / GlobalID` | the barcode of the product, with the attribute `schemeID` holding `0160`, the code of the global trade item number list | the line carries no product or the product has no barcode |
| `SpecifiedTradeProduct / SellerAssignedID` | the internal reference of the product; this is the seller item identifier | the product has no internal reference |
| `SpecifiedTradeProduct / Name` | the label of the line | the line has no label |
| `SpecifiedTradeProduct / Description` | the internal description of the product, converted from rich text to plain text | the product has no internal description |
| `SpecifiedLineTradeAgreement / GrossPriceProductTradePrice / ChargeAmount` | the raw gross unit price of the line excluding tax, in the document currency, written with at most two decimal places | never |
| `SpecifiedLineTradeAgreement / GrossPriceProductTradePrice / AppliedTradeAllowanceCharge / ChargeIndicator / Indicator` | `false` when the discount percentage is positive, `true` when it is negative | the line has no discount |
| `SpecifiedLineTradeAgreement / GrossPriceProductTradePrice / AppliedTradeAllowanceCharge / ActualAmount` | the raw gross unit price multiplied by the absolute discount percentage divided by one hundred, rounded to the currency, written with at most two decimal places | the line has no discount |
| `SpecifiedLineTradeAgreement / NetPriceProductTradePrice / ChargeAmount` | the raw total of the line excluding tax divided by its quantity, or by one when the quantity is zero, written with at most two decimal places | never |
| `SpecifiedLineTradeDelivery / BilledQuantity` | the quantity of the base line, with the attribute `unitCode` holding the unit code of its unit of measure | never |

## 5.1 Line tax

One applicable trade tax group per tax category of the line, obtained by aggregating the tax details of that line with the tax category grouping key. A group whose key is a withholding key is not written.

| Path | Value written |
|---|---|
| `SpecifiedLineTradeSettlement / ApplicableTradeTax / TypeCode` | `VAT`, the identifier of the value-added tax scheme |
| `SpecifiedLineTradeSettlement / ApplicableTradeTax / CategoryCode` | the tax category code of the key |
| `SpecifiedLineTradeSettlement / ApplicableTradeTax / RateApplicablePercent` | the percentage of the key; nothing when the category code is `O` |

The tax category grouping key of this family differs from the one of the universal business language family in one point: the tax scheme identifier is always `VAT`, the value-added tax scheme, never `GST`, the goods and services tax scheme. Everything else, including the category code prediction and the exemption reason prediction, is shared and specified in section 10 of [calculations.md](calculations.md).

## 5.2 Line billing period

| Path | Value written | Omitted when |
|---|---|---|
| `SpecifiedLineTradeSettlement / BillingSpecifiedPeriod / StartDateTime / DateTimeString` | the deferred start date of the line | the line has no deferred start date |
| `SpecifiedLineTradeSettlement / BillingSpecifiedPeriod / EndDateTime / DateTimeString` | the deferred end date of the line | the line has no deferred end date |

## 5.3 Line allowances and charges

Two kinds of group, in this order: one per recycling contribution tax of the line, then one per excise tax of the line.

| Path | Recycling contribution group | Excise group |
|---|---|---|
| `SpecifiedTradeAllowanceCharge / ChargeIndicator / Indicator` | `true` when the tax amount multiplied by the base amount is greater than zero, `false` otherwise | `true` when the tax amount is greater than zero, `false` otherwise |
| `SpecifiedTradeAllowanceCharge / ActualAmount` | the tax amount in the document currency, keeping its sign, written with at most as many decimal places as the currency has | the absolute value of the tax amount in the document currency, written with at most as many decimal places as the currency has |
| `SpecifiedTradeAllowanceCharge / ReasonCode` | `CAV` when the lower case tax name contains the text `bebat` and the group is a charge, `AEO` when it is another charge, `100` when it is an allowance | not written |
| `SpecifiedTradeAllowanceCharge / Reason` | the name of the tax | the name of the tax |

## 5.4 Line total

```
line_total = round(raw_total_excluded_currency, currency)
for every allowance or charge group already produced on this line:
    sign = +1 when its indicator is "true", otherwise -1
    line_total = line_total + sign × its actual amount
```

The value is written in `SpecifiedLineTradeSettlement / SpecifiedTradeSettlementLineMonetarySummation / LineTotalAmount` with at most two decimal places. As in the other family, the line total is computed from the amounts that were actually written, not from the amounts before presentation rounding.

---

# 6. Trade agreement

| Path | Value written | Omitted when |
|---|---|---|
| `BuyerReference` | the dedicated buyer reference field of the accounting document when a country profile defines one and it is filled, otherwise the reference of the commercial contact of the accounting document | neither is set |
| `SellerTradeParty` | the party group of section 6.1 | never |
| `BuyerTradeParty` | the party group of section 6.2 | never |
| `BuyerOrderReferencedDocument / IssuerAssignedID` | the dedicated purchase order reference field of the accounting document when a country profile defines one and it is filled, otherwise the customer reference of the accounting document, otherwise its name; this is the identifier of the order of the buyer | never, because of the fallback |
| `ContractReferencedDocument / IssuerAssignedID` | the dedicated contract reference field of the accounting document when a country profile defines one and it is filled, otherwise nothing; this is the identifier of the contract | no country profile defines the field, or it is empty |

## 6.1 Seller trade party

| Path | Value written |
|---|---|
| `Identifier` | not written for the seller |
| `Name` | the name of the supplier contact |
| `SpecifiedLegalOrganization / Identifier` | the company registry number of the commercial contact of the supplier; when that number is a valid French establishment number, only its first nine characters are written, which is the company identifier part, and the attribute `schemeID` holds `0002`; otherwise the whole number is written with no scheme attribute |
| `DefinedTradeContact / PersonName` | the name of the supplier contact |
| `DefinedTradeContact / TelephoneUniversalCommunication / CompleteNumber` | the telephone number of the supplier contact |
| `DefinedTradeContact / EmailURIUniversalCommunication / URIID` | the electronic mail address of the supplier contact |
| `PostalTradeAddress / PostcodeCode` | the postal code of the supplier contact |
| `PostalTradeAddress / LineOne` | the street of the supplier contact |
| `PostalTradeAddress / LineTwo` | the second street line of the supplier contact, omitted when empty |
| `PostalTradeAddress / CityName` | the city of the supplier contact |
| `PostalTradeAddress / CountryID` | the two letter code of the country of the supplier contact |
| `URIUniversalCommunication / URIID` | the participant endpoint of the supplier contact, with the attribute `schemeID` holding its electronic address scheme code; the whole group is omitted unless both are set |
| `SpecifiedTaxRegistration / Identifier` | the foreign tax identification number of the fiscal position of the accounting document when it has one, otherwise the tax identification number of the commercial contact of the supplier, with the attribute `schemeID` holding `VA`, the code of the value added tax register; the whole group is omitted when neither is set |

## 6.2 Buyer trade party

Identical in shape to the seller trade party, with these differences: the name, the contact and the address are taken from the customer contact rather than from the supplier contact; the legal organisation identifier is the company registry number of the commercial contact of the customer, with the same French establishment number rule; and the tax registration identifier is the tax identification number of the customer contact itself, not of its commercial contact.

## 6.3 Ship to trade party

| Path | Value written |
|---|---|
| `Identifier` | the global location number of the shipping party, with the attribute `schemeID` holding `0088`; omitted when that capability package is not installed or the party has no such number |
| `Name` | the name of the shipping party |
| `DefinedTradeContact` | the contact group built from the shipping party |
| `PostalTradeAddress` | the address group built from the shipping party |

---

# 7. Trade delivery

| Path | Value written | Omitted when |
|---|---|---|
| `ShipToTradeParty` | the party group of section 6.3 | never |
| `ActualDeliverySupplyChainEvent / OccurrenceDateTime / DateTimeString` | the delivery date of the accounting document, falling back to its invoice date | neither date is set |

---

# 8. Trade settlement

| Path | Value written | Omitted when |
|---|---|---|
| `PaymentReference` | the payment reference of the accounting document | it is empty |
| `InvoiceCurrencyCode` | the three letter code of the document currency | never |
| `SpecifiedTradeSettlementPaymentMeans / TypeCode` | `59`, the code for a direct debit under the single euro payments area scheme, when the payment capability package defines direct debit mandates and at least one payment reconciled with this document carries a mandate; `42`, the code for a payment to a bank account, otherwise | never |
| `SpecifiedTradeSettlementPaymentMeans / PayeePartyCreditorFinancialAccount / IBANID` | the sanitised account number of the recipient bank account, when that account is of the international bank account number kind | the account is of another kind, or there is no recipient bank account |
| `SpecifiedTradeSettlementPaymentMeans / PayeePartyCreditorFinancialAccount / ProprietaryID` | the sanitised account number of the recipient bank account, when that account is not of the international bank account number kind | the account is of the international kind, or there is no recipient bank account |

## 8.1 Document tax

One applicable trade tax group per tax category grouping key of the whole document, obtained by aggregating the tax details of every base line. A group whose key is a withholding key is not written.

| Path | Value written |
|---|---|
| `ApplicableTradeTax / CalculatedAmount` | the aggregated tax amount of the group in the document currency, written as exactly zero when the currency considers it zero, so that a negative zero is never produced |
| `ApplicableTradeTax / TypeCode` | `VAT`, the identifier of the value-added tax scheme |
| `ApplicableTradeTax / ExemptionReason` | the exemption reason text of the key |
| `ApplicableTradeTax / BasisAmount` | the aggregated base amount of the group in the document currency |
| `ApplicableTradeTax / CategoryCode` | the tax category code of the key |
| `ApplicableTradeTax / ExemptionReasonCode` | the exemption reason code of the key |
| `ApplicableTradeTax / DueDateTypeCode` | `5`, the code meaning that the tax becomes due on the date of issue of the invoice |
| `ApplicableTradeTax / RateApplicablePercent` | the percentage of the key; nothing when the category code is `O` |

Writing the calculated amount as exactly zero rather than as a negative zero matters, because a schema validator that compares the text of the element rejects a negative zero where a zero is expected.

## 8.2 Billing period

```
start_dates = [the invoice date of the accounting document, when it has one]
            + [the deferred start date of every line that has one]
end_dates   = [the due date of the accounting document, when it has one]
            + [the deferred end date of every line that has one]
start = the earliest of start_dates, when the list is not empty
end   = the latest of end_dates, when the list is not empty
```

The start is written in `BillingSpecifiedPeriod / StartDateTime / DateTimeString` and the end in `BillingSpecifiedPeriod / EndDateTime / DateTimeString`, each omitted when its list was empty.

## 8.3 Payment terms

| Path | Value written | Omitted when |
|---|---|---|
| `SpecifiedTradePaymentTerms / Description` | the name of the payment terms of the accounting document | the document has no payment terms |
| `SpecifiedTradePaymentTerms / DueDateDateTime / DateTimeString` | the due date of the accounting document | the document has no due date |
| `SpecifiedTradePaymentTerms / ApplicableTradePaymentDiscountTerms / BasisPeriodMeasure` | the number of days of the early payment discount of the payment terms, with the attribute `unitCode` holding `DAY` | the payment terms grant no early payment discount |
| `SpecifiedTradePaymentTerms / ApplicableTradePaymentDiscountTerms / CalculationPercent` | the percentage of the early payment discount of the payment terms | the payment terms grant no early payment discount |

## 8.4 Monetary summation

The seven elements are computed in this exact order, each reading the ones before it. The full derivation with a worked example is in section 9 of [calculations.md](calculations.md).

```
1. line total amount      = the sum, over every document tax group, of its base amount
                            in the document currency
2. tax basis total amount = the same sum
3. tax total amount       = the sum, over every document tax group, of its tax amount
                            in the document currency, carrying the currency attribute
4. rounding amount        = the sum of the totals excluding tax of the cash rounding lines;
                            the element is omitted when that sum is zero
5. grand total amount     = line total amount + tax total amount + rounding amount
6. total prepaid amount   = grand total amount - the residual amount of the accounting document
7. due payable amount     = grand total amount - total prepaid amount
```

The line total amount and the tax basis total amount are deliberately the same value, because this builder reports no document level allowance and no document level charge, so there is nothing that could make the two differ.

---

# 9. Validations

The checks run in this order and every message that is produced stops the export.

| Order | Condition under which the check runs | What is required | Rule |
|---|---|---|---|
| 1 | the accounting document is a customer invoice | a recipient bank account, and a sanitised account number on it | `EIDI-RULE-150` |
| 2 | always | a country on the commercial contact of the supplier | `EIDI-RULE-151` |
| 3 | always | a tax identification number on the supplier | `EIDI-RULE-152` |
| 4 | always | a telephone number on the commercial contact of the supplier | `EIDI-RULE-153` |
| 5 | always | an electronic mail address on the supplier | `EIDI-RULE-154` |
| 6 | always | a country on the commercial contact of the customer | `EIDI-RULE-155` |
| 7 | the collected values declare an intra-community supply | a tax identification number on the supplier and on the commercial contact of the customer | `EIDI-RULE-156` |
| 8 | the customer is established in Spain and the first two characters of its postal code are `35` or `38` | at least one tax with a rate greater than zero on every line | `EIDI-RULE-157` |
| 9 | always | at least one tax on every line that requires one | `EIDI-RULE-061` |

The exact messages are in [business-rules.md](business-rules.md). The base builder never declares an intra-community supply itself; a country profile that needs checks 7 sets that flag from its own analysis of the tax categories of the document.

---

# 10. Embedding the structured file inside the printed document

Every customer document that is sent produces a cross industry invoice file and carries it inside the printed document, whatever the profile selected for the partner. This is what makes the printed document a hybrid document: one file that a human can read and a machine can parse.

1. The printed document is rendered.
2. The structured file to embed is chosen: when the selected profile is already a cross industry invoice profile, the file that was produced for that profile is reused; otherwise a cross industry invoice file is produced silently, with no validation and with its errors discarded, so that the printed document is still interchangeable.
3. The structured file is attached to the printed document under the name `factur-x` with the extension `xml`, with the media type declared as `text/xml` and the relationship declared as `/Alternative`.
4. **Archival conversion.** When the company country is France or Germany, and either the selected profile is a cross industry invoice profile or the commercial contact of the customer is established in France or Germany with an electronic address scheme other than `0204`, the printed document is converted to the archival variant of the portable document format when it is not already one. A conversion failure is logged and the unconverted document is kept.
5. **Archival metadata.** After a successful conversion, an extension metadata block is added to the printed document declaring the document title, which is the name of the accounting document, and the current date. When the block declares the lower conformance level, it is raised to the higher one.

A deployment may nominate further printed document templates that must also carry the structured file, by listing their report names, separated by commas, in the configuration parameter named in [configuration.md](configuration.md). For a nominated template the embedding described above is applied to a single customer document that is posted.

---

# 11. Worked example

**Setup.** Company established in France, currency euro with two decimal places, customer established in France, document name `INV/2024/00010`, invoice date the tenth of March 2024, due date the ninth of April 2024, no payment made, no cash rounding, no deferred dates.

| Line | Product | Quantity | Unit price | Discount | Tax |
|---|---|---|---|---|---|
| 1 | product with internal reference `FURN_7777` | 1 | 1000.00 | 5 percent | 21 percent |
| 2 | product with internal reference `FURN_8888` | 1 | 500.00 | none | 6 percent |

**Line amounts.**

```
line 1: raw gross unit price              = 1000.000000
        applied allowance actual amount   = round(1000.000000 × 0.05) = 50.00
        raw total excluding tax           =  950.000000
        net unit price                    =  950.000000 ÷ 1 = 950.00
        line total amount                 =  950.00
line 2: raw gross unit price              =  500.000000
        no applied allowance
        raw total excluding tax           =  500.000000
        net unit price                    =  500.00
        line total amount                 =  500.00
```

**Document tax groups.**

| Group | Base amount | Tax amount | Category | Percentage |
|---|---|---|---|---|
| standard rate 21 percent | 950.00 | 199.50 | `S` | `21.0` |
| standard rate 6 percent | 500.00 | 30.00 | `S` | `6.0` |

**Monetary summation.**

```
line total amount      = 950.00 + 500.00                  = 1450.00
tax basis total amount = 950.00 + 500.00                  = 1450.00
tax total amount       = 199.50 +  30.00                  =  229.50
rounding amount        = 0.00                             → element omitted
grand total amount     = 1450.00 + 229.50 + 0.00          = 1679.50
total prepaid amount   = 1679.50 - 1679.50                =    0.00
due payable amount     = 1679.50 -    0.00                = 1679.50
```

**The produced elements, in written order.**

| Path | Value |
|---|---|
| `ExchangedDocumentContext / GuidelineSpecifiedDocumentContextParameter / Identifier` | `urn:cen.eu:en16931:2017#conformant#urn:factur-x.eu:1p0:extended` |
| `ExchangedDocument / Identifier` | `INV/2024/00010` |
| `ExchangedDocument / TypeCode` | `380` |
| `ExchangedDocument / IssueDateTime / DateTimeString` | `20240310` with the attribute `format` holding `102` |
| `IncludedSupplyChainTradeLineItem[1] / AssociatedDocumentLineDocument / LineID` | `1` |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedTradeProduct / SellerAssignedID` | `FURN_7777` |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedTradeProduct / Name` | the label of the line |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedLineTradeAgreement / GrossPriceProductTradePrice / ChargeAmount` | `1000.00` |
| `IncludedSupplyChainTradeLineItem[1] / ... / AppliedTradeAllowanceCharge / ChargeIndicator / Indicator` | `false` |
| `IncludedSupplyChainTradeLineItem[1] / ... / AppliedTradeAllowanceCharge / ActualAmount` | `50.00` |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedLineTradeAgreement / NetPriceProductTradePrice / ChargeAmount` | `950.00` |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedLineTradeDelivery / BilledQuantity` | `1.0` with the attribute `unitCode` holding `C62` |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedLineTradeSettlement / ApplicableTradeTax / TypeCode` | `VAT`, the identifier of the value-added tax scheme |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedLineTradeSettlement / ApplicableTradeTax / CategoryCode` | `S` |
| `IncludedSupplyChainTradeLineItem[1] / SpecifiedLineTradeSettlement / ApplicableTradeTax / RateApplicablePercent` | `21.0` |
| `IncludedSupplyChainTradeLineItem[1] / ... / SpecifiedTradeSettlementLineMonetarySummation / LineTotalAmount` | `950.00` |
| `IncludedSupplyChainTradeLineItem[2] / AssociatedDocumentLineDocument / LineID` | `2` |
| `IncludedSupplyChainTradeLineItem[2] / SpecifiedTradeProduct / SellerAssignedID` | `FURN_8888` |
| `IncludedSupplyChainTradeLineItem[2] / SpecifiedLineTradeAgreement / GrossPriceProductTradePrice / ChargeAmount` | `500.00` |
| `IncludedSupplyChainTradeLineItem[2] / SpecifiedLineTradeAgreement / NetPriceProductTradePrice / ChargeAmount` | `500.00` |
| `IncludedSupplyChainTradeLineItem[2] / SpecifiedLineTradeDelivery / BilledQuantity` | `1.0` with the attribute `unitCode` holding `C62` |
| `IncludedSupplyChainTradeLineItem[2] / SpecifiedLineTradeSettlement / ApplicableTradeTax / CategoryCode` | `S` |
| `IncludedSupplyChainTradeLineItem[2] / SpecifiedLineTradeSettlement / ApplicableTradeTax / RateApplicablePercent` | `6.0` |
| `IncludedSupplyChainTradeLineItem[2] / ... / SpecifiedTradeSettlementLineMonetarySummation / LineTotalAmount` | `500.00` |
| `ApplicableHeaderTradeAgreement / BuyerReference` | the reference of the commercial contact of the customer |
| `ApplicableHeaderTradeAgreement / SellerTradeParty / Name` | the name of the company contact |
| `ApplicableHeaderTradeAgreement / BuyerTradeParty / Name` | the name of the customer |
| `ApplicableHeaderTradeAgreement / BuyerOrderReferencedDocument / IssuerAssignedID` | `INV/2024/00010` |
| `ApplicableHeaderTradeDelivery / ActualDeliverySupplyChainEvent / OccurrenceDateTime / DateTimeString` | `20240310` |
| `ApplicableHeaderTradeSettlement / InvoiceCurrencyCode` | `EUR`, the code of the euro |
| `ApplicableHeaderTradeSettlement / SpecifiedTradeSettlementPaymentMeans / TypeCode` | `42` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[1] / CalculatedAmount` | `199.50` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[1] / BasisAmount` | `950.00` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[1] / CategoryCode` | `S` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[1] / DueDateTypeCode` | `5` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[1] / RateApplicablePercent` | `21.0` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[2] / CalculatedAmount` | `30.00` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[2] / BasisAmount` | `500.00` |
| `ApplicableHeaderTradeSettlement / ApplicableTradeTax[2] / RateApplicablePercent` | `6.0` |
| `ApplicableHeaderTradeSettlement / BillingSpecifiedPeriod / StartDateTime / DateTimeString` | `20240310` |
| `ApplicableHeaderTradeSettlement / BillingSpecifiedPeriod / EndDateTime / DateTimeString` | `20240409` |
| `ApplicableHeaderTradeSettlement / SpecifiedTradePaymentTerms / DueDateDateTime / DateTimeString` | `20240409` |
| `ApplicableHeaderTradeSettlement / ... / LineTotalAmount` | `1450.00` |
| `ApplicableHeaderTradeSettlement / ... / TaxBasisTotalAmount` | `1450.00` |
| `ApplicableHeaderTradeSettlement / ... / TaxTotalAmount` | `229.50` with the attribute `currencyID` holding `EUR` |
| `ApplicableHeaderTradeSettlement / ... / GrandTotalAmount` | `1679.50` |
| `ApplicableHeaderTradeSettlement / ... / TotalPrepaidAmount` | `0.00` |
| `ApplicableHeaderTradeSettlement / ... / DuePayableAmount` | `1679.50` |

Compare with the same invoice in the universal business language family, in section 21 of [universal-business-language-mapping.md](universal-business-language-mapping.md): there the document level charge of fifty is reported and the totals are higher, because that example carries a global discount line that this family would report as an ordinary document line.
