# Configuration

Every setting, parameter, shipped record, scheduled action, access group, record rule and master data prerequisite of the Electronic Invoicing and Document Interchange domain, with its data type, its default and its effect. The records these settings act on are specified in [entities.md](entities.md); the procedures are in [workflows.md](workflows.md).

---

# 1. Capability packages

The domain is delivered as ten installable capability packages. A package is named here by its function; the deployment unit boundaries matter because several behaviours only exist when a given package is present.

| Package | Requires | What it adds |
|---|---|---|
| Electronic document framework | the general ledger | The Electronic Document and Electronic Document Format records, the format list on a journal, the delivery state on an accounting document, the cancellation flow, the sending scheduled action, the embedding of format attachments into the printed document, and the deletion guard on a payload attachment. |
| Structured invoice formats | the general ledger | The whole universal business language family and the cross industry invoice family, their export builders, their import decoders, the tax category and exemption fields on a tax, the participant endpoint and scheme on a contact, and the embedding of the structured file into the printed document. Installed automatically whenever the general ledger is installed. |
| Certificates and keys | the platform foundation | The Digital Certificate and Digital Key records and their operations. |
| Interchange proxy client | the general ledger and Certificates and keys | The Electronic Interchange Proxy User record, the request signing, the token renewal and the generic proxy call. |
| Document exchange network | Interchange proxy client and Structured invoice formats | The participant registration, the participant lookup, the sending over the network, the inbox polling, the delivery state polling, the callbacks and the demonstration mode. Restricted to the countries listed in section 3.3 of [peppol-network.md](peppol-network.md). |
| Business responses | Document exchange network | The Peppol Business Response and Peppol Clarification records, the rejection wizard, the response transaction in the published document types, and the automatic acknowledgement and approval. Installed automatically whenever the exchange network package is installed. |
| Advanced document fields | the general ledger and Structured invoice formats | Seven further reference fields on an accounting document, described in section 14 of [peppol-network.md](peppol-network.md). |
| Sales order interchange | sales and Structured invoice formats | The ordering profile for a sales order, its export, its import and the two variant product matching strategies. Installed automatically whenever both prerequisites are installed. |
| Purchase order interchange | purchasing and Structured invoice formats | The ordering profile for a purchase order, its export and its import. Installed automatically whenever both prerequisites are installed. |
| Point of sale receipt interchange | point of sale and Structured invoice formats | The receipt profile and its export. |

**Removing the structured invoice formats package** clears the chosen structured format from every contact that had selected one of the six shipped keys, so that no contact is left pointing at a format that no longer exists.

**Installing the interchange proxy client package** creates the configuration parameter that names the default operating mode, with the value `demo`, so that a fresh deployment never contacts a live network by accident.

---

# 2. Configuration parameters

| Parameter | Data type | Default | Effect |
|---|---|---|---|
| `account_edi_ubl_cii.disable_pdf_in_xml` | boolean, stored as text | `False` | When true, no substitute printed document is rendered for an imported vendor bill that arrived without one. |
| `account_edi_ubl_cii.use_new_dict_to_xml_helpers` | boolean, stored as text | `True` | Selects the generation procedure of the cross industry invoice family. The procedure specified in [cross-industry-invoice-mapping.md](cross-industry-invoice-mapping.md) is the one used when the parameter is true, which is the default. |
| `account.custom_templates_facturx_list` | text, a comma separated list of printed document template names | empty | Every named template also carries the cross industry invoice file inside the printed document it renders, for a single posted customer document. |
| `account_peppol.edi.mode` | selection stored as text: `prod`, `test`, `demo` | `demo` after the interchange proxy client package is installed | The operating mode used when no credential exists yet. A deployment loaded with demonstration data also receives this parameter with the value `demo`. |
| `account_edi_proxy_client.demo` | text | `demo` | Marks the deployment as one created with the demonstration mode already selected. |
| `database.uuid` | text | supplied by the platform | The database key sent to the proxy so that it can tell two deployments apart. |

---

# 3. Settings

## 3.1 The participant settings block

Shown in the accounting settings, per company. Every field is a view of a field of the company or of its credential; the underlying fields are specified in section 16 of [entities.md](entities.md).

