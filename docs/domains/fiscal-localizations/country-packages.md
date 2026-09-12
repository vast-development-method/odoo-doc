# Fiscal Localizations: Country Package Catalogue

Every capability package this domain ships, what it contributes, and the chart data it carries.
Two hundred and twenty-nine packages are covered: one hundred and sixteen country base packages,
ninety-six country add-ons and seventeen packages that belong to a shared family rather than to one
country.

A package's technical name is reproduced in code font because it is the key a package manager, a
dependency graph and the machine-readable catalogues under `schemas/` use. Its business name is
written in words. Template codes are reproduced in code font because they are stored on the company
and read by every reload.

The generic mechanisms these packages build on are specified once and not repeated per package:
the template registry and the loading algorithm in [workflows.md](workflows.md), the fields each
package adds in [entities.md](entities.md), the rules in [business-rules.md](business-rules.md), the
arithmetic in [calculations.md](calculations.md), the ledger effects in
[accounting-effects.md](accounting-effects.md), the settings and shipped data in
[configuration.md](configuration.md), the operations and printed documents in
[interfaces.md](interfaces.md) and the state machines in [state-machines.md](state-machines.md).

Countries whose package is large enough to need its own treatment have a file of their own under
[countries/](countries/); this catalogue names it in the row of the package.

---

## 1. How to read a package row

| Column | Meaning |
|---|---|
| Package | The technical name, reproduced exactly. |
| Contributes | What installing the package adds. A base package always adds a chart of accounts template, its taxes, its tax groups and its journals; the column states what else it adds. |
| Depends on | The packages that must be installed first, reproduced exactly. A dependency on `account` means the accounting engine, on `point_of_sale` the point of sale, on `stock` inventory, on `sale` sales, on `website_sale` the online store, on `account_edi_ubl_cii` the shared payload builders and on `account_edi_proxy_client` the exchange proxy. |

A package that ships a chart of accounts template also appears in section 4 with the number of rows
in each of its data files.

---

## 2. Shared families

