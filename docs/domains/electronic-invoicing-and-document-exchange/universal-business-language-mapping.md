# Universal business language mapping

The complete element by element mapping between an accounting document and a structured file of the universal business language family, profile by profile: the layer stack, the document skeletons with their element order, every header, party, delivery, payment, allowance, charge, line, item, price, tax and total element, the value written into each one, the condition under which it is omitted, and the per profile overrides. The arithmetic behind the amounts is in [calculations.md](calculations.md); the validations that refuse a document before it is written are in [business-rules.md](business-rules.md); the reverse direction is in [import-mapping.md](import-mapping.md).

## Notation used in this file

1. **Element names are data values.** The names of the elements, the namespace identifiers and the code list values reproduced here belong to the published international standard, not to any implementation. They are quoted verbatim, in code style, because a replacement must emit them character for character. Their meaning is always given in full words in the neighbouring column. The one exception is the plain identifier element, which the standard spells with the two letters that abbreviate the word identifier. Because this specification writes no abbreviation anywhere, that element is written `Identifier` in every path of every mapping file; a replacement must emit the two letter spelling of the standard wherever `Identifier` appears as a whole path segment.
2. **Paths.** A path such as `Invoice / AccountingSupplierParty / Party / EndpointID` reads from the root element downwards. A path fragment written on its own, for example `Party / PartyName / Name`, is reused in several places and is defined once in its own section.
3. **Namespace assignment rule.** Every element that carries a text value belongs to the common basic components namespace, whose namespace identifier is `urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2`. Every element that only contains other elements belongs to the common aggregate components namespace, whose namespace identifier is `urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2`. The extension container belongs to the common extension components namespace, whose namespace identifier is `urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2`. The root element belongs to the namespace of its document type, listed in section 3. A replacement may bind any prefixes it likes to those three namespace identifiers as long as the bindings are declared on the root element.
4. **Omission rule.** An element whose value is empty is not written at all, and an aggregate element all of whose children are empty is not written either. The tables state "omitted when" only where the condition is more restrictive than that.
5. **Element order.** The order of the elements inside every parent is fixed by the standard and is reproduced in the skeleton tables of sections 4 and 5. The order in which the values are *computed* is different from the order in which they are *written*, and the computation order matters because later values read earlier ones; section 6 gives the computation order.
6. **Currency attribute.** Every monetary element carries an attribute named `currencyID` holding the three letter currency code of the amount. Quantity elements carry an attribute named `unitCode` holding the unit code of section 11 of [calculations.md](calculations.md). Identifier elements that belong to a published code list carry an attribute named `schemeID` holding the identifier of that list.
7. **Two builder generations.** Two different builders exist. The *layered builder* serves every profile derived from the European semantic standard, which is every profile except the plain 2.0 syntax, the plain 2.1 syntax and the Belgian profile. The *plain builder* serves those three. Sections 6 to 16 describe the layered builder; section 17 describes the plain builder in full.

---

# 1. The layer stack

The layered builder is assembled from five behaviour definitions. Each one may override any single node builder of the one below it, which is how a country profile is expressed as a handful of overrides rather than as a whole new document builder.

| Layer | Canonical name | What it contributes |
|---|---|---|
| 1 | Universal Business Language Base | The generic node builders: parties, addresses, lines, items, prices, allowances, charges, tax totals, monetary totals, and the grouping keys used to aggregate taxes. Writes no profile identifier and no document type code. |
| 2 | European Norm 16931 Layer | The compliance rules of the European semantic invoice standard: line level allowances and charges for discounts, recycling contribution taxes and excise taxes; removal of early payment, global discount and cash rounding lines from the line list; document level allowances and charges for early payment discounts and global discounts; suppression of the tax category for cash rounding lines, recycling contribution taxes and excise taxes; a default exemption reason for the exempt category; and the document validations of section 18.1. |
| 3 | Peppol International Billing Layer | The restrictions of the international billing model: exactly one note element, exactly one delivery element, withholding taxes reported as a prepaid amount instead of as a withholding tax total, the taxable amount of each tax subtotal recomputed from the presented line amounts, the percentage removed from the subtotal level, the electronic address of each party written as the endpoint, the country sub-entity code and the country name removed from every address, and the reduced delivery party. |
| 4 | Peppol International Billing European Union Layer | The European customization and profile identifiers, the intra-area delivery date rule, the Danish payment means code, the Netherlands credit note reference rule and the Netherlands document validations. |
| 5 | Concrete profile | The customization identifier, the file name, and the country specific overrides listed in section 16. |

The plain 2.1 builder inherits the plain 2.0 builder, and the European profile builder inherits both the plain 2.1 builder and layer 4. Where a method exists in both branches, the plain branch is consulted first and delegates to the layered branch, which is why the European profile uses the layered node builders for parties, lines, allowances, charges, tax totals and monetary totals while keeping the plain builder's outer procedure.

---

# 2. Profile catalogue

| Profile | Customization identifier written | Profile identifier written | Version identifier written | File name of the produced file |
|---|---|---|---|---|
| Peppol Billing 3.0 | `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0` for a customer document, `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:selfbilling:3.0` for a self billed document | `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0`, or `urn:fdc:peppol.eu:2017:poacc:selfbilling:01:1.0` for a self billed document | none | the document name with every solidus replaced by an underscore, followed by `_ubl_bis3` with the extension `xml` |
| German Electronic Invoice Profile | `urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0` | as above | none | the document name with every solidus replaced by an underscore, followed by `_xrechnung` with the extension `xml` |
| Netherlands Standard Invoice Profile | `urn:cen.eu:en16931:2017#compliant#urn:fdc:nen.nl:nlcius:v1.0` | as above | none | the document name with every solidus replaced by an underscore, followed by `_nlcius` with the extension `xml` |
| Australia and New Zealand Billing Profile | `urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:aunz:3.0` | as above | none | the document name with every solidus replaced by an underscore, followed by `_a_nz` with the extension `xml` |
| Singapore Billing Profile | `urn:cen.eu:en16931:2017#conformant#urn:fdc:peppol.eu:2017:poacc:billing:international:sg:3.0` | as above | none | the document name with every solidus replaced by an underscore, followed by `_sg` with the extension `xml` |
| Universal Business Language 2.0 | none | none | `2.0` | the document name with every solidus replaced by an underscore, followed by `_ubl_20` with the extension `xml` |
| Universal Business Language 2.1 | none | none | `2.1` | the document name with every solidus replaced by an underscore, followed by `_ubl_21` with the extension `xml` |
| Belgian Electronic Invoicing Profile | none | none | `2.0` | `efff_`, then the tax identification number of the commercial partner of the company when it has one, then an underscore when that number was present, then the document name with every character that is neither a letter nor a digit removed, then the extension `xml` |
| Sales Order Ordering Profile and Purchase Order Ordering Profile | `urn:fdc:peppol.eu:poacc:trns:order:3` | `urn:fdc:peppol.eu:poacc:bis:ordering:3` | none | see section 19 |

The self billing variant is only available on a profile that declares a self billing customization identifier. Only the Peppol Billing 3.0 profile does; every profile derived from it answers that it cannot export a self billed document, because its own customization identifier method returns nothing for the self billing process.

## 2.1 Document type codes

| Document | Type code element | Value for a document the company issues | Value for a self billed document |
|---|---|---|---|
| Invoice | `InvoiceTypeCode` | `380` | `389` |
| Credit note | `CreditNoteTypeCode` | `381` | `261` |
| Debit note | none in the layered builder, because a debit note is treated as an invoice | `380` | `389` |
| Order | `OrderTypeCode` | `220` for a sales order, `105` for a purchase order | not applicable |

A self billed document is a document that the customer produces on behalf of the seller. The layered builder recognises four document kinds: an invoice the company issues, a credit note the company issues, a self billed invoice and a self billed credit note. A vendor bill exported by this company is a self billed invoice, and a vendor credit note is a self billed credit note. In a self billed document the roles are swapped: the supplier is the trading partner and the customer is this company, and the delivery party is the first child contact of this company whose address kind is the delivery address, or this company itself when there is none.

---

# 3. Root elements and namespace identifiers

| Document kind | Root element | Namespace identifier of the root element |
|---|---|---|
| Invoice, including a self billed invoice and a debit note treated as an invoice | `Invoice` | the namespace identifier `urn:oasis:names:specification:ubl:schema:xsd:Invoice-2` |
| Credit note, including a self billed credit note | `CreditNote` | the namespace identifier `urn:oasis:names:specification:ubl:schema:xsd:CreditNote-2` |
| Debit note in the plain builder | `DebitNote` | the namespace identifier `urn:oasis:names:specification:ubl:schema:xsd:DebitNote-2` |
| Order | `Order` | the namespace identifier `urn:oasis:names:specification:ubl:schema:xsd:Order-2` |

The document is serialised with a declaration line stating the version and the character encoding, and the character encoding is always the eight bit unicode transformation format.

---

# 4. Document skeletons and element order

## 4.1 Invoice

In this exact order: `UBLExtensions`, `UBLVersionID`, `CustomizationID`, `ProfileID`, `ProfileExecutionID`, `Identifier`, `CopyIndicator`, `UUID`, `IssueDate`, `IssueTime`, `DueDate`, `InvoiceTypeCode`, `Note`, `TaxPointDate`, `DocumentCurrencyCode`, `TaxCurrencyCode`, `PricingCurrencyCode`, `AccountingCost`, `LineCountNumeric`, `BuyerReference`, `InvoicePeriod`, `OrderReference`, `BillingReference`, `DespatchDocumentReference`, `OriginatorDocumentReference`, `ContractDocumentReference`, `AdditionalDocumentReference`, `ProjectReference`, `Signature`, `AccountingSupplierParty`, `AccountingCustomerParty`, `SellerSupplierParty`, `Delivery`, `PaymentMeans`, `PaymentTerms`, `PrepaidPayment`, `AllowanceCharge`, `TaxExchangeRate`, `PricingExchangeRate`, `PaymentExchangeRate`, `TaxTotal`, `WithholdingTaxTotal`, `LegalMonetaryTotal`, `InvoiceLine`.

## 4.2 Credit note

