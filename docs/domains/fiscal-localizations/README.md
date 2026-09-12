# Fiscal Localizations

The Fiscal Localizations domain makes a single accounting engine legally usable in any country. It supplies the country packages that carry a chart of accounts template, the tax definitions and their reporting tags, the tax groups, the fiscal positions and their automatic detection rules, the journals, the statutory report structures, the country-specific fields that must be captured on invoices, contacts and companies, the validation rules that block a non-compliant posting, the document types and numbering series demanded by tax administrations, and the electronic invoicing exchanges with government portals and document exchange networks. It also supplies the framework on which those packages are built: the template registry and the loading algorithm that turns a template into real records for one company, the generic financial report structure used by every tax return, the withholding tax framework, the fiscal country and foreign tax identification behaviors, and the extension patterns used by point of sale and website sales.

The domain does not define what a tax is, how a tax is computed, how a journal entry is balanced, or how a payment is reconciled. Those rules belong to the [Taxes](../taxes/README.md), [General Ledger](../general-ledger/README.md) and [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) domains. This domain defines the *country data* fed into those engines and the *country rules* layered on top of them.

## Business scope

1. **Template distribution.** Each supported country ships one or more chart of accounts templates. A template is a named, versioned bundle of shipped configuration data: company-level defaults, accounts, account groups, taxes, tax groups, fiscal positions, journals, reconciliation models, and financial report definitions. A template may declare a parent template and inherit and override the parent's rows.
2. **Template instantiation.** When a company selects a template, the system materialises the template into per-company records, padding account codes to the template's code length, resolving symbolic template references to database records through per-company external identifiers, writing company-level default accounts, creating utility accounts that no template names explicitly, and loading translations of every translatable field.
3. **Template reloading.** When a company that already uses a template selects the same template again, the system performs a narrowed update that preserves user-modified data, renames superseded taxes, relinks taxes to fiscal positions, refreshes report tags, and never deletes posted accounting.
4. **Statutory reporting.** A generic report structure (report, lines, expressions, columns, external values) expresses every tax return and every statutory financial statement as data. Six computation engines resolve the numbers. Carryover rules move an unusable balance from one period into the next.
5. **Country extensions.** Country packages add fields to invoices, contacts, companies, taxes, journals, products, units of measure, transfers and point of sale records; add constraints that run at posting time; add document types and numbering series; and add electronic invoicing flows that build a payload, transmit it, and record the administration's answer as a state on the invoice.
6. **Withholding taxes.** A framework for taxes that are not recognised when the invoice is posted but when the payment is registered, with per-line numbering series, a base amount and a withheld amount, a proration rule for partial payments, and a dedicated set of journal items in the payment entry.
7. **Foreign registration.** A company registered for tax in a country other than its own records that registration on a fiscal position, which unlocks the foreign country's tax return and lets the system instantiate the foreign country's taxes into the local chart of accounts.

## Actors

| Actor | Responsibility |
|---|---|
| System Administrator | Installs a country package, selects a chart of accounts template for a company, reloads a template. |
| Accounting Manager | Configures default accounts, activates and deactivates taxes, edits fiscal positions, sets the numbering series of document types, registers foreign tax identification numbers, closes tax periods. |
| Accountant | Issues invoices and vendor bills that carry country-specific fields, transmits electronic invoices, answers rejections, files tax returns. |
| Tax Administration Gateway | An external service that receives a document, validates it, and returns an acceptance, a rejection or a registration number. Represented in the system by an exchange state on the document and, in several countries, by a dedicated document record that keeps its own history. |
| Point of Sale Cashier | Issues fiscal receipts that must carry country-specific sequencing, signatures or certification data. |

## Entities owned by this domain

The domain owns the framework abstractions and every country-specific entity. Framework abstractions:

