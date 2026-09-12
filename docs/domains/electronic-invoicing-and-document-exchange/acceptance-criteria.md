# Acceptance criteria

Given / When / Then scenarios that a replacement must pass. Every scenario is independently verifiable, uses concrete values, and cites the rule, the state transition or the formula it exercises. Numbers of the form `EIDI-RULE-nnn` refer to [business-rules.md](business-rules.md); section numbers refer to the file named with them.

Unless a scenario says otherwise, the fixture is: a company established in Belgium named Test Company, with the tax identification number `BE0477472701`, the participant endpoint `0477472701` under the electronic address scheme `0208`, the street `Rue du Test 1`, the city `Brussels`, the postal code `1000`, the telephone number `+32 2 000 00 00`, the electronic mail address `company@example.com` and the recipient bank account `BE15001559627230` held at a bank whose identifier code is `GEBABEBB`; a customer established in Belgium named Test Customer with the tax identification number `BE0246697724`, the participant endpoint `0246697724` under the scheme `0208`, the street `Chaussée du Client 2`, the city `Namur`, the postal code `5000` and the reference `CUST-1`; the currency euro with two decimal places; a product named Office Chair with the internal reference `FURN_7777` and a product named Four Person Desk with the internal reference `FURN_8888`; taxes at twenty one percent and at six percent whose price is excluded.

---

# 1. Format registration and journal configuration

**EIDI-AC-001.** Given a sale journal and a registered format that declares itself compatible with sale journals and enabled by default, When the journal is read, Then its format list contains that format and its compatible format list contains it. *(EIDI-RULE-001, EIDI-RULE-002)*

**EIDI-AC-002.** Given a sale journal whose format list contains a format that needs no remote call, and a posted invoice in that journal whose delivery record for that format is in state `to_send`, When the format is removed from the journal, Then the write succeeds and the delivery record is deleted. *(EIDI-RULE-005)*

**EIDI-AC-003.** Given the same setup but with a format that needs a remote call, When the format is removed from the journal, Then the write is refused with `Cannot deactivate (%s) on this journal because not all documents are synchronized` naming that format, and the delivery record still exists. *(EIDI-RULE-004)*

**EIDI-AC-004.** Given a purchase journal marked as the exchange reception journal, When its type is changed to a sale journal, Then the write is refused with `You can't change the type of a journal used for Peppol invoice reception to a type different than 'Purchase'.` followed by a line break and `Please change the journal used for Peppol reception before changing the type of this journal.`

**EIDI-AC-005.** Given two formats with the same code, When the second is created, Then the creation is refused because the code must be unique, with the message `This code already exists`.

**EIDI-AC-006.** Given a company that becomes able to send and that has at least one purchase journal, When the participant settings are read, Then the reception journal is the first purchase journal of that company and that journal is marked as the reception journal.

**EIDI-AC-007.** Given a company with two purchase journals, the first of which is marked as the reception journal, When the second is set as the reception journal, Then the first loses the mark and the second gains it, and the scheduled action that keeps the published document types in step is triggered.

---

# 2. The electronic document lifecycle

**EIDI-AC-010.** Given a sale journal carrying one registered format that applies to customer invoices, When a customer invoice in that journal is posted, Then exactly one delivery record exists for that invoice, in state `to_send`. *(Section 2 of [workflows.md](workflows.md))*

**EIDI-AC-011.** Given a posted invoice whose single delivery record belongs to a format that needs no remote call and whose generation succeeds, When the posting completes, Then the delivery record is in state `sent` and carries the produced attachment.

**EIDI-AC-012.** Given a posted invoice whose single delivery record belongs to a format that needs a remote call, When the sending operation runs and the remote call answers with success, Then the delivery record moves from `to_send` to `sent`.

**EIDI-AC-013.** Given the same setup and a remote call that answers with the error text `turlututu` at the warning blocking level, When the sending operation runs, Then the delivery record stays in state `to_send`, carries that error and that blocking level. When the sending operation runs again and the remote call succeeds, Then the delivery record is in state `sent` and carries no error. *(EIDI-RULE-034, EIDI-RULE-038)*

**EIDI-AC-014.** Given a posted invoice whose format answers with the error text `step1 done` at the informational blocking level, When the sending operation runs, Then the delivery record stays in state `to_send` with that error. When the format then answers with success because it recognises that the first step is done, and the sending operation runs again, Then the delivery record is in state `sent`. This proves that a multi step exchange is driven entirely by the error text and the blocking level.

**EIDI-AC-015.** Given two posted invoices of the same company, the same format and the same state, whose format declares no batching key, When the jobs are prepared, Then two jobs exist.

**EIDI-AC-016.** Given the same two invoices addressed to the same partner and a format whose batching key is the partner, When the jobs are prepared, Then exactly one job exists holding both delivery records. *(EIDI-RULE-035, EIDI-RULE-036)*

**EIDI-AC-017.** Given ninety pending delivery records for a format with no batching key and a sending scheduled action whose limit is twenty, When the scheduled action runs, Then twenty jobs are processed, seventy remain and the scheduled action is scheduled to run again as soon as possible. Then, after five runs in total, the queue is empty and no further run is scheduled. *(Section 22 of [calculations.md](calculations.md))*

**EIDI-AC-018.** Given a job whose delivery records are already locked by another process, When the sending operation runs, Then the error `This document is being sent by another process already.` is recorded on those records and the rest of the batch is still processed.

**EIDI-AC-019.** Given a job assembled from delivery records that are not all in the same state, When the job is executed, Then the execution is refused with `All electronic documents of a job should have the same state`.

**EIDI-AC-020.** Given an attachment that is the payload of a delivery record whose format needs a remote call, When the attachment is deleted, Then the deletion is refused with `You can't unlink an attachment being an electronic document sent to the government.`

---

# 3. Cancellation

**EIDI-AC-030.** Given a posted invoice whose delivery record is in state `sent` and whose format allows cancellation, When the user requests a cancellation, Then the delivery record moves to `to_cancel`.

**EIDI-AC-031.** Given a delivery record in state `to_cancel`, When the user calls the cancellation off, Then the delivery record moves back to `sent`.

**EIDI-AC-032.** Given a delivery record in state `to_cancel`, When the sending operation runs and the remote cancellation succeeds, Then the delivery record moves to `cancelled` and the accounting document is cancelled.

**EIDI-AC-033.** Given a delivery record in state `to_cancel` whose format allows a forced cancellation, When the user forces the cancellation, Then the accounting document is cancelled without any remote call.

**EIDI-AC-034.** Given a posted invoice whose delivery records are all in state `cancelled` or that has none, When the ordinary cancellation of the general ledger runs, Then no guard of this domain intervenes.

---

# 4. Export: shared validation

**EIDI-AC-040.** Given an invoice line carrying no tax on a document whose journal produces a structured format, When the file is generated, Then the generation reports `Each invoice line should have at least one tax.` *(EIDI-RULE-061)*

**EIDI-AC-041.** Given a tax whose repartition structure is invalid, When the file is generated, Then the generation raises `Tax '<the tax name>' is invalid: <the repartition error message>`. *(EIDI-RULE-060)*

**EIDI-AC-042.** Given a supplier contact with no name, When the plain 2.0 file is generated, Then the generation reports `The field 'Name' is required on <the supplier display name>.` *(EIDI-RULE-062)*

**EIDI-AC-043.** Given a line whose unit price is `-100.00` and whose quantity is `2`, When the file is generated, Then the line is written with a quantity of `-2` and a unit price of `100.00`, so that neither the item net price nor the item gross price is negative. *(EIDI-RULE-066)*