In this exact order: `UBLExtensions`, `UBLVersionID`, `CustomizationID`, `ProfileID`, `ProfileExecutionID`, `Identifier`, `CopyIndicator`, `UUID`, `IssueDate`, `TaxPointDate`, `IssueTime`, `CreditNoteTypeCode`, `Note`, `DocumentCurrencyCode`, `TaxCurrencyCode`, `PricingCurrencyCode`, `AccountingCost`, `LineCountNumeric`, `BuyerReference`, `InvoicePeriod`, `DiscrepancyResponse`, `OrderReference`, `BillingReference`, `DespatchDocumentReference`, `ContractDocumentReference`, `AdditionalDocumentReference`, `OriginatorDocumentReference`, `Signature`, `AccountingSupplierParty`, `AccountingCustomerParty`, `SellerSupplierParty`, `PrepaidPayment`, `Delivery`, `PaymentMeans`, `PaymentTerms`, `TaxExchangeRate`, `PricingExchangeRate`, `PaymentExchangeRate`, `AllowanceCharge`, `TaxTotal`, `LegalMonetaryTotal`, `CreditNoteLine`.

Note two differences from the invoice: the issue time follows the tax point date instead of preceding the due date, and the prepaid payment group precedes the delivery group.

## 4.3 Debit note

In this exact order: `UBLExtensions`, `UBLVersionID`, `CustomizationID`, `ProfileID`, `ProfileExecutionID`, `Identifier`, `CopyIndicator`, `UUID`, `IssueDate`, `IssueTime`, `Note`, `DocumentCurrencyCode`, `TaxCurrencyCode`, `PricingCurrencyCode`, `LineCountNumeric`, `InvoicePeriod`, `DiscrepancyResponse`, `OrderReference`, `BillingReference`, `AdditionalDocumentReference`, `Signature`, `AccountingSupplierParty`, `AccountingCustomerParty`, `SellerSupplierParty`, `PrepaidPayment`, `AllowanceCharge`, `Delivery`, `PaymentMeans`, `PaymentTerms`, `TaxExchangeRate`, `PricingExchangeRate`, `PaymentExchangeRate`, `TaxTotal`, `RequestedMonetaryTotal`, `DebitNoteLine`.

A debit note has no document type code element, no due date and no buyer reference, and its monetary total element is named `RequestedMonetaryTotal` instead of `LegalMonetaryTotal`.

## 4.4 Order

In this exact order: `CustomizationID`, `ProfileID`, `Identifier`, `IssueDate`, `OrderTypeCode`, `Note`, `DocumentCurrencyCode`, `ValidityPeriod`, `QuotationDocumentReference`, `OriginatorDocumentReference`, `BuyerCustomerParty`, `SellerSupplierParty`, `Delivery`, `PaymentTerms`, `AllowanceCharge`, `TaxTotal`, `AnticipatedMonetaryTotal`, `OrderLine`.

## 4.5 Document line

| Position | Invoice line | Credit note line | Debit note line | Order line |
|---|---|---|---|---|
| 1 | `Identifier` | `Identifier` | `Identifier` | `OrderLine / LineItem / Identifier` |
| 2 | `UUID` | `UUID` | `UUID` | `UUID` |
| 3 | `Note` | `Note` | `Note` | `Note` |
| 4 | `InvoicedQuantity` | `CreditedQuantity` | `DebitedQuantity` | `Quantity` |
| 5 | `LineExtensionAmount` | `LineExtensionAmount` | `LineExtensionAmount` | `LineExtensionAmount` |
| 6 | `FreeOfChargeIndicator` | `FreeOfChargeIndicator` | `FreeOfChargeIndicator` | `TotalTaxAmount` |
| 7 | `InvoicePeriod` | `InvoicePeriod` | `InvoicePeriod` | `AllowanceCharge` |
| 8 | `OrderLineReference` | `OrderLineReference` | `OrderLineReference` | `Price` |
| 9 | `BillingReference` | `BillingReference` | `BillingReference` | `Item` |
| 10 | `DocumentReference` | `DocumentReference` | `DocumentReference` | `TaxTotal` |
| 11 | `PricingReference` | `PricingReference` | `PricingReference` | `ItemPriceExtension` |
| 12 | `PaymentTerms` | `PaymentTerms` | `PaymentTerms` | |
| 13 | `AllowanceCharge` | `TaxTotal` | `TaxTotal` | |
| 14 | `TaxTotal` | `AllowanceCharge` | `AllowanceCharge` | |
| 15 | `WithholdingTaxTotal` | `Item` | `Item` | |
| 16 | `Item` | `Price` | `Price` | |
| 17 | `Price` | `ItemPriceExtension` | `ItemPriceExtension` | |
| 18 | `ItemPriceExtension` | | | |

The allowance and charge group precedes the tax total group on an invoice line and follows it on a credit note line and on a debit note line. A replacement must respect that inversion or the file will fail schema validation.

## 4.6 Reusable groups

**Address** in this order: `Identifier`, `AddressTypeCode`, `AddressFormatCode`, `StreetName`, `AdditionalStreetName`, `BuildingName`, `BuildingNumber`, `PlotIdentification`, `CitySubdivisionName`, `CityName`, `PostalZone`, `CountrySubentity`, `CountrySubentityCode`, `District`, `AddressLine / Line`, `Country / IdentificationCode`, `Country / Name`.

**Party** in this order: `EndpointID`, `IndustryClassificationCode`, `PartyIdentification / Identifier`, `PartyName / Name`, `PhysicalLocation / Address`, `PostalAddress`, `PartyTaxScheme`, `PartyLegalEntity`, `Contact`, `Person`, `PowerOfAttorney`.

**PartyTaxScheme** in this order: `RegistrationName`, `CompanyID`, `TaxLevelCode`, `RegistrationAddress`, `TaxScheme / Identifier`, `TaxScheme / Name`.

**PartyLegalEntity** in this order: `RegistrationName`, `CompanyID`, `RegistrationAddress`, `CorporateRegistrationScheme / Identifier`, `CorporateRegistrationScheme / Name`.

**Contact** in this order: `Identifier`, `Name`, `Telephone`, `ElectronicMail`.

**Supplier party wrapper** in this order: `CustomerAssignedAccountID`, `AdditionalAccountID`, `Party`, `AccountingContact`, `SellerContact`.

**Customer party wrapper** in this order: `AdditionalAccountID`, `Party`, `AccountingContact`.

**Delivery** in this order: `Identifier`, `ActualDeliveryDate`, `DeliveryLocation / Identifier`, `DeliveryLocation / Address`, `DeliveryParty`.

**Financial account** in this order: `Identifier`, `FinancialInstitutionBranch / Identifier`, `FinancialInstitutionBranch / FinancialInstitution / Identifier`, `FinancialInstitutionBranch / FinancialInstitution / Name`, `FinancialInstitutionBranch / FinancialInstitution / Address`.

**PaymentMeans** in this order: `Identifier`, `PaymentMeansCode`, `PaymentDueDate`, `InstructionID`, `InstructionNote`, `PaymentID`, `PayeeFinancialAccount`.

**PaymentTerms** in this order: `Identifier`, `PaymentMeansID`, `Note`, `PaymentPercent`, `Amount`, `PaymentDueDate`, `SettlementPeriod`.

**AllowanceCharge** in this order: `Identifier`, `ChargeIndicator`, `AllowanceChargeReasonCode`, `AllowanceChargeReason`, `MultiplierFactorNumeric`, `Amount`, `BaseAmount`, `TaxCategory`.

**TaxCategory** in this order: `Identifier`, `Name`, `Percent`, `TaxExemptionReasonCode`, `TaxExemptionReason`, `TierRange`, `TaxScheme / Identifier`, `TaxScheme / Name`, `TaxScheme / TaxTypeCode`.

**TaxTotal** in this order: `TaxAmount`, `RoundingAmount`, then one `TaxSubtotal` per subtotal, each one holding `TaxableAmount`, `TaxAmount`, `BaseUnitMeasure`, `PerUnitAmount`, `Percent`, `TaxCategory`.

**Monetary total** in this order: `LineExtensionAmount`, `TaxExclusiveAmount`, `TaxInclusiveAmount`, `AllowanceTotalAmount`, `ChargeTotalAmount`, `PrepaidAmount`, `PayableRoundingAmount`, `PayableAmount`.

**Item** in this order: `Description`, `Name`, `BrandName`, `ModelName`, `BuyersItemIdentification`, `SellersItemIdentification`, `StandardItemIdentification`, `CommodityClassification / ItemClassificationCode`, `ClassifiedTaxCategory`, `AdditionalItemProperty / Name`, `AdditionalItemProperty / Value`, `InformationContentProviderParty`. Each of the three item identification groups holds `Identifier` then `ExtendedID`.

**Price** in this order: `PriceAmount`, `BaseQuantity`, `AllowanceCharge`.

**Period** in this order: `StartDate`, `EndDate`, `DescriptionCode`, `Description`.

**Document reference** in this order: `Identifier`, `UUID`, `IssueDate`, `IssueTime`, `DocumentTypeCode`, `DocumentStatusCode`, `DocumentType`, `DocumentDescription`, `Attachment / EmbeddedDocumentBinaryObject`, `Attachment / ExternalReference / URI`, `Attachment / ExternalReference / MimeCode`, `Attachment / ExternalReference / EncodingCode`, `Attachment / ExternalReference / FileName`, `Attachment / ExternalReference / Description`, `IssuerParty / PartyIdentification / Identifier`.

**Billing reference** holds exactly one `InvoiceDocumentReference`, which is a document reference group.

---

# 5. Preparation of the export values

Before any element is written, the builder assembles a working set of values from the accounting document. The steps run in this exact order.

1. **Validate the tax structure.** Every tax used on a line of the document is checked for a valid repartition structure. A failure refuses the export with `Tax '<tax name>' is invalid: <the repartition error message>`.
2. **Decide the document kind** from the document type of the accounting document: a customer invoice gives an invoice, a customer credit note gives a credit note, a vendor bill gives a self billed invoice, a vendor credit note gives a self billed credit note. A debit note is treated as an invoice.
3. **Resolve the parties.** For a document the company issues: the supplier is the contact of the company and the customer is the partner of the document; the delivery party is the shipping address of the document, falling back to the customer. For a self billed document: the supplier is the partner of the document and the customer is the contact of the company; the delivery party is the first child contact of the customer whose address kind is the delivery address, falling back to the customer. When the company is a branch that shares the participant identification of a parent company, the supplier is replaced by the commercial contact of that parent company, so that the file carries the identity under which the group is registered on the exchange network.
4. **Resolve the currencies**: the document currency and the company currency.
5. **Build the base lines** of the document, with their tax details, using the tax engine of the [taxes](../taxes/calculations.md) domain.
6. **Turn every negative unit price positive** by negating both the quantity and the unit price of that line, because the standard forbids a negative item net price and a negative item gross price.
7. **Turn the emptying taxes into extra lines.** A fixed amount tax or a coded tax that does not increase the base of the following taxes is removed from its line and reported as an extra document line carrying the tax name as its label, the summed quantity of the lines it was removed from, and the tax amount per unit as its unit price. The discount of the lines from which the tax was removed is set to zero for that computation, because a fixed tax is not affected by a discount.
8. **Apply the six digit global rounding** to the raw amounts of every base line, in the document currency and in the company currency, in this order: round the raw total excluding tax; add and round the raw gross total excluding tax and the raw discount amount; round the raw gross total excluding tax and the raw discount amount. This is what makes the sum of the presented line amounts equal the presented document total.