These packages are not tied to one country. Several of them are the base on which a group of country packages is built.

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_account_withholding_tax` | Withholding tax framework | The country-neutral framework for taxes that become due when the payment is registered rather than when the invoice is posted: the withholding line, its numbering, its base and withheld amounts, its proration and the journal items it produces. | `account` |
| `l10n_account_withholding_tax_pos` | Withholding tax framework for point of sale | Extends the withholding framework to point of sale payments. | `l10n_account_withholding_tax`, `point_of_sale` |
| `l10n_anz_ubl_pint` | Australian and New Zealand invoice profile | Builds the international invoice payload profile adopted by Australia and New Zealand. | `account_edi_ubl_cii` |
| `l10n_din5008` | Standard letter format | The national standard for the layout of a business letter: the address block, the reference block and the fold marks, applied to every printed document. | `account` |
| `l10n_din5008_expense` | Standard letter format for expenses | Applies the letter format to expense reports. | `l10n_din5008`, `hr_expense` |
| `l10n_din5008_purchase` | Standard letter format for purchasing | Applies the letter format to purchase orders and requests for quotation. | `l10n_din5008`, `purchase` |
| `l10n_din5008_repair` | Standard letter format for repairs | Applies the letter format to repair orders. | `l10n_din5008`, `repair` |
| `l10n_din5008_sale` | Standard letter format for sales | Applies the letter format to quotations and sales orders. | `l10n_din5008`, `sale` |
| `l10n_din5008_stock` | Standard letter format for inventory | Applies the letter format to delivery notes and transfers. | `l10n_din5008`, `stock` |
| `l10n_eu_oss` | European one-stop shop | Generates, for the one-stop shop regime, one fiscal position and one destination tax per member state and per published rate. | `account` |
| `l10n_gcc_invoice` | Gulf invoice requirements | The bilingual invoice layout, the visual code and the identification fields the Gulf states share. | `account` |
| `l10n_gcc_invoice_stock_account` | Gulf invoice requirements for inventory | Adds the delivery reference the Gulf invoice must carry. | `l10n_gcc_invoice`, `stock_account` |
| `l10n_gcc_pos` | Gulf point of sale requirements | The bilingual receipt and the visual code for Gulf point of sale. | `point_of_sale`, `l10n_gcc_invoice` |
| `l10n_latam_base` | Latin American base | The identification types used across Latin America and the fields they add to contacts and companies. | `contacts`, `base_vat` |
| `l10n_latam_check` | Latin American cheque management | Tracks own cheques and third-party cheques from issuance through delivery, deposit, rejection and cancellation, with five payment methods and a mass transfer. | `account`, `base_vat` |
| `l10n_latam_invoice_document` | Latin American invoice documents | The document types used across Latin America, their numbering per journal, and the constraints that tie a document type to an invoice class. | `account`, `account_debit_note` |
| `l10n_syscohada` | West and Central African harmonised chart | The harmonised accounting chart shared by seventeen West and Central African states, in its business and its not-for-profit variants. | `account` |

---

## 3. Country packages by region

### 3.1 Western Europe

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_at` | Austria, base package | Chart of accounts template `at`, with 237 account rows, 260 tax rows and 13 fiscal position rows. Fully specified in [Austria](countries/austria.md). | `account`, `account_edi_ubl_cii`, `base_iban`, `base_vat`, `l10n_din5008` |
| `l10n_be` | Belgium, base package | Chart of accounts templates `be`, `be_asso`, `be_comp`, with 537 account rows, 432 tax rows and 12 fiscal position rows. Fully specified in [Belgium](countries/belgium.md). | `account`, `account_edi_ubl_cii`, `base_iban`, `base_vat` |
| `l10n_be_pos_restaurant` | Belgium, point of sale for restaurants | Adds the country's restaurant receipt requirements. See [Belgium](countries/belgium.md). | `pos_restaurant`, `l10n_be` |
| `l10n_be_pos_sale` | Belgium, point of sale linked to sales orders | Links point of sale orders to sales orders under the country's rules. See [Belgium](countries/belgium.md). | `pos_sale`, `l10n_be` |
| `l10n_ch` | Switzerland, base package | Chart of accounts template `ch`, with 209 account rows, 110 tax rows and 2 fiscal position rows. Fully specified in [Switzerland](countries/switzerland.md). | `account`, `account_edi_ubl_cii`, `base_iban`, `l10n_din5008` |
| `l10n_ch_pos` | Switzerland, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Switzerland](countries/switzerland.md). | `l10n_ch`, `point_of_sale` |
| `l10n_cy` | Cyprus, base package | Chart of accounts template `cy`, with 169 account rows, 124 tax rows and 5 fiscal position rows. | `account`, `account_edi_ubl_cii`, `base_vat` |
| `l10n_de` | Germany, base package | Chart of accounts templates `de_skr03`, `de_skr04`, with 2466 account rows, 484 tax rows and 60 fiscal position rows. Fully specified in [Germany](countries/germany.md). | `base_iban`, `base_vat`, `l10n_din5008`, `account`, `account_edi_ubl_cii` |
| `l10n_es` | Spain, base package | Chart of accounts templates `es_assec`, `es_canary_assoc`, `es_canary_common`, `es_canary_full`, `es_canary_pymes`, `es_common`, `es_common_mainland`, `es_coop_full`, `es_coop_pymes`, `es_full`, `es_pymes`, with 1048 account rows, 1246 tax rows and 39 fiscal position rows. Fully specified in [Spain](countries/spain.md). | `account`, `base_iban`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_es_edi_facturae` | Spain, public sector invoice payload | Builds the public sector invoice payload and its administrative centre roles. See [Spain](countries/spain.md). | `certificate`, `l10n_es` |
| `l10n_es_edi_sii` | Spain, immediate supply of invoice information | Transmits the immediate supply of invoice information to the tax administration. See [Spain](countries/spain.md). | `certificate`, `l10n_es`, `account_edi` |
| `l10n_es_edi_tbai` | Spain, provincial invoice registration | Adds the provincial invoice registration: the signed chain, the visual code and the registration and cancellation documents. See [Spain](countries/spain.md). | `l10n_es`, `certificate` |
| `l10n_es_edi_tbai_pos` | Spain, provincial invoice registration for point of sale | Extends the provincial invoice registration to point of sale receipts. See [Spain](countries/spain.md). | `l10n_es_edi_tbai`, `point_of_sale` |
| `l10n_es_edi_verifactu` | Spain, verifiable invoice regime | Adds the national verifiable-invoice regime: the chained billing records, their batch transmission and the visual code. See [Spain](countries/spain.md). | `l10n_es`, `certificate` |
| `l10n_es_edi_verifactu_pos` | Spain, verifiable invoice regime for point of sale | Extends the verifiable-invoice regime to point of sale receipts. See [Spain](countries/spain.md). | `l10n_es_edi_verifactu`, `point_of_sale` |
| `l10n_es_pos` | Spain, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Spain](countries/spain.md). | `point_of_sale`, `l10n_es` |
| `l10n_fr` | France, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. Fully specified in [France](countries/france.md). | `base` |
| `l10n_fr_account` | France, accounting package | Adds the accounting layer on top of the country base package: the chart of accounts template, the taxes, the tax groups, the fiscal positions and the journals. See [France](countries/france.md). | `base_iban`, `base_vat`, `account`, `account_edi_ubl_cii`, `l10n_fr` |
| `l10n_fr_facturx_chorus_pro` | France, public sector portal | Adds the public sector fields and the service and engagement references the public portal requires. See [France](countries/france.md). | `account`, `account_edi_ubl_cii`, `l10n_fr_account` |
| `l10n_fr_hr_holidays` | France, time off rules | Adds the country rules for time off, including the treatment of part-time work and of restricted holidays. See [France](countries/france.md). | `hr_holidays` |
| `l10n_fr_hr_work_entry_holidays` | France, work entry rules | Adds the country rules for work entries generated from time off. See [France](countries/france.md). | `l10n_fr_hr_holidays`, `hr_work_entry_holidays` |
| `l10n_fr_pdp` | France, national invoicing portal | Connects the company to the national invoicing portal: registration, identity verification, invoice transmission, lifecycle answers and the periodic transaction and payment reports. See [France](countries/france.md). | `l10n_fr_account`, `account_peppol_response`, `auth_totp_mail`, `iap` |
| `l10n_fr_pdp_pos` | France, national invoicing portal for point of sale | Adds point of sale turnover to the periodic transaction report. See [France](countries/france.md). | `l10n_fr_pdp`, `point_of_sale` |
| `l10n_fr_pos_cert` | France, point of sale certification | Makes point of sale orders inalterable by hashing them into a chain and produces the daily, monthly and annual closings. See [France](countries/france.md). | `l10n_fr_account`, `point_of_sale` |
| `l10n_gf` | French Guiana, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. | `l10n_fr_account`, `account` |
| `l10n_gp` | Guadeloupe, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. | `l10n_fr_account`, `account` |
| `l10n_gr` | Greece, base package | Chart of accounts template `gr`, with 454 account rows, 300 tax rows and 5 fiscal position rows. | `account`, `base_iban`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_gr_edi` | Greece, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. | `l10n_gr` |
| `l10n_gr_edi_e_invoo` | Greece, electronic invoicing through an accredited intermediary | Routes the country's electronic invoicing through an accredited intermediary and returns the printable document. | `account_edi_proxy_client`, `l10n_gr_edi` |
| `l10n_ie` | Ireland, base package | Chart of accounts template `ie`, with 145 account rows, 186 tax rows and 4 fiscal position rows. | `account`, `base_iban`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_it` | Italy, base package | Chart of accounts template `it`, with 184 account rows, 522 tax rows and 6 fiscal position rows. Fully specified in [Italy](countries/italy.md). | `account`, `base_iban`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_it_edi` | Italy, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. See [Italy](countries/italy.md). | `l10n_it`, `account_edi_proxy_client`, `account_debit_note` |
| `l10n_it_edi_doi` | Italy, habitual exporter declarations | Adds the habitual exporter declaration, its validity window, its threshold and the tax treatment it unlocks. See [Italy](countries/italy.md). | `l10n_it_edi`, `sale` |
| `l10n_it_edi_sale` | Italy, electronic invoicing on sales orders | Carries the electronic invoicing fields from the sales order to the invoice. See [Italy](countries/italy.md). | `l10n_it_edi`, `sale` |
| `l10n_it_stock_ddt` | Italy, goods transport notes | Adds the goods transport note, its numbering and its reference on the invoice. See [Italy](countries/italy.md). | `l10n_it_edi`, `stock_delivery`, `stock_account` |
| `l10n_lu` | Luxembourg, base package | Chart of accounts template `lu`, with 746 account rows, 1006 tax rows and 5 fiscal position rows. Fully specified in [Luxembourg](countries/luxembourg.md). | `account`, `base_iban`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_mc` | Monaco, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. | `l10n_fr_account`, `account` |
| `l10n_mq` | Martinique, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. | `l10n_fr_account`, `account` |
| `l10n_mt` | Malta, base package | Chart of accounts template `mt`, with 481 account rows, 88 tax rows and 5 fiscal position rows. | `account`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_mt_pos` | Malta, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. | `point_of_sale` |
| `l10n_nl` | the Netherlands, base package | Chart of accounts template `nl`, with 349 account rows, 144 tax rows and 21 fiscal position rows. Fully specified in [the Netherlands](countries/netherlands.md). | `base_iban`, `base_vat`, `account`, `account_edi_ubl_cii` |
| `l10n_pt` | Portugal, base package | Chart of accounts template `pt`, with 642 account rows, 204 tax rows and 4 fiscal position rows. Fully specified in [Portugal](countries/portugal.md). | `base`, `account`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_re` | Réunion, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. | `l10n_fr_account`, `account` |
| `l10n_uk` | the United Kingdom, base package | Chart of accounts templates `uk`, `xi`, with 118 account rows, 76 tax rows and 3 fiscal position rows. Fully specified in [the United Kingdom](countries/united-kingdom.md). | `account`, `base_iban`, `base_vat` |
| `l10n_yt` | Mayotte, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. | `l10n_fr_account`, `account` |

### 3.2 Northern Europe

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_dk` | Denmark, base package | Chart of accounts template `dk`, with 488 account rows, 351 tax rows and 8 fiscal position rows. Fully specified in [Denmark](countries/denmark.md). | `base_iban`, `base_vat`, `account`, `account_edi_ubl_cii` |
| `l10n_dk_fik` | Denmark, national payment reference | Adds the national payment reference form to invoices and payments. See [Denmark](countries/denmark.md). | `l10n_dk` |
| `l10n_dk_nemhandel` | Denmark, national exchange network | Registers the company on the national exchange network and sends and receives documents through it. See [Denmark](countries/denmark.md). | `account_edi_proxy_client`, `account_edi_ubl_cii`, `l10n_dk` |
| `l10n_dk_nemhandel_response` | Denmark, national exchange network answers | Sends and receives business-level answers on the national exchange network. See [Denmark](countries/denmark.md). | `l10n_dk_nemhandel` |
| `l10n_dk_oioubl` | Denmark, public-information invoice payload | Builds the national public-information invoice payload. See [Denmark](countries/denmark.md). | `account_edi_ubl_cii`, `l10n_dk` |
| `l10n_ee` | Estonia, base package | Chart of accounts template `ee`, with 211 account rows, 290 tax rows and 10 fiscal position rows. | `account`, `account_edi_ubl_cii` |
| `l10n_fi` | Finland, base package | Chart of accounts template `fi`, with 971 account rows, 239 tax rows and 6 fiscal position rows. Fully specified in [Finland](countries/finland.md). | `base_iban`, `base_vat`, `account`, `account_edi_ubl_cii` |
| `l10n_fi_sale` | Finland, sales | Propagates the country fields from the sales order to the invoice. See [Finland](countries/finland.md). | `l10n_fi`, `sale` |
| `l10n_lt` | Lithuania, base package | Chart of accounts template `lt`, with 189 account rows, 260 tax rows and 4 fiscal position rows. | `account`, `account_edi_ubl_cii` |
| `l10n_lv` | Latvia, base package | Chart of accounts template `lv`, with 242 account rows, 142 tax rows and 4 fiscal position rows. | `account`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_no` | Norway, base package | Chart of accounts template `no`, with 745 account rows, 148 tax rows and 0 fiscal position rows. Fully specified in [Norway](countries/norway.md). | `base_iban`, `base_vat`, `account`, `account_edi_ubl_cii` |
| `l10n_se` | Sweden, base package | Chart of accounts templates `se`, `se_K2`, `se_K3`, with 1214 account rows, 176 tax rows and 20 fiscal position rows. Fully specified in [Sweden](countries/sweden.md). | `account`, `base_vat`, `account_edi_ubl_cii` |

### 3.3 Central and Eastern Europe

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_bg` | Bulgaria, base package | Chart of accounts template `bg`, with 332 account rows, 112 tax rows and 4 fiscal position rows. | `account`, `base_vat` |
| `l10n_bg_ledger` | Bulgaria, statutory ledger books | Adds the statutory ledger books the country requires, with their numbering and their printed form. | `l10n_bg` |
| `l10n_cz` | the Czech Republic, base package | Chart of accounts template `cz`, with 277 account rows, 148 tax rows and 8 fiscal position rows. | `account`, `account_edi_ubl_cii`, `base_iban`, `base_vat` |
| `l10n_ge` | Georgia, base package | Chart of accounts template `ge`, with 256 account rows, 150 tax rows and 4 fiscal position rows. | `account` |
| `l10n_hr` | Croatia, base package | Chart of accounts template `hr`, with 554 account rows, 142 tax rows and 5 fiscal position rows. | `account`, `base_vat` |
| `l10n_hr_edi` | Croatia, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. | `l10n_hr`, `account_edi_ubl_cii`, `account_peppol` |
| `l10n_hr_kuna` | Croatia, pre-changeover currency | Keeps the pre-changeover currency and its conversion rules available. | `account` |
| `l10n_hu` | Hungary, base package | Chart of accounts template `hu`, with 389 account rows, 174 tax rows and 5 fiscal position rows. | `account`, `base_vat` |
| `l10n_hu_edi` | Hungary, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. | `account_debit_note`, `base_iban`, `l10n_hu` |
| `l10n_hu_edi_receive` | Hungary, inbound document retrieval | Fetches inbound documents from the tax administration and turns them into vendor bills. | `l10n_hu_edi` |
| `l10n_kz` | Kazakhstan, base package | Chart of accounts template `kz`, with 280 account rows, 96 tax rows and 3 fiscal position rows. | `account` |
| `l10n_pl` | Poland, base package | Chart of accounts template `pl`, with 238 account rows, 128 tax rows and 4 fiscal position rows. Fully specified in [Poland](countries/poland.md). | `base_iban`, `base_vat`, `account`, `account_edi_ubl_cii` |
| `l10n_pl_bank_verification` | Poland, supplier bank account verification | Checks a supplier bank account against the national register before a payment is registered. See [Poland](countries/poland.md). | `l10n_pl` |
| `l10n_pl_edi` | Poland, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. See [Poland](countries/poland.md). | `l10n_pl`, `certificate` |
| `l10n_pl_edi_jst` | Poland, public sector invoicing | Adds the public sector variant of the national invoicing system. See [Poland](countries/poland.md). | `l10n_pl`, `l10n_pl_edi` |
| `l10n_ro` | Romania, base package | Chart of accounts template `ro`, with 579 account rows, 340 tax rows and 9 fiscal position rows. Fully specified in [Romania](countries/romania.md). | `account`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_ro_cpv_code` | Romania, public procurement codes | Ships the public procurement classification codes and adds the code to products. See [Romania](countries/romania.md). | `l10n_ro_edi` |
| `l10n_ro_edi` | Romania, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. See [Romania](countries/romania.md). | `account_edi_ubl_cii`, `l10n_ro` |
| `l10n_ro_edi_stock` | Romania, goods movement declarations | Declares goods movements to the national portal. See [Romania](countries/romania.md). | `stock_delivery`, `l10n_ro_edi`, `stock_picking_batch` |
| `l10n_ro_edi_stock_batch` | Romania, batched goods movement declarations | Declares a batch of goods movements to the national portal as one declaration. See [Romania](countries/romania.md). | `l10n_ro_edi_stock`, `stock_picking_batch` |
| `l10n_rs` | Serbia, base package | Chart of accounts template `rs`, with 420 account rows, 52 tax rows and 3 fiscal position rows. | `account`, `base_vat` |
| `l10n_rs_edi` | Serbia, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. | `account_edi_ubl_cii`, `l10n_rs` |
| `l10n_si` | Slovenia, base package | Chart of accounts template `si`, with 607 account rows, 256 tax rows and 17 fiscal position rows. | `account`, `base_vat`, `account_edi_ubl_cii` |
| `l10n_sk` | Slovakia, base package | Chart of accounts template `sk`, with 345 account rows, 178 tax rows and 4 fiscal position rows. | `base_iban`, `base_vat`, `account` |
| `l10n_ua` | Ukraine, base package | Chart of accounts template `ua_psbo`, with 332 account rows, 49 tax rows and 0 fiscal position rows. | `account` |
| `l10n_uz` | Uzbekistan, base package | Chart of accounts template `uz`, with 269 account rows, 16 tax rows and 0 fiscal position rows. | `account`, `base_vat` |

### 3.4 North America

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_ca` | Canada, base package | Chart of accounts template `ca_2023`, with 341 account rows, 101 tax rows and 14 fiscal position rows. | `account`, `base_iban` |
| `l10n_mx` | Mexico, base package | Chart of accounts template `mx`, with 140 account rows, 138 tax rows and 5 fiscal position rows. Fully specified in [Mexico](countries/mexico.md). | `account` |
| `l10n_us` | the United States, base package | Country configuration without a chart data file of its own; it extends the chart of a parent package or ships its data through template functions only. Fully specified in [the United States](countries/united-states.md). | `base` |
| `l10n_us_account` | the United States, accounting package | Adds the accounting layer on top of the country base package: the chart of accounts template, the taxes, the tax groups, the fiscal positions and the journals. See [the United States](countries/united-states.md). | `l10n_us`, `account` |

