# Electronic Invoicing and Document Exchange

This domain turns accounting documents into standardised machine readable business documents, delivers them to trading partners over a regulated document exchange network, receives incoming business documents and turns them back into accounting documents, and keeps a per document delivery state with its errors, retries and cancellations. It owns the electronic document framework (one electronic document record per accounting document per registered format), the structured export and import mapping for the universal business language family of formats and for the cross industry invoice format, the connection to the document exchange network through a proxy service, the cryptographic keys and certificates used to sign and decrypt exchanged payloads, and the exchange of sales and purchase orders and of point of sale receipts in the same structured formats.

## Business scope

1. **Electronic document framework.** Every journal declares the electronic document formats it produces. When an accounting document is posted, one electronic document record is created per applicable format. Formats that need no remote call are processed immediately; formats that need a remote call are queued and processed by a scheduled action in batches, with per batch locking, per document error text and a blocking level that decides whether processing continues. A cancellation request flow lets a user ask the remote service to withdraw an already sent document, call the request off again, or force the cancellation locally.
2. **Structured document generation.** A family of document builders produces a structured markup file from an accounting document. The family is layered: a generic universal business language builder, a European Norm 16931 compliance layer, a Peppol international billing layer, a European Union variant of that layer, then concrete country profiles. A separate builder produces the cross industry invoice format, which is additionally embedded inside the printed portable document so that one file carries both a human readable and a machine readable representation.
3. **Structured document interpretation.** Incoming structured files are recognised by their root element and their customization identifier, routed to the matching builder, and converted into a draft customer invoice or vendor bill: partner matching, bank account matching, currency, dates, references, notes, document level allowances and charges, lines with quantity, unit price and discount derived from the file, product matching, unit of measure matching, account prediction, tax matching and tax total correction.
4. **Network delivery.** A company registers as a participant on the Peppol document exchange network through a proxy service operated for the platform. Registration has its own state machine (not registered, sender, pending registration to receive, receiver, rejected). Outgoing documents are handed to the proxy in batches and tracked by a message identifier; incoming documents are polled from the proxy inbox, decrypted, saved as attachments and imported. Business level responses (acknowledgement, approval, rejection with reason and action codes) are sent back to the document originator and received from document recipients.
5. **Credentials.** The proxy connection is identified by a client identifier and authenticated by signing each request, either with a rotating shared secret or with the participant private key. Private keys and certificates are stored as first class records with validity dates, chain discovery and signing operations.
6. **Order and receipt interchange.** Sales orders and purchase orders are exported in the ordering profile of the same standard, incoming orders are imported into sales orders, and point of sale receipts are exported in the universal business language 2.1 syntax.

## Capabilities delivered

- Register a document format on a journal and create, queue, retry, cancel and force cancel electronic documents per accounting document.
- Generate a structured markup file for an accounting document in any of the shipped profiles, with complete party, reference, date, currency, payment, allowance and charge, tax breakdown and monetary total mapping.
- Block posting of an accounting document whose configuration cannot produce a valid file, and report every failed validation with its exact message.
- Embed the printed portable document and user supplied attachments inside the structured file, and embed the structured file inside the portable document.
- Import a structured file into a draft vendor bill or customer invoice with partner, bank account, product, unit of measure, account and tax matching, and with automatic correction of tax and untaxed totals against the totals stated in the file.
- Register a company on the document exchange network, verify whether a trading partner is reachable and supports the chosen document type, send documents, poll their delivery state and poll the inbox for new documents.
- Send and receive business level responses, including rejection with a coded reason list and a coded suggested action list.
- Generate and store cryptographic key pairs and certificates, discover certificate chains, and sign or decrypt payloads.
- Export sales orders and purchase orders, import orders into sales orders and purchase orders, and export point of sale receipts.

## Actors

| Actor | Responsibility in this domain |
|---|---|
| Accountant | Selects the format on a journal, posts documents, resolves configuration errors, retries failed sends, requests and calls off cancellations, reviews received documents. |
| Billing clerk | Sends a posted invoice with the network sending method selected, checks the delivery state, answers received documents with an acknowledgement, approval or rejection. |
| System administrator | Registers the company on the exchange network, manages the proxy user record, the private keys and the certificates, reconnects or disconnects a database whose credentials went out of step. |
| Scheduled job runner | Executes the sending scheduled action, the inbox polling scheduled action, the delivery state polling scheduled action, the participant state polling scheduled action, the webhook keep alive scheduled action and the service registration scheduled action. |
| Exchange network proxy | Remote counterpart that stores the participant registration, accepts outgoing payloads, returns message identifiers and delivery states, stores inbound payloads until acknowledged, and calls back into the platform through webhooks. |
| Trading partner access point | Remote counterpart of the trading partner that receives and acknowledges documents on the network. |