The details of steps 5 to 8 and of every amount they produce are in section 2 of [calculations.md](calculations.md).

---

# 6. Computation order of the document nodes

The values are computed in this order. Later steps read the elements written by earlier steps, which is why the order is part of the specification.

1. Header elements (section 7).
2. Supplier party (section 8).
3. Customer party (section 8).
4. Delivery (section 9).
5. Payment means (section 10.1).
6. Payment terms (section 10.2).
7. Document lines, in the order of the base lines, numbered from one (sections 12 to 14). Within one line: the identifier, the note list, the quantity, the allowance and charge list, the line net amount, the pricing reference, the line tax total list, the item, the price.
8. Document level allowances and charges (section 11).
9. Tax totals and withholding tax totals (section 15).
10. Monetary total (section 16), whose six steps run in the order given there.
11. The optional extension elements of section 20.

Steps 4 to 6 are computed before the lines for an invoice and after the lines for a credit note and a debit note, because the plain builder that hosts the outer procedure only produces them unconditionally for an invoice and repeats them for the other two kinds. The written order is always the one of section 4, so this difference is not visible in the produced file.

---

# 7. Header elements

| Path | Value written | Omitted when |
|---|---|---|
| `UBLVersionID` | nothing in the layered builder | always in the layered builder |
| `CustomizationID` | the customization identifier of the profile, from section 2; for the Peppol Billing 3.0 profile and every profile that does not override it, the self billing identifier is written instead when the document kind is a self billed invoice or a self billed credit note | never in a profile that declares one |
| `ProfileID` | `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0` for a document the company issues, `urn:fdc:peppol.eu:2017:poacc:selfbilling:01:1.0` for a self billed document | in a profile that does not derive from the European union layer |
| `Identifier` | the name of the accounting document, which is its sequence number; this is the identifier of the invoice | never |
| `CopyIndicator` | nothing | always |
| `IssueDate` | the invoice date of the accounting document | when the document has no invoice date |
| `IssueTime` | nothing | always |
| `DueDate` | the due date of the accounting document; written only for an invoice | for a credit note and a debit note, and when there is no due date |
| `InvoiceTypeCode` | `380` for an invoice the company issues, `389` for a self billed invoice | for a credit note |
| `CreditNoteTypeCode` | `381` for a credit note the company issues, `261` for a self billed credit note | for an invoice |
| `Note` | one single text made of the sentences of section 7.1 joined by single spaces | when there is no sentence to write |
| `DocumentCurrencyCode` | the three letter code of the document currency | never |
| `TaxCurrencyCode` | the three letter code of the company currency, and only when it differs from the document currency | when the two currencies are the same, and always in the German, Netherlands, Australia and New Zealand and Singapore profiles |
| `BuyerReference` | the reference of the customer contact, falling back to the reference of its commercial contact | when neither reference is set, except in the German profile where the value `N/A` is written instead |
| `InvoicePeriod` | nothing | always |
| `OrderReference / Identifier` | the customer reference field of the accounting document, falling back to the document name; this is the identifier of the purchase order of the buyer | never, because of the fallback |
| `OrderReference / SalesOrderID` | the names of the distinct sales orders linked to the lines of the document, joined by commas; this is the identifier of the sales order of the seller | when the sales capability package is not installed, or no line is linked to a sales order |
| `BillingReference / InvoiceDocumentReference / Identifier` | for a credit note, one element per preceding invoice: the names of the accounting documents reconciled against the receivable lines of the credit note, excluding a name equal to a single solidus | for an invoice, and when the credit note is reconciled against nothing |

## 7.1 The note

The note is a single element because the international billing layer allows at most one. Its sentences, in order:

1. When the document carries withholding taxes, the sentence `The prepaid amount of <amount> corresponds to the withholding tax applied.` where the amount is the total withholding amount formatted in the document currency with its symbol, and with the non breaking space removed.
2. The terms and conditions of the accounting document, converted from rich text to plain text, when they are not empty.

## 7.2 The Netherlands credit note reference

When the supplier country is the Netherlands, the document is a credit note the company issues, the accounting document has a customer reference and no billing reference has been produced, one billing reference is added whose invoice document reference identifier is that customer reference.

---

# 8. Party elements

Two party groups are always written: the accounting supplier party and the accounting customer party, each holding one party element. The seller supplier party and the buyer customer party groups are written only by the ordering profiles.

## 8.1 Endpoint

| Path | Value written |
|---|---|
| `Party / EndpointID` | the participant endpoint of the commercial contact of the party, with the attribute `schemeID` holding its electronic address scheme code, and only when both are set |

Overrides:

- **Australia and New Zealand profile.** When the commercial contact is in Australia and has a tax identification number, the endpoint text is that number with every space removed. When the commercial contact is in New Zealand and has a company registry number, the endpoint text is that registry number. The scheme attribute keeps the value set by the international billing layer.
- **German profile.** When the endpoint is still empty and the contact has an electronic mail address, the endpoint text is that address and the scheme attribute is `EM`.

The catalogue of electronic address scheme codes per country is in section 3 of [peppol-network.md](peppol-network.md).

## 8.2 Party identification

The list is built in this order and the first rule that produces an entry wins for the second rule only.

1. When the commercial contact is in Belgium and has a company registry number, one entry whose identifier is that registry number with every separator removed and whose scheme attribute is `0208`.
2. When the list is still empty, the commercial contact has a reference and its country is not Denmark, one entry whose identifier is that reference and whose scheme attribute is not written.

Overrides:

- **Netherlands profile, supplier side.** After the two rules above, one further entry is appended holding the participant endpoint of the commercial contact, with no scheme attribute, whenever that endpoint is set.
- **Netherlands profile, customer side.** When the commercial contact is in the Netherlands and has a participant endpoint, the whole list is replaced by a single entry holding that endpoint, with the scheme attribute set to the electronic address scheme code of the contact when that code is `0106` or `0190`, and not written otherwise.

## 8.3 Party name

| Path | Value written |
|---|---|
| `Party / PartyName / Name` | the display name of the party contact when that contact has a name of its own, otherwise the display name of its commercial contact |

The fallback exists because an invoicing address or a child contact may have no name, and the standard requires a party name.

## 8.4 Postal address

| Path | Value written |
|---|---|
| `PostalAddress / StreetName` | the street of the contact |
| `PostalAddress / AdditionalStreetName` | the second street line of the contact |
| `PostalAddress / CityName` | the city of the contact |
| `PostalAddress / PostalZone` | the postal code of the contact |
| `PostalAddress / CountrySubentity` | the name of the state of the contact |
| `PostalAddress / CountrySubentityCode` | the code of the state of the contact; removed by the international billing layer, so it is never written by a profile derived from it |
| `PostalAddress / Country / IdentificationCode` | the two letter code of the country of the contact |
| `PostalAddress / Country / Name` | the name of the country of the contact; removed by the international billing layer |

## 8.5 Party tax scheme

The list is built in this order.

1. When the document contains any tax whose configured category code is `O`, meaning a service outside the scope of tax, no party tax scheme is written at all, for either party.
2. Otherwise, when the commercial contact has a country and a tax identification number that is not a single solidus, one entry is written whose `CompanyID` is that number and whose `TaxScheme / Identifier` holds the identifier `GST` when the country of the commercial contact belongs to the goods and services tax area listed in section 8.7, and the identifier `VAT` otherwise. Before it is written, the number is corrected: in Hungary a number that does not already start with `HU` is written as `HU` followed by its first eight characters; in Denmark a number that does not already start with `DK` is written as `DK` followed by the whole number.
3. When the list is still empty and the commercial contact has both a participant endpoint and an electronic address scheme code, one entry is written whose `CompanyID` is the endpoint and whose `TaxScheme / Identifier` is the scheme code.
4. On the supplier side only, when the supplier country is Norway, one further entry is appended whose `CompanyID` is the literal text `Foretaksregisteret` and whose `TaxScheme / Identifier` is `TAX`, because a Norwegian issuer must state that it is entered in the register of business enterprises. When the supplier country is Sweden, one further entry is appended whose `CompanyID` is the literal text `GODKÄND FÖR F-SKATT` and whose `TaxScheme / Identifier` is `TAX`, because a Swedish issuer must state that it is approved for the business tax regime.

Overrides:

- **German profile.** When the list is still empty and the commercial contact has an electronic address scheme code, one entry is appended whose `CompanyID` is empty and whose `TaxScheme / Identifier` is that scheme code.
- **Australia and New Zealand profile.** When the commercial contact is in Australia and has a tax identification number, or is in New Zealand and has a company registry number, the whole list is replaced by a single entry whose `CompanyID` is the endpoint value computed in section 8.1 and whose `TaxScheme / Identifier` is `GST`.

## 8.6 Party legal entity

The list is built by the first matching branch of this ordered decision. In every branch the `RegistrationName` is the name of the commercial contact.

| Order | Condition | `CompanyID` written | `schemeID` attribute |
|---|---|---|---|
| 1 | country is the Netherlands and a Netherlands legal number is available | the participant endpoint when the electronic address scheme code is `0106` or `0190`, otherwise the company registry number | `0190` when the number is exactly twenty characters long, `0106` otherwise |
| 2 | country is Luxembourg and a company registry number is set | the company registry number | not written |
| 3 | country is Sweden and a company registry number is set | the company registry number with every non digit character removed | not written |
| 4 | country is Belgium and a company registry number is set | the company registry number with every separator removed | `0208` |
| 5 | country is Denmark, the electronic address scheme code is `0184` and a participant endpoint is set | the participant endpoint | `0184` |
| 6 | country is Australia and a tax identification number that is not a single solidus is set | the tax identification number | `0151` |
| 7 | country is New Zealand and a tax identification number that is not a single solidus is set | the tax identification number | `0088` |
| 8 | a tax identification number that is not a single solidus is set | the tax identification number | not written |
| 9 | a participant endpoint is set | the participant endpoint | not written |

Overrides:

- **German profile.** When the list is still empty and the commercial contact has a name, one entry is appended holding only the registration name.
- **Australia and New Zealand profile.** When the commercial contact is in Australia and has a tax identification number, the whole list is replaced by a single entry whose `CompanyID` is the endpoint value of section 8.1 with the scheme attribute `0151`. When it is in New Zealand and has a company registry number, the whole list is replaced by a single entry whose `CompanyID` is the endpoint value with the scheme attribute `0088`.