| Setting | Data type | Default | Effect |
|---|---|---|---|
| participant role | selection: `sending_and_receiving` (Sending & Receiving), `sending_only` (Sending Only) | derived from the participant state | Writing `sending_only` on a company that is not already a sender asks the proxy to withdraw the publication. Writing `sending_and_receiving` on a company that is neither publishing nor published asks the proxy to publish it and then polls the participant state. |
| electronic address scheme | selection of ninety values, section 8 | derived from the country | The scheme half of the participant identification. |
| participant endpoint | text | derived from the scheme | The value half of the participant identification. |
| participant identification | text, read only | derived | The identification held by the credential, shown so that the user can compare it with the register. |
| contact electronic mail address | text | the electronic mail address of the company | Writing it also sends it to the proxy, when a credential exists. |
| mobile number | text | the telephone number of the company when it is a valid international number | Used for identity verification only. |
| migration key | text, readable only by a system administrator | empty | Lets an existing participant move to this access point. It is cleared as soon as it has been sent. |
| reception journal | many_to_one to Journal, restricted to purchase journals | the first purchase journal of the company once the company can send | Where documents received from the network are filed. Writing it marks that journal as the reception journal and clears the mark from every other purchase journal of the company. |
| reception journal required | boolean, derived | derived | True when the participant role is `sending_and_receiving`. |
| participant state | selection, the five values of section 1 of [peppol-network.md](peppol-network.md) | `not_registered` | The state machine of the registration. |
| external provider | text | empty | The name of the access point that already serves this identification, when the lookup found one that is not this platform. |
| operating mode | selection, read only: `prod`, `test`, `demo` | derived from the credential | The mode of the credential. |
| credentials out of step | boolean | false | When true, every call is refused until the connection is repaired. |
| uses the connection of a parent company | boolean, derived | derived | True when this company is a branch that shares the identification of an ancestor that can send. |
| name of that parent company | text, derived | derived | Shown next to the previous flag. |

**Buttons of the block:** open the registration form, reconnect this database, disconnect this database, deregister, register again, and disconnect this branch from its parent. The last one answers with the notification `Disconnected this branch company peppol configuration from <the parent name>.`

**The approved platform notice.** When the company is established in France and the approved platform package is not installed, the block shows `To use the Approved Platform for French E-Invoicing install the module '<the package name>'.` with a button that opens that package, or, when the package is not even listed, `To use the Approved Platform for French E-Invoicing install the module '<the package name>'.\nThe module was not found. Please update the app list first.` with a button that refreshes the package list.

## 3.2 The certificates and keys block

Shown in the general settings, visible only to a system administrator, per company. It holds two buttons, one opening the certificate list and one opening the key list. There is no stored setting.

## 3.3 Settings on a journal

| Setting | Data type | Default | Effect |
|---|---|---|---|
| electronic document formats | many_to_many to Electronic Document Format | derived, writable | The formats produced for documents posted in this journal. Only the compatible formats may be chosen, and a format that still has unprocessed documents may not be removed. |
| is the exchange reception journal | boolean | false | Marks the single purchase journal that receives documents from the network. |

## 3.4 Settings on a tax

| Setting | Data type | Default | Effect |
|---|---|---|---|
| tax category code | selection of ten values, section 9 | empty | Overrides the predicted tax category code. |
| tax exemption reason code | selection of the ninety one codes of the published exemption code list | empty | Overrides the predicted exemption reason code. Shown only when the chosen category requires a reason. |
| the category requires an exemption reason | boolean, derived | derived | True for the categories that demand a reason. |

## 3.5 Settings on a contact

| Setting | Data type | Default | Effect |
|---|---|---|---|
| structured format | selection extended with seven values, section 10 | empty | The format used for this contact. |
| electronic address scheme | selection of ninety values, section 8 | derived from the country | The scheme half of the participant identification of the trading partner. |
| participant endpoint | text | derived from the scheme | The value half. |
| sending method | selection extended with the network method | the ordinary default of the accounts receivable domain | The network method is offered only when the active company is established in a country where the network is available. |