### 3.5 Central America and the Caribbean

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_cr` | Costa Rica, base package | Chart of accounts template `cr`, with 98 account rows, 8 tax rows and 0 fiscal position rows. | `account` |
| `l10n_do` | the Dominican Republic, base package | Chart of accounts template `do`, with 284 account rows, 139 tax rows and 15 fiscal position rows. | `account`, `base_iban` |
| `l10n_gt` | Guatemala, base package | Chart of accounts template `gt`, with 86 account rows, 16 tax rows and 0 fiscal position rows. | `base`, `account` |
| `l10n_hn` | Honduras, base package | Chart of accounts template `hn`, with 34 account rows, 8 tax rows and 0 fiscal position rows. | `base`, `account` |
| `l10n_pa` | Panama, base package | Chart of accounts template `pa`, with 105 account rows, 8 tax rows and 0 fiscal position rows. | `account` |

### 3.6 South America

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_ar` | Argentina, base package | Chart of accounts templates `ar_base`, `ar_ex`, `ar_ri`, with 299 account rows, 292 tax rows and 4 fiscal position rows. Fully specified in [Argentina](countries/argentina.md). | `l10n_latam_invoice_document`, `l10n_latam_base`, `account` |
| `l10n_ar_pos` | Argentina, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Argentina](countries/argentina.md). | `l10n_ar`, `point_of_sale` |
| `l10n_ar_stock` | Argentina, inventory | Carries the country fields onto transfers and stock moves so that goods movements can be declared. See [Argentina](countries/argentina.md). | `l10n_ar`, `stock_account` |
| `l10n_ar_website_sale` | Argentina, online store | Adds the country's mandatory identification fields to the online checkout and validates them with the same checker as the contact form. See [Argentina](countries/argentina.md). | `website_sale`, `l10n_ar` |
| `l10n_ar_withholding` | Argentina, withholding taxes | Adds the country's withholding regime: the withholding taxes, the per-counterpart authorisations and the numbering of the withholding certificates. See [Argentina](countries/argentina.md). | `l10n_ar`, `l10n_latam_check` |
| `l10n_bo` | Bolivia, base package | Chart of accounts template `bo`, with 127 account rows, 99 tax rows and 3 fiscal position rows. | `account` |
| `l10n_br` | Brazil, base package | Chart of accounts template `br`, with 1085 account rows, 1292 tax rows and 6 fiscal position rows. Fully specified in [Brazil](countries/brazil.md). | `account`, `account_qr_code_emv`, `base_address_extended`, `l10n_latam_base`, `l10n_latam_invoice_document` |
| `l10n_br_sales` | Brazil, sales | Propagates the country fields from the sales order to the invoice and computes the country taxes on the order. See [Brazil](countries/brazil.md). | `l10n_br`, `sale` |
| `l10n_br_website_sale` | Brazil, online store | Adds the country's mandatory identification fields to the online checkout and validates them with the same checker as the contact form. See [Brazil](countries/brazil.md). | `l10n_br`, `website_sale` |
| `l10n_cl` | Chile, base package | Chart of accounts template `cl`, with 195 account rows, 108 tax rows and 25 fiscal position rows. Fully specified in [Chile](countries/chile.md). | `contacts`, `base_vat`, `l10n_latam_base`, `l10n_latam_invoice_document`, `uom`, `account` |
| `l10n_co` | Colombia, base package | Chart of accounts template `co`, with 378 account rows, 248 tax rows and 0 fiscal position rows. Fully specified in [Colombia](countries/colombia.md). | `account_debit_note`, `l10n_latam_base`, `account` |
| `l10n_co_pos` | Colombia, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Colombia](countries/colombia.md). | `l10n_co`, `point_of_sale` |
| `l10n_ec` | Ecuador, base package | Chart of accounts template `ec`, with 501 account rows, 728 tax rows and 2 fiscal position rows. Fully specified in [Ecuador](countries/ecuador.md). | `base`, `base_iban`, `account_debit_note`, `l10n_latam_invoice_document`, `l10n_latam_base`, `account` |
| `l10n_ec_sale` | Ecuador, sales | Propagates the country fields from the sales order to the invoice. See [Ecuador](countries/ecuador.md). | `l10n_ec`, `sale` |
| `l10n_ec_stock` | Ecuador, inventory | Carries the country fields onto transfers and stock moves so that goods movements can be declared. See [Ecuador](countries/ecuador.md). | `l10n_ec`, `stock` |
| `l10n_pe` | Peru, base package | Chart of accounts template `pe`, with 1228 account rows, 69 tax rows and 2 fiscal position rows. Fully specified in [Peru](countries/peru.md). | `base_vat`, `base_address_extended`, `l10n_latam_base`, `l10n_latam_invoice_document`, `account_debit_note`, `account` |
| `l10n_pe_pos` | Peru, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Peru](countries/peru.md). | `l10n_pe`, `point_of_sale` |
| `l10n_uy` | Uruguay, base package | Chart of accounts template `uy`, with 166 account rows, 48 tax rows and 2 fiscal position rows. Fully specified in [Uruguay](countries/uruguay.md). | `account`, `l10n_latam_invoice_document`, `l10n_latam_base` |
| `l10n_uy_pos` | Uruguay, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Uruguay](countries/uruguay.md). | `l10n_uy`, `point_of_sale` |
| `l10n_ve` | Venezuela, base package | Chart of accounts template `ve`, with 266 account rows, 32 tax rows and 0 fiscal position rows. | `account` |