**EIDI-AC-044.** Given a line carrying a fixed amount tax of `1.00` that does not increase the base of the following taxes, on two lines of quantity `3` and `2` respectively, When the file is generated, Then the two lines carry that tax no longer, and one extra document line exists whose label is the tax name, whose quantity is `5` and whose unit price is `1.00`. *(EIDI-RULE-068, section 2.3 of [calculations.md](calculations.md))*

---

# 5. Export: the universal business language mapping

**EIDI-AC-050.** Given the fixture invoice with line one of quantity one, unit price `1000.00`, discount five percent, tax twenty one percent, line two of quantity one, unit price `500.00`, tax six percent, and a global discount line of `50.00` at twenty one percent, When the billing profile file is generated, Then the produced elements are exactly those listed in section 21 of [universal-business-language-mapping.md](universal-business-language-mapping.md). In particular the line extension amount is `1450.00`, the tax exclusive amount is `1500.00`, the tax inclusive amount is `1740.00`, the charge total amount is `50.00`, the allowance total amount and the payable rounding amount are absent, the prepaid amount is `0.00` and the payable amount is `1740.00`.

**EIDI-AC-051.** Given the same invoice, When the tax breakdown is read, Then exactly two subtotals exist: one whose taxable amount is `1000.00`, whose tax amount is `210.00` and whose category percentage is `21.0`, and one whose taxable amount is `500.00`, whose tax amount is `30.00` and whose category percentage is `6.0`; and the total tax amount is `240.00`. *(Section 7.1 of [calculations.md](calculations.md))*

**EIDI-AC-052.** Given the same invoice, When line one is read, Then it carries one allowance whose charge indicator is `false`, whose reason code is `95`, whose reason is `Discount`, whose multiplier factor is `5.0`, whose amount is `50.00` and whose base amount is `1000.00`; its line net amount is `950.00` and its unit price is `1000.0`. *(Section 3.5 of [calculations.md](calculations.md))*

**EIDI-AC-053.** Given the same invoice, When the global discount line is read in the produced file, Then it is not a document line but a document level charge whose reason code is `ADK`, whose reason is `General upsell`, whose amount is `50.00` and whose tax category is `S` at `21.0` percent. *(Section 11.2 of [universal-business-language-mapping.md](universal-business-language-mapping.md))*

**EIDI-AC-054.** Given two lines of `33.33` at twenty one percent and a document allowance of `6.66` at twenty one percent, When the file is generated, Then the taxable amount of the twenty one percent subtotal is `60.00`, the sum of the presented line amounts minus the presented allowance, even when the aggregated base amounts would give `59.99`. *(Section 8 of [calculations.md](calculations.md))*

**EIDI-AC-055.** Given a customer invoice whose company currency is the euro and whose document currency is the United States dollar, When the billing profile file is generated, Then the tax currency code element carries `EUR`, the code of the euro, and two tax totals exist, one per currency, the one in the company currency carrying no subtotal.

**EIDI-AC-056.** Given the same invoice exported with the German electronic invoice profile, the Netherlands profile, the Australia and New Zealand profile or the Singapore profile, When the file is generated, Then the tax currency code element is absent and only the document currency tax total is written.

**EIDI-AC-057.** Given a Belgian company and a Belgian customer that has the company registry number `0246697724`, When the billing profile file is generated, Then the customer party identification carries that number compacted with the scheme attribute `0208`, and the customer legal entity carries the same number with the same scheme.

**EIDI-AC-058.** Given a customer established in the Netherlands whose electronic address scheme is `0106` and whose endpoint is `12345678`, When the billing profile file is generated, Then the customer legal entity identifier is `12345678` with the scheme attribute `0106`. Given instead an endpoint of twenty characters, Then the scheme attribute is `0190`.

**EIDI-AC-059.** Given a supplier established in Norway, When the billing profile file is generated, Then a further party tax scheme group is written whose identifier is `Foretaksregisteret` and whose scheme is `TAX`. Given a supplier established in Sweden, Then the further group carries `GODKÄND FÖR F-SKATT` with the scheme `TAX`.

**EIDI-AC-060.** Given a supplier established in Hungary whose tax identification number is `12345678901` and does not begin with the country prefix, When the file is generated, Then the party tax scheme identifier is `HU12345678`, the country prefix followed by the first eight characters.

**EIDI-AC-061.** Given a customer established in Denmark, When the billing profile file is generated, Then the payment means code is `1` with the attribute `name` holding `unknown`.

**EIDI-AC-062.** Given a customer invoice with no recipient bank account, When the billing profile file is generated, Then the payment means code is `ZZZ` with the attribute `name` holding `mutually defined` and no payee financial account is written.

**EIDI-AC-063.** Given a vendor bill exported as a self billed invoice, When the file is generated, Then the customization identifier is the self billing identifier, the profile identifier is the self billing profile identifier, the invoice type code is `389`, the supplier is the trading partner and the customer is the company.

**EIDI-AC-064.** Given a credit note that is fully reconciled against the invoice named `INV/2024/00001`, When the billing profile file is generated, Then one billing reference is written whose invoice document reference identifier is `INV/2024/00001`.

**EIDI-AC-065.** Given a company and a customer established in two different countries of the European economic area, When the billing profile file is generated, Then the actual delivery date of the delivery group equals the invoice date of the document, even when the document carries no delivery date. *(Section 9 of [universal-business-language-mapping.md](universal-business-language-mapping.md))*

**EIDI-AC-066.** Given the same setup and a document whose delivery location cannot be written, When the file is generated, Then the generation reports `For intracommunity supply, the delivery address should be included.` *(EIDI-RULE-099)*

**EIDI-AC-067.** Given a line whose product carries the variant attribute value "Colour: black", When the file is generated, Then the item carries one additional item property whose name is `Colour` and whose value is `black`.

**EIDI-AC-068.** Given a line whose product carries the barcode `1234567890128`, When the file is generated, Then the standard item identification carries that barcode with the scheme attribute `0160`.

**EIDI-AC-069.** Given a line whose label is `FURN_7777 Office Chair` and whose product display name is `FURN_7777 Office Chair`, When the file is generated, Then the item name is the product display name and the item description is empty. Given instead a label of `FURN_7777 Office Chair with armrests`, Then the item description is `with armrests`.

**EIDI-AC-070.** Given a line carrying a recycling contribution tax named `RECUPEL` whose amount is `1.00`, When the file is generated, Then the line carries one charge whose reason code is `AEO`, whose reason is `RECUPEL` and whose amount is `1.00`, and the line net amount includes it. Given the tax is named `BEBAT`, Then the reason code is `CAV`.

**EIDI-AC-071.** Given a line carrying an excise tax of `2.50`, When the file is generated, Then the line carries one charge with no reason code, whose reason is the tax name and whose amount is `2.50`.

**EIDI-AC-072.** Given an invoice with a mixed early payment discount of five percent on a base of `100.00` at twenty one percent, When the file is generated, Then two document level groups exist: an allowance of `5.00` with the reason code `64` and the reason `Conditional cash/payment discount` carrying the twenty one percent tax category, and a charge of `5.00` with the reason code `ZZZ` and the same reason carrying the zero rate category; and neither early payment line appears as a document line.

**EIDI-AC-073.** Given an invoice with a cash rounding line of `0.02`, When the file is generated, Then no document line carries it and the payable rounding amount is `0.02`.

**EIDI-AC-074.** Given an invoice whose only tax is a withholding tax of minus ten percent on a base of `1000.00`, When the billing profile file is generated, Then no withholding tax total is written, the prepaid amount is increased by `100.00`, the payable amount is increased by `100.00`, and the note begins with `The prepaid amount of` followed by the formatted amount and ` corresponds to the withholding tax applied.`