## Entities owned by this domain

The folder owns twenty nine entities. The table gives, for each of them, the canonical full name used in prose throughout this folder, the identifier used inside this folder, the transport name and the storage name that a compatible rebuild has to reproduce character for character, the kind, and the purpose. The generated reference page of each entity is linked from the transport name.

Every scope entity of this domain is owned here: none of them is a generic platform entity, so nothing in the candidate list is delegated to the platform foundation folder. The entities this domain merely extends are listed in the next section and are owned elsewhere.

| Canonical name | Identifier in this folder | Transport name | Storage name | Kind | Purpose |
|---|---|---|---|---|---|
| Electronic Document | `electronic_document` | [`account.edi.document`](../../references/entities/account.edi.document.md) | `account_edi_document` | stored record | One delivery record per accounting document per format, carrying the produced file, the state, the last error and the blocking level. |
| Electronic Document Format | `electronic_document_format` | [`account.edi.format`](../../references/entities/account.edi.format.md) | `account_edi_format` | stored record | A registered format that may be switched on per journal and that decides applicability, batching and remote call behaviour. |
| Electronic Interchange Proxy User | `electronic_interchange_proxy_user` | [`account_edi_proxy_client.user`](../../references/entities/account_edi_proxy_client.user.md) | `account_edi_proxy_client_user` | stored record | The company credential on a proxy service: client identifier, participant identification, private key, rotating token, operating mode. |
| Digital Certificate | `digital_certificate` | [`certificate.certificate`](../../references/entities/certificate.certificate.md) | `certificate_certificate` | stored record | A loaded certificate with validity window, subject name, serial number, issuer link and signing operations. |
| Digital Key | `digital_key` | [`certificate.key`](../../references/entities/certificate.key.md) | `certificate_key` | stored record | A loaded public or private key with signing, verification and decryption operations. |
| Peppol Business Response | `peppol_business_response` | [`account.peppol.response`](../../references/entities/account.peppol.response.md) | `account_peppol_response` | stored record | One business level response exchanged about one document, with its response code and its delivery state. |
| Peppol Clarification | `peppol_clarification` | [`account.peppol.clarification`](../../references/entities/account.peppol.clarification.md) | `account_peppol_clarification` | stored record | A shipped code from the rejection reason list or the suggested action list. |
| Peppol Rejection Wizard | `peppol_rejection_wizard` | [`account.peppol.rejection.wizard`](../../references/entities/account.peppol.rejection.wizard.md) | `account_peppol_rejection_wizard` | transient record | Collects the reasons and suggested actions for rejecting one or more received documents. |
| Peppol Registration Wizard | `peppol_registration_wizard` | [`peppol.registration`](../../references/entities/peppol.registration.md) | `peppol_registration` | transient record | Collects the participant identification, contact details and authentication choice used to register on the exchange network. |
| Peppol Configuration Wizard | `peppol_configuration_wizard` | [`peppol.config.wizard`](../../references/entities/peppol.config.wizard.md) | `peppol_config_wizard` | transient record | Advanced participant maintenance: contact email synchronisation, deregistration, downgrade to sender, upgrade to receiver. |
| Peppol Service | `peppol_service` | [`account_peppol.service`](../../references/entities/account_peppol.service.md) | `account_peppol_service` | transient record | One selectable document type of a registered receiver inside the configuration wizard. |
| Electronic Document Common Base | `electronic_document_common_base` | [`account.edi.common`](../../references/entities/account.edi.common.md) | `account_edi_common` | behaviour definition | Shared helpers: unit code mapping, electronic address scheme mapping, tax category prediction, exemption reason prediction, required field checks, import helpers. |
| Universal Business Language Base | `universal_business_language_base` | [`account.edi.ubl`](../../references/entities/account.edi.ubl.md) | `account_edi_ubl` | behaviour definition | Generic node builders and import steps for the universal business language syntax. |
| European Norm 16931 Layer | `european_norm_16931_layer` | [`account.edi.ubl_cen_en16931`](../../references/entities/account.edi.ubl_cen_en16931.md) | `account_edi_ubl_cen_en16931` | behaviour definition | Compliance rules of the European semantic invoice standard on top of the generic builder. |
| Peppol International Billing Layer | `peppol_international_billing_layer` | [`account.edi.ubl_pint`](../../references/entities/account.edi.ubl_pint.md) | `account_edi_ubl_pint` | behaviour definition | Restrictions of the international Peppol billing model: single note, single delivery, withholding reported as prepaid, taxable amount recomputed from line amounts. |
| Peppol International Billing European Union Layer | `peppol_international_billing_european_union_layer` | [`account.edi.ubl_pint_eu`](../../references/entities/account.edi.ubl_pint_eu.md) | `account_edi_ubl_pint_eu` | behaviour definition | Combination of the two previous layers with the European customization and profile identifiers. |
| Universal Business Language 2.0 | `universal_business_language_2_0` | [`account.edi.xml.ubl_20`](../../references/entities/account.edi.xml.ubl_20.md) | `account_edi_xml_ubl_20` | behaviour definition | The 2.0 syntax builder and the plain import mapping. |
| Universal Business Language 2.1 | `universal_business_language_2_1` | [`account.edi.xml.ubl_21`](../../references/entities/account.edi.xml.ubl_21.md) | `account_edi_xml_ubl_21` | behaviour definition | The 2.1 syntax builder. |
| Peppol Billing 3.0 | `peppol_billing_3_0` | [`account.edi.xml.ubl_bis3`](../../references/entities/account.edi.xml.ubl_bis3.md) | `account_edi_xml_ubl_bis3` | behaviour definition | The European business interoperability specification billing profile, version 3.0.12. |
| German Electronic Invoice Profile | `xrechnung` | [`account.edi.xml.ubl_de`](../../references/entities/account.edi.xml.ubl_de.md) | `account_edi_xml_ubl_de` | behaviour definition | German public sector profile on top of the billing profile. |
| Netherlands Standard Invoice Profile | `nlcius` | [`account.edi.xml.ubl_nl`](../../references/entities/account.edi.xml.ubl_nl.md) | `account_edi_xml_ubl_nl` | behaviour definition | Netherlands profile on top of the billing profile. |
| Australia and New Zealand Billing Profile | `australia_new_zealand_billing` | [`account.edi.xml.ubl_a_nz`](../../references/entities/account.edi.xml.ubl_a_nz.md) | `account_edi_xml_ubl_a_nz` | behaviour definition | Australia and New Zealand profile on top of the billing profile. |
| Singapore Billing Profile | `singapore_billing` | [`account.edi.xml.ubl_sg`](../../references/entities/account.edi.xml.ubl_sg.md) | `account_edi_xml_ubl_sg` | behaviour definition | Singapore profile on top of the billing profile. |
| Belgian Electronic Invoicing Profile | `e_fff` | [`account.edi.xml.ubl_efff`](../../references/entities/account.edi.xml.ubl_efff.md) | `account_edi_xml_ubl_efff` | behaviour definition | Belgian profile on top of the 2.0 syntax, used only for file naming and export. |
| Cross Industry Invoice Base | `cross_industry_invoice_base` | [`account.edi.cii`](../../references/entities/account.edi.cii.md) | `account_edi_cii` | behaviour definition | Shared node builders and import steps for the cross industry invoice syntax. |
| Cross Industry Invoice Profile | `cross_industry_invoice` | [`account.edi.xml.cii`](../../references/entities/account.edi.xml.cii.md) | `account_edi_xml_cii` | behaviour definition | The concrete cross industry invoice builder, version 2.2.0, used for both the French and the German hybrid invoice. |
| Sales Order Ordering Profile | `sale_ordering_profile` | [`sale.edi.xml.ubl_bis3`](../../references/entities/sale.edi.xml.ubl_bis3.md) | `sale_edi_xml_ubl_bis3` | behaviour definition | Export and import of sales orders in the ordering profile. |
| Purchase Order Ordering Profile | `purchase_ordering_profile` | [`purchase.edi.xml.ubl_bis3`](../../references/entities/purchase.edi.xml.ubl_bis3.md) | `purchase_edi_xml_ubl_bis3` | behaviour definition | Export and import of purchase orders in the ordering profile. |
| Point of Sale Receipt Profile | `point_of_sale_receipt_profile` | [`pos.edi.xml.ubl_21`](../../references/entities/pos.edi.xml.ubl_21.md) | `pos_edi_xml_ubl_21` | behaviour definition | Export of a point of sale receipt in the 2.1 syntax. |