Both the scheme and the endpoint are writable from the customer portal, so that a customer can declare how it wants to be invoiced. The portal additionally requires, when the customer selects the network sending method, that its country is one where the network is available, that the endpoint passes the validity rules, and that the resulting verification state is `valid`; otherwise it answers `That country is not available for Peppol.`, the endpoint message of the failing rule, or `If you want to be invoiced by Peppol, your configuration must be valid.`

---

# 4. Master data prerequisites

| Prerequisite | Why it is needed |
|---|---|
| a company with a country, a name, a street, a city, a postal code, a tax identification number, a contact electronic mail address, a telephone number and a mobile number | every export validation and the registration require them |
| a participant identification on the company | the registration and the endpoint element of every exported file |
| a recipient bank account on a customer invoice | required by the European semantic standard when the payment means is a credit transfer, and by the cross industry invoice family for every customer invoice |
| at least one purchase journal | to receive documents from the network |
| one purchase journal marked as the reception journal | the inbox polling refuses to run without it |
| a sale journal marked as a self billing journal, when self billed documents are received | otherwise a self billed document falls back to the first sale journal of the company |
| a tax on every invoice line that requires one | every export validation refuses a line without a tax |
| a tax category code on a tax whose category the prediction would get wrong | the prediction is documented in section 10 of [calculations.md](calculations.md) and is explicitly described as approximate |
| a unit of measure whose stable name is in the unit code table | otherwise the unit code `C62` is written |
| an internal reference or a barcode on a product | the item identification elements, and the product matching at import |

---

# 5. Shipped records

| Record kind | Shipped records |
|---|---|
| rejection reason codes | the fourteen entries of section 13.4 of [peppol-network.md](peppol-network.md) |
| suggested action codes | the seven entries of section 13.5 of [peppol-network.md](peppol-network.md) |
| printed document template | the preview template used when an imported vendor bill arrives without a printed document, together with its layout and its report action named "Invoice report generated from the received file" |
| archival metadata block | the metadata written into a printed document after it has been converted to the archival variant |
| electronic mail layouts | the layout that adds the network footnote to an invoice message, for the countries listed in section 3.3 of [peppol-network.md](peppol-network.md) |
| welcome message template | sent to a company that has just become a sender or a receiver |
| default value | the participant verification state of a contact defaults to `not_verified`, set as a default value record for every company that is created |
| configuration parameters | the six parameters of section 2 |
| server action | the "(un)group lines by tax" action, bound to the form of an accounting document and reserved to the invoicing group |

---

# 6. Scheduled actions

| Scheduled action | Interval | Active on installation | What it does | Retrigger rule |
|---|---|---|---|---|
| process the electronic documents that need a remote call | every day | no | Runs the sending and cancellation batches, with a limit of twenty jobs per run | schedules itself again as soon as possible when jobs were left over |
| retrieve new documents from the network | every four hours | yes | Polls the inbox of every receiver credential | schedules itself again as soon as possible when more messages than the batch size were waiting |
| update the message state | every day | yes | Polls the delivery state of every document and every response awaiting one | schedules itself again in five minutes when more records than the batch size were waiting |
| update the participant state | every week | yes | Polls the participant state of every credential | schedules itself again in one hour while any company is in the `smp_registration` state |
| keep the callbacks alive | every two weeks | yes | Re-registers the callback address and a fresh token for every company in the `sender` or `receiver` state | none |
| keep the published document types in step | every nine hundred and ninety nine months, which means it effectively only runs when triggered | yes | Compares the published set of every receiver with the target set and corrects it | schedules itself again in four hours when any receiver failed |

The last one is also triggered whenever the reception journal of a company is written. The three polling actions are also triggered by the corresponding callback.

**Batch sizes.** The sending scheduled action takes twenty jobs per run. The three polling actions take fifty records per run; a caller may override that number through the execution context, which is how a deployment that handles very large documents lowers it.

**Limits.** A single payload handed to the network may not exceed sixty four million bytes.

---

# 7. Access groups and record rules

## 7.1 What each group may do

| Group | Electronic Document Format | Electronic Document | Credential | Certificate and key | Registration, configuration and rejection wizards, service lines, clarification codes, business responses |
|---|---|---|---|---|---|
| any internal user | read | read | none | none | none |
| invoicing group | read, create, write, delete | read, create, write, delete | read | none | read, create, write, delete |
| system administrator | read, create, write, delete | read, create, write, delete | read, create, write, delete | read, create, write, delete | inherited from the invoicing group |