**EIDI-AC-075.** Given a line whose unit price is `12.3456789`, When the file is generated, Then the price amount is written as `12.3456789`. Given a unit price of `1000.0`, Then the price amount is written as `1000.0`. *(Section 1 of [calculations.md](calculations.md))*

**EIDI-AC-076.** Given a line whose unit of measure is the dozen, When the file is generated, Then the quantity element carries the attribute `unitCode` holding `DZN`. Given a unit of measure that is not in the table, Then the attribute holds `C62`.

**EIDI-AC-077.** Given an invoice generated with the plain 2.1 syntax, When the file is read, Then the version identifier element carries `2.1`, the buyer reference carries the reference of the commercial contact of the customer, and the party identification carries that same reference.

**EIDI-AC-078.** Given the same invoice generated with the Belgian profile, When the file name is read, Then it is `efff_BE0477472701_INV20240000001` with the extension `xml` for a document named `INV/2024/00001`, that is the prefix, the tax identification number of the company, an underscore, and the document name with every character that is neither a letter nor a digit removed.

**EIDI-AC-079.** Given a document with a quantity of five, a unit price of `100.00` and a discount of twenty percent generated with the plain 2.0 syntax, When the file is read, Then the line net amount is `400.00`, the gross unit price is `100.00` and the discount allowance amount is `100.00`. *(Section 4 of [calculations.md](calculations.md))*

**EIDI-AC-080.** Given the delivery location code package installed and a customer whose delivery address carries the code `222222222222`, and a posted customer invoice for that customer with one line of quantity one, unit price `100.00` and a tax at twenty one percent, When the file is generated with the Peppol billing 3.0 profile, Then the delivery node carries a delivery location whose identifier element holds the text `222222222222` with the attribute `schemeID` holding `0088`, and the address of that delivery location carries the street, the city, the postal code and the country of the delivery address. *(Section 15.6 of [entities.md](entities.md))*

**EIDI-AC-081.** Given the same invoice but with the delivery location code of the delivery address left empty, or with the delivery location code package not installed, When the file is generated, Then the delivery location carries its address and no identifier element at all, and the generation reports no error. *(Section 15.6 of [entities.md](entities.md))*

**EIDI-AC-082.** Given the same invoice generated with the plain 2.0 syntax instead, When the file is generated, Then the delivery location of the delivery node carries the same identifier element, with the same text and the same scheme attribute, taken from the shipping address of the document. *(Section 15.6 of [entities.md](entities.md))*

---

# 6. Export: the cross industry invoice mapping

**EIDI-AC-090.** Given a company established in France, a customer established in France, line one of quantity one, unit price `1000.00`, discount five percent, tax twenty one percent, and line two of quantity one, unit price `500.00`, tax six percent, When the cross industry invoice file is generated, Then the line total amount is `1450.00`, the tax basis total amount is `1450.00`, the tax total amount is `229.50` carrying the currency attribute, the grand total amount is `1679.50`, the total prepaid amount is `0.00` and the due payable amount is `1679.50`; the rounding amount element is absent. *(Section 11 of [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md))*

**EIDI-AC-091.** Given the same invoice, When line one is read, Then its gross price charge amount is `1000.00`, its applied allowance indicator is `false`, its applied allowance actual amount is `50.00`, its net price charge amount is `950.00` and its line total amount is `950.00`.

**EIDI-AC-092.** Given an invoice date of the tenth of March 2024, When the file is generated, Then the issue date and time element carries the text `20240310` and the attribute `format` holding `102`.

**EIDI-AC-093.** Given a company whose company registry number is a valid French establishment identification number of fourteen characters, When the file is generated, Then the seller legal organisation identifier carries only its first nine characters and the attribute `schemeID` holds `0002`.

**EIDI-AC-094.** Given a customer invoice whose recipient bank account is of the international kind, When the file is generated, Then the payee creditor financial account carries the international account element; given an account of another kind, Then it carries the proprietary account element.

**EIDI-AC-095.** Given a customer invoice reconciled with a payment that carries a direct debit mandate, When the file is generated, Then the payment means type code is `59`; otherwise it is `42`.

**EIDI-AC-096.** Given a customer invoice with no recipient bank account, When the cross industry invoice file is generated, Then the generation reports the required field message for the recipient bank account and, when a bank account exists without a sanitised account number, `The field 'Sanitized Account Number' is required on the Recipient Bank.` *(EIDI-RULE-150)*

**EIDI-AC-097.** Given a customer established in Spain whose postal code begins with `35` and an invoice line whose only tax has a rate of zero, When the file is generated, Then the generation reports `When the Canary Island General Indirect Tax (IGIC) applies, the tax rate on each invoice line should be greater than 0.` *(EIDI-RULE-157)*

**EIDI-AC-098.** Given a supplier with no telephone number, When the cross industry invoice file is generated, Then the generation reports the required field message for the telephone number. *(EIDI-RULE-153)*

**EIDI-AC-099.** Given an invoice whose only payment terms grant an early payment discount of two percent within ten days, When the file is generated, Then the payment discount terms carry a basis period measure of `10` with the attribute `unitCode` holding `DAY` and a calculation percentage of `2.0`.

**EIDI-AC-100.** Given an invoice with a cash rounding line of `-0.03`, When the file is generated, Then the rounding amount is `-0.03` and the grand total amount includes it.

**EIDI-AC-101.** Given a tax group whose aggregated tax amount rounds to a negative zero, When the file is generated, Then the calculated amount element carries exactly `0.00`.

**EIDI-AC-102.** Given a customer invoice sent to a commercial contact established in Germany, When the file name is read, Then it ends with `_zugferd` with the extension `xml`; for any other country it ends with `_factur_x` with the extension `xml`; and the copy embedded inside the printed document is always named `factur-x` with the extension `xml`.

**EIDI-AC-103.** Given a company established in France, a customer established in France whose delivery address is a child contact named `FR Partner Delivery` at `Rue Napoléon, 55`, `75000` `Paris`, carrying the delivery location code `5412345000008`, and a posted invoice for that customer, When the cross industry invoice file is generated, Then the ship-to trade party carries an identifier element holding the text `5412345000008` with the attribute `schemeID` holding `0088`, followed by the name element holding `FR Partner Delivery`. *(Section 15.6 of [entities.md](entities.md))*

**EIDI-AC-104.** Given the same invoice, When the seller trade party and the buyer trade party are read, Then neither carries an identifier element, because both pass an empty delivery location code to the same node builder, whatever code their own contacts carry. *(Section 15.6 of [entities.md](entities.md))*

**EIDI-AC-105.** Given the same invoice with the delivery location code of the delivery address left empty, When the file is generated, Then the ship-to trade party carries no identifier element and begins directly with its name element. *(Section 15.6 of [entities.md](entities.md))*

---

# 7. Export: tax category and exemption prediction

**EIDI-AC-110.** Given a supplier established in Belgium with a tax identification number, a customer established in France with a tax identification number, and one line of `1000.00` with a tax at zero percent carrying no configured category code, When the file is generated, Then the tax category code is `K`, the exemption reason code is `VATEX-EU-IC` and the exemption reason is `Intra-Community supply`. *(Section 10.1 of [calculations.md](calculations.md))*

**EIDI-AC-111.** Given a supplier and a customer established in the same country and a tax at twenty one percent, When the file is generated, Then the tax category code is `S`.

**EIDI-AC-112.** Given a supplier and a customer established in the same country and a tax at zero percent, When the file is generated, Then the tax category code is `E` and the exemption reason is `Exempt from tax`.