"Behaviour definition" means a named set of rules and node builders that has no stored records of its own. A replacement may implement them as classes, strategy objects or configuration; what matters is that the layering and the override points described in this folder are preserved, because concrete country profiles are built by overriding single node builders.

## Entities from other domains that this domain extends

| Entity | Owning domain | What this domain adds |
|---|---|---|
| Journal | [general ledger](../general-ledger/entities.md) | `electronic_document_formats`, `compatible_electronic_document_formats`, `participant_state`, `is_exchange_reception_journal`, plus guards that forbid switching off a format that still has unprocessed documents and forbid changing the type of the reception journal. |
| Journal Entry | [general ledger](../general-ledger/entities.md) | `electronic_documents`, `electronic_document_state`, `electronic_document_error_count`, `electronic_document_blocking_level`, `electronic_document_error_message`, `electronic_document_pending_remote_formats`, three cancellation button visibility flags, `electronic_invoice_markup_attachment`, `electronic_invoice_markup_file`, `electronic_invoice_markup_filename`, `network_message_identifier`, `network_document_state`, `network_is_sent`, `network_responses`, `network_can_send_response`, and the posting, cancellation, reset to draft and resequencing guards. |
| Document Sending Service | [accounts receivable](../accounts-receivable/workflows.md) | Generation of the structured file before the printed document is rendered, embedding of the printed document into the structured file after rendering, alerts about missing participant configuration, and the network sending method. |
| Tax | [taxes](../taxes/entities.md) | `electronic_invoicing_tax_category_code`, `electronic_invoicing_tax_exemption_reason_code`, `electronic_invoicing_requires_exemption_reason`. |
| Contact | [contacts and organizations](../contacts-and-organizations/entities.md) | `peppol_electronic_address_scheme`, `peppol_endpoint`, `available_peppol_electronic_address_schemes`, `invoice_electronic_format` extension values, `is_universal_business_language_format`, `participant_verification_state`, `participant_supported_documents`, `participant_supports_responses`, `available_network_sending_methods`, `available_network_formats`. |
| Company | [contacts and organizations](../contacts-and-organizations/entities.md) | `electronic_interchange_proxy_users`, `participant_state`, `participant_contact_email`, `participant_phone_number`, `participant_migration_key`, `participant_proxy_user`, `peppol_electronic_address_scheme`, `peppol_endpoint`, `reception_journal`, `external_access_point_provider`, `participant_can_send`, `participant_parent_company`, `participant_metadata`, `participant_metadata_updated_on`. |
| Configuration Settings | [platform foundation](../platform-foundation/configuration.md) | The participant settings block and its buttons. |
| Report Action | [platform foundation](../platform-foundation/interfaces.md) | Embedding of format attachments and of the cross industry invoice file into the rendered portable document. |
| Attachment | [platform foundation](../platform-foundation/entities.md) | Deletion guard for attachments that are the payload of an electronic document sent to an authority. |
| Product Variant | [point of sale](../point-of-sale/entities.md) | Two extra product matching strategies during import, using the variant level item identifiers. |
| Purchase Order | [purchasing](../purchasing/workflows.md) | Export builder registration, file type recognition, decoder registration and activity creation for partially imported orders. |
| Sales Order | [sales](../sales/workflows.md) | Export builder registration, file type recognition, decoder registration and activity creation for partially imported orders. |
| Resequencing Wizard | [general ledger](../general-ledger/workflows.md) | Guard that forbids resequencing documents already sent through a remote service. |