The Netherlands rule exists because the tax identification number may serve as the network endpoint there while the legal entity must be identified by the chamber of commerce number or by the public body number, and the two are distinguished by their length.

## 8.7 The goods and services tax area

A commercial contact whose country belongs to this set uses the goods and services tax scheme identifier `GST` instead of the value-added tax scheme identifier `VAT`: Australia, New Zealand, India, Singapore, Malaysia, Pakistan, Bangladesh, Sri Lanka, Nepal, Bhutan, Papua New Guinea, Saudi Arabia, Antigua and Barbuda, Bahamas, Barbados, Dominica, Grenada, Jamaica, Saint Kitts and Nevis, Saint Lucia, Saint Vincent and the Grenadines, Trinidad and Tobago.

## 8.8 Contact

| Path | Value written |
|---|---|
| `Party / Contact / Identifier` | nothing |
| `Party / Contact / Name` | the name of the party contact |
| `Party / Contact / Telephone` | the telephone number of the party contact |
| `Party / Contact / ElectronicMail` | the electronic mail address of the party contact |

---

# 9. Delivery

The international billing layer allows at most one delivery group, so the list produced by the base layer is reduced to its first entry, and to nothing when the list is empty. The group is produced only when a delivery party was resolved.

| Path | Value written | Omitted when |
|---|---|---|
| `Delivery / ActualDeliveryDate` | the delivery date of the accounting document | the accounting document has no delivery date, except under the intra-area rule below |
| `Delivery / DeliveryLocation / Identifier` | the global location number of the delivery party, with the scheme attribute `0088` | the global location number capability package is not installed, or the delivery party has none |
| `Delivery / DeliveryLocation / Address` | the postal address group of section 8.4 built from the delivery party | never, when a delivery party exists |
| `Delivery / DeliveryParty / PartyName / Name` | the party name of section 8.3 built from the delivery party | never, when a delivery party exists |

The international billing layer suppresses the endpoint, the party identification, the postal address, the party tax scheme, the party legal entity and the contact of the delivery party, so the delivery party carries only its name.

**Intra-area delivery date rule.** When the customer country and the supplier country both belong to the European economic area and they differ, the actual delivery date is forced to the invoice date of the accounting document. This satisfies the semantic rule that an intra-community supply must carry either an actual delivery date or an invoicing period.

**The European economic area** is: Austria, Belgium, Bulgaria, Croatia, Cyprus, Czechia, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, the Netherlands, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden, Iceland, Liechtenstein and Norway.

---

# 10. Payment

## 10.1 Payment means

One payment means group is produced for an invoice, a credit note, a self billed invoice and a self billed credit note.

| Path | Value written |
|---|---|
| `PaymentMeans / PaymentMeansCode` | `30` with the attribute `name` holding `credit transfer` when the accounting document is a customer invoice and carries a recipient bank account; `ZZZ` with the attribute `name` holding `mutually defined` when it is a customer invoice without a recipient bank account; `57` with the attribute `name` holding `standing agreement` in every other case |
| `PaymentMeans / PaymentID` | the payment reference of the accounting document, falling back to its name |
| `PaymentMeans / PayeeFinancialAccount / Identifier` | the sanitised account number of the recipient bank account; omitted with the whole group when there is no recipient bank account |
| `PaymentMeans / PayeeFinancialAccount / FinancialInstitutionBranch / Identifier` | the bank identifier code of the bank of that account, with the scheme attribute removed by the international billing layer; omitted when the account has no bank |

The international billing layer removes the financial institution sub-group entirely, so the bank name and the bank address are never written by a profile derived from it.

Overrides:

- **European union layer.** When the commercial contact of the customer is in Denmark, the payment means code becomes `1` with the attribute `name` holding `unknown`, because the credit transfer code is not accepted there and the true means cannot be deduced from the accounting document.
- **Netherlands profile.** The attribute `name` and the attribute `listID` of the payment means code are cleared, because a payment means text is not recommended there.
- **Singapore profile.** The payment means code of the first group is replaced by `54` with the attribute `name` holding `Credit Card`.

## 10.2 Payment terms

| Path | Value written | Omitted when |
|---|---|---|
| `PaymentTerms / Note` | the note of the payment terms of the accounting document, converted from rich text to plain text | the document has no payment terms, or their note is empty |

---

# 11. Document level allowances and charges

Two kinds of document level allowance or charge are produced, in this order. Both are produced by grouping the base lines of the corresponding special kind with the tax grouping key of section 5 of [calculations.md](calculations.md).

## 11.1 Early payment discount

One group per tax grouping key of the early payment lines.

| Path | Value written |
|---|---|
| `AllowanceCharge / ChargeIndicator` | `true` when the grouped total excluding tax is greater than zero, `false` otherwise |
| `AllowanceCharge / AllowanceChargeReasonCode` | `ZZZ` when it is a charge, `64` when it is an allowance |
| `AllowanceCharge / AllowanceChargeReason` | `Conditional cash/payment discount` |
| `AllowanceCharge / Amount` | the absolute value of the grouped total excluding tax, rounded to the currency |
| `AllowanceCharge / TaxCategory / Identifier` | the tax category code of the grouping key |
| `AllowanceCharge / TaxCategory / Percent` | the percentage of the grouping key |
| `AllowanceCharge / TaxCategory / TaxScheme / Identifier` | the tax scheme code of the grouping key |

The exemption reason is deliberately removed from the grouping key of an early payment allowance whose category is the exempt category, so that the positive and the negative halves of a mixed early payment are not merged into one group.

## 11.2 Global discount

One group per tax grouping key of the global discount lines.

| Path | Value written |
|---|---|
| `AllowanceCharge / ChargeIndicator` | `true` when the grouped total excluding tax is greater than zero, `false` otherwise |
| `AllowanceCharge / AllowanceChargeReasonCode` | `ADK` when it is a charge, `95` when it is an allowance |
| `AllowanceCharge / AllowanceChargeReason` | `General upsell` when it is a charge, `General discount` when it is an allowance |
| `AllowanceCharge / Amount` | the absolute value of the grouped total excluding tax, rounded to the currency |
| `AllowanceCharge / TaxCategory / Identifier` | the tax category code of the grouping key |
| `AllowanceCharge / TaxCategory / Percent` | the percentage of the grouping key |
| `AllowanceCharge / TaxCategory / TaxScheme / Identifier` | the tax scheme code of the grouping key |

Neither kind is produced when its grouping key is a withholding key.

---

# 12. Document lines

## 12.1 Which base lines become document lines

A base line becomes a document line unless it is one of the following, which are reported elsewhere:

| Kind of base line | Where it is reported instead |
|---|---|
| an early payment discount line | a document level allowance or charge, section 11.1 |
| a global discount line | a document level allowance or charge, section 11.2 |
| a cash rounding line | the payable rounding amount of the monetary total, section 16 |

Every other base line becomes a document line, including the extra lines created in step 7 of section 5 from the emptying taxes. The lines are numbered from one in the order in which they appear, and the numbering counts only the lines that are written.

## 12.2 Line elements

| Path | Value written | Omitted when |
|---|---|---|
| `<line> / Identifier` | the position of the line, starting at one; this is the identifier of the line | never |
| `<line> / Note` | nothing | always |
| `<line> / InvoicedQuantity`, `<line> / CreditedQuantity`, `<line> / DebitedQuantity` | the quantity of the base line, with the attribute `unitCode` holding the unit code of its unit of measure | never |
| `<line> / AllowanceCharge` | the list of section 12.3 | when the list is empty |
| `<line> / LineExtensionAmount` | the line net amount of section 3.3 of [calculations.md](calculations.md), with the attribute `currencyID` | never |
| `<line> / PricingReference` | nothing | always |
| `<line> / TaxTotal` | nothing at line level in the layered builder | always |
| `<line> / Item` | the item group of section 13 | never |
| `<line> / Price` | the price group of section 14 | never |

## 12.3 Line level allowances and charges

Three kinds of group are produced on a line, in this order: the discount, then one group per recycling contribution tax, then one group per excise tax. The exact amounts and conditions are in section 3.2 of [calculations.md](calculations.md). The elements are:

| Path | Discount group | Recycling contribution group | Excise group |
|---|---|---|---|
| `AllowanceCharge / ChargeIndicator` | `false` for a positive discount, `true` for a negative discount | `true` when the tax amount is positive, `false` otherwise | `true` when the tax amount is positive, `false` otherwise |
| `AllowanceCharge / AllowanceChargeReasonCode` | `95` when it is an allowance, `ADK` when it is a charge | `CAV` when the lower case tax name contains the text `bebat` and it is a charge, `AEO` when it is another charge, `100` when it is an allowance | not written |
| `AllowanceCharge / AllowanceChargeReason` | `Discount` | the name of the tax | the name of the tax |
| `AllowanceCharge / MultiplierFactorNumeric` | the absolute value of the discount percentage of the line | not written | not written |
| `AllowanceCharge / Amount` | the absolute value of the discount amount, with the attribute `currencyID` | the absolute value of the tax amount, with the attribute `currencyID` | the absolute value of the tax amount, with the attribute `currencyID` |
| `AllowanceCharge / BaseAmount` | the absolute value of the gross total of the line excluding tax, with the attribute `currencyID` | not written | not written |

Overrides that reduce the discount group:

| Profile | Elements cleared from the discount group |
|---|---|
| Netherlands Standard Invoice Profile | the reason code, the multiplier factor and the base amount |
| Australia and New Zealand Billing Profile | the reason, the multiplier factor and the base amount |
| Singapore Billing Profile | the reason, the multiplier factor and the base amount |
| German Electronic Invoice Profile | the reason, the multiplier factor and the base amount |
| Sales Order Ordering Profile and Purchase Order Ordering Profile | the reason, the multiplier factor and the base amount |

---

# 13. The item group

| Path | Value written | Omitted when |
|---|---|---|
| `Item / Description` | the label of the line with the display name of the product removed and the result trimmed, when the line carries a product; nothing otherwise | the result is empty |
| `Item / Name` | the display name of the product of the line; when the line carries no product but was created from an emptying tax, the name of that tax; when the line carries neither, the label of the line | the result is empty |
| `Item / SellersItemIdentification / Identifier` | the internal reference of the product; this is the seller item identifier | the product has no internal reference |
| `Item / StandardItemIdentification / Identifier` | the barcode of the product, with the scheme attribute `0160`, which is the code of the global trade item number list | the product has no barcode |
| `Item / CommodityClassification / ItemClassificationCode` | one element per applicable classification, see section 13.1 | no classification applies |
| `Item / ClassifiedTaxCategory` | one group per tax category of the line, see section 13.2 | the line has no tax category |
| `Item / AdditionalItemProperty / Name` and `Item / AdditionalItemProperty / Value` | one pair per variant attribute value of the product: the attribute name and the value name | the product has no variant attribute value |