**EIDI-AC-113.** Given a supplier established in Belgium with a tax identification number and a customer established outside the European economic area, and a tax at zero percent, When the file is generated, Then the tax category code is `G`, the exemption reason code is `VATEX-EU-G` and the exemption reason is `Export outside the EU`, with the acronym expanding to "European Union".

**EIDI-AC-114.** Given a customer established in Spain whose postal code begins with `38`, When the file is generated, Then the tax category code is `L`. Given a postal code beginning with `52`, Then it is `M`.

**EIDI-AC-115.** Given a tax carrying the configured category code `AE` and the configured exemption reason code `VATEX-EU-AE`, When the file is generated, Then those two values are written unchanged and the exemption reason is `Reverse charge`.

**EIDI-AC-116.** Given a Belgian company invoicing a Belgian customer under the co-contractor fiscal position with a zero rate tax, When the file is generated, Then the exemption reason code is `VATEX-EU-AE` and the exemption reason is the note of that fiscal position, or, when the fiscal position carries no note, the sentence beginning `Reverse charge: In the absence of a written objection within one month of receipt of the invoice,`.

**EIDI-AC-117.** Given a document in which one line carries the category `O` and another carries the category `S`, When the file is generated, Then the generation reports `Taxes of category 'Service outside scope of tax' shall not be mixed with tax from other categories. You should split your invoice in two` *(EIDI-RULE-093)*, and no party tax scheme group is written for either party.

**EIDI-AC-118.** Given a document exported with the Singapore profile and one line with a tax at seven percent and one line with no tax, When the file is generated, Then the first line carries the category `SR` and the second carries `ZR`, and no exemption reason and no exemption reason code are written anywhere.

---

# 8. Import: routing and document direction

**EIDI-AC-130.** Given a file whose root element is the invoice root and whose tax inclusive amount is positive, dropped on a purchase journal, When it is imported, Then a vendor bill is produced and the quantities keep their sign.

**EIDI-AC-131.** Given a file whose root element is the invoice root and whose tax inclusive amount is negative, When it is imported, Then a credit note is produced, every quantity is negated and the message `The invoice has been converted into a credit note and the quantities have been reverted.` is logged.

**EIDI-AC-132.** Given a file whose root element is the credit note root, When it is imported into a sale journal, Then a customer credit note is produced with the quantities unchanged.

**EIDI-AC-133.** Given a cross industry invoice file whose document type code is `381`, When it is imported, Then a credit note is produced.

**EIDI-AC-134.** Given a file whose customization identifier contains `xrechnung`, When the file type is recognised, Then the German electronic invoice profile decodes it. Given a customization identifier equal to the Netherlands standard invoice identifier, Then the Netherlands profile decodes it. Given a version element of `2.0` and no recognised customization identifier, Then the plain 2.0 profile decodes it. *(Section 12.10 of [entities.md](entities.md))*

**EIDI-AC-135.** Given a file whose root element is the wrapper element and that holds an embedded markup object, When the file type is recognised, Then the wrapper is unwrapped, the embedded file is parsed and the recognition is applied again to it.

**EIDI-AC-136.** Given an accounting document that already has lines, When a file is dropped on it, Then the decoder refuses and reports that the document already has lines.

---

# 9. Import: document level values

**EIDI-AC-140.** Given a file whose supplier tax identification number is `BE0477472701` and a vendor that carries that number, When the file is imported into a purchase journal, Then that vendor is the partner of the produced bill and no new partner is created.

**EIDI-AC-141.** Given a file whose supplier name is `New Vendor` and whose tax identification number is `BE0999999999`, and no partner carrying either, When the file is imported, Then a new partner is created carrying the name, the tax identification number, the electronic mail address, the telephone number, the street, the second street line, the postal code, the city and the country from the file, marked as a company, and the message `Could not retrieve a partner corresponding to 'New Vendor'. A new partner was created.` is logged.

**EIDI-AC-142.** Given a matched partner whose tax identification number differs from the one in the file, When the file is imported, Then a second partner is created and the message `Could not retrieve a partner corresponding to '<name>' with the same tax identification number. A new partner was created.` is logged.

**EIDI-AC-143.** Given a matched partner established in Switzerland whose stored number is `CHE-123.456.789 MWST` and a file carrying `CHE123456789`, When the file is imported, Then the two are considered equal and no partner is created.

**EIDI-AC-144.** Given a file whose currency code is `CHF`, the code of the Swiss franc, and that currency is archived, When the file is imported, Then the currency is still used and the message `The currency 'CHF' is not active.` is logged.

**EIDI-AC-145.** Given a file whose currency code names no known currency, When the file is imported, Then the company currency is used and the message `Could not retrieve currency: <code>. Did you enable the multicurrency option and activate the currency?` is logged.

**EIDI-AC-146.** Given a vendor bill file carrying the payee account number `BE15001559627230`, When it is imported, Then that account is found on or created for the vendor and becomes the recipient bank account of the bill.

**EIDI-AC-147.** Given a customer invoice file carrying a payee account number, When it is imported into a sale journal, Then the account is attached to the company, not to the customer.

**EIDI-AC-148.** Given a file whose order reference identifier is empty, whose item descriptions contain the text `P00042`, and a purchase order sequence whose prefix is `P` and whose padding is five, When the file is imported, Then the origin of the produced bill is `P00042`.

**EIDI-AC-149.** Given a file with a note and a payment terms note, When it is imported, Then the terms and conditions of the produced document hold both texts, each escaped and wrapped in its own paragraph, in that order.

**EIDI-AC-150.** Given a file whose prepaid amount is `100.00`, When it is imported, Then the message `A payment of <the formatted amount> was detected.` is logged.

**EIDI-AC-151.** Given a file carrying two payment identifier elements holding `REF1` and `REF1` and `REF2`, When it is imported, Then the payment reference of the produced document is `REF1,REF2`.

**EIDI-AC-152.** Given a file carrying exactly one delivery terms code that names a known delivery terms record, When it is imported, Then that record is written on the document. Given two different codes, Then no delivery terms are written.

---

# 10. Import: line level values

**EIDI-AC-160.** Given a line whose line net amount is `950.00`, whose invoiced quantity is `1.0`, whose price amount is `1000.0` and whose line allowance amount is `50.00` with a base amount of `1000.00`, When the file is imported, Then the produced line has a quantity of `1.0`, a unit price of `1000.00` and a discount of `5.0` percent. *(Section 13 of [import-mapping.md](import-mapping.md))*

**EIDI-AC-161.** Given a line whose line net amount is `1000.00`, whose invoiced quantity is absent, whose price amount is `250.00` and whose base quantity is `5.0`, When the file is imported, Then the produced line has a quantity of `20.0` and a unit price of `250.00` so that the subtotal is `1000.00`.

**EIDI-AC-162.** Given a line whose line net amount is `0.00`, whose invoiced quantity is `0.0` and whose price amount is absent, When the file is imported, Then the produced line has a quantity of zero, a unit price of zero and a discount of zero, and, because its total including tax is zero and it carries no discount, it is dropped from the produced document.

**EIDI-AC-163.** Given a line whose item name is `Office Chair` and whose seller item identifier is `FURN_7777`, When the file is imported, Then the label of the produced line is `[FURN_7777] Office Chair`.

**EIDI-AC-164.** Given the same line plus two item description elements, When the file is imported, Then the label is `[FURN_7777] Office Chair` followed by a line break and the two descriptions joined by line breaks.

**EIDI-AC-165.** Given a line whose seller item identifier is `FURN_7777` and a product with that internal reference, When the file is imported, Then that product is on the produced line and no other matching strategy is consulted. *(Section 16 of [calculations.md](calculations.md))*