### 3.7 Middle East and North Africa

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_ae` | United Arab Emirates, base package | Chart of accounts template `ae`, with 170 account rows, 74 tax rows and 9 fiscal position rows. Fully specified in [United Arab Emirates](countries/united-arab-emirates.md). | `account`, `l10n_gcc_invoice` |
| `l10n_ae_pos` | United Arab Emirates, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [United Arab Emirates](countries/united-arab-emirates.md). | `l10n_gcc_pos`, `l10n_ae` |
| `l10n_bh` | Bahrain, base package | Chart of accounts template `bh`, with 137 account rows, 56 tax rows and 3 fiscal position rows. | `account`, `l10n_gcc_invoice` |
| `l10n_dz` | Algeria, base package | Chart of accounts template `dz`, with 294 account rows, 168 tax rows and 4 fiscal position rows. | `base_vat`, `account` |
| `l10n_eg` | Egypt, base package | Chart of accounts template `eg`, with 205 account rows, 120 tax rows and 2 fiscal position rows. Fully specified in [Egypt](countries/egypt.md). | `account` |
| `l10n_eg_edi_eta` | Egypt, tax administration invoice integration | Adds the tax administration invoice integration, including the signing device and the activity and unit codes. See [Egypt](countries/egypt.md). | nothing beyond the platform |
| `l10n_il` | Israel, base package | Chart of accounts template `il`, with 86 account rows, 58 tax rows and 6 fiscal position rows. | `account` |
| `l10n_iq` | Iraq, base package | Chart of accounts template `iq`, with 137 account rows, 20 tax rows and 0 fiscal position rows. | `account` |
| `l10n_jo` | Jordan, base package | Chart of accounts template `jo_standard`, with 140 account rows, 116 tax rows and 2 fiscal position rows. | `account` |
| `l10n_jo_edi` | Jordan, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. | `account_edi_ubl_cii`, `l10n_jo` |
| `l10n_jo_edi_pos` | Jordan, electronic invoicing for point of sale | Extends the country's electronic invoicing flow to point of sale receipts. | `l10n_jo_edi`, `pos_edi_ubl` |
| `l10n_kw` | Kuwait, base package | Chart of accounts template `kw`, with 136 account rows, 0 tax rows and 0 fiscal position rows. | `account`, `l10n_gcc_invoice` |
| `l10n_lb_account` | Lebanon, accounting package | Adds the accounting layer on top of the country base package: the chart of accounts template, the taxes, the tax groups, the fiscal positions and the journals. | `account` |
| `l10n_ma` | Morocco, base package | Chart of accounts template `ma`, with 634 account rows, 318 tax rows and 8 fiscal position rows. | `base`, `account` |
| `l10n_om` | Oman, base package | Chart of accounts template `om`, with 139 account rows, 56 tax rows and 2 fiscal position rows. | `account`, `l10n_gcc_invoice` |
| `l10n_qa` | Qatar, base package | Chart of accounts template `qa`, with 136 account rows, 0 tax rows and 0 fiscal position rows. | `account`, `l10n_gcc_invoice` |
| `l10n_sa` | Saudi Arabia, base package | Chart of accounts template `sa`, with 169 account rows, 174 tax rows and 2 fiscal position rows. Fully specified in [Saudi Arabia](countries/saudi-arabia.md). | `l10n_gcc_invoice`, `account`, `account_debit_note` |
| `l10n_sa_edi` | Saudi Arabia, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. See [Saudi Arabia](countries/saudi-arabia.md). | nothing beyond the platform |
| `l10n_sa_edi_pos` | Saudi Arabia, electronic invoicing for point of sale | Extends the country's electronic invoicing flow to point of sale receipts. See [Saudi Arabia](countries/saudi-arabia.md). | nothing beyond the platform |
| `l10n_sa_pos` | Saudi Arabia, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Saudi Arabia](countries/saudi-arabia.md). | `l10n_gcc_pos`, `l10n_sa` |
| `l10n_sa_withholding_tax` | Saudi Arabia, withholding taxes | Adds the country's withholding regime on top of the shared withholding framework. See [Saudi Arabia](countries/saudi-arabia.md). | `l10n_account_withholding_tax`, `l10n_sa` |
| `l10n_tn` | Tunisia, base package | Chart of accounts template `tn`, with 450 account rows, 116 tax rows and 3 fiscal position rows. | `account` |
| `l10n_tr` | Türkiye, base package | Chart of accounts template `tr`, with 267 account rows, 110 tax rows and 0 fiscal position rows. Fully specified in [Türkiye](countries/turkey.md). | `account` |
| `l10n_tr_nilvera` | Türkiye, accredited intermediary connection | Connects the company to the accredited intermediary and keeps the counterpart directory and the unit codes. See [Türkiye](countries/turkey.md). | `l10n_tr`, `account_edi_ubl_cii` |
| `l10n_tr_nilvera_base_vat` | Türkiye, identification number formats | Adds the national identification number formats for individuals and for legal entities. See [Türkiye](countries/turkey.md). | `l10n_tr_nilvera`, `base_vat` |
| `l10n_tr_nilvera_edispatch` | Türkiye, electronic dispatch notes | Sends the electronic dispatch note with its trailer plates and driver data. See [Türkiye](countries/turkey.md). | `l10n_tr_nilvera`, `stock` |
| `l10n_tr_nilvera_einvoice` | Türkiye, electronic and archive invoices | Sends electronic invoices and archive invoices through the accredited intermediary and fetches inbound ones. See [Türkiye](countries/turkey.md). | `l10n_tr_nilvera`, `account_edi_ubl_cii` |
| `l10n_tr_nilvera_einvoice_extended` | Türkiye, extended invoice fields and tax codes | Adds the administration tax codes, the tax offices and the extended invoice fields. See [Türkiye](countries/turkey.md). | `l10n_tr_nilvera_einvoice`, `contacts` |

### 3.8 Sub-Saharan Africa

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_bf` | Burkina Faso, base package | Chart of accounts templates `bf`, `bf_syscebnl`, with 0 account rows, 76 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_bj` | Benin, base package | Chart of accounts templates `bj`, `bj_syscebnl`, with 0 account rows, 84 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_cd` | the Democratic Republic of the Congo, base package | Chart of accounts templates `cd`, `cd_syscebnl`, with 0 account rows, 112 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_cf` | the Central African Republic, base package | Chart of accounts templates `cf`, `cf_syscebnl`, with 0 account rows, 64 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_cg` | the Republic of the Congo, base package | Chart of accounts templates `cg`, `cg_syscebnl`, with 0 account rows, 68 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_ci` | Côte d'Ivoire, base package | Chart of accounts templates `ci`, `ci_syscebnl`, with 0 account rows, 120 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_cm` | Cameroon, base package | Chart of accounts templates `cm`, `cm_syscebnl`, with 0 account rows, 56 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_et` | Ethiopia, base package | Chart of accounts template `et`, with 100 account rows, 52 tax rows and 0 fiscal position rows. | `account` |
| `l10n_ga` | Gabon, base package | Chart of accounts templates `ga`, `ga_syscebnl`, with 0 account rows, 154 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_gn` | Guinea, base package | Chart of accounts templates `gn`, `gn_syscebnl`, with 0 account rows, 56 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_gq` | Equatorial Guinea, base package | Chart of accounts templates `gq`, `gq_syscebnl`, with 0 account rows, 80 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_gw` | Guinea-Bissau, base package | Chart of accounts templates `gw`, `gw_syscebnl`, with 0 account rows, 96 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_ke` | Kenya, base package | Chart of accounts template `ke`, with 127 account rows, 116 tax rows and 2 fiscal position rows. Fully specified in [Kenya](countries/kenya.md). | `account` |
| `l10n_ke_edi_tremol` | Kenya, certified fiscal device | Signs invoices with the certified fiscal device and transmits them. See [Kenya](countries/kenya.md). | `l10n_ke` |
| `l10n_km` | the Comoros, base package | Chart of accounts templates `km`, `km_syscebnl`, with 0 account rows, 64 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_ml` | Mali, base package | Chart of accounts templates `ml`, `ml_syscebnl`, with 0 account rows, 104 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_mr` | Mauritania, base package | Chart of accounts template `mr`, with 611 account rows, 60 tax rows and 2 fiscal position rows. | `account`, `base_vat` |
| `l10n_mu_account` | Mauritius, accounting package | Adds the accounting layer on top of the country base package: the chart of accounts template, the taxes, the tax groups, the fiscal positions and the journals. | `account` |
| `l10n_mz` | Mozambique, base package | Chart of accounts template `mz`, with 311 account rows, 44 tax rows and 2 fiscal position rows. | `base`, `account` |
| `l10n_ne` | Niger, base package | Chart of accounts templates `ne`, `ne_syscebnl`, with 0 account rows, 92 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_ng` | Nigeria, base package | Chart of accounts template `ng`, with 0 account rows, 48 tax rows and 2 fiscal position rows. | `base_vat`, `account` |
| `l10n_rw` | Rwanda, base package | Chart of accounts template `rw`, with 164 account rows, 36 tax rows and 2 fiscal position rows. | `account` |
| `l10n_sn` | Senegal, base package | Chart of accounts templates `sn`, `sn_syscebnl`, with 0 account rows, 80 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_td` | Chad, base package | Chart of accounts templates `td`, `td_syscebnl`, with 0 account rows, 64 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_tg` | Togo, base package | Chart of accounts templates `tg`, `tg_syscebnl`, with 0 account rows, 88 tax rows and 4 fiscal position rows. | `l10n_syscohada`, `account` |
| `l10n_tz_account` | Tanzania, accounting package | Adds the accounting layer on top of the country base package: the chart of accounts template, the taxes, the tax groups, the fiscal positions and the journals. | `account` |
| `l10n_ug` | Uganda, base package | Chart of accounts template `ug`, with 124 account rows, 80 tax rows and 2 fiscal position rows. | `account` |
| `l10n_za` | South Africa, base package | Chart of accounts template `za`, with 117 account rows, 64 tax rows and 0 fiscal position rows. | `account`, `base_vat` |
| `l10n_zm_account` | Zambia, accounting package | Adds the accounting layer on top of the country base package: the chart of accounts template, the taxes, the tax groups, the fiscal positions and the journals. | `account` |