| Canonical name | Identifier | Kind | Purpose |
|---|---|---|---|
| Chart of Accounts Template | `account.chart.template` | abstract mechanism | Registry of country templates and the loading algorithm that instantiates one for a company. |
| Withholding Line | `account.withholding.line` | abstract | Common behavior of a line that withholds tax at payment time: base amount, withheld amount, numbering, account, currency conversion. |
| Payment Withholding Line | `account.payment.withholding.line` | persistent | A withholding line stored on a payment. |
| Payment Register Withholding Line | `account.payment.register.withholding.line` | transient | A withholding line being edited inside the payment registration wizard. |

Country-specific entities owned by this domain, grouped by the country that introduces them:

| Canonical name | Identifier | Kind | Purpose |
|---|---|---|---|
| Latam Document Type | `l10n_latam.document.type` | persistent | A legally defined invoice document class (invoice, credit note, debit note, receipt, export invoice) with its own numbering series and validation rules, used across Latin America. |
| Latam Identification Type | `l10n_latam.identification.type` | persistent | A legally defined class of person identification number (tax identification number, national identity number, passport, foreign identifier). |
| Latam Check | `l10n_latam.check` | persistent | A physical or electronic check tracked from issuance through delivery, deposit, rejection and cancellation. |
| Argentina Responsibility Type | `l10n_ar.afip.responsibility.type` | persistent | The tax responsibility category assigned to a person or legal entity by the Argentine tax administration. |
| Argentina Earnings Scale | `l10n_ar.earnings.scale` | persistent | A progressive scale used to compute income tax withholding. |
| Argentina Earnings Scale Line | `l10n_ar.earnings.scale.line` | persistent | One bracket of a progressive withholding scale. |
| Argentina Partner Tax | `l10n_ar.partner.tax` | persistent | A dated authorisation that a given withholding tax applies to a given contact. |
| Argentina Payment Register Withholding | `l10n_ar.payment.register.withholding` | transient | Withholding line inside the Argentine payment registration wizard. |
| Brazil Zip Range | `l10n_br.zip.range` | persistent | The postal code interval that identifies a Brazilian city. |
| Czech Tax Office | `l10n_cz.tax_office` | persistent | A territorial tax office with its workplace code, code, name and region. |
| Ecuador Payment Method | `l10n_ec.sri.payment` | persistent | A payment method code published by the Ecuadorian tax administration. |
| Egypt Activity Type | `l10n_eg_edi.activity.type` | persistent | An economic activity code required on Egyptian electronic invoices. |
| Egypt Signing Device | `l10n_eg_edi.thumb.drive` | persistent | A per-user signing device holding the certificate and personal identification number used to sign Egyptian electronic invoices. |
| Egypt Unit of Measure Code | `l10n_eg_edi.uom.code` | persistent | The mapping from an internal unit of measure to the Egyptian administration's unit code. |
| Spain Administrative Centre Role Type | `l10n_es_edi_facturae.ac_role_type` | persistent | The role of an administrative centre on a Spanish public sector invoice. |
| Spain Basque Country Document | `l10n_es_edi_tbai.document` | persistent | One submission to the Basque Country invoice registry, with its chain index, signature and state. |
| Spain Verifiable Invoice Document | `l10n_es_edi_verifactu.document` | persistent | One submission to the Spanish invoice registration service, with its chain data, state and errors. |
| France Sale Closing | `account.sale.closing` | persistent | An immutable daily, monthly or annual point of sale turnover closing with a cumulative grand total. |
| France Reporting Flow | `l10n.fr.pdp.reports.flow` | persistent | A periodic transaction or payment report addressed to the French public invoicing portal. |
| France Send Reporting Flow Wizard | `l10n.fr.pdp.reports.send.wizard` | transient | Confirmation step before transmitting a reporting flow that contains invalid invoices. |
| Greece Interchange Document | `l10n_gr_edi.document` | persistent | One submission to the Greek tax administration's invoice registry. |
| Greece Preferred Classification | `l10n_gr_edi.preferred_classification` | persistent | The income or expense classification pair remembered for a product. |
| Croatia Tax Category | `l10n.hr.tax.category` | persistent | A Croatian expense tax category with its international trade data code. |
| Croatia Product Category Code | `l10n_hr.kpd.category` | persistent | The Croatian product classification code carried by a product. |
| Croatia Interchange Addendum | `l10n_hr_edi.addendum` | persistent | Fiscalisation and business status data attached to a Croatian invoice. |
| India Electronic Way Bill | `l10n.in.ewaybill` | persistent | A goods movement permit obtained from the Indian portal, with transport data, validity and cancellation. |
| India Electronic Way Bill Type | `l10n.in.ewaybill.type` | persistent | A document type and sub-type accepted by the Indian way bill portal. |
| India Permanent Account Number Entity | `l10n_in.pan.entity` | persistent | The national tax account number shared by several contacts, carrying the aggregated withholding thresholds. |
| India Port Code | `l10n_in.port.code` | persistent | A customs port code used on export invoices. |
| India Section Alert | `l10n_in.section.alert` | persistent | A withholding section with its threshold and the alert raised when the threshold is crossed. |
| India Withhold Wizard | `l10n_in.withhold.wizard` | transient | Creation of a withholding entry against an invoice or a payment. |
| India Optional Holiday | `l10n.in.hr.leave.optional.holiday` | persistent | A date an employee may choose as a restricted holiday. |
| Indonesia Quick Response Transaction | `l10n_id.qris.transaction` | persistent | A payment initiated through the Indonesian quick response payment scheme. |
| Indonesia Electronic Invoice Document | `l10n_id_efaktur_coretax.document` | persistent | One Indonesian electronic invoice submission with its state and returned number. |
| Indonesia Product Code | `l10n_id_efaktur_coretax.product.code` | persistent | The Indonesian goods or services classification of a product. |
| Indonesia Unit of Measure Code | `l10n_id_efaktur_coretax.uom.code` | persistent | The Indonesian administration's unit code for a unit of measure. |
| Italy Transport Document | `l10n_it.ddt` | persistent | A goods transport note number and date referenced by an Italian invoice. |
| Italy Document Type | `l10n_it.document.type` | persistent | An Italian fiscal document type code. |
| Italy Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | persistent | A customer's habitual exporter declaration with a protocol number, a validity window and a consumable threshold. |
| Kenya Item Code | `l10n_ke.item.code` | persistent | A code that justifies a tax rate or exemption on a Kenyan invoice. |
| Malaysia Industry Classification | `l10n_my_edi.industry_classification` | persistent | The Malaysian industry classification of a company. |
| Malaysia Interchange Document | `myinvois.document` | persistent | One Malaysian portal submission, including consolidated receipts. |
| Peru District | `l10n_pe.res.city.district` | persistent | The third administrative level of a Peruvian address. |
| Poland Bank Account Verification | `l10n_pl.bank.account.verification` | persistent | The result of checking a supplier bank account against the national white list. |
| Poland Tax Office | `l10n_pl.l10n_pl_tax_office` | persistent | A Polish tax office with its code. |
| Romania Common Procurement Code | `l10n_ro.cpv.code` | persistent | A public procurement classification code carried by a product. |
| Romania Interchange Document | `l10n_ro_edi.document` | persistent | One Romanian portal submission with its state, index and answer. |
| Turkey Alias | `l10n_tr.nilvera.alias` | persistent | An electronic invoicing address registered for a Turkish contact. |
| Turkey Trailer Plate | `l10n_tr.nilvera.trailer.plate` | persistent | A trailer registration plate used on a Turkish electronic dispatch. |
| Turkey Tax Code | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | persistent | A Turkish administration tax code attached to a tax. |
| Turkey Tax Office | `l10n_tr_nilvera_einvoice_extended.tax.office` | persistent | A Turkish tax office. |
| Denmark Business Response | `nemhandel.response` | persistent | A business level response received for a Danish electronic document. |
| Vietnam Invoice Symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | persistent | An invoice series symbol allocated by the Vietnamese invoicing service. |
| Vietnam Invoice Template | `l10n_vn_edi_viettel.sinvoice.template` | persistent | An invoice template registered with the Vietnamese invoicing service. |