## Cross domain dependencies

These domains must exist before this one:

1. [general ledger](../general-ledger/README.md): Journal, Journal Entry, Journal Item, posting, cancellation, reset to draft, fiscal lock dates, sequence and resequencing. The electronic document framework hooks into posting and cancellation.
2. [accounts receivable](../accounts-receivable/README.md): the Document Sending Service and its wizards, the printed invoice report, the sending method selection and the mail attachment handling.
3. [taxes](../taxes/calculations.md): the base line preparation, the tax detail computation, the aggregation helpers and the rounding helpers. Every monetary amount in an exported file is produced by those helpers; this domain only chooses the grouping keys and the rounding presentation.
4. [contacts and organizations](../contacts-and-organizations/README.md): Contact, Company, bank account records, country and state records, tax identification number validation.
5. [platform foundation](../platform-foundation/README.md): attachments, report rendering, scheduled actions, configuration parameters, access groups, record rules, signed hash helpers and the message bus.
6. [messaging and activities](../messaging-and-activities/README.md): the discussion thread used to log import results, delivery state changes and response messages, and the activity used to report partially imported orders.
7. [purchasing](../purchasing/README.md), [sales](../sales/README.md), [point of sale](../point-of-sale/README.md), [units of measure and packaging](../units-of-measure-and-packaging/README.md): only for the order and receipt interchange packages and for the unit code mapping.