### 3.9 South and Central Asia

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_bd` | Bangladesh, base package | Chart of accounts template `bd`, with 135 account rows, 64 tax rows and 2 fiscal position rows. | `account` |
| `l10n_in` | India, base package | Chart of accounts template `in`, with 102 account rows, 1741 tax rows and 2 fiscal position rows. Fully specified in [India](countries/india.md). | `account_tax_python`, `base_vat`, `account_debit_note`, `account`, `iap` |
| `l10n_in_edi` | India, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. See [India](countries/india.md). | `account_edi`, `l10n_in` |
| `l10n_in_ewaybill` | India, goods movement permits | Obtains the goods movement permit from the national portal, with transport data, validity and cancellation. See [India](countries/india.md). | `l10n_in` |
| `l10n_in_ewaybill_irn` | India, goods movement permits from the registration number | Obtains the goods movement permit by quoting the invoice registration number instead of resending the whole document. See [India](countries/india.md). | `l10n_in_ewaybill`, `l10n_in_edi` |
| `l10n_in_ewaybill_stock` | India, goods movement permits from transfers | Obtains the goods movement permit from a transfer rather than from an invoice. See [India](countries/india.md). | `l10n_in_stock`, `l10n_in_ewaybill` |
| `l10n_in_hr_holidays` | India, time off rules | Adds the country rules for time off, including the treatment of part-time work and of restricted holidays. See [India](countries/india.md). | `hr_holidays` |
| `l10n_in_pos` | India, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [India](countries/india.md). | `l10n_in`, `point_of_sale` |
| `l10n_in_purchase_stock` | India, purchasing and inventory | Carries the country fields from the purchase order onto the receipt. See [India](countries/india.md). | `l10n_in_stock`, `purchase_stock` |
| `l10n_in_sale` | India, sales | Propagates the country fields from the sales order to the invoice. See [India](countries/india.md). | `l10n_in`, `sale` |
| `l10n_in_sale_stock` | India, sales and inventory | Carries the country fields from the sales order onto the delivery. See [India](countries/india.md). | `l10n_in_sale`, `l10n_in_stock`, `sale_stock` |
| `l10n_in_stock` | India, inventory | Carries the country fields onto transfers and stock moves so that goods movements can be declared. See [India](countries/india.md). | `l10n_in`, `stock`, `stock_account` |
| `l10n_lk` | Sri Lanka, base package | Chart of accounts template `lk`, with 100 account rows, 236 tax rows and 2 fiscal position rows. | `account`, `l10n_account_withholding_tax` |
| `l10n_lk_invoice` | Sri Lanka, invoice requirements | Adds the invoice fields, the printed wording and the numbering the country demands. | `l10n_lk` |
| `l10n_mn` | Mongolia, base package | Chart of accounts template `mn`, with 304 account rows, 164 tax rows and 4 fiscal position rows. | `account` |
| `l10n_pk` | Pakistan, base package | Chart of accounts template `pk`, with 123 account rows, 244 tax rows and 0 fiscal position rows. | `account` |

### 3.10 East and South-East Asia

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_cn` | China, base package | Chart of accounts templates `cn`, `cn_common`, `cn_large_bis`, with 361 account rows, 48 tax rows and 0 fiscal position rows. | `base`, `account` |
| `l10n_cn_city` | China, city list | Ships the country's city list so that addresses can be selected rather than typed. | `l10n_cn`, `base_address_extended` |
| `l10n_hk` | Hong Kong, base package | Chart of accounts template `hk`, with 75 account rows, 0 tax rows and 0 fiscal position rows. | `account_qr_code_emv`, `account` |
| `l10n_id` | Indonesia, base package | Chart of accounts template `id`, with 114 account rows, 78 tax rows and 0 fiscal position rows. Fully specified in [Indonesia](countries/indonesia.md). | `account`, `base_iban`, `base_vat` |
| `l10n_id_efaktur_coretax` | Indonesia, electronic invoice documents | Produces the electronic invoice document that is uploaded to the administration, with product and unit codes. See [Indonesia](countries/indonesia.md). | `l10n_id` |
| `l10n_id_pos` | Indonesia, point of sale | Extends the point of sale with the country's receipt requirements: identification of the customer, fiscal numbering and the wording the receipt must carry. See [Indonesia](countries/indonesia.md). | `l10n_id`, `point_of_sale` |
| `l10n_jp` | Japan, base package | Chart of accounts template `jp`, with 128 account rows, 32 tax rows and 3 fiscal position rows. | `account` |
| `l10n_jp_ubl_pint` | Japan, international invoice profile | Builds the international invoice payload profile the country has adopted. | `account_edi_ubl_cii` |
| `l10n_kh` | Cambodia, base package | Chart of accounts template `kh`, with 106 account rows, 197 tax rows and 3 fiscal position rows. | `account_qr_code_emv`, `l10n_account_withholding_tax` |
| `l10n_kr` | South Korea, base package | Chart of accounts template `kr`, with 181 account rows, 256 tax rows and 0 fiscal position rows. | `account` |
| `l10n_my` | Malaysia, base package | Chart of accounts template `my`, with 77 account rows, 140 tax rows and 5 fiscal position rows. Fully specified in [Malaysia](countries/malaysia.md). | `account`, `account_tax_python` |
| `l10n_my_edi` | Malaysia, electronic invoicing | Adds the country's electronic invoicing flow: the payload, the transmission, the polling of the answer and the state on the invoice. See [Malaysia](countries/malaysia.md). | `l10n_my`, `l10n_my_ubl_pint`, `account_edi_proxy_client` |
| `l10n_my_edi_pos` | Malaysia, electronic invoicing for point of sale | Extends the country's electronic invoicing flow to point of sale receipts. See [Malaysia](countries/malaysia.md). | `l10n_my_edi`, `point_of_sale` |
| `l10n_my_ubl_pint` | Malaysia, international invoice profile | Builds the international invoice payload profile the country has adopted. See [Malaysia](countries/malaysia.md). | `account_edi_ubl_cii` |
| `l10n_ph` | the Philippines, base package | Chart of accounts template `ph`, with 104 account rows, 254 tax rows and 2 fiscal position rows. | `account`, `base_vat`, `l10n_account_withholding_tax` |
| `l10n_sg` | Singapore, base package | Chart of accounts template `sg`, with 135 account rows, 149 tax rows and 2 fiscal position rows. Fully specified in [Singapore](countries/singapore.md). | `account_qr_code_emv`, `account` |
| `l10n_sg_ubl_pint` | Singapore, international invoice profile | Builds the international invoice payload profile the country has adopted. See [Singapore](countries/singapore.md). | `account_edi_ubl_cii` |
| `l10n_th` | Thailand, base package | Chart of accounts template `th`, with 144 account rows, 72 tax rows and 0 fiscal position rows. | `account_qr_code_emv`, `account` |
| `l10n_tw` | Taiwan, base package | Chart of accounts template `tw`, with 135 account rows, 290 tax rows and 8 fiscal position rows. | `account`, `base_address_extended` |
| `l10n_tw_edi_ecpay` | Taiwan, electronic invoicing service | Adds the country's electronic invoicing through the accredited service: issuance, status query, cancellation and printing. | `l10n_tw`, `base_vat` |
| `l10n_tw_edi_ecpay_pos` | Taiwan, electronic invoicing service for point of sale | Extends that service to point of sale receipts. | `point_of_sale`, `l10n_tw_edi_ecpay` |
| `l10n_tw_edi_ecpay_website_sale` | Taiwan, electronic invoicing service for the online store | Adds the carrier and donation choices the service offers to online buyers. | `website_sale`, `l10n_tw_edi_ecpay` |
| `l10n_vn` | Vietnam, base package | Chart of accounts template `vn`, with 216 account rows, 60 tax rows and 0 fiscal position rows. Fully specified in [Vietnam](countries/vietnam.md). | `account_qr_code_emv`, `base_iban`, `account` |
| `l10n_vn_edi_viettel` | Vietnam, national invoicing service | Issues invoices through the national invoicing service, with symbols, templates, adjustments, replacements and cancellations. See [Vietnam](countries/vietnam.md). | `l10n_vn` |
| `l10n_vn_edi_viettel_pos` | Vietnam, national invoicing service for point of sale | Extends that service to point of sale receipts. See [Vietnam](countries/vietnam.md). | `l10n_vn_edi_viettel`, `point_of_sale` |