The description is produced by removing the product display name from the label of the line because the label usually begins with the product name, and repeating it in both elements is redundant.

## 13.1 Commodity classification

The classifications are appended in this fixed order, and each one is written only when the corresponding capability package is installed and the product carries the code.

| Order | Source code on the product | `listID` attribute |
|---|---|---|
| 1 | the intra-community trade statistics commodity code | `HS` |
| 2 | the standard products and services classification code | `TST` |
| 3 | the common procurement vocabulary code | `STI` |
| 4 | the national product classification code of the local profile that defines it, taken from the line when the line carries one and from the product otherwise | `CG` |

The attribute holding the list version is present but never given a value.

## 13.2 Classified tax category

One group per tax category of the line, obtained by aggregating the tax details of that line with the tax category grouping key. Groups whose key is a withholding key are not written.

| Path | Value written |
|---|---|
| `ClassifiedTaxCategory / Identifier` | the tax category code of the key |
| `ClassifiedTaxCategory / Name` | nothing |
| `ClassifiedTaxCategory / Percent` | the percentage of the key; nothing when the category code is `O` |
| `ClassifiedTaxCategory / TaxExemptionReasonCode` | nothing at line level |
| `ClassifiedTaxCategory / TaxExemptionReason` | nothing at line level |
| `ClassifiedTaxCategory / TaxScheme / Identifier` | the tax scheme identifier of the key, `VAT` or `GST` |

The European semantic layer refuses a document in which a line has zero tax categories or more than one, and refuses a document that mixes the category `O` with any other category. See section 18.

---

# 14. The price group

| Path | Value written |
|---|---|
| `Price / PriceAmount` | the gross unit price of the line excluding every tax, in the chosen currency, written with at least one and at most ten decimal places, with the attribute `currencyID` |

The gross unit price is the price before the line discount, because the discount is reported separately as a line allowance. Its derivation is in section 3.4 of [calculations.md](calculations.md). The base quantity element and the price level allowance element are not written by the layered builder.

---

# 15. Tax totals

Two lists are produced: the tax total list and the withholding tax total list. The grouping is three levels deep and every level is computed independently from the base lines, so that a level can be suppressed by a profile without disturbing the others. The three grouping keys and the aggregation are specified in section 5 of [calculations.md](calculations.md).

| Level | Key |
|---|---|
| tax total | whether the group is a withholding group, and the currency |
| tax subtotal | the currency, whether it is a withholding group, the tax category code, the tax scheme code and the percentage |
| tax category inside the subtotal | everything above plus the exemption reason code and the exemption reason text |

A group whose key says it is a withholding group goes into the withholding tax total list and its amounts are negated; every other group goes into the tax total list.

When the document currency differs from the company currency, the whole computation is run twice, once per currency, and both results are written, so the file carries one tax total in the document currency and one in the company currency.

| Path | Value written |
|---|---|
| `TaxTotal / TaxAmount` | the aggregated tax amount of the group, with the attribute `currencyID` |
| `TaxTotal / TaxSubtotal / TaxableAmount` | the aggregated base amount of the subtotal, with the attribute `currencyID`; replaced by the recomputation of section 15.1 by the international billing layer |
| `TaxTotal / TaxSubtotal / TaxAmount` | the aggregated tax amount of the subtotal, with the attribute `currencyID` |
| `TaxTotal / TaxSubtotal / Percent` | the percentage of the subtotal key; cleared by the international billing layer |
| `TaxTotal / TaxSubtotal / TaxCategory / Identifier` | the tax category code |
| `TaxTotal / TaxSubtotal / TaxCategory / Name` | nothing |
| `TaxTotal / TaxSubtotal / TaxCategory / Percent` | the percentage of the key; nothing when the category code is `O` |
| `TaxTotal / TaxSubtotal / TaxCategory / TaxExemptionReasonCode` | the exemption reason code of the key |
| `TaxTotal / TaxSubtotal / TaxCategory / TaxExemptionReason` | the exemption reason text of the key |
| `TaxTotal / TaxSubtotal / TaxCategory / TaxScheme / Identifier` | the tax scheme code |

When neither list produced any group, one empty tax total is written holding a tax amount of zero in the chosen currency and no subtotal, because the standard requires at least one tax total.

## 15.1 Overrides of the tax total grouping

| Layer or profile | Override |
|---|---|
| European semantic layer | a subtotal whose category is the exempt category and that has no exemption reason text receives the default text `Exempt from tax` |
| International billing layer | the withholding tax total list is suppressed entirely; the withholding amounts are reported as a prepaid amount instead |
| International billing layer | in a multi currency document, the subtotal level of the group expressed in the company currency is suppressed, so the company currency tax total carries only its total amount |
| International billing layer | the taxable amount of every subtotal is recomputed from the presented amounts, as described in section 8 of [calculations.md](calculations.md) |
| International billing layer | the percentage at subtotal level is cleared |
| German, Netherlands, Australia and New Zealand, Singapore profiles | the whole tax total of the group expressed in the company currency is suppressed in a multi currency document, so only the document currency tax total is written |
| Netherlands profile | the exemption reason code is cleared from every key |
| Singapore profile | both the exemption reason code and the exemption reason text are cleared from every key, and the category code becomes `ZR` when the line has no tax or a zero rate tax and `SR` otherwise |

---

# 16. Monetary total

The element is named `LegalMonetaryTotal` on an invoice and on a credit note, `RequestedMonetaryTotal` on a debit note and `AnticipatedMonetaryTotal` on an order. Its six computation steps and their formulas are in section 7 of [calculations.md](calculations.md); the elements are:

| Path | Value written | Omitted when |
|---|---|---|
| `LineExtensionAmount` | the sum of the line net amounts of every document line | never |
| `TaxExclusiveAmount` | the line extension amount plus every document charge minus every document allowance | never |
| `TaxInclusiveAmount` | the tax exclusive amount plus the tax amounts of the tax totals of the chosen currency minus the tax amounts of the withholding tax totals of the same currency | never |
| `AllowanceTotalAmount` | the sum of the amounts of the document level allowances | the sum is zero |
| `ChargeTotalAmount` | the sum of the amounts of the document level charges | the sum is zero |
| `PrepaidAmount` | the total of the accounting document minus its residual amount, increased by the withholding total under the international billing layer | never; it is written as zero when there is nothing prepaid |
| `PayableRoundingAmount` | the difference between the true document total and the tax inclusive amount just written, reduced by the withholding total under the international billing layer | the difference is zero in the currency |
| `PayableAmount` | the residual amount of the accounting document | never |

When the amounts are expressed in the company currency, the prepaid amount and the payable amount are read from the signed company currency totals of the accounting document instead of from the document currency totals.

---

# 17. The plain builder

The plain 2.0 syntax, the plain 2.1 syntax and the Belgian profile use a separate, older builder. It is simpler, it applies no European semantic rule, and it computes its line amounts with the formulas of section 4 of [calculations.md](calculations.md) rather than with the six digit rounding.

## 17.1 Preparation

1. Validate the tax structure, exactly as in section 5 step 1.
2. Decide the document kind: a debit note when the accounting document was created as a debit note of another document, a credit note when it is a customer or vendor credit note, an invoice otherwise. For a vendor document the supplier and the customer are swapped and the shipping party becomes the customer.
3. Set two switches: amounts are expressed in the document currency, and fixed amount taxes are reported as line level allowances and charges.
4. Build the base lines with their tax details.
5. For each base line, keep the label of the line with every line break replaced by a space.
6. Move every recycling contribution tax out of the tax list of its line and into a separate list on that line, adding its amount to the total excluding tax of the line, in both currencies and in both the raw and the rounded form.
7. Turn the emptying taxes into extra lines, as in section 5 step 7.
8. Split the cash rounding lines out of the line list into their own list.

## 17.2 Header elements

| Path | Value written |
|---|---|
| `UBLVersionID` | `2.0` in the plain 2.0 syntax and in the Belgian profile, `2.1` in the plain 2.1 syntax |
| `Identifier` | the name of the accounting document |
| `IssueDate` | the invoice date |
| `DueDate` | the due date, written only in the plain 2.1 syntax and only for an invoice |
| `InvoiceTypeCode` | `389` for a self billed document, `380` otherwise; written only for an invoice |
| `CreditNoteTypeCode` | `261` for a self billed document, `381` otherwise; written only in the plain 2.1 syntax and only for a credit note |
| `Note` | the terms and conditions of the accounting document converted to plain text |
| `DocumentCurrencyCode` | the three letter code of the document currency |
| `BuyerReference` | the reference of the commercial contact of the customer; written only in the plain 2.1 syntax |
| `OrderReference / Identifier` | the customer reference of the accounting document, falling back to its name |
| `OrderReference / SalesOrderID` | the names of the sales orders linked to the lines, joined by commas; written only when the sales capability package is installed |

## 17.3 Party elements

The plain builder writes one party group per role with these elements, and does not apply any of the country specific rules of section 8.

| Path | Value written |
|---|---|
| `Party / EndpointID` | nothing |
| `Party / PartyIdentification / Identifier` | the reference of the commercial contact |
| `Party / PartyName / Name` | the display name of the contact when it has a name, otherwise the display name of its commercial contact |
| `Party / PostalAddress` | the address group built from the contact, with all seven elements of section 8.4 including the country sub-entity code and the country name |
| `Party / PartyTaxScheme / RegistrationName` | the name of the commercial contact; the whole group is written only when the contact has a tax identification number that is not a single solidus |
| `Party / PartyTaxScheme / CompanyID` | the tax identification number of the commercial contact |
| `Party / PartyTaxScheme / RegistrationAddress` | the address group built from the commercial contact |
| `Party / PartyTaxScheme / TaxScheme / Identifier` | the identifier `NOT_EU_VAT` when the commercial contact has a country and a tax identification number whose first two characters are not letters, the identifier `VAT` otherwise |
| `Party / PartyLegalEntity / RegistrationName` | the name of the commercial contact |
| `Party / PartyLegalEntity / CompanyID` | the tax identification number of the commercial contact |
| `Party / PartyLegalEntity / RegistrationAddress` | the address group built from the commercial contact |
| `Party / Contact / Identifier` | the internal key of the contact record |
| `Party / Contact / Name` | the name of the contact |
| `Party / Contact / Telephone` | the telephone number of the contact |
| `Party / Contact / ElectronicMail` | the electronic mail address of the contact |