**EIDI-AC-166.** Given a line whose standard item identifier with the scheme `0160` is `1234567890128` and a product with that barcode, When the file is imported, Then that product is on the produced line, because the barcode strategy ranks before the internal reference strategy.

**EIDI-AC-167.** Given a line whose extended seller item identifier is `FURN_7777_BLACK` and a product variant with that internal reference, and the sales capability package installed, When the file is imported, Then the variant is matched rather than the template level product.

**EIDI-AC-168.** Given a line whose quantity element carries the unit code `DZN` and a product whose unit of measure is the unit, both belonging to the same reference unit, When the file is imported, Then the produced line carries the dozen.

**EIDI-AC-169.** Given a line whose quantity element carries the unit code `KGM` and a product whose unit of measure is the unit, which belong to different reference units, When the file is imported, Then the produced line carries no unit of measure and the message `The Unit of Measure 'kg' (from unit code 'KGM') was ignored on the line for product 'Office Chair' because it is not compatible with the product's Unit of Measure 'Units'. The unit of measure was left empty.` is logged.

**EIDI-AC-170.** Given a line whose classified tax category is `S` at twenty one percent and a document tax total carrying the same pair, and a purchase tax at twenty one percent existing in the company, When the file is imported, Then that tax is on the produced line.

**EIDI-AC-171.** Given a line whose classified tax category is `S` at twenty one percent while the document declares no tax total for that pair, When the file is imported, Then no tax value is produced for that line at all.

**EIDI-AC-172.** Given a line whose classified tax category resolves no tax, When the file is imported, Then the message `Could not retrieve the tax: 21.0 % for line '<the label>'.` is logged.

**EIDI-AC-173.** Given a line carrying a charge whose reason code is `AEO`, whose reason is `RECUPEL` and whose amount is `10.00`, on a line of quantity `10`, When the file is imported and a fixed amount tax of `1.00` named `RECUPEL` exists, Then that fixed tax is on the produced line, the unit price is reduced by `1.00` and the discount is recomputed so that the subtotal before the fixed tax is unchanged. *(Section 13 of [calculations.md](calculations.md))*

**EIDI-AC-174.** Given a line whose taxes include one whose price is included, When the file is imported, Then the unit price of the produced line is increased by the raw tax amount per unit divided by one minus the discount, so that the stated line net amount is reproduced.

---

# 11. Import: document level allowances, charges and corrections

**EIDI-AC-180.** Given a document level allowance whose amount is `50.00`, whose base amount is `1000.00`, whose multiplier factor is `5.0` and whose tax percentage is `21.0`, When the file is imported, Then one line is produced whose label is the reason, whose unit price is `-1000.00` and whose quantity is `0.05`.

**EIDI-AC-181.** Given a document level charge whose amount is `50.00` and that carries no base amount, When the file is imported, Then one line is produced whose unit price is `50.00` and whose quantity is `1`.

**EIDI-AC-182.** Given a file whose tax totals state `199.50` for the twenty one percent group, and a produced document whose computed tax amount for that group is `199.48`, When the corrections run, Then the tax lines are rewritten so that the tax amount is `199.50`, because the difference of `0.02` is within the tolerance of `0.03`. *(Section 14 of [calculations.md](calculations.md))*

**EIDI-AC-183.** Given the same file and a produced document whose computed tax amount is `199.44`, When the corrections run, Then nothing is corrected, because the difference of `0.06` exceeds the tolerance.

**EIDI-AC-184.** Given a file one of whose tax groups resolved no tax at all, When the corrections run, Then neither the tax amount correction nor the untaxed amount correction runs.

**EIDI-AC-185.** Given a file whose tax exclusive amount is `950.00` and whose payable rounding amount is `0.02`, and a produced document whose untaxed amount is `950.00`, When the corrections run, Then one line labelled `Rounding` is appended with a quantity of one, a unit price of `0.02` and no tax. *(Section 15 of [calculations.md](calculations.md))*

**EIDI-AC-186.** Given a file that carries an embedded printed document inside an additional document reference, When it is imported, Then that document is attached to the produced accounting document and becomes its main attachment when the current main attachment is a markup file.

**EIDI-AC-187.** Given a file with no embedded printed document, a produced vendor bill with no main attachment, and the parameter that disables the substitute rendering left at its default, When the import finishes, Then a printed document is rendered from the imported values, stamped with the word `Preview`, and becomes the main attachment.

**EIDI-AC-188.** Given any successful import, When the discussion thread of the produced document is read, Then exactly one message exists whose bold title is `Format used to import the invoice: <the display name of the profile>` and whose list holds the collected messages without duplicates.

**EIDI-AC-189.** Given an import of a document that arrived from the exchange network with the message identifier `abc-123`, When the discussion thread is read, Then the title is `Peppol invoice received` and the first list item is `Peppol document UUID: abc-123`, with the acronym expanded.

---

# 12. The exchange network: registration

**EIDI-AC-200.** Given a company with no country, When the registration is attempted, Then it is refused with `Please select a country for your company.`

**EIDI-AC-201.** Given a company with a country but no contact electronic mail address, When the registration is attempted, Then it is refused with `Contact email and phone number are required.`

**EIDI-AC-202.** Given a company whose mobile number is `0470123456` without an international prefix, When it is written, Then it is refused with `Please enter the mobile number in the correct international format.` followed by a line break and `For example: +32123456789, where +32 is the country code.`

**EIDI-AC-203.** Given a company whose participant identification is already published on the network by another provider named `Other Access Point`, When the registration as a receiver is attempted, Then it is refused with `A participant with these details has already been registered on the network. If you have previously registered to a Peppol service, please deregister.The Peppol service that is used is Other Access Point.` and the provider name is stored on the company.

**EIDI-AC-204.** Given a company in the `not_registered` state and a proxy that requires no identity verification, When the registration runs and the proxy answers with the state `sender`, Then a credential is created carrying the returned client identifier and token, a private key is generated, the participant state becomes `sender`, a welcome message is sent and the notification `You can now send electronic invoices via Peppol.` is shown.

**EIDI-AC-205.** Given a company in the `sender` state, When it asks to receive as well, Then the proxy is asked to publish it with the full target document type set, the migration key is cleared, the participant state becomes `smp_registration` and the participant state poll is scheduled in one hour.

**EIDI-AC-206.** Given a company in the `smp_registration` state, When the participant state poll answers `receiver`, Then the participant state becomes `receiver`.

**EIDI-AC-207.** Given a company in the `receiver` state, When it asks to stop receiving, Then the pending delivery states and the pending inbox are drained first, the proxy is asked to withdraw the publication and the participant state becomes `sender`.

**EIDI-AC-208.** Given a company in any state with a credential, When it deregisters, Then the pending states and the inbox are drained, the registration is cancelled, the participant configuration is reset, the credential is deleted and the participant state is `not_registered`.

**EIDI-AC-209.** Given a participant state poll that fails with the code saying the client is gone, When it runs, Then the participant configuration is reset and the credential is archived, and no other error causes a deregistration.

**EIDI-AC-210.** Given a company whose registration was rejected, When the participant settings are read, Then the participant state is `rejected` and no sending operation applies to it.

**EIDI-AC-211.** Given a company that already has an active credential of a different kind, When a registration on the exchange network is attempted, Then it is refused with `A connection to '<the other kind>' already exists.`

**EIDI-AC-212.** Given an archived credential of the same company and the same identification, When a new registration runs, Then the archived credential is not resurrected and exactly one active credential exists afterwards.

---

# 13. The exchange network: lookup and sending