### 3.11 Oceania

| Package | Business name | Contributes | Depends on |
|---|---|---|---|
| `l10n_au` | Australia, base package | Chart of accounts template `au`, with 114 account rows, 130 tax rows and 4 fiscal position rows. Fully specified in [Australia](countries/australia.md). | `account` |
| `l10n_nz` | New Zealand, base package | Chart of accounts template `nz`, with 100 account rows, 24 tax rows and 2 fiscal position rows. | `account` |

---

## 4. Chart data shipped by each template

Each row gives one template code and the number of rows in each of its delimited data files. A
blank cell means the template ships no file of that kind, either because the parent template
supplies it or because the package builds those records in template functions rather than in a
file. The numbers are the size of the shipped configuration a rebuild must reproduce.

| Package | Template code | Accounts | Account groups | Taxes | Tax groups | Fiscal positions |
|---|---|---|---|---|---|---|
| `l10n_ae` | `ae` | 170 |  | 74 | 5 | 9 |
| `l10n_ar` | `ar_base` | 228 | 58 | 104 | 40 |  |
| `l10n_ar` | `ar_ex` | 63 |  | 108 | 40 |  |
| `l10n_ar` | `ar_ri` | 8 |  | 80 | 40 | 4 |
| `l10n_ar_withholding` | `ar_base` | 1 |  | 125 | 25 |  |
| `l10n_ar_withholding` | `ar_ex` |  |  | 278 | 2 |  |
| `l10n_ar_withholding` | `ar_ri` |  |  | 4 |  |  |
| `l10n_at` | `at` | 237 |  | 260 | 7 | 13 |
| `l10n_au` | `au` | 114 |  | 130 | 4 | 4 |
| `l10n_bd` | `bd` | 135 | 31 | 64 | 7 | 2 |
| `l10n_be` | `be` | 426 | 252 | 432 | 4 | 12 |
| `l10n_be` | `be_asso` | 43 | 42 |  |  |  |
| `l10n_be` | `be_comp` | 68 | 46 |  |  |  |
| `l10n_be_pos_restaurant` | `be` |  |  | 4 |  |  |
| `l10n_bf` | `bf` |  |  | 38 | 3 | 2 |
| `l10n_bf` | `bf_syscebnl` |  |  | 38 | 3 | 2 |
| `l10n_bg` | `bg` | 332 | 67 | 112 | 3 | 4 |
| `l10n_bh` | `bh` | 137 | 31 | 56 | 4 | 3 |
| `l10n_bj` | `bj` |  |  | 42 | 2 | 2 |
| `l10n_bj` | `bj_syscebnl` |  |  | 42 | 2 | 2 |
| `l10n_bo` | `bo` | 127 | 43 | 99 | 9 | 3 |
| `l10n_br` | `br` | 1085 |  | 1292 | 160 | 6 |
| `l10n_ca` | `ca_2023` | 341 | 168 | 101 | 8 | 14 |
| `l10n_cd` | `cd` |  |  | 56 | 2 | 2 |
| `l10n_cd` | `cd_syscebnl` |  |  | 56 | 2 | 2 |
| `l10n_cf` | `cf` |  |  | 32 | 3 | 2 |
| `l10n_cf` | `cf_syscebnl` |  |  | 32 | 3 | 2 |
| `l10n_cg` | `cg` |  |  | 34 | 4 | 2 |
| `l10n_cg` | `cg_syscebnl` |  |  | 34 | 4 | 2 |
| `l10n_ch` | `ch` | 209 |  | 110 | 8 | 2 |
| `l10n_ci` | `ci` |  |  | 60 | 3 | 2 |
| `l10n_ci` | `ci_syscebnl` |  |  | 60 | 3 | 2 |
| `l10n_cl` | `cl` | 195 |  | 108 | 5 | 25 |
| `l10n_cm` | `cm` |  |  | 28 | 2 | 2 |
| `l10n_cm` | `cm_syscebnl` |  |  | 28 | 2 | 2 |
| `l10n_cn` | `cn` | 71 |  | 24 |  |  |
| `l10n_cn` | `cn_common` | 88 |  |  | 3 |  |
| `l10n_cn` | `cn_large_bis` | 202 |  | 24 |  |  |
| `l10n_co` | `co` | 378 | 401 | 248 | 43 |  |
| `l10n_cr` | `cr` | 98 |  | 8 | 1 |  |
| `l10n_cy` | `cy` | 169 |  | 124 | 5 | 5 |
| `l10n_cz` | `cz` | 277 | 313 | 148 | 4 | 8 |
| `l10n_de` | `de_skr03` | 1274 |  | 242 | 6 | 30 |
| `l10n_de` | `de_skr04` | 1192 |  | 242 | 6 | 30 |
| `l10n_dk` | `dk` | 488 | 41 | 351 | 1 | 8 |
| `l10n_do` | `do` | 284 | 95 | 139 | 14 | 15 |
| `l10n_dz` | `dz` | 294 | 68 | 168 | 3 | 4 |
| `l10n_ec` | `ec` | 501 | 112 | 728 | 17 | 2 |
| `l10n_ee` | `ee` | 211 | 103 | 290 | 7 | 10 |
| `l10n_eg` | `eg` | 205 |  | 120 | 14 | 2 |
| `l10n_es` | `es_assec` | 61 |  |  |  |  |
| `l10n_es` | `es_canary_assoc` |  |  |  |  |  |
| `l10n_es` | `es_canary_common` | 8 |  | 518 | 17 | 10 |
| `l10n_es` | `es_canary_full` |  |  |  |  |  |
| `l10n_es` | `es_canary_pymes` |  |  |  |  |  |
| `l10n_es` | `es_common` | 598 | 930 |  | 15 |  |
| `l10n_es` | `es_common_mainland` | 93 |  | 728 | 17 | 29 |
| `l10n_es` | `es_coop_full` | 106 |  |  |  |  |
| `l10n_es` | `es_coop_pymes` | 64 | 16 |  |  |  |
| `l10n_es` | `es_full` | 109 |  |  |  |  |
| `l10n_es` | `es_pymes` | 9 |  |  |  |  |
| `l10n_es_edi_facturae` | `es_canary_common` |  |  | 118 |  |  |
| `l10n_es_edi_facturae` | `es_common_mainland` |  |  | 182 |  |  |
| `l10n_es_edi_verifactu` | `es_canary_common` |  |  | 106 |  |  |
| `l10n_es_edi_verifactu` | `es_common_mainland` |  |  | 167 |  |  |
| `l10n_et` | `et` | 100 |  | 52 | 5 |  |
| `l10n_fi` | `fi` | 971 |  | 239 | 6 | 6 |
| `l10n_fr_account` | `fr` | 656 | 182 | 304 | 6 | 10 |
| `l10n_fr_account` | `gf` |  |  |  |  |  |
| `l10n_fr_account` | `gp` |  |  |  |  |  |
| `l10n_fr_account` | `mc` |  |  |  |  |  |
| `l10n_fr_account` | `mq` |  |  |  |  |  |
| `l10n_fr_account` | `re` |  |  |  |  |  |
| `l10n_fr_account` | `yt` |  |  |  |  |  |
| `l10n_ga` | `ga` |  |  | 77 | 8 | 2 |
| `l10n_ga` | `ga_syscebnl` |  |  | 77 | 8 | 2 |
| `l10n_ge` | `ge` | 256 | 50 | 150 | 5 | 4 |
| `l10n_gn` | `gn` |  |  | 28 | 2 | 2 |
| `l10n_gn` | `gn_syscebnl` |  |  | 28 | 2 | 2 |
| `l10n_gq` | `gq` |  |  | 40 | 4 | 2 |
| `l10n_gq` | `gq_syscebnl` |  |  | 40 | 4 | 2 |
| `l10n_gr` | `gr` | 454 | 187 | 300 | 7 | 5 |
| `l10n_gt` | `gt` | 86 |  | 16 | 3 |  |
| `l10n_gw` | `gw` |  |  | 48 | 4 | 2 |
| `l10n_gw` | `gw_syscebnl` |  |  | 48 | 4 | 2 |
| `l10n_hk` | `hk` | 75 |  |  |  |  |
| `l10n_hn` | `hn` | 34 |  | 8 | 1 |  |
| `l10n_hr` | `hr` | 554 | 89 | 142 | 4 | 5 |
| `l10n_hr_edi` | `hr` |  |  | 15 |  |  |
| `l10n_hr_kuna` | `hr_kuna` | 1674 |  | 140 | 5 | 7 |
| `l10n_hu` | `hu` | 389 | 74 | 174 | 5 | 5 |
| `l10n_hu_edi` | `hu` |  |  | 39 |  |  |
| `l10n_id` | `id` | 114 |  | 78 | 6 |  |
| `l10n_ie` | `ie` | 145 |  | 186 | 5 | 4 |
| `l10n_il` | `il` | 86 | 10 | 58 | 4 | 6 |
| `l10n_in` | `in` | 102 |  | 1741 | 12 | 2 |
| `l10n_iq` | `iq` | 137 | 32 | 20 | 6 |  |
| `l10n_it` | `it` | 184 |  | 522 | 14 | 6 |
| `l10n_it_edi` | `it` |  |  | 14 |  |  |
| `l10n_it_edi_doi` | `it` |  |  | 4 |  | 1 |
| `l10n_jo` | `jo_standard` | 140 | 31 | 116 | 5 | 2 |
| `l10n_jp` | `jp` | 128 |  | 32 | 3 | 3 |
| `l10n_ke` | `ke` | 127 |  | 116 | 5 | 2 |
| `l10n_kh` | `kh` | 106 | 85 | 197 | 13 | 3 |
| `l10n_km` | `km` |  |  | 32 | 6 | 2 |
| `l10n_km` | `km_syscebnl` |  |  | 32 | 6 | 2 |
| `l10n_kr` | `kr` | 181 | 69 | 256 | 1 |  |
| `l10n_kw` | `kw` | 136 | 31 |  |  |  |
| `l10n_kz` | `kz` | 280 |  | 96 | 4 | 3 |
| `l10n_lb_account` | `lb` | 373 | 234 | 32 | 3 | 2 |
| `l10n_lk` | `lk` | 100 |  | 236 | 5 | 2 |
| `l10n_lt` | `lt` | 189 |  | 260 | 5 | 4 |
| `l10n_lu` | `lu` | 746 | 973 | 1006 | 12 | 5 |
| `l10n_lv` | `lv` | 242 | 55 | 142 | 5 | 4 |
| `l10n_ma` | `ma` | 634 | 147 | 318 | 5 | 8 |
| `l10n_ml` | `ml` |  |  | 52 | 3 | 2 |
| `l10n_ml` | `ml_syscebnl` |  |  | 52 | 3 | 2 |
| `l10n_mn` | `mn` | 304 |  | 164 | 6 | 4 |
| `l10n_mr` | `mr` | 611 | 129 | 60 | 4 | 2 |
| `l10n_mt` | `mt` | 481 |  | 88 | 4 | 5 |
| `l10n_mu_account` | `mu` | 42 |  | 48 | 2 | 2 |
| `l10n_mx` | `mx` | 140 | 1079 | 138 | 14 | 5 |
| `l10n_my` | `my` | 77 |  | 140 | 10 | 5 |
| `l10n_mz` | `mz` | 311 | 101 | 44 | 3 | 2 |
| `l10n_ne` | `ne` |  |  | 46 | 2 | 2 |
| `l10n_ne` | `ne_syscebnl` |  |  | 46 | 2 | 2 |
| `l10n_ng` | `ng` |  |  | 48 | 4 | 2 |
| `l10n_nl` | `nl` | 349 |  | 144 | 3 | 21 |
| `l10n_no` | `no` | 745 |  | 148 | 5 |  |
| `l10n_nz` | `nz` | 100 |  | 24 | 3 | 2 |
| `l10n_om` | `om` | 139 | 31 | 56 | 4 | 2 |
| `l10n_pa` | `pa` | 105 |  | 8 | 1 |  |
| `l10n_pe` | `pe` | 1228 | 83 | 69 | 15 | 2 |
| `l10n_ph` | `ph` | 104 |  | 254 | 4 | 2 |
| `l10n_pk` | `pk` | 123 | 44 | 244 | 13 |  |
| `l10n_pl` | `pl` | 238 | 136 | 128 | 4 | 4 |
| `l10n_pt` | `pt` | 642 | 211 | 204 | 10 | 4 |
| `l10n_qa` | `qa` | 136 | 31 |  |  |  |
| `l10n_ro` | `ro` | 579 | 139 | 340 | 18 | 9 |
| `l10n_rs` | `rs` | 420 | 86 | 52 | 4 | 3 |
| `l10n_rw` | `rw` | 164 |  | 36 | 2 | 2 |
| `l10n_sa` | `sa` | 169 |  | 174 | 5 | 2 |
| `l10n_sa_edi` | `sa` |  |  | 15 |  |  |
| `l10n_se` | `se` | 332 | 537 | 176 | 4 | 20 |
| `l10n_se` | `se_K2` | 856 |  |  | 4 |  |
| `l10n_se` | `se_K3` | 26 |  |  | 4 |  |
| `l10n_sg` | `sg` | 135 |  | 149 | 5 | 2 |
| `l10n_si` | `si` | 607 | 78 | 256 | 8 | 17 |
| `l10n_sk` | `sk` | 345 | 296 | 178 | 4 | 4 |
| `l10n_sn` | `sn` |  |  | 40 | 3 | 2 |
| `l10n_sn` | `sn_syscebnl` |  |  | 40 | 3 | 2 |
| `l10n_syscohada` | `syscebnl` | 452 | 91 |  |  |  |
| `l10n_syscohada` | `syscohada` | 1134 |  |  |  |  |
| `l10n_td` | `td` |  |  | 32 | 3 | 2 |
| `l10n_td` | `td_syscebnl` |  |  | 32 | 3 | 2 |
| `l10n_tg` | `tg` |  |  | 44 | 2 | 2 |
| `l10n_tg` | `tg_syscebnl` |  |  | 44 | 2 | 2 |
| `l10n_th` | `th` | 144 |  | 72 | 5 |  |
| `l10n_tn` | `tn` | 450 | 162 | 116 | 9 | 3 |
| `l10n_tr` | `tr` | 267 | 64 | 110 | 13 |  |
| `l10n_tr_nilvera_einvoice_extended` | `tr` |  |  | 270 |  |  |
| `l10n_tw` | `tw` | 135 |  | 290 | 19 | 8 |
| `l10n_tz_account` | `tz` | 164 |  | 40 | 2 | 4 |
| `l10n_ua` | `ua_psbo` | 332 | 51 | 49 | 7 |  |
| `l10n_ug` | `ug` | 124 |  | 80 | 1 | 2 |
| `l10n_uk` | `uk` | 118 |  | 76 | 4 | 3 |
| `l10n_uk` | `xi` |  |  |  |  |  |
| `l10n_us_account` | `us` | 101 | 14 | 184 | 23 | 2 |
| `l10n_uy` | `uy` | 166 |  | 48 | 4 | 2 |
| `l10n_uz` | `uz` | 269 | 90 | 16 | 3 |  |
| `l10n_ve` | `ve` | 266 |  | 32 | 4 |  |
| `l10n_vn` | `vn` | 216 | 106 | 60 | 6 |  |
| `l10n_za` | `za` | 117 |  | 64 | 2 |  |
| `l10n_zm_account` | `zm` | 89 | 47 | 76 | 3 | 2 |