## Reading order

A reader coming to this domain for the first time should read the files in this order.

1. This file, for the scope, the actors, the entities and the dependencies.
2. [glossary.md](glossary.md), because the whole domain turns on a small vocabulary of standard terms that are used without further explanation everywhere else.
3. [entities.md](entities.md), for the records and every one of their fields.
4. [state-machines.md](state-machines.md), for the eleven state fields, their transitions and their guards; it is the shortest complete map of what the domain actually does.
5. [workflows.md](workflows.md), for the end to end procedures that fire those transitions.
6. [business-rules.md](business-rules.md), for the numbered catalogue of everything the system refuses and the exact words it refuses with.
7. [calculations.md](calculations.md), for the arithmetic of the amounts written into a file and reconstructed from one.
8. The three mapping files, in the order in which a rebuild would need them: [universal-business-language-mapping.md](universal-business-language-mapping.md), [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md), [import-mapping.md](import-mapping.md).
9. [peppol-network.md](peppol-network.md), for the network, the proxy contract and the registration.
10. [configuration.md](configuration.md) and [interfaces.md](interfaces.md), for the settings, the shipped records, the scheduled jobs and the screens.
11. [accounting-effects.md](accounting-effects.md), for the boundary with the ledger.
12. [acceptance-criteria.md](acceptance-criteria.md), last, to check a rebuild against the numbered scenarios.

## Files in this folder

| File | Content |
|---|---|
| [entities.md](entities.md) | Every field of every owned entity and every field added to entities owned elsewhere, with types, defaults, derivations, constraints, validation messages and lifecycles. |
| [state-machines.md](state-machines.md) | The eleven state fields of the domain: every state with its stored value, label and meaning; every transition with origin, destination, trigger, guards in evaluation order and side effects; the exact refusal of every guard; and a diagram per machine. |
| [workflows.md](workflows.md) | End to end procedures: posting with format generation, the sending scheduled action, cancellation, sending an invoice over the network, receiving documents, business responses, registration, import of a received file, order interchange. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with exact messages, prefixed `EIDI-RULE-`. |
| [calculations.md](calculations.md) | Every formula: line amounts, discount inference, tax grouping and aggregation, monetary totals, rounding presentation, import price and discount reconstruction, tax amount correction. Each with a worked example. |
| [accounting-effects.md](accounting-effects.md) | The boundary statement: this domain writes no journal entries of its own, and the exact list of accounting side effects it does cause. |
| [configuration.md](configuration.md) | Every setting, parameter, shipped record, scheduled action, access group and record rule. |
| [interfaces.md](interfaces.md) | Service operations, request endpoints, the proxy call contract, reports, exported files, notifications, scheduled jobs and the screens described as workflows on views. |
| [acceptance-criteria.md](acceptance-criteria.md) | Given / When / Then scenarios covering every rule, transition and formula. |
| [glossary.md](glossary.md) | Domain terms in full words. |
| [universal-business-language-mapping.md](universal-business-language-mapping.md) | The complete element by element export mapping of the universal business language family, profile by profile. |
| [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md) | The complete element by element export mapping of the cross industry invoice format. |
| [import-mapping.md](import-mapping.md) | The complete import mapping and every matching strategy with its fallbacks. |
| [peppol-network.md](peppol-network.md) | Registration, participant lookup, sending, receiving, responses, the proxy call contract, the demonstration mode and the out of step recovery. |