## 17.4 Delivery, payment means, payment terms

Written for an invoice in both syntaxes, and additionally for a credit note and a debit note in the plain 2.1 syntax.

| Path | Value written |
|---|---|
| `Delivery / ActualDeliveryDate` | the delivery date of the accounting document |
| `Delivery / DeliveryLocation / Identifier` | the global location number of the shipping party with the scheme attribute `0088`, when that capability package is installed and the number is set |
| `Delivery / DeliveryLocation / Address` | the address group built from the shipping party |
| `PaymentMeans / PaymentMeansCode` | `30` with the attribute `name` holding `credit transfer` for a customer invoice with a recipient bank account; `ZZZ` with `mutually defined` for a customer invoice without one; `57` with `standing agreement` otherwise; and `1` with `unknown` when the customer country is Denmark |
| `PaymentMeans / PaymentDueDate` | the due date of the accounting document, falling back to its invoice date |
| `PaymentMeans / InstructionID` | the payment reference of the accounting document |
| `PaymentMeans / PaymentID` | the payment reference of the accounting document, falling back to its name |
| `PaymentMeans / PayeeFinancialAccount / Identifier` | the account number of the recipient bank account with every space removed |
| `PaymentMeans / PayeeFinancialAccount / FinancialInstitutionBranch / Identifier` | the bank identifier code of the bank, with the scheme attribute `BIC` |
| `PaymentMeans / PayeeFinancialAccount / FinancialInstitutionBranch / FinancialInstitution / Identifier` | the same bank identifier code with the same scheme attribute |
| `PaymentMeans / PayeeFinancialAccount / FinancialInstitutionBranch / FinancialInstitution / Name` | the name of the bank |
| `PaymentMeans / PayeeFinancialAccount / FinancialInstitutionBranch / FinancialInstitution / Address` | the address group built from the bank record |
| `PaymentTerms / Note` | the note of the payment terms converted to plain text |

## 17.5 Document level allowances and charges

Only the early payment lines become document level groups, and only for an invoice and a credit note in the plain 2.0 syntax; the plain 2.1 syntax adds them for a debit note as well.

| Path | Value written |
|---|---|
| `AllowanceCharge / ChargeIndicator` | `false` when the total excluding tax of the line is negative, `true` otherwise |
| `AllowanceCharge / AllowanceChargeReasonCode` | `66` when it is an allowance, `ZZZ` when it is a charge |
| `AllowanceCharge / AllowanceChargeReason` | `Conditional cash/payment discount` |
| `AllowanceCharge / Amount` | the absolute value of the total excluding tax of the line |
| `AllowanceCharge / TaxCategory` | one group per tax grouping key of that line, built as in section 17.7 |

## 17.6 Line elements

| Path | Value written |
|---|---|
| `<line> / Identifier` | the position of the line, starting at one |
| `<line> / InvoicedQuantity`, `<line> / CreditedQuantity`, `<line> / DebitedQuantity`, `<line> / Quantity` | the quantity of the base line with the attribute `unitCode` |
| `<line> / LineExtensionAmount` | the total of the line excluding tax, increased by the delta of that total and by the fixed tax amounts |
| `<line> / AllowanceCharge` | the discount group and one group per recycling contribution tax; written for an invoice in the plain 2.0 syntax, and for every document kind in the plain 2.1 syntax |
| `<line> / TaxTotal` | a full tax total group built from the tax details of that line, with one subtotal per tax grouping key |
| `<line> / Item / Description` | the sale description of the product, replaced by the label of the line when the line has one |
| `<line> / Item / Name` | the name of the product, replaced by the label of the line when the product has no name |
| `<line> / Item / SellersItemIdentification / Identifier` | the internal reference of the product |
| `<line> / Item / StandardItemIdentification / Identifier` | the barcode of the product with the scheme attribute `0160` |
| `<line> / Item / AdditionalItemProperty` | one pair per variant attribute value of the product |
| `<line> / Item / ClassifiedTaxCategory` | one group per tax grouping key of the line, built as in section 17.7 |
| `<line> / Price / PriceAmount` | the gross unit price of the line rounded to the product price precision, with the attribute `currencyID` |

The line discount group holds the charge indicator `false` when the discount amount is positive and `true` when it is negative, the reason code `95`, and the absolute discount amount. It carries no reason text, no multiplier and no base amount. The recycling contribution group holds the charge indicator, the reason code `AEO` for a charge and `100` for an allowance, the tax name as the reason, and the absolute tax amount.

## 17.7 Tax grouping, tax totals and monetary total of the plain builder

The plain builder uses a single grouping key made of the tax category code, the exemption reason code, the exemption reason text, the rate of the tax, and the kind of the tax amount. The document tax total holds the summed tax amount and one subtotal per key; each subtotal holds the taxable amount, the tax amount, the percentage when the tax is a percentage tax, and a tax category group holding the category code, the name of the key when it has one, the percentage when the tax is a percentage tax, the exemption reason code, the exemption reason text and the tax scheme identifier `VAT`. The withholding tax total element is written empty.

The monetary total is computed as follows.

Take each base line in turn and compute its line total.

```formula
line total of a base line = total excluding tax + rounding adjustment of the total excluding tax
                            + sum of the fixed tax amounts of that line
```

A base line that is an early payment discount line contributes to the allowance total when its line total is negative, and to the charge total when its line total is not negative. A base line that is not an early payment discount line contributes to the total of the lines instead.

```formula
total allowance      = sum over early payment discount lines whose line total is negative of ( − line total )
total charge         = sum over early payment discount lines whose line total is not negative of ( line total )
total of the lines   = sum over the other base lines of ( line total )
tax exclusive amount = total of the lines + total charge − total allowance
total tax amount     = aggregated tax amount of every tax that is not a fixed tax
tax inclusive amount = tax exclusive amount + total tax amount
cash rounding amount = sum over cash rounding lines of ( total excluding tax of the line )
```

| Path | Value written | Omitted when |
|---|---|---|
| `LineExtensionAmount` | `total_lines` | never |
| `TaxExclusiveAmount` | `tax_exclusive_amount` | never |
| `TaxInclusiveAmount` | `tax_inclusive_amount` | never |
| `AllowanceTotalAmount` | `total_allowance` | it is zero |
| `ChargeTotalAmount` | `total_charge` | it is zero |
| `PrepaidAmount` | the total of the accounting document minus its residual amount | never |
| `PayableRoundingAmount` | `cash_rounding_amount` | it is zero |
| `PayableAmount` | the residual amount of the accounting document | never |

## 17.8 Validations of the plain builder

Four checks, in addition to the shared check that every line carries at least one tax:

| Identifier of the check | Message |
|---|---|
| supplier name | `The field 'Name' is required on <the supplier>.` |
| customer name | `The field 'Name' is required on <the commercial contact of the customer>.` |
| document name | `The field 'Number' is required on <the document>.` |
| document date | `The field 'Invoice/Bill Date' is required on <the document>.` |

---

# 18. Which validations apply to which profile

Every validation is specified once, with its exact message, in [business-rules.md](business-rules.md). This table says which set applies to which profile. A validation that produces a message stops the export and, in the sending flow, records the message on the accounting document without stopping the rest of the batch.

| Profile | Validations applied |
|---|---|
| Universal Business Language 2.0, Universal Business Language 2.1, Belgian Electronic Invoicing Profile | `EIDI-RULE-060`, `EIDI-RULE-061`, `EIDI-RULE-062`, `EIDI-RULE-063`, `EIDI-RULE-064` |
| Peppol Billing 3.0 | the five above, plus `EIDI-RULE-090` to `EIDI-RULE-104`, plus `EIDI-RULE-105` and `EIDI-RULE-106` when the file is produced for the exchange network, plus `EIDI-RULE-132` to `EIDI-RULE-137` when the supplier is in the Netherlands |
| German Electronic Invoice Profile | everything of the Peppol Billing 3.0 profile, plus `EIDI-RULE-130` and `EIDI-RULE-131` |
| Netherlands Standard Invoice Profile, Australia and New Zealand Billing Profile, Singapore Billing Profile | everything of the Peppol Billing 3.0 profile |
| Sales Order Ordering Profile, Purchase Order Ordering Profile | no document validation is run on an exported order |

---

# 19. The ordering profiles

Two profiles export an order rather than an accounting document. Both derive from the Peppol Billing 3.0 profile, so they reuse its party, line, item, price, allowance, charge and tax total builders, and both write the ordering customization identifier `urn:fdc:peppol.eu:poacc:trns:order:3` and the ordering profile identifier `urn:fdc:peppol.eu:poacc:bis:ordering:3`.

## 19.1 Preparation

1. Resolve the parties. For a sales order: the supplier is the commercial contact of the company and the customer is the partner of the order; the delivery party is the shipping address of the order, falling back to the first child contact of the customer whose address kind is the delivery address, falling back to the customer. For a purchase order: the supplier is the partner of the order and the customer is the commercial contact of the company; the delivery party is the destination address of the order, falling back to the first child delivery contact of the customer, falling back to the customer.
2. Build one base line per order line that is not a display line, add the tax details and round them.
3. Turn every negative unit price positive.
4. Turn the emptying taxes into extra lines.
5. Apply the six digit global rounding, exactly as in section 5 step 8.
6. For a purchase order only, resolve for each line the vendor pricelist entry of that product for that vendor, preferring the entry that names the variant over the entry that names only the template, and keeping only an entry that carries a vendor product code or a vendor product name.

## 19.2 Header elements

| Path | Sales order | Purchase order |
|---|---|---|
| `CustomizationID` | `urn:fdc:peppol.eu:poacc:trns:order:3` | the same |
| `ProfileID` | `urn:fdc:peppol.eu:poacc:bis:ordering:3` | the same |
| `Identifier` | the name of the order | the name of the order |
| `IssueDate` | the date part of the creation timestamp of the order | the date part of the creation timestamp of the order |
| `OrderTypeCode` | `220` | `105` |
| `Note` | the terms and conditions of the order converted to plain text | the internal note of the order converted to plain text |
| `DocumentCurrencyCode` | the three letter code of the order currency | the same |
| `ValidityPeriod / EndDate` | the expiration date of the order | not written |
| `QuotationDocumentReference / Identifier` | not written | the vendor reference of the order |
| `OriginatorDocumentReference / Identifier` | the customer reference of the order | not written |

## 19.3 Body