---

## 5. Other data files shipped with a template

Besides accounts, groups, taxes, tax groups and fiscal positions, templates ship the files below.
Each is read by the same delimited-file reader and merged by the same rules, described in
[calculations.md](calculations.md).

| File kind | What it creates | Packages that ship it |
|---|---|---|
| `account.asset` | Fixed asset models with their depreciation method and duration | `l10n_be`, `l10n_eg`, `l10n_es`, `l10n_gt`, `l10n_mr`, `l10n_mx`, `l10n_pk`, `l10n_sk`, `l10n_th`, `l10n_uk`, `l10n_us_account`, `l10n_uy` |
| `l10n_tr_nilvera_einvoice_extended.account.tax.code` | Records of this kind | `l10n_tr_nilvera_einvoice_extended` |


### 5.1 Records built by a template function rather than by a file

A template may build records in a function instead of a delimited file, which is how a value that
depends on the company is produced: a code that must include the company's own subdivision, a rate
that differs per branch, an account whose code is derived from a prefix setting. The composition
order of functions and files differs, and that difference is specified in
[calculations.md](calculations.md) section 5.

| Entity | What the function creates | Packages | Which packages |
|---|---|---|---|
| `account.account` | Accounts built in a template function rather than read from a file, usually because their code depends on the company | 82 | `l10n_ae`, `l10n_ar`, `l10n_ar_withholding`, `l10n_at`, `l10n_au`, `l10n_bd`, `l10n_be`, `l10n_bf`, `l10n_bj`, `l10n_bo`, `l10n_br`, `l10n_ca`, `l10n_cd`, `l10n_cf`, `l10n_cg`, `l10n_ch`, `l10n_ci`, `l10n_cl`, `l10n_cm`, `l10n_cn`, `l10n_co`, `l10n_cr`, `l10n_cy`, `l10n_cz`, `l10n_de`, `l10n_dz`, `l10n_ec`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_fi`, `l10n_fr_account`, `l10n_ga`, `l10n_ge`, `l10n_gn`, `l10n_gq`, `l10n_gt`, `l10n_gw`, `l10n_hk`, `l10n_hn`, `l10n_hu`, `l10n_id`, `l10n_ie`, `l10n_il`, `l10n_it`, `l10n_jo`, `l10n_ke`, `l10n_km`, `l10n_lt`, `l10n_ma`, `l10n_ml`, `l10n_mu_account`, `l10n_mx`, `l10n_my`, `l10n_ne`, `l10n_ng`, `l10n_nl`, `l10n_om`, `l10n_pe`, `l10n_ph`, `l10n_pk`, `l10n_pl`, `l10n_pt`, `l10n_qa`, `l10n_ro`, `l10n_sa`, `l10n_sg`, `l10n_sk`, `l10n_sn`, `l10n_td`, `l10n_tg`, `l10n_tn`, `l10n_tr`, `l10n_tw`, `l10n_tz_account`, `l10n_ua`, `l10n_ug`, `l10n_uk`, `l10n_us_account`, `l10n_uy`, `l10n_ve`, `l10n_vn` |
| `account.asset` | Fixed asset models built in a template function | 1 | `l10n_es` |
| `account.cash.rounding` | Cash rounding rules, used where coins below a threshold are no longer in circulation | 1 | `l10n_in` |
| `account.fiscal.position` | Fiscal positions built in a template function, usually because one of them must name the company's own subdivision | 3 | `l10n_ae`, `l10n_in`, `l10n_it_edi_doi` |
| `account.journal` | Journals beyond the six that every template creates, such as a second sales journal or a tax journal | 24 | `l10n_ae`, `l10n_ar`, `l10n_bd`, `l10n_be`, `l10n_br`, `l10n_cl`, `l10n_cn`, `l10n_ec`, `l10n_eg`, `l10n_fr_account`, `l10n_ge`, `l10n_id`, `l10n_jo`, `l10n_kh`, `l10n_kz`, `l10n_lk`, `l10n_lu`, `l10n_mx`, `l10n_ph`, `l10n_sa`, `l10n_tr`, `l10n_tw`, `l10n_uy`, `l10n_vn` |
| `account.reconcile.model` | Reconciliation models that propose a counterpart line when a bank statement line is matched | 4 | `l10n_be`, `l10n_de`, `l10n_lu`, `l10n_ro` |
| `account.tax` | Taxes built in a template function, usually because their rate or their repartition depends on the company | 10 | `l10n_ar_withholding`, `l10n_be_pos_restaurant`, `l10n_es_edi_facturae`, `l10n_es_edi_verifactu`, `l10n_hr_edi`, `l10n_hu_edi`, `l10n_it_edi`, `l10n_it_edi_doi`, `l10n_sa_edi`, `l10n_tr_nilvera_einvoice_extended` |
| `account.tax.group` | Tax groups built in a template function | 1 | `l10n_ar_withholding` |
| `l10n_tr_nilvera_einvoice_extended.account.tax.code` | Administration tax codes attached to taxes | 1 | `l10n_tr_nilvera_einvoice_extended` |
| `res.company` | Company-level defaults: the code length, the currency, the fiscal country, the default accounts, the rounding method, the tax inclusion setting and the account code prefixes | 118 | `l10n_ae`, `l10n_ar`, `l10n_ar_withholding`, `l10n_at`, `l10n_au`, `l10n_bd`, `l10n_be`, `l10n_bf`, `l10n_bg`, `l10n_bh`, `l10n_bj`, `l10n_bo`, `l10n_br`, `l10n_ca`, `l10n_cd`, `l10n_cf`, `l10n_cg`, `l10n_ch`, `l10n_ci`, `l10n_cl`, `l10n_cm`, `l10n_cn`, `l10n_co`, `l10n_cr`, `l10n_cy`, `l10n_cz`, `l10n_de`, `l10n_dk`, `l10n_do`, `l10n_dz`, `l10n_ec`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_et`, `l10n_fi`, `l10n_fr_account`, `l10n_ga`, `l10n_ge`, `l10n_gn`, `l10n_gq`, `l10n_gr`, `l10n_gt`, `l10n_gw`, `l10n_hk`, `l10n_hn`, `l10n_hr`, `l10n_hr_kuna`, `l10n_hu`, `l10n_id`, `l10n_ie`, `l10n_il`, `l10n_in`, `l10n_iq`, `l10n_it`, `l10n_it_edi_doi`, `l10n_jo`, `l10n_jp`, `l10n_ke`, `l10n_kh`, `l10n_km`, `l10n_kr`, `l10n_kw`, `l10n_kz`, `l10n_lb_account`, `l10n_lk`, `l10n_lt`, `l10n_lu`, `l10n_lv`, `l10n_ma`, `l10n_ml`, `l10n_mn`, `l10n_mr`, `l10n_mt`, `l10n_mu_account`, `l10n_mx`, `l10n_my`, `l10n_mz`, `l10n_ne`, `l10n_ng`, `l10n_nl`, `l10n_no`, `l10n_nz`, `l10n_om`, `l10n_pa`, `l10n_pe`, `l10n_ph`, `l10n_pk`, `l10n_pl`, `l10n_pt`, `l10n_qa`, `l10n_ro`, `l10n_rs`, `l10n_rw`, `l10n_sa`, `l10n_se`, `l10n_sg`, `l10n_si`, `l10n_sk`, `l10n_sn`, `l10n_syscohada`, `l10n_td`, `l10n_tg`, `l10n_th`, `l10n_tn`, `l10n_tr`, `l10n_tw`, `l10n_tz_account`, `l10n_ua`, `l10n_ug`, `l10n_uk`, `l10n_us_account`, `l10n_uy`, `l10n_uz`, `l10n_ve`, `l10n_vn`, `l10n_za`, `l10n_zm_account` |

