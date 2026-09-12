# Fiscal Localizations: Glossary

Terms used across this domain, in full words. Where a term is a translation of a national concept, the national name is given as a data value and the meaning is explained in full words.

---

**Accredited intermediary.** A private company licensed by a tax administration to receive documents on behalf of taxpayers, validate them and forward them to the administration. Several countries make an intermediary mandatory; the system then talks to the intermediary rather than to the administration.

**Account code padding.** Extending an account code on the right with zero characters until it reaches the template's declared code length. See [calculations.md](calculations.md) section 1.

**Account group.** A record that names a range of account code prefixes and gives that range a display name, so that a chart of accounts can be presented and reported as a hierarchy without the accounts themselves carrying a parent link.

**Advance tax payment account.** The account of a tax group where payments made to the administration before the return is filed are held. The closing entry clears it before deciding whether the period is payable or receivable.

**Archive channel.** In several countries, the channel used to issue a document to a counterpart that is not registered for the mandatory electronic channel. The document is still registered with the administration but is delivered to the counterpart by ordinary means.

**Base line.** A journal item that carries the taxable amount of a tax, as opposed to the tax line that carries the tax itself. A base line is reported on the return through its report tags.

**Carryover.** The mechanism by which an unusable balance of one return period, typically a credit that the administration does not refund, is stored as an external value and consumed by the following period.

**Cash basis tax.** A tax that becomes exigible when the invoice is paid rather than when it is issued. Between issuance and payment the amount sits on a cash basis transition account and carries no report tag.

**Cash basis transition account.** The holding account of a cash basis tax.

**Certification chain.** A sequence of documents in which each document carries a fingerprint computed over its own immutable data and over the fingerprint of the previous document, so that a missing or altered document can be detected.

**Chart of accounts template.** A named bundle of shipped configuration data for one country or one accounting plan: company-level defaults, accounts, account groups, taxes, tax groups, fiscal positions, journals and reconciliation models. A template is not a stored record; it is assembled at load time from the contributions of one or more capability packages.

**Code digits.** The total character length to which every account code of a template is padded.

**Company-qualified external identifier.** The stable name under which a record created by a template load is addressable: the accounting package's name, a dot, the company identifier, an underscore and the template's symbolic row identifier.

**Consolidated document.** One electronic document that carries many small receipts issued to unidentified consumers over a period, allowed by several countries as an alternative to issuing one document per receipt.

**Country group.** A named set of countries used by a rule that applies to a region rather than to one country, for example the European Union group used by intra-union tax rules and the Gulf Cooperation Council group used by the dual-language invoice layout.

**Debit note.** A document that increases an amount already invoiced, as opposed to a credit note that decreases it. Several countries require a distinct document class and numbering series for it.

**Declaration of intent.** An authorisation issued by a habitual exporter allowing its suppliers to invoice without value-added tax up to a declared threshold, within a declared validity window.

**Destination code.** The address of an invoice recipient inside a country's exchange system. A six-character code marks a public administration; a seven-character code marks a private business.

**Document type.** A legally defined class of invoice-like document, with its own code, its own numbering series and its own validation rules. Used across Latin America and, under other names, in Italy, Bulgaria and Greece.

**Dual-language layout.** A printed invoice layout that renders every label and every description in both the local language and English, required in the Gulf Cooperation Council countries.

**Emission point.** In Ecuador, the numbered outlet from which a document is issued. The number forms the middle part of the legal document number.

**Exchange proxy user.** The record that holds a company's registration with an exchange service: its identifier, its operating mode and its credentials.

**Exigibility.** The moment at which a tax becomes due to the administration: on invoice, or on payment.

**External value.** A figure stored against a report expression for a company and a date, either typed by an accountant or written by a carryover.

**Fingerprint.** A one-way value computed over a document's immutable data, used to detect alteration and to chain documents together. Also called a hash.

**Fiscal country.** The country whose tax rules a company follows. It defaults to the company's country and may differ from it.

**Fiscal device.** A physical unit certified by a tax administration that signs receipts and keeps its own counters. Kenya's registry flow uses one.

**Fiscal position.** A record that rewrites the taxes and the accounts of a document line, used to express a tax regime such as intra-union supply, export, reverse charge or a registration in another country.

**Foreign registration.** A tax registration a company holds in a country other than its fiscal country, recorded on a fiscal position that carries the foreign tax identification number and the country.

**Gross income tax.** In Argentina, a provincial turnover tax collected by withholding and by perception.

**Habitual exporter.** A taxpayer whose exports exceed a legal proportion of its turnover and who may therefore buy without value-added tax under a declaration of intent.

**Harmonised commodity code.** The internationally agreed classification of goods, required on Indian invoices under the name of the harmonised system of nomenclature, and used by other countries for customs declarations.