| Element | Content |
|---|---|
| `BuyerCustomerParty / Party` | the customer party built exactly as in section 8 |
| `SellerSupplierParty / Party` | the supplier party built exactly as in section 8 |
| `Delivery` | the delivery group of section 9 reduced to its first entry, with the actual delivery date and the delivery location removed, so only the delivery party name survives |
| `PaymentTerms / Note` | the name of the payment terms of the order, when it has payment terms |
| `OrderLine / LineItem` | one per base line: the identifier, the allowance and charge list of section 12.3, the quantity, the line net amount, the item group of section 13 and the price group of section 14 |
| `AllowanceCharge` | the document level groups of section 11, plus the early payment discount groups |
| `TaxTotal` | the tax totals of section 15 |
| `AnticipatedMonetaryTotal` | the monetary total of section 16, with the line extension amount computed as the sum of the line net amounts of the order lines |

For a purchase order, the item name is replaced by the vendor product name when the vendor pricelist entry has one, and the seller item identifier is replaced by the vendor product code when the entry has one, so that the vendor receives its own names and codes.

---

# 20. Embedded documents and extension elements

## 20.1 The printed document inside the structured file

After the printed document has been rendered and before the structured file is stored, the printed document and the user supplied attachments are inserted into the structured file as additional document reference groups. This happens for every profile of this family; the cross industry invoice family does the reverse, embedding the structured file inside the printed document.

1. Parse the structured file.
2. Find the insertion anchor, which is the first element of the document that matches, in order of preference:
   - in an invoice: the project reference, the signature, the accounting supplier party;
   - in a credit note: the statement document reference, the originator document reference, the signature, the accounting supplier party;
   - in a debit note: the signature, the accounting supplier party.
   When no anchor is found, nothing is embedded and the file is left unchanged, because inserting at an arbitrary place would break the element order.
3. Build the list of attachments to embed: first the user supplied attachments of the sending wizard whose media type is one of the supported media types of section 20.2, and only when the profile declares that it embeds attachments; then the printed document itself.
4. For each attachment, in that order, insert immediately before the anchor one additional document reference group holding:

| Path | Value written |
|---|---|
| `AdditionalDocumentReference / Identifier` | the file name of the attachment; this is the identifier of the referenced document |
| `AdditionalDocumentReference / DocumentTypeCode` | the document type code element of the profile, when the profile defines one; only the printed document carries it |
| `AdditionalDocumentReference / Attachment / EmbeddedDocumentBinaryObject` | the content of the attachment encoded in base sixty four, with the attribute `mimeCode` holding its media type and the attribute `filename` holding its file name |

5. Serialise the tree again.

## 20.2 Supported media types for embedding