## 7.2 Record rules

| Record | Rule |
|---|---|
| Electronic Interchange Proxy User | visible when its company is the active company or one of its ancestors |
| Digital Certificate | visible when it has no company, or when its company is the active company or one of its ancestors |
| Digital Key | visible when it has no company, or when its company is the active company or one of its ancestors |

## 7.3 Field level visibility

| Field | Restriction |
|---|---|
| the payload attachment of an Electronic Document | readable only by a system administrator; ordinary users reach it through the accounting document |
| the rotating token of a credential | readable only by a system administrator |
| the migration key of a company | readable only by a system administrator |

## 7.4 Menu visibility

The list of credentials is reachable only by a user who has switched the technical features on. The certificate list and the key list are reachable from the general settings, which are visible only to a system administrator. The electronic document page of an accounting document is visible only to a user who has switched the technical features on, and only when the document has at least one electronic document.

---

# 8. The electronic address scheme catalogue

The ninety values offered on a contact and on a company, in the order in which they are declared. A value marked as withdrawn is not offered any more but stays visible on a contact that already carries it.

| Code | Label | Withdrawn |
|---|---|---|
| `9923` | Albania value added tax number | no |
| `9922` | Andorra value added tax number | no |
| `0151` | Australia business number | no |
| `9914` | Austria identification number | no |
| `9915` | Austria government identification code | no |
| `0208` | Belgian company registry number | no |
| `9925` | Belgian value added tax number | no |
| `9924` | Bosnia and Herzegovina value added tax number | no |
| `9926` | Bulgaria value added tax number | no |
| `9934` | Croatia value added tax number | no |
| `9928` | Cyprus value added tax number | no |
| `9929` | Czech Republic value added tax number | no |
| `0096` | Denmark production unit number | no |
| `0184` | Denmark central business register number | no |
| `0198` | Denmark value added tax register number | no |
| `0191` | Estonia company code | no |
| `9931` | Estonia value added tax number | no |
| `0037` | Finland business identity code | yes |
| `0216` | Finland electronic invoicing address | no |
| `0213` | Finland value added tax number | yes |
| `0002` | France company identification register | no |
| `0009` | France establishment identification register | no |
| `9957` | France value added tax number | no |
| `0225` | France electronic invoicing address | no |
| `0240` | France register of legal persons | no |
| `0246` | German electronic business address | no |
| `0204` | Germany public sector routing identifier | no |
| `9930` | Germany value added tax number | no |
| `9933` | Greece value added tax number | no |
| `9910` | Hungary value added tax number | no |
| `0196` | Iceland identification number | no |
| `9935` | Ireland value added tax number | no |
| `0211` | Italy value added tax number | no |
| `0097` | Italy interchange identifier | no |
| `0188` | Japan corporate number | no |
| `0221` | Japan invoice issuer registration number | no |
| `0218` | Latvia unified registration number | no |
| `9939` | Latvia value added tax number | no |
| `9936` | Liechtenstein value added tax number | no |
| `0200` | Lithuania legal entity code | no |
| `9937` | Lithuania value added tax number | no |
| `9938` | Luxembourg value added tax number | no |
| `9942` | North Macedonia value added tax number | no |
| `0230` | Malaysia identification number | no |
| `9943` | Malta value added tax number | no |
| `9940` | Monaco value added tax number | no |
| `9941` | Montenegro value added tax number | no |
| `0106` | Netherlands chamber of commerce number | no |
| `0190` | Netherlands public body identification number | no |
| `9944` | Netherlands value added tax number | no |
| `0244` | Nigeria tax identification number | no |
| `0192` | Norway organisation number | no |
| `9945` | Poland value added tax number | no |
| `9946` | Portugal value added tax number | no |
| `9947` | Romania value added tax number | no |
| `9948` | Serbia value added tax number | no |
| `0195` | Singapore unique entity number | no |
| `0245` | Slovakia tax identification number | no |
| `9949` | Slovenia value added tax number | no |
| `9950` | Slovakia value added tax number | no |
| `9920` | Spain value added tax number | no |
| `0007` | Sweden organisation number | no |
| `9955` | Sweden value added tax number | yes |
| `9927` | Switzerland value added tax number | no |
| `0183` | Switzerland enterprise identification number | no |
| `9952` | Turkey value added tax number | no |
| `0235` | United Arab Emirates tax identification number | no |
| `9932` | United Kingdom value added tax number | no |
| `9959` | United States employer identification number | no |
| `0060` | Universal numbering system identifier | no |
| `0088` | International article number location code | no |
| `0130` | Directorates of the European Commission | no |
| `0135` | Object identifiers of the Italian banking association | no |
| `0142` | Object identifiers of the Italian interbank services company | no |
| `0193` | Belgian universal business language party identifier | yes |
| `0199` | Legal entity identifier | no |
| `0201` | Italian public administration organisational unit code | no |
| `0202` | Italian certified electronic mail address | no |
| `0209` | Global standards identification keys | no |
| `0210` | Italian fiscal code | no |
| `9913` | Business registers network identifier | no |
| `9918` | Society for worldwide interbank financial telecommunication identifier | no |
| `9919` | Austrian enterprise register number | no |
| `9951` | San Marino value added tax number | no |
| `9953` | Vatican value added tax number | no |
| `AN` | File transfer protocol of the European automotive industry | no |
| `AQ` | Message handling address for mail text | no |
| `AS` | Applicability statement exchange address | no |
| `AU` | File transfer protocol address | no |
| `EM` | Electronic mail address | no |