**Integrated tax, central tax and state tax.** The three components of the Indian goods and services tax. A supply inside one state carries the central and the state components; a supply across states carries the integrated component.

**Interchange document.** A record that holds the history of one submission of one document to one administration or intermediary: the payload, the transmission identifier, the state, the errors and the answer.

**Letter of undertaking.** An Indian authorisation that lets an exporter invoice at zero rate without paying the tax and claiming it back.

**Liquidity account.** An account that represents money: a bank account, a cash account, a credit card account, an outstanding receipts or outstanding payments account, or the inter-banks transfer account. A withholding line may not use one.

**Loading a template.** Materialising a chart of accounts template into per-company records. See [workflows.md](workflows.md) section 3.

**Numbering series.** A record that produces the next value of a counter, with a prefix and a suffix that may interpolate date parts, a padding width and an increment step.

**One-stop shop.** The European Union regime under which a business selling to private consumers in other member states declares and pays the destination country's tax through a single return in its own country.

**Outstanding receipts account and outstanding payments account.** The two intermediary accounts that hold a payment between the moment it is registered and the moment it is matched against a bank statement line.

**Perception.** A tax collected in advance on behalf of an administration by the seller, added to the amount due without changing the taxable base of the ordinary tax. Used in Argentina.

**Place of supply.** The jurisdiction whose tax applies to a transaction. In India it decides whether the integrated component or the central and state components apply.

**Point of sale number.** In Argentina and Ecuador, the number the administration assigns to an issuing outlet, which forms the first part of the legal document number.

**Purchase liquidation.** In Ecuador, a document a buyer issues on behalf of a seller who cannot issue one, for example an unregistered producer.

**Registration mark.** The identifier a tax administration returns when it accepts a document, which must be printed on the document and quoted in later correspondence.

**Reload.** Selecting a company's current template again, which performs a narrowed update that refreshes report tags and shipped configuration without destroying user data. See [workflows.md](workflows.md) section 4.

**Report tag.** A named marker attached to a tax repartition line and carried onto the journal items it produces, which a report expression sums to build one line of a tax return. Every tag exists as a positive and a negative variant.

**Repartition line.** An instruction that says which proportion of a tax's base or of a tax's amount goes to which account and carries which report tags, separately for an invoice and for a credit note.

**Responsibility type.** In Argentina, the tax category assigned to a person or legal entity, which together with the issuer's own category decides the letter and class of the document that may be issued.

**Reverse charge.** A regime in which the buyer, not the seller, accounts for the tax. The invoice carries no tax and the buyer books a matching debit and credit.

**Self-billed invoice.** A document the buyer issues on behalf of the seller. Several countries require a distinct document class for it.

**Simplified invoice.** A receipt that omits the customer's identification, allowed below a legal threshold.

**Storno accounting.** The convention by which a reversal is booked as a negative amount on the original side rather than as a positive amount on the opposite side, so that turnover figures are not inflated. Mandatory in eleven countries and optional in four.

**Symbolic row identifier.** The name a template gives to one of its rows, used to reference that row from another row before either exists in the database.

**Tax deducted at source and tax collected at source.** The two Indian withholding families: the first is withheld by the payer from a payment, the second is collected by the seller in addition to the price.

**Tax group.** A grouping of taxes used to subtotal a document and to decide the payable and receivable accounts of the closing entry.

**Tax lock date.** The date up to and including which no journal item carrying a tax may be created, modified or deleted, set when a return period is closed.

**Tax unit.** A group of companies that file one return under one registration.

**Template code.** The lowercase identifier of a chart of accounts template.

**Transport note.** A document that accompanies goods and states what is being moved, from where and to where. Italy's version is referenced on the invoice; India's and Romania's are declared to a portal.

**Unaffected earnings account.** The account of type current year earnings that holds the result of the year before it is allocated. Every company must have exactly one.

**Verifiable invoice registry.** The Spanish system in which every invoice is registered in a chain and carries a printed visual code that lets the recipient check the registration.

**Visual code.** A two-dimensional printed code that encodes a document's key data or a web address where it may be verified.

**Way bill.** A permit obtained before goods move, quoting the vehicle, the distance and the value of the goods. India's version is the electronic way bill; Romania's transport declaration serves the same purpose.

**Withholding tax.** A tax that the payer deducts from a payment and remits to the administration on the payee's behalf. It is not recognised when the invoice is posted but when the payment is registered.

**Withholding number.** The number of the certificate the payer must give the payee as proof of the amount withheld.

**Zero rated and exempt.** Two different things. A zero-rated supply carries a tax at zero percent and the input tax on it remains deductible. An exempt supply carries no tax and the input tax attached to it is generally not deductible. Country packages ship separate taxes for the two, with different report tags.