---

## 6. Countries with a file of their own

The countries below carry enough country-specific behaviour — added fields, posting constraints,
document types, numbering series, electronic invoicing flows, point of sale certification — that
their package is specified in full in its own file.

- [Argentina](countries/argentina.md)
- [Australia](countries/australia.md)
- [Austria](countries/austria.md)
- [Belgium](countries/belgium.md)
- [Brazil](countries/brazil.md)
- [Chile](countries/chile.md)
- [Colombia](countries/colombia.md)
- [Denmark](countries/denmark.md)
- [Ecuador](countries/ecuador.md)
- [Egypt](countries/egypt.md)
- [Finland](countries/finland.md)
- [France](countries/france.md)
- [Germany](countries/germany.md)
- [India](countries/india.md)
- [Indonesia](countries/indonesia.md)
- [Italy](countries/italy.md)
- [Kenya](countries/kenya.md)
- [Luxembourg](countries/luxembourg.md)
- [Malaysia](countries/malaysia.md)
- [Mexico](countries/mexico.md)
- [The Netherlands](countries/netherlands.md)
- [Norway](countries/norway.md)
- [Peru](countries/peru.md)
- [Poland](countries/poland.md)
- [Portugal](countries/portugal.md)
- [Romania](countries/romania.md)
- [Saudi Arabia](countries/saudi-arabia.md)
- [Singapore](countries/singapore.md)
- [Spain](countries/spain.md)
- [Sweden](countries/sweden.md)
- [Switzerland](countries/switzerland.md)
- [Türkiye](countries/turkey.md)
- [The United Kingdom](countries/united-kingdom.md)
- [The United States](countries/united-states.md)
- [United Arab Emirates](countries/united-arab-emirates.md)
- [Uruguay](countries/uruguay.md)
- [Vietnam](countries/vietnam.md)

Every other package in this catalogue contributes configuration only: a chart of accounts template,
its taxes and tax groups, its fiscal positions, its journals and, where the country publishes one, a
statutory report structure. Those packages add no field, no constraint and no state machine, so the
rows above are their complete specification, together with the generic loading behaviour in
[workflows.md](workflows.md).