**EIDI-AC-220.** Given a contact with the scheme `0208`, the endpoint `0246697724` and the billing profile chosen, and a lookup that returns that identification with a service whose document type identifier contains the billing customization identifier, When the verification runs, Then the verification state is `valid`.

**EIDI-AC-221.** Given the same contact and a lookup that returns the identification but no service carrying the billing customization identifier, When the verification runs, Then the verification state is `not_valid_format`.

**EIDI-AC-222.** Given a lookup that returns nothing, When the verification runs, Then the verification state is `not_valid`.

**EIDI-AC-223.** Given a contact with no endpoint, When the verification runs, Then the verification state is `not_verified` and no lookup is made.

**EIDI-AC-224.** Given a Belgian identification whose published service address belongs to the national pre-registration register, When the verification runs, Then the participant is considered not to exist and the state is `not_valid`.

**EIDI-AC-225.** Given a verification state that changes from `not_verified` to `valid`, When the change is written, Then one message is posted on the discussion thread of the contact showing both labels, the field name and the company name.

**EIDI-AC-226.** Given a posted customer invoice whose partner is a valid participant and whose company can send, When the sending wizard is used with the network method, Then the payload holds the file name, the receiver written as `0208:0246697724` and the file content encoded in base sixty four, the answer supplies a message identifier, the network state becomes `processing` and the delivery state poll is scheduled in five minutes.

**EIDI-AC-227.** Given three documents sent in one call, When the answer holds three message entries, Then the first entry is paired with the first document, the second with the second and the third with the third.

**EIDI-AC-228.** Given a document whose structured file is sixty five million bytes long, When the payload is built, Then the document is skipped with the error `Invoice <name> exceeds the size limit of 64 MB to be sent via Peppol.`, with the unit expanded.

**EIDI-AC-229.** Given a document for which no structured file could be produced and that has none stored, When the payload is built, Then the network state becomes `error` and the error title is `Errors occurred while creating the EDI document (format: <the profile description>):`, with the acronym expanded.

**EIDI-AC-230.** Given a partner whose verification state is not valid, When the sending wizard is confirmed with the network method, Then it is refused with `Partner doesn't have a valid Peppol configuration.`

**EIDI-AC-231.** Given a partner whose scheme is `0208`, whose endpoint is `0246697724` and whose verification state is not valid, and a lookup that would answer `valid` for the scheme `9925` with the endpoint `BE0246697724`, When the applicability of the network method is evaluated, Then the partner is rewritten with the scheme `9925` and that endpoint and is verified again.

**EIDI-AC-232.** Given a company whose operating mode is the acceptance mode, When the sending wizard is opened, Then the label of the network check box ends with ` (Test)`. Given the demonstration mode, Then it ends with ` (Demo)`.

**EIDI-AC-233.** Given a document sent over the network whose delivery state poll answers `done`, When the poll runs, Then the network state becomes `done`, the message `Peppol status update: done` is logged and the message identifier is acknowledged.

**EIDI-AC-234.** Given a delivery state poll that answers with the error code `702`, When the poll runs, Then nothing is written and the message identifier is not acknowledged, so the record is polled again.

**EIDI-AC-235.** Given a delivery state poll that answers with any other error, When the poll runs, Then the network state becomes `error`, the translated error message is logged and the identifier is acknowledged.

**EIDI-AC-236.** Given fifty one documents awaiting a delivery state and a batch size of fifty, When the poll runs, Then fifty are polled and the scheduled action is scheduled again in five minutes.

**EIDI-AC-237.** Given a document whose network state is `done`, When a reset to draft is attempted, Then the button is not offered and the document stays posted. When a resequencing is attempted, Then it is refused with `The following documents have already been sent and cannot be resequenced: %s` naming that document.

**EIDI-AC-238.** Given a document whose network state is `to_send` and that is not a draft, When the cancellation of the queued send is pressed, Then the network state and the queued sending data are cleared. Given a document whose network state is `processing`, Then the cancellation is refused with `Cannot cancel an entry that has already been sent to PEPPOL`.

---

# 14. The exchange network: receiving

**EIDI-AC-250.** Given a company in the `receiver` state with a reception journal and one waiting message holding an encrypted billing profile invoice, When the inbox poll runs, Then the message is decrypted, an attachment is created named after the message file name with the extension `xml`, a vendor bill is created in the reception journal carrying the message identifier and the state of the message, the file is imported into it, the identifier is acknowledged and the reception journal is notified.

**EIDI-AC-251.** Given a waiting message whose document type code is `389`, and a sale journal marked as a self billing journal, When the inbox poll runs, Then a customer invoice is created in that self billing journal.

**EIDI-AC-252.** Given the same message and no self billing sale journal, When the inbox poll runs, Then a customer invoice is created in the first sale journal of the company. Given no sale journal at all, Then nothing is created and the identifier is not acknowledged.

**EIDI-AC-253.** Given a waiting message whose identifier is already carried by an accounting document of the company and whose sender differs from its receiver, When the inbox poll runs, Then the identifier is acknowledged as a duplicate, no document is created and the discarded identifiers are logged.

**EIDI-AC-254.** Given a waiting message whose sender equals its receiver, that is a company invoicing itself, When the inbox poll runs, Then it is not treated as a duplicate and a vendor bill is created even though an outgoing invoice already carries that identifier.

**EIDI-AC-255.** Given a company in the `receiver` state with no reception journal, When the inbox poll runs as a scheduled action, Then a warning is logged and the credential is skipped. When it is run manually, Then it is refused with `Please set a journal for Peppol invoices on <company> before receiving documents.`

**EIDI-AC-256.** Given fifty one waiting messages and a batch size of fifty, When the inbox poll runs, Then fifty are fetched and the scheduled action is scheduled again.

**EIDI-AC-257.** Given an imported document whose partner had an unchecked verification state, When the inbox batch finishes, Then the participant lookup runs for that partner.

**EIDI-AC-258.** Given the business response package installed and an imported document that may be answered, When the inbox batch finishes, Then an acknowledgement response with the code `AB` is sent for it.

---

# 15. Business responses

**EIDI-AC-270.** Given an imported vendor bill that may be answered, When it is posted, Then an approval response with the code identifier `AP` is sent and the message `A Peppol response was sent to the Peppol Access Point declaring you accepted this document.` is logged.

**EIDI-AC-271.** Given the same bill, When it is cancelled, Then the rejection form opens carrying the result of the cancellation, the default reason is the code `UNR`, and confirming it sends a rejection response with the code `RE` and the chosen clarification list, after which the cancellation completes.

**EIDI-AC-272.** Given the rejection form with no reason selected, When it is confirmed, Then it is refused with `At least one reason must be given when rejecting a Peppol invoice.`

**EIDI-AC-273.** Given a response whose delivery state poll answers with the error code `207`, When the poll runs, Then the delivery state of the response becomes `not_serviced` and the identifier is acknowledged.

**EIDI-AC-274.** Given a response whose delivery state poll answers with any other error, When the poll runs, Then the delivery state becomes `error` and the message `Peppol business response error: <the error text>` is logged on the accounting document.

**EIDI-AC-275.** Given an inbound response document whose response code is `RE`, carrying one status whose reason code is `PRI` in the reason list and one whose reason code is `NIN` in the action list, When it is processed, Then a business response record is created with the code `RE`, and the accounting document receives the message `The Peppol receiver of this document has rejected it with the following information:` followed by a line break, the bold title `Reasons:`, a line break, `Prices incorrect`, a line break, the bold title `Suggested actions:`, a line break and `Issue new invoice`.