The exchange network package adds one further value, `odemo`, labelled as the demonstration identifier, which is offered only while the company operates in the demonstration mode.

Which of these values a given country prefers, and which field of the contact supplies the endpoint value, is in section 3.1 of [peppol-network.md](peppol-network.md).

---

# 9. Tax category codes

The ten values offered on a tax.

| Code | Meaning |
|---|---|
| `S` | standard rate |
| `Z` | zero rated goods |
| `E` | exempt from tax |
| `AE` | reverse charge |
| `K` | intra-community supply |
| `G` | free export item, tax not charged |
| `O` | services outside the scope of tax |
| `L` | Canary Islands general indirect tax |
| `M` | tax of Ceuta and Melilla |
| `B` | transferred, for the Italian split payment regime |

The categories that demand an exemption reason are `E`, `AE`, `K`, `G` and `O`; only for those is the exemption reason code field shown on the tax.

---

# 10. The structured formats

| Format key | Label shown to the user | Countries for which it is offered | Accepted by the exchange network | Ordering sequence | Embeds attachments |
|---|---|---|---|---|---|
| `ubl_bis3` | EU Standard (Peppol Bis 3.0) | every country where the network is the default sending method, listed in section 3.3 of [peppol-network.md](peppol-network.md) | yes | 200 | yes |
| `xrechnung` | Germany (XRechnung) | Germany | yes | 200 | no |
| `ubl_a_nz` | Australia (BIS Billing 3.0 A-NZ) | New Zealand, Australia | no | 100 | no |
| `nlcius` | Netherlands (NLCIUS) | the Netherlands | yes | 100 | no |
| `ubl_sg` | Singapore (BIS Billing 3.0 SG) | Singapore | no | 100 | no |
| `facturx` | France (FacturX) | France | no | 100 | no |
| `zugferd` | Germany (ZUGFeRD) | Germany | no | 100 | no |

A format that declares no ordering sequence is treated as if it declared one hundred. When a country offers several formats, the one with the smallest sequence wins, except that a contact whose electronic address scheme is the German public sector routing identifier `0204` always resolves to the German electronic invoice profile.

---

# 11. Presentation precision

| Element family | Minimum decimal places | Maximum decimal places |
|---|---|---|
| every monetary amount of the universal business language family, except the unit price | the decimal places of the currency | none |
| the unit price of the universal business language family | 1 | 10 |
| every monetary amount of the cross industry invoice family, except the allowance and charge amounts | 2 | 2 |
| the allowance and charge amounts of the cross industry invoice family | 2 | the decimal places of the currency |
| the unit price of the plain builder | the product price precision | the same |

The rounding sequence that turns an amount into text is in section 1 of [calculations.md](calculations.md).
