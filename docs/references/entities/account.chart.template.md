# Account Chart Template (`account.chart.template`)

**Transport name:** `account.chart.template`  
**Storage name:** `account_chart_template`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account`  
**Extended by packages:** `sale`, `stock_account`, `l10n_ae`, `l10n_latam_invoice_document`, `l10n_ar`, `l10n_ar`, `l10n_ar`, `l10n_ar`, `l10n_latam_check`, `l10n_ar_withholding`, `l10n_at`, `l10n_au`, `l10n_bd`, `l10n_be`, `l10n_be`, `l10n_be`, `l10n_be_pos_restaurant`, `l10n_syscohada`, `l10n_syscohada`, `l10n_bf`, `l10n_bf`, `l10n_bg`, `l10n_bh`, `l10n_bj`, `l10n_bj`, `l10n_bo`, `l10n_br`, `l10n_ca`, `l10n_cd`, `l10n_cd`, `l10n_cf`, `l10n_cf`, `l10n_cg`, `l10n_cg`, `l10n_ch`, `l10n_ci`, `l10n_ci`, `l10n_cl`, `l10n_cm`, `l10n_cm`, `l10n_cn`, `l10n_cn`, `l10n_cn`, `l10n_co`, `l10n_cr`, `l10n_cy`, `l10n_cz`, `l10n_de`, `l10n_de`, `l10n_de`, `l10n_dk`, `l10n_do`, `l10n_dz`, `l10n_ec`, `l10n_ec_stock`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es`, `l10n_es_edi_facturae`, `l10n_es_edi_verifactu`, `l10n_et`, `l10n_fi`, `l10n_fr_account`, `l10n_fr_account`, `l10n_fr_account`, `l10n_fr_account`, `l10n_fr_account`, `l10n_fr_account`, `l10n_fr_account`, `l10n_ga`, `l10n_ga`, `l10n_ge`, `l10n_gn`, `l10n_gn`, `l10n_gq`, `l10n_gq`, `l10n_gr`, `l10n_gt`, `l10n_gw`, `l10n_gw`, `l10n_hk`, `l10n_hn`, `l10n_hr`, `l10n_hr_edi`, `l10n_hr_kuna`, `l10n_hu`, `l10n_hu_edi`, `l10n_id`, `l10n_ie`, `l10n_ie`, `l10n_il`, `l10n_in`, `l10n_iq`, `l10n_it`, `l10n_it_edi`, `l10n_it_edi_doi`, `l10n_jo`, `l10n_jp`, `l10n_ke`, `l10n_kh`, `l10n_km`, `l10n_km`, `l10n_kr`, `l10n_kw`, `l10n_kz`, `l10n_lb_account`, `l10n_lk`, `l10n_lt`, `l10n_lu`, `l10n_lv`, `l10n_ma`, `l10n_ml`, `l10n_ml`, `l10n_mn`, `l10n_mr`, `l10n_mt`, `l10n_mu_account`, `l10n_mx`, `l10n_my`, `l10n_mz`, `l10n_ne`, `l10n_ne`, `l10n_ng`, `l10n_nl`, `l10n_nl`, `l10n_no`, `l10n_nz`, `l10n_om`, `l10n_pa`, `l10n_pe`, `l10n_ph`, `l10n_pk`, `l10n_pl`, `l10n_pt`, `l10n_qa`, `l10n_ro`, `l10n_rs`, `l10n_rw`, `l10n_sa`, `l10n_sa_edi`, `l10n_se`, `l10n_se`, `l10n_se`, `l10n_sg`, `l10n_si`, `l10n_sk`, `l10n_sn`, `l10n_sn`, `l10n_td`, `l10n_td`, `l10n_tg`, `l10n_tg`, `l10n_th`, `l10n_tn`, `l10n_tr`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw`, `l10n_tz_account`, `l10n_ua`, `l10n_ug`, `l10n_uk`, `l10n_uk`, `l10n_us_account`, `l10n_uy`, `l10n_uz`, `l10n_ve`, `l10n_vn`, `l10n_za`, `l10n_zm_account`

Description: Account Chart Template

## Operations (523)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_generic_coa_template_data` | preparation rule | self | `account` |  | Return the data necessary for the chart template.  :return: all the values that are not stored but are used to instancieate          the chart of accounts. Common keys are:          * property_*          * code_digits :rtype: dict |
| `_get_generic_coa_res_company` | preparation rule | self | `account` |  | Return the data to be written on the company.  The data is a mapping the XMLID to the create/write values of a record.  :rtype: dict[(str, int), dict] |
| `_get_generic_coa_account_account` | preparation rule | self | `account` |  |  |
| `_template_register` | internal rule | self | `account` |  |  |
| `_post_model_setup__` | internal rule | self | `account` |  |  |
| `_get_chart_template_mapping` | preparation rule | self, get_all | `account` |  | Get basic information about available CoA and their modules.  :return: a mapping between the template code and a dictionary containing the          name, country id, country name, module dependencies and parent template :rtype: dict[str, dict] |
| `_select_chart_template` | internal rule | self, country | `account` |  | Get the available templates in a format suited for Selection fields. |
| `_guess_chart_template` | internal rule | self, country | `account` |  | Guess the most appropriate template based on the country. |
| `try_loading` | operation | self, template_code, company, install_demo, force_create | `account`, `l10n_ar` |  | Check if the chart template can be loaded then proceeds installing it.  :param template_code: code of the chart template to be loaded. :type template_code: str :param company: the company we try to load the chart template on.     If not provided, it is retrieved from the context. :type company: int, Model<res.company> :param install_demo: whether or not we should load demo data right after loading the     chart template. :type install_demo: bool :param force_create: Determines the loading behavior. If True, forces the creation of new entries;     if False, prevents new creations and performs u |
| `_load` | internal rule | self, template_code, company, install_demo, force_create | `account`, `l10n_ar`, `l10n_ec_stock`, `l10n_hu_edi`, `l10n_in`, `l10n_uy` |  | Install this chart of accounts for the current company.  :param template_code: code of the chart template to be loaded. :param company: the company we try to load the chart template on.     If not provided, it is retrieved from the context. :param install_demo: whether or not we should load demo data right after loading the     chart template. |
| `_install_demo` | internal rule | self, companies | `account` | model |  |
| `_pre_reload_data` | internal rule | self, company, template_data, data, force_create | `account` |  | Pre-process the data in case of reloading the chart of accounts.  When we reload the chart of accounts, we only want to update fields that are main configuration, like: - tax tags |
| `_pre_load_data` | internal rule | self, template_code, company, template_data, data | `account` |  | Pre-process the data and preload some values.  Some of the data needs special pre_process before being fed to the database. e.g. the account codes' width must be standardized to the code_digits applied. The fiscal country code must be put in place before taxes are generated. |
| `_load_data` | internal rule | self, data | `account` |  | Load all the data linked to the template into the database.  The data can contain translation values (i.e. `name@fr_FR` to translate the name in French) An xml_id that doesn't contain a `.` will be treated as being linked to `account` and prefixed with the company's id (i.e. `cash` is interpreted as `account.1_cash` if the company's id is 1)  :param data: Basically all the final data of records to create/update for the chart              of accounts. It is a mapping {model: {xml_id: values}}. :type data: dict[str, dict[(str, int), dict]] |
| `_post_load_data` | internal rule | self, template_code, company, template_data | `account`, `l10n_be`, `l10n_ec`, `l10n_ie`, `l10n_in`, `l10n_ma`, `l10n_nl`, `l10n_uk` |  | If the company is located in Northern Ireland, activate the relevant taxes and fiscal postions. |
| `_get_bank_fees_reco_account` | preparation rule | self, company | `account`, `l10n_be`, `l10n_fr_account`, `l10n_lu` |  |  |
| `_get_property_accounts` | preparation rule | self, additional_properties | `account`, `sale` |  |  |
| `_get_chart_template_model_data` | preparation rule | self, template_code, model | `account` |  | Lightweight version of `_get_chart_template_data` targeting only one model. |
| `_get_chart_template_data` | preparation rule | self, template_code | `account` |  |  |
| `_get_accounts_data_values` | preparation rule | self, company, template_data, bank_prefix, code_digits | `account`, `l10n_mx`, `l10n_us_account` |  |  |
| `_setup_utility_bank_accounts` | internal rule | self, template_code, company, template_data | `account`, `l10n_at`, `l10n_de`, `l10n_dk`, `l10n_lt` |  | Define basic bank accounts for the company.  - Suspense Account - Outstanding Receipts/Payments Accounts - Cash Difference Gain/Loss Accounts - Liquidity Transfer Account |
| `_create_outstanding_accounts` | internal rule | self, company, bank_prefix, code_digits | `account` |  |  |
| `_instantiate_foreign_taxes` | internal rule | self, country, company | `account` | model | Create and configure foreign taxes from the provided country.  Instantiate the taxes as they would be for the foreign localization only replacing the accounts used by the most probable account we can retrieve from the company's localization. This method is intended as a shortcut for instantiation, accelerating it, not as an out-of-the-box solution 100% correct solution. |
| `_get_account_account` | preparation rule | self, template_code | `account` |  |  |
| `_get_account_group` | preparation rule | self, template_code | `account` |  |  |
| `_get_account_tax_group` | preparation rule | self, template_code | `account` |  |  |
| `_get_account_tax` | preparation rule | self, template_code | `account` |  |  |
| `_get_account_fiscal_position` | preparation rule | self, template_code | `account` |  |  |
| `_get_account_journal` | preparation rule | self, template_code | `account`, `l10n_pt` |  |  |
| `_get_account_reconcile_model` | preparation rule | self, template_code | `account` |  |  |
| `company_xmlid` | operation | self, xmlid, company | `account` |  |  |
| `ref` | operation | self, xmlid, raise_if_not_found | `account` |  |  |
| `_get_parent_template` | preparation rule | self, code | `account` |  |  |
| `_get_tag_mapper` | preparation rule | self, country_id | `account` |  |  |
| `_deref_account_tags` | internal rule | self, template_code, tax_data | `account`, `l10n_fr_account`, `l10n_uk` |  |  |
| `_parse_csv` | internal rule | self, template_code, model, module | `account` |  |  |
| `_get_untranslatable_fields_target_language` | preparation rule | self, template_code, company | `account` |  | Return the code of the language we want to translate the untranslatable fields into. |
| `_get_untranslatable_fields_to_translate` | preparation rule | self | `account` |  | Return information about the untranslatable fields we want to translate anyway.  :return: Dictionary mapping the model name to the list of all its untranslatable fields          that we want to translate anyway :rtype: dict[str, list[str]] |
| `_get_translatable_template_model_fields` | preparation rule | self | `account` |  |  |
| `_get_untranslated_translatable_template_model_records` | preparation rule | self, langs, companies | `account` |  | Return information about the records of any model in TEMPLATE_MODELS (and belonging to companies) that need to be translated. Records are in need of translation if they have a translatable field which is missing a translation (into any of the languages given in langs).  :param langs: The codes of the languages into which we want to translate the records. :type langs: list[str] :param companies: Records belonging to these companies will be considered. :type companies: Model<res.company> :return: The records which information will be returned are those records that have at least 1 untranslated t |
| `_get_field_translation` | preparation rule | self, record, fname, lang | `account` |  | Return the value for language lang for field with fname from record (or None if none exists).  :param record: record formatted like in the template data (generated by _get_chart_template_data) :type record: dict :param fname: the name of a field (in record) as string :type str :param lang: the code of a res.lang :type str :return record[fname] translated into lang (or None) :rtype str |
| `_load_translations` | internal rule | self, langs, companies, template_data | `account` |  | Load the translations of the chart template.  :param langs: the lang code to load the translations for. If one of the codes is not present,               we are looking for it more generic locale (i.e. `en` instead of `en_US`) :type langs: list[str] :param companies: the companies to load the translations for :type companies: Model<res.company> |
| `_get_stock_account_res_company` | preparation rule | self, template_code | `stock_account` |  |  |
| `_get_stock_account_account` | preparation rule | self, template_code | `stock_account` |  |  |
| `_get_stock_account_journal` | preparation rule | self, template_code | `stock_account` |  |  |
| `_get_stock_template_data` | preparation rule | self, template_code | `stock_account` |  |  |
| `_get_ae_template_data` | preparation rule | self | `l10n_ae` |  |  |
| `_get_ae_res_company` | preparation rule | self | `l10n_ae` |  |  |
| `_get_ae_account_journal` | preparation rule | self | `l10n_ae` |  | If UAE chart, we add 2 new journals TA and IFRS |
| `_get_ae_account_fiscal_position` | preparation rule | self | `l10n_ae` |  |  |
| `_get_ae_account_account` | preparation rule | self | `l10n_ae` |  |  |
| `_get_latam_document_account_journal` | preparation rule | self, template_code | `l10n_latam_invoice_document` |  | We add use_documents or not depending on the context |
| `_get_ar_base_template_data` | preparation rule | self | `l10n_ar` |  |  |
| `_get_ar_base_res_company` | preparation rule | self | `l10n_ar_withholding`, `l10n_ar` |  |  |
| `_get_ar_account_journal` | preparation rule | self | `l10n_ar` |  | In case of an Argentinean CoA, we modify the default values of the sales journal to be a preprinted journal |
| `_get_ar_base_account_account` | preparation rule | self | `l10n_ar` |  |  |
| `_get_ar_ex_template_data` | preparation rule | self | `l10n_ar` |  |  |
| `_get_ar_ex_res_company` | preparation rule | self | `l10n_ar` |  |  |
| `_get_ar_ri_template_data` | preparation rule | self | `l10n_ar` |  |  |
| `_get_ar_ri_res_company` | preparation rule | self | `l10n_ar` |  |  |
| `_get_ar_responsibility_match` | preparation rule | self, chart_template | `l10n_ar` | model | return responsibility type that match with the given chart_template code |
| `_get_third_party_checks_country_codes` | preparation rule | self | `l10n_latam_check` | model | Return the list of country codes for the countries where third party checks journals should be created when installing the COA |
| `_get_latam_check_account_journal` | preparation rule | self, template_code | `l10n_latam_check` |  |  |
| `_get_latam_check_outstanding_account_account` | preparation rule | self, template_code | `l10n_latam_check` |  |  |
| `_get_ar_base_withholding_account_account` | preparation rule | self | `l10n_ar_withholding` |  |  |
| `_get_ar_ri_withholding_account_tax_group` | preparation rule | self | `l10n_ar_withholding` |  |  |
| `_get_ar_ri_withholding_account_tax` | preparation rule | self | `l10n_ar_withholding` |  |  |
| `_get_ar_ex_withholding_account_tax_group` | preparation rule | self | `l10n_ar_withholding` |  |  |
| `_get_ar_ex_withholding_account_tax` | preparation rule | self | `l10n_ar_withholding` |  |  |
| `_get_at_template_data` | preparation rule | self | `l10n_at` |  |  |
| `_get_at_res_company` | preparation rule | self | `l10n_at` |  |  |
| `_get_at_account_account` | preparation rule | self | `l10n_at` |  |  |
| `_get_au_template_data` | preparation rule | self | `l10n_au` |  |  |
| `_get_au_res_company` | preparation rule | self | `l10n_au` |  |  |
| `_get_au_account_account` | preparation rule | self | `l10n_au` |  |  |
| `_get_bd_template_data` | preparation rule | self | `l10n_bd` |  |  |
| `_get_bd_res_company` | preparation rule | self | `l10n_bd` |  |  |
| `_get_bd_account_journal` | preparation rule | self | `l10n_bd` |  |  |
| `_get_bd_account_account` | preparation rule | self | `l10n_bd` |  |  |
| `_get_be_template_data` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_res_company` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_account_journal` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_reconcile_model` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_account_account` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_asso_template_data` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_asso_res_company` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_comp_template_data` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_comp_res_company` | preparation rule | self | `l10n_be` |  |  |
| `_get_be_pos_restaurant_account_tax` | preparation rule | self | `l10n_be_pos_restaurant` |  |  |
| `_get_syscohada_template_data` | preparation rule | self | `l10n_syscohada` |  |  |
| `_get_syscohada_res_company` | preparation rule | self | `l10n_syscohada` |  |  |
| `_get_syscebnl_template_data` | preparation rule | self | `l10n_syscohada` |  |  |
| `_get_syscebnl_res_company` | preparation rule | self | `l10n_syscohada` |  |  |
| `_get_bf_template_data` | preparation rule | self | `l10n_bf` |  |  |
| `_get_bf_res_company` | preparation rule | self | `l10n_bf` |  |  |
| `_get_bf_account_account` | preparation rule | self | `l10n_bf` |  |  |
| `_get_bf_syscebnl_template_data` | preparation rule | self | `l10n_bf` |  |  |
| `_get_bf_syscebnl_res_company` | preparation rule | self | `l10n_bf` |  |  |
| `_get_bf_syscebnl_account_account` | preparation rule | self | `l10n_bf` |  |  |
| `_get_bg_template_data` | preparation rule | self | `l10n_bg` |  |  |
| `_get_bg_res_company` | preparation rule | self | `l10n_bg` |  |  |
| `_get_bh_template_data` | preparation rule | self | `l10n_bh` |  |  |
| `_get_bh_res_company` | preparation rule | self | `l10n_bh` |  |  |
| `_get_bj_template_data` | preparation rule | self | `l10n_bj` |  |  |
| `_get_bj_res_company` | preparation rule | self | `l10n_bj` |  |  |
| `_get_bj_account_account` | preparation rule | self | `l10n_bj` |  |  |
| `_get_bj_syscebnl_template_data` | preparation rule | self | `l10n_bj` |  |  |
| `_get_bj_syscebnl_res_company` | preparation rule | self | `l10n_bj` |  |  |
| `_get_bj_syscebnl_account_account` | preparation rule | self | `l10n_bj` |  |  |
| `_get_bo_template_data` | preparation rule | self | `l10n_bo` |  |  |
| `_get_bo_res_company` | preparation rule | self | `l10n_bo` |  |  |
| `_get_bo_account_account` | preparation rule | self | `l10n_bo` |  |  |
| `_get_br_template_data` | preparation rule | self | `l10n_br` |  |  |
| `_get_br_res_company` | preparation rule | self | `l10n_br` |  |  |
| `_get_br_account_journal` | preparation rule | self | `l10n_br` |  |  |
| `_get_br_account_account` | preparation rule | self | `l10n_br` |  |  |
| `_get_ca_template_data` | preparation rule | self | `l10n_ca` |  |  |
| `_get_ca_res_company` | preparation rule | self | `l10n_ca` |  |  |
| `_get_ca_account_account` | preparation rule | self | `l10n_ca` |  |  |
| `_get_cd_template_data` | preparation rule | self | `l10n_cd` |  |  |
| `_get_cd_res_company` | preparation rule | self | `l10n_cd` |  |  |
| `_get_cd_account_account` | preparation rule | self | `l10n_cd` |  |  |
| `_get_cd_syscebnl_template_data` | preparation rule | self | `l10n_cd` |  |  |
| `_get_cd_syscebnl_res_company` | preparation rule | self | `l10n_cd` |  |  |
| `_get_cd_syscebnl_account_account` | preparation rule | self | `l10n_cd` |  |  |
| `_get_cf_template_data` | preparation rule | self | `l10n_cf` |  |  |
| `_get_cf_res_company` | preparation rule | self | `l10n_cf` |  |  |
| `_get_cf_account_account` | preparation rule | self | `l10n_cf` |  |  |
| `_get_cf_syscebnl_res_company` | preparation rule | self | `l10n_cf` |  |  |
| `_get_cf_syscebnl_account_account` | preparation rule | self | `l10n_cf` |  |  |
| `_get_cg_template_data` | preparation rule | self | `l10n_cg` |  |  |
| `_get_cg_res_company` | preparation rule | self | `l10n_cg` |  |  |
| `_get_cg_account_account` | preparation rule | self | `l10n_cg` |  |  |
| `_get_cg_syscebnl_template_data` | preparation rule | self | `l10n_cg` |  |  |
| `_get_cg_syscebnl_res_company` | preparation rule | self | `l10n_cg` |  |  |
| `_get_cg_syscebnl_account_account` | preparation rule | self | `l10n_cg` |  |  |
| `_get_ch_template_data` | preparation rule | self | `l10n_ch` |  |  |
| `_get_ch_res_company` | preparation rule | self | `l10n_ch` |  |  |
| `_get_ch_account_account` | preparation rule | self | `l10n_ch` |  |  |
| `_get_ci_template_data` | preparation rule | self | `l10n_ci` |  |  |
| `_get_ci_res_company` | preparation rule | self | `l10n_ci` |  |  |
| `_get_ci_account_account` | preparation rule | self | `l10n_ci` |  |  |
| `_get_ci_syscebnl_template_data` | preparation rule | self | `l10n_ci` |  |  |
| `_get_ci_syscebnl_res_company` | preparation rule | self | `l10n_ci` |  |  |
| `_get_ci_syscebnl_account_account` | preparation rule | self | `l10n_ci` |  |  |
| `_get_cl_template_data` | preparation rule | self | `l10n_cl` |  |  |
| `_get_cl_res_company` | preparation rule | self | `l10n_cl` |  |  |
| `_get_cl_account_account` | preparation rule | self | `l10n_cl` |  |  |
| `_get_cl_account_journal` | preparation rule | self | `l10n_cl` |  |  |
| `_get_cm_template_data` | preparation rule | self | `l10n_cm` |  |  |
| `_get_cm_res_company` | preparation rule | self | `l10n_cm` |  |  |
| `_get_cm_account_account` | preparation rule | self | `l10n_cm` |  |  |
| `_get_cm_syscebnl_template_data` | preparation rule | self | `l10n_cm` |  |  |
| `_get_cm_syscebnl_res_company` | preparation rule | self | `l10n_cm` |  |  |
| `_get_cm_syscebnl_account_account` | preparation rule | self | `l10n_cm` |  |  |
| `_get_cn_template_data` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_res_company` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_account_account` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_common_template_data` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_common_res_company` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_account_journal` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_large_bis_template_data` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_large_bis_company` | preparation rule | self | `l10n_cn` |  |  |
| `_get_cn_large_bis_account_account` | preparation rule | self | `l10n_cn` |  |  |
| `_get_co_template_data` | preparation rule | self | `l10n_co` |  |  |
| `_get_co_res_company` | preparation rule | self | `l10n_co` |  |  |
| `_get_co_account_account` | preparation rule | self | `l10n_co` |  |  |
| `_get_cr_template_data` | preparation rule | self | `l10n_cr` |  |  |
| `_get_cr_res_company` | preparation rule | self | `l10n_cr` |  |  |
| `_get_cr_account_account` | preparation rule | self | `l10n_cr` |  |  |
| `_get_cy_template_data` | preparation rule | self | `l10n_cy` |  |  |
| `_get_cy_res_company` | preparation rule | self | `l10n_cy` |  |  |
| `_get_cy_account_account` | preparation rule | self | `l10n_cy` |  |  |
| `_get_cz_template_data` | preparation rule | self | `l10n_cz` |  |  |
| `_get_cz_res_company` | preparation rule | self | `l10n_cz` |  |  |
| `_get_demo_data_move` | preparation rule | self, company | `l10n_cz` | model |  |
| `_get_cz_account_account` | preparation rule | self | `l10n_cz` |  |  |
| `_get_de_res_company` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr03_template_data` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr03_res_company` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr03_reconcile_model` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr03_account_account` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr04_template_data` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr04_res_company` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr04_reconcile_model` | preparation rule | self | `l10n_de` |  |  |
| `_get_de_skr04_account_account` | preparation rule | self | `l10n_de` |  |  |
| `_get_dk_template_data` | preparation rule | self | `l10n_dk` |  |  |
| `_get_dk_res_company` | preparation rule | self | `l10n_dk` |  |  |
| `_get_do_template_data` | preparation rule | self | `l10n_do` |  |  |
| `_get_do_res_company` | preparation rule | self | `l10n_do` |  |  |
| `_get_dz_template_data` | preparation rule | self | `l10n_dz` |  |  |
| `_get_dz_res_company` | preparation rule | self | `l10n_dz` |  |  |
| `_get_dz_account_account` | preparation rule | self | `l10n_dz` |  |  |
| `_get_ec_template_data` | preparation rule | self | `l10n_ec` |  |  |
| `_get_ec_res_company` | preparation rule | self | `l10n_ec` |  |  |
| `_get_ec_account_journal` | preparation rule | self | `l10n_ec` |  | In case of an Ecuador, we modified the sales journal |
| `_get_ec_account_account` | preparation rule | self | `l10n_ec` |  |  |
| `_l10n_ec_setup_location_accounts` | internal rule | self, companies | `l10n_ec_stock` |  |  |
| `_get_ee_template_data` | preparation rule | self | `l10n_ee` |  |  |
| `_get_ee_res_company` | preparation rule | self | `l10n_ee` |  |  |
| `_get_ee_account_account` | preparation rule | self | `l10n_ee` |  |  |
| `_get_eg_template_data` | preparation rule | self | `l10n_eg` |  |  |
| `_get_eg_res_company` | preparation rule | self | `l10n_eg` |  |  |
| `_get_eg_account_journal` | preparation rule | self | `l10n_eg` |  | If EGYPT chart, we add 2 new journals TA and IFRS |
| `_get_eg_account_account` | preparation rule | self | `l10n_eg` |  |  |
| `_get_es_coop_full_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_coop_full_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_coop_pymes_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_coop_pymes_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_assec_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_assec_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_assoc_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_assoc_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_assoc_account_account` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_assoc_account_asset` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_common_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_common_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_full_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_full_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_full_account_account` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_full_account_asset` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_pymes_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_pymes_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_pymes_account_account` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_canary_pymes_account_asset` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_common_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_common_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_common_account_account` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_common_mainland_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_common_mainland_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_product` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_full_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_full_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_pymes_template_data` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_pymes_res_company` | preparation rule | self | `l10n_es` |  |  |
| `_get_es_facturae_account_tax_es_common_mainland` | preparation rule | self | `l10n_es_edi_facturae` |  |  |
| `_get_es_facturae_account_tax_es_canary_common` | preparation rule | self | `l10n_es_edi_facturae` |  |  |
| `_get_es_verifactu_account_tax_es_common_mainland` | preparation rule | self | `l10n_es_edi_verifactu` |  |  |
| `_get_es_verifactu_account_tax_es_canary_common` | preparation rule | self | `l10n_es_edi_verifactu` |  |  |
| `_get_et_template_data` | preparation rule | self | `l10n_et` |  |  |
| `_get_et_res_company` | preparation rule | self | `l10n_et` |  |  |
| `_get_fi_template_data` | preparation rule | self | `l10n_fi` |  |  |
| `_get_fi_res_company` | preparation rule | self | `l10n_fi` |  |  |
| `_get_fi_account_account` | preparation rule | self | `l10n_fi` |  |  |
| `_get_gf_template_data` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_gp_template_data` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_mq_template_data` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_re_template_data` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_yt_template_data` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_fr_template_data` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_fr_res_company` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_fr_account_journal` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_fr_account_account` | preparation rule | self | `l10n_fr_account` |  |  |
| `_get_mc_template_data` | preparation rule | self | `l10n_fr_account`, `l10n_uk` |  |  |
| `_get_ga_template_data` | preparation rule | self | `l10n_ga` |  |  |
| `_get_ga_res_company` | preparation rule | self | `l10n_ga` |  |  |
| `_get_ga_account_account` | preparation rule | self | `l10n_ga` |  |  |
| `_get_ga_syscebnl_template_data` | preparation rule | self | `l10n_ga` |  |  |
| `_get_ga_syscebnl_res_company` | preparation rule | self | `l10n_ga` |  |  |
| `_get_ga_syscebnl_account_account` | preparation rule | self | `l10n_ga` |  |  |
| `_get_ge_account_journal` | preparation rule | self | `l10n_ge` |  |  |
| `_get_ge_coa_account_account` | preparation rule | self | `l10n_ge` |  |  |
| `_get_ge_res_company` | preparation rule | self | `l10n_ge` |  |  |
| `_get_ge_template_data` | preparation rule | self | `l10n_ge` |  |  |
| `_get_gn_template_data` | preparation rule | self | `l10n_gn` |  |  |
| `_get_gn_res_company` | preparation rule | self | `l10n_gn` |  |  |
| `_get_gn_account_account` | preparation rule | self | `l10n_gn` |  |  |
| `_get_gn_syscebnl_template_data` | preparation rule | self | `l10n_gn` |  |  |
| `_get_gn_syscebnl_res_company` | preparation rule | self | `l10n_gn` |  |  |
| `_get_gn_syscebnl_account_account` | preparation rule | self | `l10n_gn` |  |  |
| `_get_gq_template_data` | preparation rule | self | `l10n_gq` |  |  |
| `_get_gq_res_company` | preparation rule | self | `l10n_gq` |  |  |
| `_get_gq_account_account` | preparation rule | self | `l10n_gq` |  |  |
| `_get_gq_syscebnl_template_data` | preparation rule | self | `l10n_gq` |  |  |
| `_get_gq_syscebnl_res_company` | preparation rule | self | `l10n_gq` |  |  |
| `_get_gq_syscebnl_account_account` | preparation rule | self | `l10n_gq` |  |  |
| `_get_gr_template_data` | preparation rule | self | `l10n_gr` |  |  |
| `_get_gr_res_company` | preparation rule | self | `l10n_gr` |  |  |
| `_get_gt_template_data` | preparation rule | self | `l10n_gt` |  |  |
| `_get_gt_res_company` | preparation rule | self | `l10n_gt` |  |  |
| `_get_gt_account_account` | preparation rule | self | `l10n_gt` |  |  |
| `_get_gw_template_data` | preparation rule | self | `l10n_gw` |  |  |
| `_get_gw_res_company` | preparation rule | self | `l10n_gw` |  |  |
| `_get_gw_account_account` | preparation rule | self | `l10n_gw` |  |  |
| `_get_gw_syscebnl_template_data` | preparation rule | self | `l10n_gw` |  |  |
| `_get_gw_syscebnl_res_company` | preparation rule | self | `l10n_gw` |  |  |
| `_get_gw_syscebnl_account_account` | preparation rule | self | `l10n_gw` |  |  |
| `_get_hk_template_data` | preparation rule | self | `l10n_hk` |  |  |
| `_get_hk_res_company` | preparation rule | self | `l10n_hk` |  |  |
| `_get_hk_account_account` | preparation rule | self | `l10n_hk` |  |  |
| `_get_hn_template_data` | preparation rule | self | `l10n_hn` |  |  |
| `_get_hn_res_company` | preparation rule | self | `l10n_hn` |  |  |
| `_get_hn_account_account` | preparation rule | self | `l10n_hn` |  |  |
| `_get_hr_template_data` | preparation rule | self | `l10n_hr` |  |  |
| `_get_hr_res_company` | preparation rule | self | `l10n_hr` |  |  |
| `_get_hr_edi_account_tax` | preparation rule | self | `l10n_hr_edi` |  |  |
| `_get_hr_kuna_template_data` | preparation rule | self | `l10n_hr_kuna` |  |  |
| `_get_hr_kuna_res_company` | preparation rule | self | `l10n_hr_kuna` |  |  |
| `_get_hu_template_data` | preparation rule | self | `l10n_hu` |  |  |
| `_get_hu_res_company` | preparation rule | self | `l10n_hu` |  |  |
| `_get_hu_account_account` | preparation rule | self | `l10n_hu` |  |  |
| `_get_hu_account_tax` | preparation rule | self | `l10n_hu_edi` |  |  |
| `_get_id_template_data` | preparation rule | self | `l10n_id` |  |  |
| `_get_id_res_company` | preparation rule | self | `l10n_id` |  |  |
| `_get_id_account_account` | preparation rule | self | `l10n_id` |  |  |
| `_get_id_account_journal` | preparation rule | self | `l10n_id` |  |  |
| `_get_ie_template_data` | preparation rule | self | `l10n_ie` |  |  |
| `_get_ie_res_company` | preparation rule | self | `l10n_ie` |  |  |
| `_get_ie_account_account` | preparation rule | self | `l10n_ie` |  |  |
| `_get_il_template_data` | preparation rule | self | `l10n_il` |  |  |
| `_get_il_res_company` | preparation rule | self | `l10n_il` |  |  |
| `_get_il_account_account` | preparation rule | self | `l10n_il` |  |  |
| `_get_in_template_data` | preparation rule | self | `l10n_in` |  |  |
| `_get_in_res_company` | preparation rule | self | `l10n_in` |  |  |
| `_get_in_account_cash_rounding` | preparation rule | self | `l10n_in` |  |  |
| `_get_in_account_fiscal_position` | preparation rule | self | `l10n_in` |  |  |
| `_get_l10n_in_fiscal_tax_vals` | preparation rule | self, fiscal_position_xml_ids | `l10n_in` |  |  |
| `_get_iq_template_data` | preparation rule | self | `l10n_iq` |  |  |
| `_get_iq_res_company` | preparation rule | self | `l10n_iq` |  |  |
| `_get_it_template_data` | preparation rule | self | `l10n_it` |  |  |
| `_get_it_res_company` | preparation rule | self | `l10n_it` |  |  |
| `_get_it_account_account` | preparation rule | self | `l10n_it` |  |  |
| `_get_it_account_tax` | preparation rule | self | `l10n_it_edi` |  |  |
| `_get_it_edi_doi_account_tax` | preparation rule | self | `l10n_it_edi_doi` |  |  |
| `_get_it_edi_doi_account_fiscal_position` | preparation rule | self | `l10n_it_edi_doi` |  |  |
| `_get_it_edi_doi_res_company` | preparation rule | self | `l10n_it_edi_doi` |  |  |
| `_get_jo_standard_template_data` | preparation rule | self | `l10n_jo` |  |  |
| `_get_jo_standard_res_company` | preparation rule | self | `l10n_jo` |  |  |
| `_get_jo_standard_account_journal` | preparation rule | self | `l10n_jo` |  |  |
| `_get_jo_standard_account_account` | preparation rule | self | `l10n_jo` |  |  |
| `_get_jp_template_data` | preparation rule | self | `l10n_jp` |  |  |
| `_get_jp_res_company` | preparation rule | self | `l10n_jp` |  |  |
| `_get_ke_template_data` | preparation rule | self | `l10n_ke` |  |  |
| `_get_ke_res_company` | preparation rule | self | `l10n_ke` |  |  |
| `_get_ke_account_account` | preparation rule | self | `l10n_ke` |  |  |
| `_get_kh_template_data` | preparation rule | self | `l10n_kh` |  |  |
| `_get_kh_res_company` | preparation rule | self | `l10n_kh` |  |  |
| `_get_kh_account_journal` | preparation rule | self | `l10n_kh` |  |  |
| `_get_km_template_data` | preparation rule | self | `l10n_km` |  |  |
| `_get_km_res_company` | preparation rule | self | `l10n_km` |  |  |
| `_get_km_account_account` | preparation rule | self | `l10n_km` |  |  |
| `_get_km_syscebnl_template_data` | preparation rule | self | `l10n_km` |  |  |
| `_get_km_syscebnl_res_company` | preparation rule | self | `l10n_km` |  |  |
| `_get_km_syscebnl_account_account` | preparation rule | self | `l10n_km` |  |  |
| `_get_kr_template_data` | preparation rule | self | `l10n_kr` |  |  |
| `_get_kr_res_company` | preparation rule | self | `l10n_kr` |  |  |
| `_get_kw_template_data` | preparation rule | self | `l10n_kw` |  |  |
| `_get_kw_res_company` | preparation rule | self | `l10n_kw` |  |  |
| `_get_kz_template_data` | preparation rule | self | `l10n_kz` |  |  |
| `_get_kz_res_company` | preparation rule | self | `l10n_kz` |  |  |
| `_get_kz_account_journal` | preparation rule | self | `l10n_kz` |  |  |
| `_get_lb_template_data` | preparation rule | self | `l10n_lb_account` |  |  |
| `_get_leb_res_company` | preparation rule | self | `l10n_lb_account` |  |  |
| `_get_lk_template_data` | preparation rule | self | `l10n_lk` |  |  |
| `_get_lk_res_company` | preparation rule | self | `l10n_lk` |  |  |
| `_get_lk_account_journal` | preparation rule | self | `l10n_lk` |  |  |
| `_get_lt_template_data` | preparation rule | self | `l10n_lt` |  |  |
| `_get_lt_res_company` | preparation rule | self | `l10n_lt` |  |  |
| `_get_lt_account_account` | preparation rule | self | `l10n_lt` |  |  |
| `_get_lu_template_data` | preparation rule | self | `l10n_lu` |  |  |
| `_get_lu_res_company` | preparation rule | self | `l10n_lu` |  |  |
| `_get_lu_account_journal` | preparation rule | self | `l10n_lu` |  |  |
| `_get_lu_reconcile_model` | preparation rule | self | `l10n_lu` |  |  |
| `_get_lv_template_data` | preparation rule | self | `l10n_lv` |  |  |
| `_get_lv_res_company` | preparation rule | self | `l10n_lv` |  |  |
| `_get_ma_template_data` | preparation rule | self | `l10n_ma` |  |  |
| `_get_ma_res_company` | preparation rule | self | `l10n_ma` |  |  |
| `_get_ma_account_account` | preparation rule | self | `l10n_ma` |  |  |
| `_get_ml_template_data` | preparation rule | self | `l10n_ml` |  |  |
| `_get_ml_res_company` | preparation rule | self | `l10n_ml` |  |  |
| `_get_ml_account_account` | preparation rule | self | `l10n_ml` |  |  |
| `_get_ml_syscebnl_template_data` | preparation rule | self | `l10n_ml` |  |  |
| `_get_ml_syscebnl_res_company` | preparation rule | self | `l10n_ml` |  |  |
| `_get_ml_syscebnl_account_account` | preparation rule | self | `l10n_ml` |  |  |
| `_get_mn_template_data` | preparation rule | self | `l10n_mn` |  |  |
| `_get_mn_res_company` | preparation rule | self | `l10n_mn` |  |  |
| `_get_mr_template_data` | preparation rule | self | `l10n_mr` |  |  |
| `_get_mr_res_company` | preparation rule | self | `l10n_mr` |  |  |
| `_get_mt_template_data` | preparation rule | self | `l10n_mt` |  |  |
| `_get_mt_res_company` | preparation rule | self | `l10n_mt` |  |  |
| `_get_mu_template_data` | preparation rule | self | `l10n_mu_account` |  |  |
| `_get_mu_res_company` | preparation rule | self | `l10n_mu_account` |  |  |
| `_get_mu_account_account` | preparation rule | self | `l10n_mu_account` |  |  |
| `_get_mx_template_data` | preparation rule | self | `l10n_mx` |  |  |
| `_get_mx_res_company` | preparation rule | self | `l10n_mx` |  |  |
| `_get_mx_account_journal` | preparation rule | self | `l10n_mx` |  |  |
| `_get_mx_account_account` | preparation rule | self | `l10n_mx` |  |  |
| `_get_my_template_data` | preparation rule | self | `l10n_my` |  |  |
| `_get_my_res_company` | preparation rule | self | `l10n_my` |  |  |
| `_get_my_account_account` | preparation rule | self | `l10n_my` |  |  |
| `_get_mz_template_data` | preparation rule | self | `l10n_mz` |  |  |
| `_get_mz_res_company` | preparation rule | self | `l10n_mz` |  |  |
| `_get_ne_template_data` | preparation rule | self | `l10n_ne` |  |  |
| `_get_ne_res_company` | preparation rule | self | `l10n_ne` |  |  |
| `_get_ne_account_account` | preparation rule | self | `l10n_ne` |  |  |
| `_get_ne_syscebnl_template_data` | preparation rule | self | `l10n_ne` |  |  |
| `_get_ne_syscebnl_res_company` | preparation rule | self | `l10n_ne` |  |  |
| `_get_ne_syscebnl_account_account` | preparation rule | self | `l10n_ne` |  |  |
| `_get_ng_account_account` | preparation rule | self | `l10n_ng` |  | Nigerian companies are fine with using the generic COA but we need to add Nigeria-specific taxes and a tax report |
| `_get_ng_template_data` | preparation rule | self | `l10n_ng` |  | Copies the generic CoA template data. Changes to it will be reflected here as well. We remove the name and country to use the default values, whereas the generic CoA has to override these. |
| `_get_ng_res_company` | preparation rule | self | `l10n_ng` |  |  |
| `_get_nl_template_data` | preparation rule | self | `l10n_nl` |  |  |
| `_get_nl_res_company` | preparation rule | self | `l10n_nl` |  |  |
| `_get_nl_account_account` | preparation rule | self | `l10n_nl` |  |  |
| `_get_no_template_data` | preparation rule | self | `l10n_no` |  |  |
| `_get_no_res_company` | preparation rule | self | `l10n_no` |  |  |
| `_get_nz_template_data` | preparation rule | self | `l10n_nz` |  |  |
| `_get_nz_res_company` | preparation rule | self | `l10n_nz` |  |  |
| `_get_om_template_data` | preparation rule | self | `l10n_om` |  |  |
| `_get_om_res_company` | preparation rule | self | `l10n_om` |  |  |
| `_get_om_account_account` | preparation rule | self | `l10n_om` |  |  |
| `_get_pa_template_data` | preparation rule | self | `l10n_pa` |  |  |
| `_get_pa_res_company` | preparation rule | self | `l10n_pa` |  |  |
| `_get_pe_template_data` | preparation rule | self | `l10n_pe` |  |  |
| `_get_pe_res_company` | preparation rule | self | `l10n_pe` |  |  |
| `_get_pe_account_account` | preparation rule | self | `l10n_pe` |  |  |
| `_get_ph_template_data` | preparation rule | self | `l10n_ph` |  |  |
| `_get_ph_res_company` | preparation rule | self | `l10n_ph` |  |  |
| `_get_ph_account_journal` | preparation rule | self | `l10n_ph` |  |  |
| `_get_ph_account_account` | preparation rule | self | `l10n_ph` |  |  |
| `_get_pk_template_data` | preparation rule | self | `l10n_pk` |  |  |
| `_get_pk_res_company` | preparation rule | self | `l10n_pk` |  |  |
| `_get_pk_account_account` | preparation rule | self | `l10n_pk` |  |  |
| `_get_pl_template_data` | preparation rule | self | `l10n_pl` |  |  |
| `_get_pl_res_company` | preparation rule | self | `l10n_pl` |  |  |
| `_get_pl_account_account` | preparation rule | self | `l10n_pl` |  |  |
| `_get_pt_template_data` | preparation rule | self | `l10n_pt` |  |  |
| `_get_pt_res_company` | preparation rule | self | `l10n_pt` |  |  |
| `_get_pt_account_account` | preparation rule | self | `l10n_pt` |  |  |
| `_get_qa_template_data` | preparation rule | self | `l10n_qa` |  |  |
| `_get_qa_res_company` | preparation rule | self | `l10n_qa` |  |  |
| `_get_qa_account_account` | preparation rule | self | `l10n_qa` |  |  |
| `_get_ro_template_data` | preparation rule | self | `l10n_ro` |  |  |
| `_get_ro_res_company` | preparation rule | self | `l10n_ro` |  |  |
| `_get_ro_reconcile_model` | preparation rule | self | `l10n_ro` |  |  |
| `_get_ro_account_account` | preparation rule | self | `l10n_ro` |  |  |
| `_get_rs_template_data` | preparation rule | self | `l10n_rs` |  |  |
| `_get_rs_res_company` | preparation rule | self | `l10n_rs` |  |  |
| `_get_rw_template_data` | preparation rule | self | `l10n_rw` |  |  |
| `_get_rw_res_company` | preparation rule | self | `l10n_rw` |  |  |
| `_get_sa_template_data` | preparation rule | self | `l10n_sa` |  |  |
| `_get_sa_res_company` | preparation rule | self | `l10n_sa` |  |  |
| `_get_sa_account_journal` | preparation rule | self | `l10n_sa` |  | If Saudi Arabia chart, we add 3 new journals Tax Adjustments, IFRS 16 and Zakat |
| `_get_sa_account_account` | preparation rule | self | `l10n_sa` |  |  |
| `_get_sa_edi_account_tax` | preparation rule | self | `l10n_sa_edi` |  |  |
| `_get_se_template_data` | preparation rule | self | `l10n_se` |  |  |
| `_get_se_res_company` | preparation rule | self | `l10n_se` |  |  |
| `_get_se_K2_template_data` | preparation rule | self | `l10n_se` |  |  |
| `_get_se_K2_res_company` | preparation rule | self | `l10n_se` |  |  |
| `_get_se_K3_template_data` | preparation rule | self | `l10n_se` |  |  |
| `_get_se_K3_res_company` | preparation rule | self | `l10n_se` |  |  |
| `_get_sg_template_data` | preparation rule | self | `l10n_sg` |  |  |
| `_get_sg_res_company` | preparation rule | self | `l10n_sg` |  |  |
| `_get_sg_account_account` | preparation rule | self | `l10n_sg` |  |  |
| `_get_si_template_data` | preparation rule | self | `l10n_si` |  |  |
| `_get_si_res_company` | preparation rule | self | `l10n_si` |  |  |
| `_get_sk_template_data` | preparation rule | self | `l10n_sk` |  |  |
| `_get_sk_res_company` | preparation rule | self | `l10n_sk` |  |  |
| `_get_sk_account_account` | preparation rule | self | `l10n_sk` |  |  |
| `_get_sn_template_data` | preparation rule | self | `l10n_sn` |  |  |
| `_get_sn_res_company` | preparation rule | self | `l10n_sn` |  |  |
| `_get_sn_account_account` | preparation rule | self | `l10n_sn` |  |  |
| `_get_sn_syscebnl_template_data` | preparation rule | self | `l10n_sn` |  |  |
| `_get_sn_syscebnl_res_company` | preparation rule | self | `l10n_sn` |  |  |
| `_get_sn_syscebnl_account_account` | preparation rule | self | `l10n_sn` |  |  |
| `_get_td_template_data` | preparation rule | self | `l10n_td` |  |  |
| `_get_td_res_company` | preparation rule | self | `l10n_td` |  |  |
| `_get_td_account_account` | preparation rule | self | `l10n_td` |  |  |
| `_get_td_syscebnl_template_data` | preparation rule | self | `l10n_td` |  |  |
| `_get_td_syscebnl_res_company` | preparation rule | self | `l10n_td` |  |  |
| `_get_td_syscebnl_account_account` | preparation rule | self | `l10n_td` |  |  |
| `_get_tg_template_data` | preparation rule | self | `l10n_tg` |  |  |
| `_get_tg_res_company` | preparation rule | self | `l10n_tg` |  |  |
| `_get_tg_account_account` | preparation rule | self | `l10n_tg` |  |  |
| `_get_tg_syscebnl_res_company` | preparation rule | self | `l10n_tg` |  |  |
| `_get_tg_syscebnl_account_account` | preparation rule | self | `l10n_tg` |  |  |
| `_get_th_template_data` | preparation rule | self | `l10n_th` |  |  |
| `_get_th_res_company` | preparation rule | self | `l10n_th` |  |  |
| `_get_tn_template_data` | preparation rule | self | `l10n_tn` |  |  |
| `_get_tn_res_company` | preparation rule | self | `l10n_tn` |  |  |
| `_get_tn_account_account` | preparation rule | self | `l10n_tn` |  |  |
| `_get_tr_template_data` | preparation rule | self | `l10n_tr` |  |  |
| `_get_tr_res_company` | preparation rule | self | `l10n_tr` |  |  |
| `_get_tr_account_journal` | preparation rule | self | `l10n_tr` |  |  |
| `_get_tr_account_account` | preparation rule | self | `l10n_tr` |  |  |
| `_get_tr_withholding_account_tax` | preparation rule | self | `l10n_tr_nilvera_einvoice_extended` |  |  |
| `_get_tr_withholding_account_tax_code` | preparation rule | self | `l10n_tr_nilvera_einvoice_extended` |  |  |
| `_get_tw_template_data` | preparation rule | self | `l10n_tw` |  |  |
| `_get_tw_res_company` | preparation rule | self | `l10n_tw` |  |  |
| `_get_tw_account_journal` | preparation rule | self | `l10n_tw` |  |  |
| `_get_tw_account_account` | preparation rule | self | `l10n_tw` |  |  |
| `_get_tz_template_data` | preparation rule | self | `l10n_tz_account` |  |  |
| `_get_tz_res_company` | preparation rule | self | `l10n_tz_account` |  |  |
| `_get_tz_account_account` | preparation rule | self | `l10n_tz_account` |  |  |
| `_get_ua_psbo_template_data` | preparation rule | self | `l10n_ua` |  |  |
| `_get_ua_psbo_res_company` | preparation rule | self | `l10n_ua` |  |  |
| `_get_ua_psbo_account_account` | preparation rule | self | `l10n_ua` |  |  |
| `_get_ug_template_data` | preparation rule | self | `l10n_ug` |  |  |
| `_get_ug_res_company` | preparation rule | self | `l10n_ug` |  |  |
| `_get_ug_account_account` | preparation rule | self | `l10n_ug` |  |  |
| `_get_uk_template_data` | preparation rule | self | `l10n_uk` |  |  |
| `_get_uk_res_company` | preparation rule | self | `l10n_uk` |  |  |
| `_get_uk_account_account` | preparation rule | self | `l10n_uk` |  |  |
| `_get_us_template_data` | preparation rule | self | `l10n_us_account` |  |  |
| `_get_us_res_company` | preparation rule | self | `l10n_us_account` |  |  |
| `_get_us_account_account` | preparation rule | self | `l10n_us_account` |  |  |
| `_get_uy_template_data` | preparation rule | self | `l10n_uy` |  |  |
| `_get_uy_res_company` | preparation rule | self | `l10n_uy` |  |  |
| `_get_uy_account_journal` | preparation rule | self | `l10n_uy` |  |  |
| `_get_uy_account_account` | preparation rule | self | `l10n_uy` |  |  |
| `_get_uz_template_data` | preparation rule | self | `l10n_uz` |  |  |
| `_get_uz_res_company` | preparation rule | self | `l10n_uz` |  |  |
| `_get_ve_template_data` | preparation rule | self | `l10n_ve` |  |  |
| `_get_ve_res_company` | preparation rule | self | `l10n_ve` |  |  |
| `_get_ve_account_account` | preparation rule | self | `l10n_ve` |  |  |
| `_get_vn_template_data` | preparation rule | self | `l10n_vn` |  |  |
| `_get_vn_res_company` | preparation rule | self | `l10n_vn` |  |  |
| `_get_vn_account_account` | preparation rule | self | `l10n_vn` |  |  |
| `_get_vn_account_journal` | preparation rule | self | `l10n_vn` |  |  |
| `_get_za_template_data` | preparation rule | self | `l10n_za` |  |  |
| `_get_za_res_company` | preparation rule | self | `l10n_za` |  |  |
| `_get_zm_template_data` | preparation rule | self | `l10n_zm_account` |  |  |
| `_get_zm_res_company` | preparation rule | self | `l10n_zm_account` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `try_loading` | UserError | The %s chart template shouldn't be selected directly. Instead, you should directly select the chart template related to your country. | `account` |
| `_load` | AccessError | Only administrators can install chart templates | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.chart.template.json`.