Wizards owned by this domain that drive a country workflow are listed in [interfaces.md](interfaces.md).

## Entities from other domains that this domain extends

| Host entity | Owning domain | What this domain adds |
|---|---|---|
| Company | [Contacts and Organizations](../contacts-and-organizations/README.md) | Chart of accounts template code, fiscal country, bank, cash and transfer account code prefixes, default accounts, country registration numbers, electronic invoicing credentials and modes, country reporting options. |
| Contact | [Contacts and Organizations](../contacts-and-organizations/README.md) | Identification type, responsibility type, foreign registration data, electronic invoicing addresses, national registers, withholding authorisations. |
| Journal Entry | [General Ledger](../general-ledger/README.md) | Document type, document number, country reporting codes, electronic invoicing state and attachments, exchange identifiers, withholding links, place of supply. |
| Journal Item | [General Ledger](../general-ledger/README.md) | Country classification codes, tax codes, withholding markers. |
| Journal | [General Ledger](../general-ledger/README.md) | Document type restrictions, numbering behavior, electronic invoicing flags, country office codes. |
| Tax | [Taxes](../taxes/README.md) | Withholding flags and numbering series, country exemption codes, country tax categories, report classification codes. |
| Tax Group | [Taxes](../taxes/README.md) | Country type codes used in tax returns. |
| Fiscal Position | [Taxes](../taxes/README.md) | Country classification used to choose a document type or a reporting code. |
| Financial Report and Financial Report Expression | [Taxes](../taxes/README.md) | Country availability and country-specific computation engines. |
| Payment and Payment Registration Wizard | [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) | Withholding lines, net amount, outstanding account selection, check management. |
| Product Template and Product Variant | [Products and Catalog](../products-and-catalog/README.md) | Country classification codes, commodity codes, service codes. |
| Unit of Measure | [Units of Measure and Packaging](../units-of-measure-and-packaging/README.md) | Country unit codes. |
| Transfer and Stock Move | [Inventory Operations](../inventory-operations/README.md) | Transport documents, way bills, electronic dispatch data. |
| Point of Sale Configuration, Order, Session, Payment Method | [Point of Sale](../point-of-sale/README.md) | Fiscal receipt numbering, certification hashes, electronic receipt transmission, simplified invoice limits. |
| Sales Order | [Sales](../sales/README.md) | Country fields propagated to the invoice. |
| Bank Account | [Contacts and Organizations](../contacts-and-organizations/README.md) | Country account number formats and validation. |