**EIDI-AC-276.** Given an inbound response whose only status carries a reason text that matches no shipped entry, When it is processed, Then that text appears under the bold title `Miscellaneous:`.

**EIDI-AC-277.** Given an accounting document with one delivered acknowledgement and one delivered approval, When its network state is derived, Then it is the approval code identifier `AP`. Given a delivered rejection as well, Then it is `RE`.

**EIDI-AC-278.** Given an accounting document that already carries a delivered approval, When the sending of a further response is attempted, Then the document is not eligible and no response is sent.

**EIDI-AC-279.** Given a receiver with no reception journal, When the scheduled action that keeps the published document types in step runs, Then the response transaction identifier is removed from the published set.

---

# 16. Credentials and recovery

**EIDI-AC-290.** Given a credential whose rotating token has expired, When a call is made, Then the token is renewed, the renewal is committed, the original call is retried once and it succeeds.

**EIDI-AC-291.** Given a credential whose renewal cannot take the lock because another process holds it, When the renewal is attempted, Then it is abandoned silently.

**EIDI-AC-292.** Given a call that fails with the invalid signature code, When it is handled, Then the credential is marked as out of step, its token is cleared, the proxy is told with a call signed with the participant key, and the refusal `Failed to connect to Peppol Access Point. This might happen if you restored a database from a backup or copied it without neutralization. To fix this, please go to Settings > Accounting > Peppol Settings and click on 'Reconnect this database'.` is raised.

**EIDI-AC-293.** Given a credential already marked as out of step, When any call is attempted, Then it is refused before any request is sent, with the same message.

**EIDI-AC-294.** Given a credential marked as out of step, When the reconnection is requested and the proxy accepts it, Then the synchronisation counter is increased by one, the returned token is stored, the out of step mark is cleared and the participant state poll is scheduled.

**EIDI-AC-295.** Given a credential marked as out of step, When the reconnection is requested and the proxy answers that the connection has been superseded, Then this database is disconnected and the refusal `This connection has been superseded by another database. Register again.` is raised.

**EIDI-AC-296.** Given a credential marked as out of step, When the disconnection of this database is requested, Then the participant configuration is soft reset, the credential is deleted and the connection held by the proxy is left untouched.

**EIDI-AC-297.** Given two active credentials for the same company, the same kind and the same operating mode, When the second is created, Then the creation is refused, because that combination must be unique among active credentials.

**EIDI-AC-298.** Given two credentials with the same client identifier, When the second is created, Then the creation is refused with `This id_client is already used on another user.`, expressed in the replacement as a refusal saying that the client identifier is already used by another credential.

**EIDI-AC-299.** Given a company operating in the demonstration mode, When any proxy call is made, Then no request leaves the deployment and the answer comes from the table of section 11 of [peppol-network.md](peppol-network.md).

**EIDI-AC-300.** Given a company operating in the demonstration mode that has just registered, When the inbox poll runs, Then exactly one demonstration vendor bill is created; and when it runs a second time, Then nothing is created, because a document already carries that message identifier.

**EIDI-AC-301.** Given a callback request carrying a token that fails verification, When the callback runs, Then nothing is triggered and the answer is still an empty success.

**EIDI-AC-302.** Given a callback request carrying a valid token, When the callback runs, Then the corresponding scheduled action is triggered and the answer is an empty success.

---

# 17. Certificates and keys

**EIDI-AC-310.** Given a certificate file in the binary encoding uploaded with no password, When it is loaded, Then the original format, the normalised certificate, the subject common name, the serial number and the validity window are filled.

**EIDI-AC-311.** Given a container file uploaded with the wrong password, When it is loaded, Then the load is refused with `This certificate could not be loaded. Either the content or the password is erroneous.`

**EIDI-AC-312.** Given a container file that holds a private key, When it is loaded, Then a key record is created for that private key, or an existing key of the same company with the same content is reused.

**EIDI-AC-313.** Given a bundle that holds an issuing certificate that is not yet known, When it is loaded, Then that issuing certificate is created as an archived record named after its subject common name followed by ` (certificate authority)`.

**EIDI-AC-314.** Given a certificate whose issuer common name matches two known candidates, When the issuer link is derived, Then the candidate with the furthest expiration date wins, and only when either the cryptographic verification succeeds or the authority key identifier of the certificate equals the subject key identifier of the candidate.

**EIDI-AC-315.** Given a certificate whose linked key does not match the key inside it, When it is saved, Then the save is refused.

**EIDI-AC-316.** Given a certificate outside its validity window, When a signature is requested, Then the request is refused.

**EIDI-AC-317.** Given a user who is not a system administrator, When a certificate or a key is read, Then access is refused.

---

# 18. Orders and point of sale receipts

**EIDI-AC-330.** Given a confirmed purchase order with one line of quantity two, unit price `100.00` and a tax at twenty one percent, When the order file is produced, Then the customization identifier is `urn:fdc:peppol.eu:poacc:trns:order:3`, the profile identifier is `urn:fdc:peppol.eu:poacc:bis:ordering:3`, the order type code is `105`, and the anticipated monetary total line extension amount is `200.00`.

**EIDI-AC-331.** Given a sales order with an expiration date, When the order file is produced, Then the order type code is `220` and the validity period end date carries that expiration date.

**EIDI-AC-332.** Given a purchase order line whose product has a vendor pricelist entry for that vendor carrying the vendor product code `V-7777` and the vendor product name `Vendor Chair`, When the order file is produced, Then the item name is `Vendor Chair` and the seller item identifier is `V-7777`.

**EIDI-AC-333.** Given an order file whose lines all match a product, When it is imported into a sales order, Then the partner comes from the buyer customer party, the customer reference comes from the document identifier, the origin comes from the quotation document reference, the shipping address comes from the delivery party, the discounts of the file are discarded and the unit price and the discount of every line are recomputed from the pricing rules.

**EIDI-AC-334.** Given an order file with one line whose product cannot be matched, When it is imported, Then the message `Could not retrieve the product named: <the label>` is collected, one message is posted on the order naming the profile, and an activity of the "to do" kind is created whose note is `Some information could not be imported:` followed by the list.

**EIDI-AC-335.** Given a point of sale receipt whose total is positive, with one line of quantity one, unit price `100.00` and a tax at twenty one percent, and an amount paid of `121.00`, When the receipt file is produced, Then the version identifier is `2.0`, the invoice type code is `380`, the tax exclusive amount is `100.00`, the prepaid amount is `100.00 - 121.00 = -21.00` and the payable amount is `121.00`.

**EIDI-AC-336.** Given a point of sale receipt whose total is negative, When the receipt file is produced, Then the document is a credit note and no invoice type code is written.

---

# 19. Permissions and visibility

**EIDI-AC-350.** Given a user who is only an internal user, When an Electronic Document is read, Then it is visible; When it is written, Then the write is refused. *(EIDI-RULE-330)*

**EIDI-AC-351.** Given a user of the invoicing group, When a registration wizard, a configuration wizard, a rejection wizard, a clarification code or a business response is created, read, written or deleted, Then the operation succeeds. *(EIDI-RULE-333)*

**EIDI-AC-352.** Given a user who is not a system administrator, When the payload attachment of an Electronic Document is read directly, Then access is refused; When the printed document of the accounting document is rendered, Then the payload is still embedded in it. *(EIDI-RULE-336)*

**EIDI-AC-353.** Given a credential belonging to a company that is neither the active company nor one of its ancestors, When credentials are listed, Then it is not visible. *(EIDI-RULE-334)*

**EIDI-AC-354.** Given a certificate with no company, When it is listed from any company, Then it is visible. *(EIDI-RULE-335)*