| Media type | Extension used when the file is extracted on import |
|---|---|
| the media type identifier `application/pdf` | `pdf` |
| the media type identifier `application/vnd.oasis.opendocument.spreadsheet` | `ods` |
| the media type identifier `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | `xlsx` |
| the media type identifier `image/jpeg` | `jpeg` |
| the media type identifier `image/png` | `png` |
| the media type identifier `text/csv` | `csv` |

An attachment of any other media type is not embedded; the sending flow reports which attachments could not be embedded.

## 20.3 User defined extension elements

A deployment may add user defined fields to the accounting document and to its lines whose name begins with the reserved prefix for network extension fields. Each such field is declared in a shipped table that gives, per field, the path of the element to create and the attributes to write. When the document is an invoice the invoice table is used, when it is a credit note the credit note table is used, and the same distinction applies at line level. For every declared field that has a value on the record, the builder walks the declared path, creating each missing container on the way, and writes the declared attributes at the end of it. This is the only mechanism by which an element outside the mapping of this file can appear in the produced document.

---

# 21. Worked example: a complete file

**Setup.** Company established in Belgium, currency euro with two decimal places, customer established in Belgium, profile Peppol Billing 3.0, document kind invoice, document name `INV/2024/00001`, invoice date the first of March 2024, due date the thirty first of March 2024, no payment made.

| Line | Product | Quantity | Unit price | Discount | Tax |
|---|---|---|---|---|---|
| 1 | product with internal reference `FURN_7777`, label `FURN_7777 Office Chair` | 1 | 1000.00 | 5 percent | 21 percent standard rate |
| 2 | product with internal reference `FURN_8888`, label `FURN_8888 Four Person Desk` | 1 | 500.00 | none | 6 percent standard rate |
| global discount line | none, label `General upsell` | 1 | 50.00 | none | 21 percent standard rate |

The company has the tax identification number `BE0477472701`, the participant endpoint `0477472701` under the scheme `0208`, the street `Rue du Test 1`, the city `Brussels`, the postal code `1000`, the telephone number `+32 2 000 00 00` and the electronic mail address `company@example.com`. The customer has the tax identification number `BE0246697724`, the participant endpoint `0246697724` under the scheme `0208`, the street `Chaussée du Client 2`, the city `Namur`, the postal code `5000` and the reference `CUST-1`. The recipient bank account is `BE15001559627230`, held at a bank whose bank identifier code is `GEBABEBB`.

**The produced elements, in written order.**

| Path | Value |
|---|---|
| `CustomizationID` | `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0` |
| `ProfileID` | `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0` |
| `Identifier` | `INV/2024/00001` |
| `IssueDate` | `2024-03-01` |
| `DueDate` | `2024-03-31` |
| `InvoiceTypeCode` | `380` |
| `DocumentCurrencyCode` | `EUR`, the code of the euro |
| `BuyerReference` | `CUST-1` |
| `OrderReference / Identifier` | `INV/2024/00001`, because the document carries no customer reference |
| `AccountingSupplierParty / Party / EndpointID` | `0477472701` with the attribute `schemeID` holding `0208` |
| `AccountingSupplierParty / Party / PartyIdentification / Identifier` | omitted, because the company has no company registry number and the endpoint rule already applied |
| `AccountingSupplierParty / Party / PartyName / Name` | the display name of the company |
| `AccountingSupplierParty / Party / PostalAddress / StreetName` | `Rue du Test 1` |
| `AccountingSupplierParty / Party / PostalAddress / CityName` | `Brussels` |
| `AccountingSupplierParty / Party / PostalAddress / PostalZone` | `1000` |
| `AccountingSupplierParty / Party / PostalAddress / Country / IdentificationCode` | `BE`, the code of Belgium |
| `AccountingSupplierParty / Party / PartyTaxScheme / CompanyID` | `BE0477472701` |
| `AccountingSupplierParty / Party / PartyTaxScheme / TaxScheme / Identifier` | `VAT`, the value-added tax scheme identifier |
| `AccountingSupplierParty / Party / PartyLegalEntity / RegistrationName` | the name of the company |
| `AccountingSupplierParty / Party / PartyLegalEntity / CompanyID` | `BE0477472701`, without a scheme attribute, by branch 8 of section 8.6 |
| `AccountingSupplierParty / Party / Contact / Name` | the name of the company contact |
| `AccountingSupplierParty / Party / Contact / Telephone` | `+32 2 000 00 00` |
| `AccountingSupplierParty / Party / Contact / ElectronicMail` | `company@example.com` |
| `AccountingCustomerParty / Party / EndpointID` | `0246697724` with the attribute `schemeID` holding `0208` |
| `AccountingCustomerParty / Party / PartyIdentification / Identifier` | `CUST-1`, because the customer has a reference and no company registry number |
| `AccountingCustomerParty / Party / PartyName / Name` | the display name of the customer |
| `AccountingCustomerParty / Party / PostalAddress / StreetName` | `Chaussée du Client 2` |
| `AccountingCustomerParty / Party / PostalAddress / CityName` | `Namur` |
| `AccountingCustomerParty / Party / PostalAddress / PostalZone` | `5000` |
| `AccountingCustomerParty / Party / PostalAddress / Country / IdentificationCode` | `BE` |
| `AccountingCustomerParty / Party / PartyTaxScheme / CompanyID` | `BE0246697724` |
| `AccountingCustomerParty / Party / PartyTaxScheme / TaxScheme / Identifier` | `VAT`, the value-added tax scheme identifier |
| `AccountingCustomerParty / Party / PartyLegalEntity / RegistrationName` | the name of the customer |
| `AccountingCustomerParty / Party / PartyLegalEntity / CompanyID` | `BE0246697724` |
| `Delivery / DeliveryParty / PartyName / Name` | the display name of the customer, because no separate shipping address exists |
| `PaymentMeans / PaymentMeansCode` | `30` with the attribute `name` holding `credit transfer` |
| `PaymentMeans / PaymentID` | `INV/2024/00001` |
| `PaymentMeans / PayeeFinancialAccount / Identifier` | `BE15001559627230` |
| `PaymentMeans / PayeeFinancialAccount / FinancialInstitutionBranch / Identifier` | `GEBABEBB` |
| `AllowanceCharge / ChargeIndicator` | `true` |
| `AllowanceCharge / AllowanceChargeReasonCode` | `ADK` |
| `AllowanceCharge / AllowanceChargeReason` | `General upsell` |
| `AllowanceCharge / Amount` | `50.00` with the attribute `currencyID` holding `EUR` |
| `AllowanceCharge / TaxCategory / Identifier` | `S`, the standard rate category |
| `AllowanceCharge / TaxCategory / Percent` | `21.0` |
| `AllowanceCharge / TaxCategory / TaxScheme / Identifier` | `VAT`, the value-added tax scheme identifier |
| `TaxTotal / TaxAmount` | `240.00` with the attribute `currencyID` holding `EUR` |
| `TaxTotal / TaxSubtotal[1] / TaxableAmount` | `1000.00` |
| `TaxTotal / TaxSubtotal[1] / TaxAmount` | `210.00` |
| `TaxTotal / TaxSubtotal[1] / TaxCategory / Identifier` | `S` |
| `TaxTotal / TaxSubtotal[1] / TaxCategory / Percent` | `21.0` |
| `TaxTotal / TaxSubtotal[1] / TaxCategory / TaxScheme / Identifier` | `VAT`, the value-added tax scheme identifier |
| `TaxTotal / TaxSubtotal[2] / TaxableAmount` | `500.00` |
| `TaxTotal / TaxSubtotal[2] / TaxAmount` | `30.00` |
| `TaxTotal / TaxSubtotal[2] / TaxCategory / Identifier` | `S` |
| `TaxTotal / TaxSubtotal[2] / TaxCategory / Percent` | `6.0` |
| `TaxTotal / TaxSubtotal[2] / TaxCategory / TaxScheme / Identifier` | `VAT`, the value-added tax scheme identifier |
| `LegalMonetaryTotal / LineExtensionAmount` | `1450.00` |
| `LegalMonetaryTotal / TaxExclusiveAmount` | `1500.00` |
| `LegalMonetaryTotal / TaxInclusiveAmount` | `1740.00` |
| `LegalMonetaryTotal / AllowanceTotalAmount` | omitted, the sum of the allowances is zero |
| `LegalMonetaryTotal / ChargeTotalAmount` | `50.00` |
| `LegalMonetaryTotal / PrepaidAmount` | `0.00` |
| `LegalMonetaryTotal / PayableRoundingAmount` | omitted, the difference is zero |
| `LegalMonetaryTotal / PayableAmount` | `1740.00` |
| `InvoiceLine[1] / Identifier` | `1` |
| `InvoiceLine[1] / InvoicedQuantity` | `1.0` with the attribute `unitCode` holding `C62`, the code for one piece |
| `InvoiceLine[1] / AllowanceCharge / ChargeIndicator` | `false` |
| `InvoiceLine[1] / AllowanceCharge / AllowanceChargeReasonCode` | `95` |
| `InvoiceLine[1] / AllowanceCharge / AllowanceChargeReason` | `Discount` |
| `InvoiceLine[1] / AllowanceCharge / MultiplierFactorNumeric` | `5.0` |
| `InvoiceLine[1] / AllowanceCharge / Amount` | `50.00` |
| `InvoiceLine[1] / AllowanceCharge / BaseAmount` | `1000.00` |
| `InvoiceLine[1] / LineExtensionAmount` | `950.00` |
| `InvoiceLine[1] / Item / Description` | `Office Chair` |
| `InvoiceLine[1] / Item / Name` | the display name of the product |
| `InvoiceLine[1] / Item / SellersItemIdentification / Identifier` | `FURN_7777` |
| `InvoiceLine[1] / Item / ClassifiedTaxCategory / Identifier` | `S` |
| `InvoiceLine[1] / Item / ClassifiedTaxCategory / Percent` | `21.0` |
| `InvoiceLine[1] / Item / ClassifiedTaxCategory / TaxScheme / Identifier` | `VAT`, the value-added tax scheme identifier |
| `InvoiceLine[1] / Price / PriceAmount` | `1000.0` |
| `InvoiceLine[2] / Identifier` | `2` |
| `InvoiceLine[2] / InvoicedQuantity` | `1.0` with the attribute `unitCode` holding `C62` |
| `InvoiceLine[2] / LineExtensionAmount` | `500.00` |
| `InvoiceLine[2] / Item / Description` | `Four Person Desk` |
| `InvoiceLine[2] / Item / Name` | the display name of the product |
| `InvoiceLine[2] / Item / SellersItemIdentification / Identifier` | `FURN_8888` |
| `InvoiceLine[2] / Item / ClassifiedTaxCategory / Identifier` | `S` |
| `InvoiceLine[2] / Item / ClassifiedTaxCategory / Percent` | `6.0` |
| `InvoiceLine[2] / Item / ClassifiedTaxCategory / TaxScheme / Identifier` | `VAT`, the value-added tax scheme identifier |
| `InvoiceLine[2] / Price / PriceAmount` | `500.0` |

The global discount line does not become a document line; it is the document level charge listed above. The arithmetic of every figure is worked through step by step in section 7.1 of [calculations.md](calculations.md).

---

# 22. The point of sale receipt profile

A point of sale receipt is exported with the plain 2.1 builder, with these differences.

1. **Document kind.** An invoice when the total of the receipt is zero or positive, a credit note when it is negative.
2. **Parties.** The supplier is the commercial contact of the company; the customer is the partner of the receipt.
3. **Base lines** are built from the tax base values of the receipt, with their tax details added and rounded. No emptying tax handling, no recycling contribution extraction and no cash rounding extraction are applied.
4. **Header elements.** The version identifier is `2.0`; the identifier is the name of the receipt; the issue date is the order date of the receipt; the invoice type code is `380` for an invoice and is not written for a credit note; the note is the general customer note of the receipt; the document currency code is the currency of the receipt; the order reference identifier is the name of the receipt.
5. **Body.** The supplier party, the customer party, the document level allowances and charges, the tax totals, the monetary total and the lines are produced exactly as in the plain builder. No payment means group is produced.
6. **Monetary total.** After the plain monetary total has been produced, two of its elements are replaced: the prepaid amount becomes the tax exclusive amount minus the amount paid on the receipt, and the payable amount becomes the amount paid on the receipt.
7. **Item naming.** The description of the item is the label of the line with every line break replaced by a space, and the name of the item falls back to that same label when the product has no name.
8. **Validation.** No validation is run on a point of sale receipt.


---

# 23. Tax exemption reason codes

The ninety one codes that may be configured on a tax, and the sentence each one writes into the exemption reason element. Sixty two belong to the European list and twenty nine to the French list. A code that is configured on a tax but is not in this table writes the sentence `Exempt from tax` when its category demands a reason, and writes no sentence otherwise. When no code is configured at all, the prediction of section 10.2 of [calculations.md](calculations.md) decides both the code and the sentence.

| Code | Sentence written as the exemption reason |
|---|---|
| `VATEX-EU-79-C` | Exempt based on article 79, point c of Council Directive 2006/112/EC |
| `VATEX-EU-132` | Exempt based on article 132 of Council Directive 2006/112/EC |
| `VATEX-EU-132-1A` | Exempt based on article 132, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1B` | Exempt based on article 132, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1C` | Exempt based on article 132, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1D` | Exempt based on article 132, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1E` | Exempt based on article 132, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1F` | Exempt based on article 132, section 1 (f) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1G` | Exempt based on article 132, section 1 (g) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1H` | Exempt based on article 132, section 1 (h) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1I` | Exempt based on article 132, section 1 (i) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1J` | Exempt based on article 132, section 1 (j) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1K` | Exempt based on article 132, section 1 (k) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1L` | Exempt based on article 132, section 1 (l) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1M` | Exempt based on article 132, section 1 (m) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1N` | Exempt based on article 132, section 1 (n) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1O` | Exempt based on article 132, section 1 (o) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1P` | Exempt based on article 132, section 1 (p) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1Q` | Exempt based on article 132, section 1 (q) of Council Directive 2006/112/EC |
| `VATEX-EU-135-1` | Exempt based on article 135, section 1 of Council Directive 2006/112/EC |
| `VATEX-EU-143` | Exempt based on article 143 of Council Directive 2006/112/EC |
| `VATEX-EU-143-1A` | Exempt based on article 143, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1B` | Exempt based on article 143, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1C` | Exempt based on article 143, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1D` | Exempt based on article 143, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1E` | Exempt based on article 143, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1F` | Exempt based on article 143, section 1 (f) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1FA` | Exempt based on article 143, section 1 (fa) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1G` | Exempt based on article 143, section 1 (g) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1H` | Exempt based on article 143, section 1 (h) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1I` | Exempt based on article 143, section 1 (i) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1J` | Exempt based on article 143, section 1 (j) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1K` | Exempt based on article 143, section 1 (k) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1L` | Exempt based on article 143, section 1 (l) of Council Directive 2006/112/EC |
| `VATEX-EU-144` | Exempt based on article 144 of Council Directive 2006/112/EC |
| `VATEX-EU-146-1E` | Exempt based on article 146 section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-148` | Exempt based on article 148 of Council Directive 2006/112/EC |
| `VATEX-EU-148-A` | Exempt based on article 148, section (a) of Council Directive 2006/112/EC |
| `VATEX-EU-148-B` | Exempt based on article 148, section (b) of Council Directive 2006/112/EC |
| `VATEX-EU-148-C` | Exempt based on article 148, section (c) of Council Directive 2006/112/EC |
| `VATEX-EU-148-D` | Exempt based on article 148, section (d) of Council Directive 2006/112/EC |
| `VATEX-EU-148-E` | Exempt based on article 148, section (e) of Council Directive 2006/112/EC |
| `VATEX-EU-148-F` | Exempt based on article 148, section (f) of Council Directive 2006/112/EC |
| `VATEX-EU-148-G` | Exempt based on article 148, section (g) of Council Directive 2006/112/EC |
| `VATEX-EU-151` | Exempt based on article 151 of Council Directive 2006/112/EC |
| `VATEX-EU-151-1A` | Exempt based on article 151, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1AA` | Exempt based on article 151, section 1 (aa) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1B` | Exempt based on article 151, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1C` | Exempt based on article 151, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1D` | Exempt based on article 151, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1E` | Exempt based on article 151, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-153` | Exempt based on article 153 of Council Directive 2006/112/EC |
| `VATEX-EU-159` | Exempt based on article 159 of Council Directive 2006/112/EC |
| `VATEX-EU-309` | Exempt based on article 309 of Council Directive 2006/112/EC |
| `VATEX-EU-AE` | Reverse charge |
| `VATEX-EU-D` | Intra-Community acquisition from second hand means of transport |
| `VATEX-EU-F` | Intra-Community acquisition of second hand goods |
| `VATEX-EU-G` | Export outside the EU |
| `VATEX-EU-I` | Intra-Community acquisition of works of art |
| `VATEX-EU-IC` | Intra-Community supply |
| `VATEX-EU-O` | Not subject to value-added tax |
| `VATEX-EU-J` | Intra-Community acquisition of collectors items and antiques |
| `VATEX-FR-FRANCHISE` | France domestic value-added tax franchise in base |
| `VATEX-FR-CNWVAT` | France domestic Credit Notes without value-added tax, due to supplier forfeit of value-added tax for discount |
| `VATEX-FR-CGI261-1` | Exempt based on 1 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-2` | Exempt based on 2 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-3` | Exempt based on 3 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-4` | Exempt based on 4 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-5` | Exempt based on 5 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-7` | Exempt based on 7 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-8` | Exempt based on 8 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261A` | Exempt based on article 261 A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261B` | Exempt based on article 261 B of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-1` | Exempt based on 1° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-2` | Exempt based on 2° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-3` | Exempt based on 3° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-1` | Exempt based on 1° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-1BIS` | Exempt based on 1°bis of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-2` | Exempt based on 2° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-3` | Exempt based on 3° of article 261 D of the Code Général des Impôts (CGI ; General tax code) Exonération de TVA - Article 261 D-3° du Code Général des Impôts |
| `VATEX-FR-CGI261D-4` | Exempt based on 4° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261E-1` | Exempt based on 1° of article 261 E of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261E-2` | Exempt based on 2° of article 261 E of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI277A` | Exempt based on article 277 A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI275` | Exempt based on article 275 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-298SEXDECIESA` | Exempt based on article 298 sexdecies A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI295` | Exempt based on article 295 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-AE` | Exempt based on 2 of article 283 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-F` | VATEX-FR-F - Second-hand sales |
| `VATEX-FR-I` | VATEX-FR-I - Sales of works of art |
| `VATEX-FR-J` | VATEX-FR-J - Sales of antiques |