The complete field-by-field list of every added field is in [entities.md](entities.md).

## Cross-domain dependencies

The following domains must exist before this one can function:

1. [General Ledger](../general-ledger/README.md): Account, Account Group, Journal, Journal Entry, Journal Item, the posting engine, the sequence mixin that numbers documents, and the company record.
2. [Taxes](../taxes/README.md): Tax, Tax Repartition Line, Tax Group, Account Tag, Fiscal Position, the tax computation engine, and the financial report engine that tax returns are built on.
3. [Accounts Receivable](../accounts-receivable/README.md) and [Accounts Payable](../accounts-payable/README.md): customer invoice and vendor bill lifecycles that country rules constrain.
4. [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md): the payment record and the payment registration wizard that withholding lines extend.
5. [Electronic Invoicing and Document Exchange](../electronic-invoicing-and-document-exchange/README.md): the document builders, the exchange proxy client, and the document sending service that country flows specialise.
6. [Contacts and Organizations](../contacts-and-organizations/README.md): Contact, Company, Country, Country Subdivision, Country Group, Bank Account.
7. [the platform overview](../../overview/README.md): external identifiers, translations, scheduled jobs, attachments, report actions, access groups.
8. [Multi-Currency](../multi-currency/README.md): currency activation and conversion used when a template sets the company currency and when a withholding line converts a foreign currency amount.
9. [Products and Catalog](../products-and-catalog/README.md), [Units of Measure and Packaging](../units-of-measure-and-packaging/README.md), [Inventory Operations](../inventory-operations/README.md), [Point of Sale](../point-of-sale/README.md), [Sales](../sales/README.md): hosts of country-specific fields.