**EIDI-AC-355.** Given a customer using the customer portal, When the participant endpoint and the electronic address scheme are submitted, Then they are accepted; When any other participant field is submitted, Then it is ignored. *(EIDI-RULE-339)*

**EIDI-AC-356.** Given a customer using the customer portal who selects the network sending method with an endpoint that fails the validity rule of its scheme, When the address is submitted, Then the submission is refused with the message of that rule.

---

# 20. State machines: aggregation, edge cases and the two compatibility findings

These scenarios exercise the derivations and the edge cases specified in [state-machines.md](state-machines.md). They are stated separately from the lifecycle scenarios of sections 2 and 3 because they concern the values a replacement computes rather than the operations a user invokes.

**EIDI-AC-360.** Given a posted invoice carrying two delivery records of formats that both need a remote call, one in state `sent` and one in state `to_send`, When the aggregated delivery state of the accounting document is read, Then it is `to_send`, because the rule that requires every collected state to be `sent` did not hold and the rule that looks for `to_send` is evaluated before the one that looks for `to_cancel`.

**EIDI-AC-361.** Given a posted invoice carrying two delivery records of formats that both need a remote call, one in state `sent` and one in state `cancelled`, When the aggregated delivery state is read, Then it is **empty**, not `sent` and not `cancelled`, because both of the first two rules demand that the collected set hold exactly one value and no later rule matches.

**EIDI-AC-362.** Given a posted invoice carrying two delivery records of formats that both need a remote call, one in state `to_send` and one in state `to_cancel`, When the aggregated delivery state is read, Then it is `to_send`.

**EIDI-AC-363.** Given a posted invoice carrying only delivery records of formats that need no remote call, all in state `sent`, When the aggregated delivery state is read, Then it is empty, because only records of formats that need a remote call are collected.

**EIDI-AC-364.** Given a posted invoice with exactly one delivery record in error, carrying the error text `turlututu` at the warning blocking level, When the aggregated error message and the aggregated blocking level are read, Then the message is `turlututu` and the level is `warning`.

**EIDI-AC-365.** Given a posted invoice with three delivery records carrying an error text, of which one is at the error level and two are at the warning level, When the aggregated error message is read, Then it is `3 Electronic invoicing error(s)` and the aggregated level is `error`.

**EIDI-AC-366.** Given a posted invoice with two delivery records carrying an error text, both at the informational level, and a third record that carries no error text but is left at the warning level, When the aggregated error message is read, Then the message is `2 Electronic invoicing warning(s)` and the level is `warning`, even though neither record that carries a text is at the warning level. This is the **compatibility finding** recorded in section 4.2 of [state-machines.md](state-machines.md); a corrected behaviour would produce `2 Electronic invoicing info(s)` at the informational level.

**EIDI-AC-367.** Given a posted customer invoice of a company whose registration state is `receiver`, addressed to a commercial partner whose verification state for that company is `valid`, and whose network state is empty, When the network state is recomputed, Then it becomes `ready`.

**EIDI-AC-368.** Given the same invoice once it is in network state `processing`, When it is reset to draft, Then the network state is **not** cleared, because the document already counts as having left the platform; and When the reset is attempted from the interface, Then the reset button is not offered at all.

**EIDI-AC-369.** Given a customer invoice in network state `to_send` whose structured file would be seventy million bytes, When the sending service builds the payload, Then the sending result records `Invoice <the document number> exceeds the size limit of 64 MB to be sent via Peppol.` and the network state stays `to_send`. This is the **compatibility finding** recorded in section 5.7 of [state-machines.md](state-machines.md); a corrected behaviour would write `error` into the network state so that the document leaves the queue.

**EIDI-AC-370.** Given a customer invoice in network state `processing`, When the delivery state poll answers with an error object whose code is `702`, Then nothing is written, no log entry is created, and the message is not acknowledged, so the same message is polled again on the next run.

**EIDI-AC-371.** Given a customer invoice in network state `processing`, When the delivery state poll answers with an error object whose code is `500`, Then the network state becomes `error`, a log entry carrying the message built from the proxy error catalogue is written, and the message is acknowledged.

**EIDI-AC-372.** Given a sent customer invoice in network state `done`, When a business response carrying the code `AB` reaches delivery state `done`, Then the network state becomes `AB`; When a later business response carrying the code `AP` reaches delivery state `done`, Then the network state becomes `AP`; and When a later business response carrying the code `RE` reaches delivery state `done`, Then the network state becomes `RE`.

**EIDI-AC-373.** Given a sent customer invoice about which two business responses exist, one carrying `RE` in delivery state `done` and one carrying `AP` in delivery state `processing`, When the network state is recomputed, Then it is `RE`, because only responses in delivery state `done` are collected and the rejection code wins.

**EIDI-AC-374.** Given a received vendor bill that has been acknowledged and whose acknowledgement response is in delivery state `processing`, When the delivery state poll answers with an error object whose code is `207`, Then that response moves to `not_serviced`, no log entry is written, and the vendor bill can no longer be answered at all, so posting it sends no approval response.

**EIDI-AC-375.** Given a received vendor bill whose approval response is in delivery state `error`, When the bill is cancelled, Then the rejection form is still offered, because a response in the error state does not close the door.

**EIDI-AC-376.** Given a contact in Belgium whose electronic address scheme is `0208` and whose endpoint is `0477472701`, and whose lookup under that pair answers that the participant does not exist, When the verification runs, Then the alternative pair, scheme `9925` with endpoint `BE0477472701`, is tried; and when that pair answers `valid`, Then the scheme and the endpoint of the contact are rewritten to the alternative pair and the verification state becomes `valid`.

**EIDI-AC-377.** Given a contact whose verification state for the active company is `valid` and whose chosen profile is then changed to one the participant does not publish, When the verification runs again, Then the verification state becomes `not_valid_format` and a log entry is written in the discussion thread of the contact showing the old label, the new label, the label of the field and the display name of the company.

**EIDI-AC-378.** Given a certificate whose start of validity is tomorrow and whose end of validity is next year, When the validity flag is read today, Then it is false; When a signature is requested today, Then the request is refused; and When the flag is read tomorrow, Then it is true.

**EIDI-AC-379.** Given a certificate whose content is set and whose normalised certificate is empty because the upload could not be parsed, and that carries no loading error, When it is saved, Then the save is refused with `This certificate could not be loaded. Please provide the certificate password.`

**EIDI-AC-380.** Given a company in registration state `receiver`, When the downgrade to sender only is requested, Then the pending delivery states and the pending inbox are drained first, the proxy is then asked to remove the publication, and the registration state becomes `sender`; and When the inbox polling scheduled action next runs, Then that company is no longer among the companies polled.

**EIDI-AC-381.** Given a company in registration state `smp_registration`, When the upgrade to receiver is requested again, Then the request is refused with `Cannot register a user with a Can send, pending registration to receive application`, the placeholder carrying the translated label of the current state.

**EIDI-AC-382.** Given a company in registration state `receiver`, When the participant state poll answers with the draft state, Then the participant configuration is reset in its full form, the credential is archived rather than deleted, and the registration state becomes `not_registered`.

**EIDI-AC-383.** Given the same company, When the participant state poll instead fails with a transport error that is not the client-gone code, Then nothing is written and the failure is only logged, so that a transient failure of the proxy cannot deregister a working participant.

**EIDI-AC-384.** Given a company whose credential was archived by the client-gone handling and that holds no other credential of the same kind, When a further call is made on that archived credential and the proxy answers that no such user exists, Then the registration state becomes `not_registered`, the migration key is cleared, the change is committed and the refusal `We could not find a user with this information on our server. Please check your information.` is raised.