## Navigation

| File | Contents |
|---|---|
| [entities.md](entities.md) | Every field of every entity this domain owns, and every field this domain adds to entities owned elsewhere. |
| [workflows.md](workflows.md) | Template selection, template loading, template reloading, foreign tax instantiation, tax return preparation and closing, withholding at payment, electronic invoicing send and answer cycles, point of sale certification, with full state machine tables. |
| [business-rules.md](business-rules.md) | The numbered rule catalog with exact messages: template guards, account code rules, tax tag rules, fiscal position rules, foreign registration rules, document type rules, posting constraints per country. |
| [calculations.md](calculations.md) | Account code padding, template composition, report expression engines, carryover, withholding proration and currency conversion, progressive scale withholding, rounding, and every country formula with worked examples. |
| [accounting-effects.md](accounting-effects.md) | The journal entries produced by withholding at payment, by tax period closing, by cash basis taxes created from templates, and the account selection precedence used by fiscal positions. |
| [configuration.md](configuration.md) | Every setting, default, master data prerequisite, sequence, access group and scheduled job of the domain. |
| [interfaces.md](interfaces.md) | Service operations, request endpoints, screens described as workflows on views, printed documents, exported files, notifications and scheduled jobs. |
| [acceptance-criteria.md](acceptance-criteria.md) | Given/When/Then scenarios covering every rule, transition and formula. |
| [glossary.md](glossary.md) | Domain terms in full words. |
| [country-packages.md](country-packages.md) | The catalog of all shipped country packages, grouped by region, with template names, account counts, code lengths, taxes, tax groups, fiscal positions, added fields, validations, documents, exchange flows and numbering series. |
| [countries/](countries/) | One file per country with the complete detail of that country's package. |

### Country files

[Argentina](countries/argentina.md) ·
[Australia](countries/australia.md) ·
[Austria](countries/austria.md) ·
[Belgium](countries/belgium.md) ·
[Brazil](countries/brazil.md) ·
[Chile](countries/chile.md) ·
[Colombia](countries/colombia.md) ·
[Denmark](countries/denmark.md) ·
[Ecuador](countries/ecuador.md) ·
[Egypt](countries/egypt.md) ·
[Finland](countries/finland.md) ·
[France](countries/france.md) ·
[Germany](countries/germany.md) ·
[India](countries/india.md) ·
[Indonesia](countries/indonesia.md) ·
[Italy](countries/italy.md) ·
[Kenya](countries/kenya.md) ·
[Luxembourg](countries/luxembourg.md) ·
[Malaysia](countries/malaysia.md) ·
[Mexico](countries/mexico.md) ·
[Netherlands](countries/netherlands.md) ·
[Norway](countries/norway.md) ·
[Peru](countries/peru.md) ·
[Poland](countries/poland.md) ·
[Portugal](countries/portugal.md) ·
[Romania](countries/romania.md) ·
[Saudi Arabia](countries/saudi-arabia.md) ·
[Singapore](countries/singapore.md) ·
[Spain](countries/spain.md) ·
[Sweden](countries/sweden.md) ·
[Switzerland](countries/switzerland.md) ·
[Turkey](countries/turkey.md) ·
[United Arab Emirates](countries/united-arab-emirates.md) ·
[United Kingdom](countries/united-kingdom.md) ·
[United States](countries/united-states.md) ·
[Uruguay](countries/uruguay.md) ·
[Vietnam](countries/vietnam.md)
