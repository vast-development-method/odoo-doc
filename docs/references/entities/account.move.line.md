# Journal Item (`account.move.line`)

**Transport name:** `account.move.line`  
**Storage name:** `account_move_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_fleet`, `sale`, `stock_account`, `sale_stock`, `hr_expense`, `point_of_sale`, `l10n_gcc_invoice`, `l10n_latam_invoice_document`, `l10n_ar`, `l10n_latam_check`, `pos_sale`, `l10n_cl`, `purchase`, `repair`, `l10n_es_edi_tbai`, `l10n_gr_edi`, `l10n_hr_edi`, `l10n_id_efaktur_coretax`, `l10n_in`, `l10n_in_edi`, `l10n_in_pos`, `purchase_stock`, `l10n_mx`, `l10n_my_edi`, `l10n_sa_edi`, `l10n_tr`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw_edi_ecpay`, `mrp_account`, `stock_landed_costs`, `mrp_subcontracting_purchase`, `pos_discount`, `pos_loyalty`, `sale_mrp`, `sale_project`, `sale_expense`, `project_sale_expense`, `sale_expense_margin`, `sale_loyalty`, `sale_timesheet`

Description: Journal Item

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`
- Default ordering: `date desc, move_name desc, id`
- Display name search fields: `["name", "move_id", "product_id"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (127)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Journal Entry | many to one | `account.move` | required; read only; indexed; on delete of the target: cascade; must belong to the same company |
| `journal_id` | Journal | many to one |  | related through path `move_id.journal_id` and stored; indexed; not copied on duplication; precomputed before insertion |
| `journal_group_id` | Ledger | many to one | `account.journal.group` | searchable through a search rule |
| `company_id` | Company | many to one |  | read only; related through path `move_id.company_id` and stored; indexed; precomputed before insertion |
| `company_currency_id` | Company Currency | many to one |  | read only; related through path `move_id.company_currency_id` and stored; precomputed before insertion |
| `move_name` | Number | single line text |  | related through path `move_id.name` and stored; indexed (btree) |
| `parent_state` | Parent State | selection |  | related through path `move_id.state` and stored |
| `date` | Date | date |  | related through path `move_id.date` and stored; not copied on duplication; aggregated with min |
| `invoice_date` | Invoice Date | date |  | related through path `move_id.invoice_date` and stored; not copied on duplication; aggregated with min |
| `ref` | Ref | single line text |  | related through path `move_id.ref` and stored; indexed (trigram); not copied on duplication |
| `is_storno` | Company Storno Accounting | boolean |  | computed by rule `_compute_is_storno` and stored; precomputed before insertion; Help: Utility field to express whether the journal item is subject to storno accounting |
| `sequence` | Sequence | integer |  | computed by rule `_compute_sequence` and stored; precomputed before insertion |
| `move_type` | Move Type | selection |  | related through path `move_id.move_type` |
| `account_id` | Account | many to one | `account.account` | computed by rule `_compute_account_id` and stored; writable through an inverse rule; changes are tracked in the message thread; on delete of the target: restrict; restricted by domain `[('account_type', '!=', 'off_balance')]`; must belong to the same company; precomputed before insertion |
| `account_name` | Account Name | single line text |  | related through path `account_id.name` |
| `account_code` | Account Code | single line text |  | related through path `account_id.code` |
| `search_account_id` | Search Account | many to one | `account.account` | searchable through a search rule |
| `name` | Label | single line text |  | computed by rule `_compute_name` and stored; changes are tracked in the message thread; precomputed before insertion |
| `translated_product_name` | Translated Product Name | multi line text |  | computed by rule `_compute_translated_product_name` (not stored) |
| `debit` | Debit | monetary |  | computed by rule `_compute_debit_credit` and stored; writable through an inverse rule; currency taken from `company_currency_id`; precomputed before insertion |
| `credit` | Credit | monetary |  | computed by rule `_compute_debit_credit` and stored; writable through an inverse rule; currency taken from `company_currency_id`; precomputed before insertion |
| `balance` | Balance | monetary |  | computed by rule `_compute_balance` and stored; changes are tracked in the message thread; currency taken from `company_currency_id`; precomputed before insertion |
| `cumulated_balance` | Cumulated Balance | monetary |  | computed by rule `_compute_cumulated_balance` (not stored); currency taken from `company_currency_id`; Help: Cumulated balance depending on the domain and the order chosen in the view. |
| `currency_rate` | Currency Rate | float |  | computed by rule `_compute_currency_rate` (not stored); Help: Currency rate from company currency to document currency. |
| `amount_currency` | Amount in Currency | monetary |  | computed by rule `_compute_amount_currency` and stored; writable through an inverse rule; precomputed before insertion; Help: The amount expressed in an optional other currency if it is a multi-currency entry. |
| `currency_id` | Currency | many to one | `res.currency` | required; computed by rule `_compute_currency_id` and stored; precomputed before insertion |
| `is_same_currency` | Is Same Currency | boolean |  | computed by rule `_compute_same_currency` (not stored) |
| `partner_id` | Partner | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; writable through an inverse rule; on delete of the target: restrict; precomputed before insertion |
| `is_imported` | Is Imported | boolean |  |  |
| `reconcile_model_id` | Reconciliation Model | many to one | `account.reconcile.model` | read only; not copied on duplication; must belong to the same company |
| `payment_id` | Originator Payment | many to one | `account.payment` | related through path `move_id.origin_payment_id` and stored; indexed (btree_not_null); Help: The payment that created this entry |
| `statement_line_id` | Originator Statement Line | many to one | `account.bank.statement.line` | related through path `move_id.statement_line_id` and stored; indexed (btree_not_null); Help: The statement line that created this entry |
| `statement_id` | Statement | many to one |  | related through path `statement_line_id.statement_id` and stored; indexed (btree_not_null); not copied on duplication; Help: The bank statement used for bank reconciliation |
| `commercial_partner_country` | Commercial Partner Country | many to one |  | related through path `move_id.commercial_partner_id.country_id` |
| `tax_ids` | Taxes | many to many | `account.tax` | computed by rule `_compute_tax_ids` and stored; changes are tracked in the message thread; must belong to the same company; association table `account_move_line_account_tax_rel`; precomputed before insertion |
| `group_tax_id` | Originator Group of Taxes | many to one | `account.tax` | indexed (btree_not_null); must belong to the same company |
| `tax_line_id` | Originator Tax | many to one | `account.tax` | related through path `tax_repartition_line_id.tax_id` and stored; on delete of the target: restrict; precomputed before insertion; Help: Indicates that this journal item is a tax line |
| `tax_group_id` | Originator tax group | many to one |  | related through path `tax_line_id.tax_group_id` and stored; precomputed before insertion |
| `tax_base_amount` | Base Amount | monetary |  | read only; currency taken from `company_currency_id` |
| `tax_repartition_line_id` | Originator Tax Distribution Line | many to one | `account.tax.repartition.line` | read only; on delete of the target: restrict; must belong to the same company; Help: Tax distribution line that caused the creation of this move line, if any |
| `tax_tag_ids` | Tags | many to many | `account.account.tag` | changes are tracked in the message thread; on delete of the target: restrict; Help: Tags assigned to this line by the tax creating it, if any. It determines its impact on financial reports. |
| `extra_tax_data` | Extra Tax Data | structured document |  |  |
| `amount_residual` | Residual Amount | monetary |  | computed by rule `_compute_amount_residual` and stored; currency taken from `company_currency_id`; Help: The residual amount on a journal item expressed in the company currency. |
| `amount_residual_currency` | Residual Amount in Currency | monetary |  | computed by rule `_compute_amount_residual` and stored; Help: The residual amount on a journal item expressed in its currency (possibly not the company currency). |
| `reconciled` | Reconciled | boolean |  | computed by rule `_compute_amount_residual` and stored |
| `full_reconcile_id` | Matching | many to one | `account.full.reconcile` | read only; indexed (btree_not_null); not copied on duplication |
| `matched_debit_ids` | Matched Debits | one to many | `account.partial.reconcile` | read only; inverse field `credit_move_id`; Help: Debit journal items that are matched with this journal item. |
| `matched_credit_ids` | Matched Credits | one to many | `account.partial.reconcile` | read only; inverse field `debit_move_id`; Help: Credit journal items that are matched with this journal item. |
| `reconciled_lines_ids` | Reconciled Lines | many to many | `account.move.line` | computed by rule `_compute_reconciled_lines_ids` (not stored); writable through an inverse rule |
| `reconciled_lines_excluding_exchange_diff_ids` | Reconciled Lines Excluding Exchange Diff | many to many | `account.move.line` | computed by rule `_compute_reconciled_lines_excluding_exchange_diff_ids` (not stored) |
| `first_reconciled_lines_id` | First Reconciled Lines | many to one | `account.move.line` | computed by rule `_compute_reconciled_lines_ids` (not stored) |
| `count_reconciled_lines` | Count Reconciled Lines | integer |  | computed by rule `_compute_reconciled_lines_ids` (not stored) |
| `first_reconciled_lines_excluding_exchange_diff_id` | First Reconciled Lines Excluding Exchange Diff | many to one | `account.move.line` | computed by rule `_compute_reconciled_lines_excluding_exchange_diff_ids` (not stored) |
| `count_reconciled_lines_excluding_exchange_diff` | Count Reconciled Lines Excluding Exchange Diff | integer |  | computed by rule `_compute_reconciled_lines_excluding_exchange_diff_ids` (not stored) |
| `exchange_move_ids` | Exchange Move | many to many | `account.move` | computed by rule `_compute_exchange_move` (not stored) |
| `matching_number` | Matching # | single line text |  | indexed (btree); not copied on duplication; Help: Matching number for this line, 'P' if it is only partially reconcile, or the name of the full reconcile if it exists. |
| `is_account_reconcile` | Account Reconcile | boolean |  | related through path `account_id.reconcile` |
| `account_type` | Internal Type | selection |  | related through path `account_id.account_type` |
| `account_internal_group` | Account Internal Group | selection |  | related through path `account_id.internal_group` |
| `account_root_id` | Account Root | many to one |  | related through path `account_id.root_id` |
| `product_category_id` | Product Category | many to one |  | related through path `product_id.product_tmpl_id.categ_id` |
| `display_type` | Display Type | selection |  | required; computed by rule `_compute_display_type` and stored; precomputed before insertion |
| `collapse_composition` | Hide Composition | boolean |  | Help: If checked, the lines below this section will not be displayed in reports and portal. |
| `collapse_prices` | Hide Prices | boolean |  | Help: If checked, the prices of the lines below this section will not be displayed in reports and portal. |
| `parent_id` | Parent Section Line | many to one | `account.move.line` | computed by rule `_compute_parent_id` (not stored) |
| `product_id` | Product | many to one | `product.product` | writable through an inverse rule; indexed; on delete of the target: restrict; must belong to the same company |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom_id` | Unit | many to one | `uom.uom` | computed by rule `_compute_product_uom_id` and stored; on delete of the target: restrict; restricted by domain `[('id', 'in', allowed_uom_ids)]`; precomputed before insertion |
| `quantity` | Quantity | float |  | computed by rule `_compute_quantity` and stored; precision `Product Unit`; precomputed before insertion; Help: The optional quantity expressed by this line, eg: number of product sold. The quantity is not a legal requirement but is very useful for some reports. |
| `date_maturity` | Due Date | date |  | changes are tracked in the message thread; indexed; Help: This field is used for payable and receivable journal entries. You can put the limit date for the payment of this line. |
| `price_unit` | Unit Price | float |  | computed by rule `_compute_price_unit` and stored; precomputed before insertion |
| `price_subtotal` | Subtotal | monetary |  | computed by rule `_compute_totals` and stored; currency taken from `currency_id` |
| `price_total` | Total | monetary |  | computed by rule `_compute_totals` and stored; currency taken from `currency_id` |
| `discount` | Discount (%) | float |  | default ; precision `Discount` |
| `tax_calculation_rounding_method` | Tax calculation rounding method | selection |  | read only; related through path `company_id.tax_calculation_rounding_method` |
| `deductible_amount` | Deductibility | float |  | default `100` |
| `term_key` | Term Key | binary |  | computed by rule `_compute_term_key` (not stored) |
| `epd_key` | Epd Key | binary |  | computed by rule `_compute_epd_key` (not stored) |
| `epd_needed` | Epd Needed | binary |  | computed by rule `_compute_epd_needed` (not stored) |
| `epd_dirty` | Epd Dirty | boolean |  | computed by rule `_compute_epd_needed` (not stored) |
| `discount_allocation_key` | Discount Allocation Key | binary |  | computed by rule `_compute_discount_allocation_key` (not stored) |
| `discount_allocation_needed` | Discount Allocation Needed | binary |  | computed by rule `_compute_discount_allocation_needed` (not stored) |
| `discount_allocation_dirty` | Discount Allocation Dirty | boolean |  | computed by rule `_compute_discount_allocation_needed` (not stored) |
| `analytic_line_ids` | Analytic lines | one to many | `account.analytic.line` | inverse field `move_line_id` |
| `analytic_distribution` | Analytic Distribution | structured document |  | writable through an inverse rule |
| `has_invalid_analytics` | Has Invalid Analytics | boolean |  | computed by rule `_compute_has_invalid_analytics` (not stored) |
| `discount_date` | Discount Date | date |  | read only; Help: Last date at which the discounted amount must be paid in order for the Early Payment Discount to be granted |
| `discount_amount_currency` | Discount amount in Currency | monetary |  | currency taken from `currency_id` |
| `discount_balance` | Discount Balance | monetary |  | currency taken from `company_currency_id` |
| `payment_date` | Next Payment Date | date |  | computed by rule `_compute_payment_date` (not stored); searchable through a search rule |
| `is_refund` | Is Refund | boolean |  | computed by rule `_compute_is_refund` (not stored) |
| `no_followup` | No Follow-Up | boolean |  | computed by rule `_compute_no_followup` and stored; writable through an inverse rule; Help: Exclude this journal item from follow-up reports. |
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` | indexed (btree_not_null) |
| `need_vehicle` | Need Vehicle | boolean |  | computed by rule `_compute_need_vehicle` (not stored) |
| `vehicle_log_service_ids` | Vehicle Log Service | one to many | `fleet.vehicle.log.services` | inverse field `account_move_line_id` |
| `is_downpayment` | Is Downpayment | boolean |  | extended by packages `purchase` |
| `sale_line_ids` | Sales Order Lines | many to many | `sale.order.line` | read only; not copied on duplication; association table `sale_order_line_invoice_rel` |
| `sale_line_warn_msg` | Sale Line Warn Msg | multi line text |  | computed by rule `_compute_sale_line_warn_msg` (not stored) |
| `cogs_origin_id` | Cost of goods sold Origin | many to one | `account.move.line` | indexed (btree_not_null); not copied on duplication |
| `expense_id` | Expense | many to one | `hr.expense` | indexed (btree_not_null) |
| `l10n_gcc_invoice_tax_amount` | Tax Amount | float |  | computed by rule `_compute_tax_amount` (not stored) |
| `l10n_gcc_line_name` | Localization Gcc Line Name | single line text |  | computed by rule `_compute_l10n_gcc_line_name` (not stored) |
| `l10n_latam_document_type_id` | Localization Latam Document Type | many to one |  | related through path `move_id.l10n_latam_document_type_id` and stored; indexed (btree_not_null) |
| `l10n_latam_use_documents` | Localization Latam Use Documents | boolean |  | related through path `move_id.l10n_latam_use_documents` |
| `l10n_latam_check_ids` | Checks | one to many | `l10n_latam.check` | inverse field `outstanding_line_id` |
| `purchase_line_id` | Purchase Order Line | many to one | `purchase.order.line` | indexed (btree_not_null); not copied on duplication; on delete of the target: set null |
| `purchase_order_id` | Purchase Order | many to one | `purchase.order` | read only; related through path `purchase_line_id.order_id` |
| `purchase_line_warn_msg` | Purchase Line Warn Msg | multi line text |  | computed by rule `_compute_purchase_line_warn_msg` (not stored) |
| `l10n_gr_edi_available_cls_category` | Localization Gr Electronic data interchange Available Cls Category | single line text |  | computed by rule `_compute_l10n_gr_edi_available_cls_category` (not stored) |
| `l10n_gr_edi_available_cls_type` | Localization Gr Electronic data interchange Available Cls Type | single line text |  | computed by rule `_compute_l10n_gr_edi_available_cls_type` (not stored) |
| `l10n_gr_edi_available_cls_vat` | Localization Gr Electronic data interchange Available Cls Value-added tax | single line text |  | computed by rule `_compute_l10n_gr_edi_available_cls_type` (not stored) |
| `l10n_gr_edi_need_exemption_category` | Localization Gr Electronic data interchange Need Exemption Category | boolean |  | computed by rule `_compute_l10n_gr_edi_need_exemption_category` (not stored); default  |
| `l10n_gr_edi_detail_type` | Detail Type | selection |  | computed by rule `_compute_l10n_gr_edi_detail_type` and stored |
| `l10n_gr_edi_cls_category` | myDATA Category | selection |  | computed by rule `_compute_l10n_gr_edi_cls_category` and stored |
| `l10n_gr_edi_cls_type` | myDATA Type | selection |  | computed by rule `_compute_l10n_gr_edi_cls_type` and stored |
| `l10n_gr_edi_cls_vat` | value-added tax Classification | selection |  | computed by rule `_compute_l10n_gr_edi_cls_vat` and stored |
| `l10n_gr_edi_tax_exemption_category` | Tax Exemption Category | selection |  | computed by rule `_compute_l10n_gr_edi_tax_exemption_category` and stored |
| `l10n_hr_kpd_category_id` | KPD category | many to one | `l10n_hr.kpd.category` | computed by rule `_compute_l10n_hr_product_id_kpd` and stored |
| `l10n_in_hsn_code` | harmonized system nomenclature/SAC Code | single line text |  | computed by rule `_compute_l10n_in_hsn_code` and stored; not copied on duplication |
| `l10n_in_gstr_section` | GSTR Section | selection |  | indexed (btree_not_null) |
| `l10n_in_withhold_tax_amount` | tax deducted at source Tax Amount | monetary |  | computed by rule `_compute_l10n_in_withhold_tax_amount` (not stored) |
| `l10n_in_tds_tcs_section_id` | Localization In Tax deducted at source Tax collected at source Section | many to one |  | related through path `account_id.l10n_in_tds_tcs_section_id` |
| `l10n_my_edi_classification_code` | Malaysian classification code | selection |  | computed by rule `_compute_l10n_my_edi_classification_code` and stored; not copied on duplication |
| `l10n_tr_ctsp_number` | CTSP Number | single line text |  | computed by rule `_compute_l10n_tr_ctsp_number` and stored |
| `l10n_tw_edi_ecpay_item_sequence` | Item Sequence | integer |  | read only |
| `product_type` | Product Type | selection |  | read only; related through path `product_id.type` |
| `is_landed_costs_line` | Is Landed Costs Line | boolean |  |  |

## Selection values

### `display_type` (Display Type)

| Value | Label |
|---|---|
| `product` | Product |
| `cogs` | Cost of Goods Sold |
| `tax` | Tax |
| `discount` | Discount |
| `rounding` | Rounding |
| `payment_term` | Payment Term |
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |
| `epd` | Early Payment Discount |
| `non_deductible_product_total` | Non Deductible Products Total |
| `non_deductible_product` | Non Deductible Products |
| `non_deductible_tax` | Non Deductible Tax |

### `l10n_gr_edi_detail_type` (Detail Type)

| Value | Label |
|---|---|
| `1` | 1 |
| `2` | 2 |

### `l10n_in_gstr_section` (GSTR Section)

| Value | Label |
|---|---|
| `sale_b2b_rcm` | B2B RCM |
| `sale_b2b_regular` | B2B Regular |
| `sale_b2cl` | B2CL |
| `sale_b2cs` | B2CS |
| `sale_exp_wp` | EXP(WP) |
| `sale_exp_wop` | EXP(WOP) |
| `sale_sez_wp` | SEZ(WP) |
| `sale_sez_wop` | SEZ(WOP) |
| `sale_deemed_export` | Deemed Export |
| `sale_cdnr_rcm` | CDNR RCM |
| `sale_cdnr_regular` | CDNR Regular |
| `sale_cdnr_deemed_export` | CDNR(Deemed Export) |
| `sale_cdnr_sez_wp` | CDNR(SEZ-WP) |
| `sale_cdnr_sez_wop` | CDNR(SEZ-WOP) |
| `sale_cdnur_b2cl` | CDNUR(B2CL) |
| `sale_cdnur_exp_wp` | CDNUR(EXP-WP) |
| `sale_cdnur_exp_wop` | CDNUR(EXP-WOP) |
| `sale_nil_rated` | Nil Rated |
| `sale_exempt` | Exempt |
| `sale_non_gst_supplies` | Non-GST Supplies |
| `sale_eco_9_5` | ECO 9(5) |
| `sale_out_of_scope` | Out of Scope |
| `purchase_b2b_regular` | B2B Regular |
| `purchase_b2c_regular` | B2C Regular |
| `purchase_b2b_rcm` | B2B RCM |
| `purchase_b2c_rcm` | B2C RCM |
| `purchase_imp_services` | IMP(services-RCM) |
| `purchase_imp_goods` | IMP(goods) |
| `purchase_cdnr_regular` | CDNR Regular |
| `purchase_cdnur_regular` | CDNUR Regular |
| `purchase_cdnr_rcm` | CDNR RCM |
| `purchase_cdnur_rcm` | CDNUR RCM |
| `purchase_nil_rated` | Nil Rated |
| `purchase_exempt` | Exempt |
| `purchase_non_gst_supplies` | Non-GST Supplies |
| `purchase_composition_supplies` | Composition Supplies |
| `purchase_out_of_scope` | Out of Scope |

## State fields

State machine fields of this entity: `parent_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (9)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_credit_debit` | Constraint | `CHECK(display_type IN ('line_section', 'line_subsection', 'line_note') OR credit * debit=0)` | Wrong credit or debit value in accounting entry! | `account` |
| `_check_amount_currency_balance_sign` | Constraint | `CHECK(                 display_type IN ('line_section', 'line_subsection', 'line_note')                 OR (                     (balance <= 0 AND amount_currency <= 0)                     OR                     (balance >= 0 AND amount_currency >= 0)                 )             )` | The amount expressed in the secondary currency must be positive when account is debited and negative when account is credited. If the currency is the same as the one from the company, this amount must strictly be equal to the balance. | `account` |
| `_check_accountable_required_fields` | Constraint | `CHECK(display_type IN ('line_section', 'line_subsection', 'line_note') OR account_id IS NOT NULL)` | Missing required account on accountable line. | `account` |
| `_check_non_accountable_fields_null` | Constraint | `CHECK(display_type NOT IN ('line_section', 'line_subsection', 'line_note') OR (amount_currency = 0 AND debit = 0 AND credit = 0 AND account_id IS NULL))` | Forbidden balance or account on non-accountable line | `account` |
| `_partner_id_ref_idx` | Index | `(partner_id, ref)` |  | `account` |
| `_date_name_id_idx` | Index | `(date desc, move_name desc, id)` |  | `account` |
| `_unreconciled_index` | Index | `(account_id, partner_id) WHERE reconciled IS NOT TRUE` |  | `account` |
| `_journal_id_neg_amnt_residual_idx` | Index | `(journal_id) WHERE amount_residual < 0` |  | `account` |
| `_account_id_date_idx` | Index | `(account_id, date)` |  | `account` |

## Operations (209)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_query_tax_details_from_domain` | preparation rule | self, domain, fallback | `account` | model | Create the tax details sub-query based on the orm domain passed as parameter.  :param domain:      An orm domain on account.move.line. :param fallback:    Fallback on an approximated mapping if the mapping failed. :return:            query as SQL object |
| `_get_extra_query_base_tax_line_mapping` | preparation rule | self | `account`, `hr_expense` | model |  |
| `_get_query_tax_details` | preparation rule | self, table_references, search_condition, fallback | `account` | model | Create the tax details sub-query based on the orm domain passed as parameter.  :param table_references:    The query to inject after the FROM, as an SQL object. :param search_condition:    The query to inject in the WHERE clause, as an SQL object. :param fallback:            Fallback on an approximated mapping if the mapping failed. :return:                    query as an SQL object |
| `get_views` | operation | self, views, options | `account` | model |  |
| `_compute_display_type` | computation | self | `account` | depends: `move_id` |  |
| `_compute_partner_id` | computation | self | `account`, `hr_expense` |  |  |
| `_compute_currency_id` | computation | self | `account` | depends: `move_id.currency_id` |  |
| `_compute_name` | computation | self | `account`, `point_of_sale` | depends: `product_id`, `move_id.ref`, `move_id.payment_reference` |  |
| `_compute_translated_product_name` | computation | self | `account` | depends: `product_id` |  |
| `_compute_account_id` | computation | self | `account`, `l10n_mx`, `l10n_tr`, `stock_account` | depends: `move_id.country_code`, `move_id.move_type`, `display_type` |  |
| `_search_account_id` | search rule | self, operator, value | `account` | model | Search method that can be a drop-in replacement for searching on `account_id`. Resolves the domain and inlines the resulting ids to yield better final queries and avoids joining on the `account.account` model. This should be a net positive on average, under the assertion that the cardinality of `account.account` doesn't grow too large. (e.g. <10k rows) |
| `_compute_is_storno` | computation | self | `account`, `sale` | depends: `move_id.is_storno`, `price_unit`, `quantity`; depends: `balance` |  |
| `_compute_balance` | computation | self | `account` | depends: `move_id` |  |
| `_compute_debit_credit` | computation | self | `account` | depends: `balance` |  |
| `_compute_currency_rate` | computation | self | `account` | depends: `currency_id`, `company_id`, `move_id.invoice_currency_rate`, `move_id.date` |  |
| `_compute_same_currency` | computation | self | `account` | depends: `currency_id`, `company_currency_id` |  |
| `_compute_amount_currency` | computation | self | `account` | depends: `currency_rate`, `balance` |  |
| `_compute_cumulated_balance` | computation | self | `account` | depends_context: `order_cumulated_balance`, `domain_cumulated_balance` |  |
| `_compute_amount_residual` | computation | self | `account` | depends: `debit`, `credit`, `amount_currency`, `account_id`, `currency_id`, `company_id`, `matched_debit_ids`, `matched_credit_ids` | Computes the residual amount of a move line from a reconcilable account in the company currency and the line's currency. This amount will be 0 for fully reconciled lines or lines from a non-reconcilable account, the original line amount for unreconciled lines, and something in-between for partially reconciled lines. |
| `_compute_allowed_uom_ids` | computation | self | `account` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids` |  |
| `_compute_product_uom_id` | computation | self | `account` | depends: `product_id` |  |
| `_compute_quantity` | computation | self | `account` | depends: `display_type` |  |
| `_compute_sequence` | computation | self | `account` | depends: `display_type` |  |
| `_compute_totals` | computation | self | `account`, `hr_expense` | depends: `quantity`, `discount`, `price_unit`, `tax_ids`, `currency_id`, `amount_currency` | Compute 'price_subtotal' / 'price_total' outside of `_sync_tax_lines` because those values must be visible for the user on the UI with draft moves and the dynamic lines are synchronized only when saving the record. |
| `_compute_price_unit` | computation | self | `account` | depends: `product_id`, `product_uom_id` |  |
| `_compute_tax_ids` | computation | self | `account` | depends: `product_id`, `product_uom_id` |  |
| `_get_computed_taxes` | preparation rule | self | `account` |  |  |
| `_compute_discount_allocation_key` | computation | self | `account` | depends: `account_id`, `company_id` |  |
| `_compute_discount_allocation_needed` | computation | self | `account` | depends: `account_id`, `company_id`, `price_unit`, `quantity`, `currency_rate`, `move_id.line_ids.discount`, `move_id.line_ids.analytic_distribution` |  |
| `_compute_epd_key` | computation | self | `account` | depends: `tax_ids`, `account_id`, `company_id` |  |
| `_compute_epd_needed` | computation | self | `account` | depends: `move_id.needed_terms`, `account_id`, `analytic_distribution`, `tax_ids`, `tax_tag_ids`, `company_id`, `price_subtotal` |  |
| `_compute_is_refund` | computation | self | `account` | depends: `move_id.move_type`, `balance`, `tax_repartition_line_id`, `tax_ids` |  |
| `_compute_term_key` | computation | self | `account` | depends: `date_maturity` |  |
| `_compute_analytic_distribution` | computation | self | `account`, `sale_project` | depends: `account_id`, `partner_id`, `product_id` |  |
| `_get_analytic_distribution_arguments` | preparation rule | self, root_plans | `account` |  | Get arguments to determine analytic distribution. This function aims to be overridden by partner submodules :param root_plans: account.analytic.plan recordset :return: dict |
| `_compute_payment_date` | computation | self | `account` | depends: `discount_date`, `date_maturity` |  |
| `_compute_exchange_move` | computation | self | `account` | depends: `matched_debit_ids`, `matched_credit_ids` |  |
| `_compute_sql_payment_date` | computation | self, table | `account` |  |  |
| `_compute_reconciled_lines_ids` | computation | self | `account` | depends: `matched_debit_ids`, `matched_credit_ids` |  |
| `_compute_reconciled_lines_excluding_exchange_diff_ids` | computation | self | `account` | depends: `reconciled_lines_ids`, `matched_debit_ids`, `matched_credit_ids` |  |
| `_compute_parent_id` | computation | self | `account` |  |  |
| `_compute_no_followup` | computation | self | `account` | depends: `journal_id.type` |  |
| `_inverse_no_followup` | inverse computation | self | `account` |  |  |
| `_search_payment_date` | search rule | self, operator, value | `account` |  |  |
| `action_payment_items_register_payment` | user action | self | `account` |  |  |
| `action_register_payment` | user action | self, ctx | `account` |  | Open the account.payment.register wizard to pay the selected journal items. :return: An action opening the account.payment.register wizard. |
| `_search_journal_group_id` | search rule | self, operator, value | `account` |  |  |
| `_inverse_partner_id` | on change | self | `account` | onchange: `partner_id` |  |
| `_inverse_product_id` | on change | self | `account`, `stock_account` | onchange: `product_id` |  |
| `_inverse_amount_currency` | on change | self | `account` | onchange: `amount_currency`, `currency_id` |  |
| `_inverse_debit` | on change | self | `account` | onchange: `debit` |  |
| `_inverse_credit` | on change | self | `account` | onchange: `credit` |  |
| `_inverse_analytic_distribution` | inverse computation | self | `account` |  | Unlink and recreate analytic_lines when modifying the distribution. |
| `_inverse_account_id` | on change | self | `account` | onchange: `account_id` |  |
| `_inverse_reconciled_lines_ids` | inverse computation | self | `account` |  |  |
| `_check_constrains_account_id_journal_id` | validation | self | `account` |  |  |
| `_check_off_balance` | validation | self | `account` | constrains: `account_id`, `tax_ids`, `tax_line_id`, `reconciled` |  |
| `_check_payable_receivable` | validation | self | `account`, `hr_expense` | constrains: `account_id`, `display_type` |  |
| `_affect_tax_report` | internal rule | self | `account` |  |  |
| `_check_tax_lock_date` | validation | self | `account` |  |  |
| `_check_reconciliation` | validation | self | `account` |  |  |
| `_check_caba_non_caba_shared_tags` | validation | self | `account` | constrains: `tax_ids`, `tax_repartition_line_id` | When mixing cash basis and non cash basis taxes, it is important that those taxes don't share tags on the repartition creating a single account.move.line.  Shared tags in this context cannot work, as the tags would need to be present on both the invoice and cash basis move, leading to the same base amount to be taken into account twice; which is wrong.This is why we don't support that. A workaround may be provided by the use of a group of taxes, whose children are type_tax_use=None, and only one of them uses the common tag.  Note that taxes of the same exigibility are allowed to share tags. |
| `_constrains_matching_number` | validation | self | `account` | constrains: `matching_number`, `matched_debit_ids`, `matched_credit_ids`, `full_reconcile_id` |  |
| `_constrains_deductible_amount` | validation | self | `account` | constrains: `deductible_amount` |  |
| `check_field_access_rights` | operation | self, operation, field_names | `account` | model |  |
| `_get_default_read_fields` | preparation rule | self | `account` | model |  |
| `read` | lifecycle override | self, fields, load | `account` |  |  |
| `search_read` | lifecycle override | self, domain, fields, offset, limit, order, **read_kwargs | `account` | model |  |
| `invalidate_model` | operation | self, fnames, flush | `account` |  |  |
| `invalidate_recordset` | operation | self, fnames, flush | `account` |  |  |
| `search_fetch` | operation | self, domain, field_names, offset, limit, order | `account` | model |  |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_sanitize_vals` | internal rule | self, vals | `account` |  |  |
| `_prepare_create_values` | preparation rule | self, vals_list | `account` |  |  |
| `_sync_invoice` | internal rule | self, container | `account` |  |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account_fleet`, `account` |  |  |
| `_parse_flush_fnames` | internal rule | self, fnames | `account` |  |  |
| `flush_recordset` | operation | self, fnames | `account` |  |  |
| `flush_model` | operation | self, fnames | `account` |  |  |
| `_valid_field_parameter` | lifecycle override | self, field, name | `account` |  |  |
| `_unlink_except_posted` | internal rule | self | `account` | ondelete |  |
| `_prevent_automatic_line_deletion` | internal rule | self | `account` | ondelete |  |
| `_except_hashed_entry_lines` | internal rule | self | `account` | ondelete | Lines belonginig to a hashed (locked) entry should not be allowed to be deleted in order to protect the hash chain. |
| `unlink` | lifecycle override | self | `account_fleet`, `account`, `sale_timesheet` |  |  |
| `_format_aml_name` | internal rule | self, line_name, move_ref, move_name | `account` | model | Format the display of an account.move.line record. As its very costly to fetch the account.move.line records, only line_name, move_ref, move_name are passed as parameters to deal with sql-queries more easily.  :param line_name:   The name of the account.move.line record. :param move_ref:    The reference of the account.move record. :param move_name:   The name of the account.move record. :return:            The formatted name of the account.move.line record. |
| `_compute_display_name` | computation | self | `account` | depends: `move_id`, `ref`, `product_id` |  |
| `_compute_has_invalid_analytics` | computation | self | `account` | depends: `account_id`, `company_id`, `move_id`, `product_id`, `display_type`, `analytic_distribution` |  |
| `copy_data` | lifecycle override | self, default | `account`, `l10n_tr` |  |  |
| `_field_to_sql` | internal rule | self, alias, field_expr, query | `account` |  |  |
| `_search_panel_domain_image` | search rule | self, field_name, domain, set_count, limit | `account` |  |  |
| `_get_reconciliation_aml_field_value` | preparation rule | self, field, shadowed_aml_values | `account` |  |  |
| `_prepare_move_line_residual_amounts` | preparation rule | self, aml_values, counterpart_currency, shadowed_aml_values, other_aml_values | `account` | model | Prepare the available residual amounts for each currency. :param aml_values: The values of account.move.line to consider. :param counterpart_currency: The currency of the opposite line this line will be reconciled with. :param shadowed_aml_values: A mapping aml -> dictionary to replace some original aml values to something else.                             This is usefull if you want to preview the reconciliation before doing some changes                             on amls like changing a date or an account. :param other_aml_values:    The other aml values to be reconciled with the current on |
| `_prepare_reconciliation_single_partial` | preparation rule | self, debit_values, credit_values, shadowed_aml_values | `account` | model | Prepare the values to create an account.partial.reconcile later when reconciling the dictionaries passed as parameters, each one representing an account.move.line. :param debit_values:  The values of account.move.line to consider for a debit line. :param credit_values: The values of account.move.line to consider for a credit line. :param shadowed_aml_values: A mapping aml -> dictionary to replace some original aml values to something else.                             This is usefull if you want to preview the reconciliation before doing some changes                             on amls like cha |
| `_prepare_reconciliation_amls` | preparation rule | self, values_list, shadowed_aml_values | `account` | model | Prepare the partials on the current journal items to perform the reconciliation. Note: The order of records in self is important because the journal items will be reconciled using this order.  :param values_list: A list of dictionaries, one for each aml. :param shadowed_aml_values: A mapping aml -> dictionary to replace some original aml values to something else.                             This is usefull if you want to preview the reconciliation before doing some changes                             on amls like changing a date or an account. :return: a tuple of     1) list of vals for partia |
| `_prepare_reconciliation_plan` | preparation rule | self, plan, amls_values_map, shadowed_aml_values | `account` | model | Perform virtually the reconciliation of the plan passed as parameter.  :param plan: The plan to know which lines to reconcile in which order. :param amls_values_map: A mapping aml => amount_residual/amount_residual_currency :param shadowed_aml_values: A mapping aml -> dictionary to replace some original aml values to something else.                             This is usefull if you want to preview the reconciliation before doing some changes                             on amls like changing a date or an account. :return: A list of all results returned by the '_prepare_reconciliation_amls' met |
| `_check_amls_exigibility_for_reconciliation` | validation | self, shadowed_aml_values | `account` |  | Ensure the current journal items are eligible to be reconciled together. :param shadowed_aml_values: A mapping aml -> dictionary to replace some original aml values to something else.                             This is usefull if you want to preview the reconciliation before doing some changes                             on amls like changing a date or an account. |
| `_optimize_reconciliation_plan` | internal rule | self, reconciliation_plan, shadowed_aml_values | `account` | model | Decode the initial reconciliation plan passed as parameter and converted it into a list of tree depicting the way the reconciliation should be done. Also, this method is responsible sorting the amls and splitting them by currency. Then, this method checks the parameter to ensure we are not going to perform any invalid reconciliation like a cross-account/cross-company partial.  The split by currencies is made as follows. Suppose account.move.line(1, 2) are expressed in currency1 and account.move.line(3, 4) are expressed in currency2. If the reconciliation plan is [account.move.line(1, 2, 3, 4)] |
| `_reconcile_pre_hook` | internal rule | self | `account` |  |  |
| `_reconcile_post_hook` | internal rule | self, data | `account` |  |  |
| `_reconcile_plan` | internal rule | self, reconciliation_plan | `account` | model | Reconcile the amls following the reconciliation plan. The plan passed as parameter is a list of either a recordset of amls, either another plan.  For example: [account.move.line(1, 2), account.move.line(3, 4)] means: - account.move.line(1, 2) will be reconciled first. - account.move.line(3, 4) will be reconciled after.  [[account.move.line(1, 2), account.move.line(3, 4)]] means: - account.move.line(1, 2) will be reconciled first. - account.move.line(3, 4) will be reconciled after. - account.move.line(1, 2, 3, 4).filtered(lambda x: not x.reconciled) will be reconciled at the end.  :param reconc |
| `_reconcile_plan_with_sync` | internal rule | self, plan_list, all_amls | `account` |  |  |
| `_get_exchange_journal` | preparation rule | self, company | `account` |  |  |
| `_get_exchange_account` | preparation rule | self, company, amount | `account` |  |  |
| `_prepare_exchange_difference_move_vals` | preparation rule | self, amounts_list, company, exchange_date, **kwargs | `account` |  | Prepare values to create later the exchange difference journal entry. The exchange difference journal entry is there to fix the debit/credit of lines when the journal items are fully reconciled in foreign currency. :param amounts_list:    A list of dict, one for each aml. :param company:         The company in case there is no aml in self. :param exchange_date:   Optional date object providing the date to consider for the exchange difference. :return:                A python dictionary containing:     * move_vals:    A dictionary to be passed to the account.move.create method.     * to_reconci |
| `_create_exchange_difference_moves` | internal rule | self, exchange_diff_values_list | `account` | model | Create the exchange difference journal entry on the current journal items.  :param exchange_diff_values_list:   A list of values to create and reconcile the exchange differences                                     See the '_prepare_exchange_difference_move_vals' method. :return: An account.move recordset. |
| `reconcile` | operation | self | `account` |  | Reconcile the current move lines all together. |
| `remove_move_reconcile` | operation | self | `account` |  | Undo a reconciliation |
| `action_unreconcile_match_entries` | user action | self | `account` |  | This method will do the unreconcile action in the list view of the moves |
| `_reconcile_marked` | internal rule | self | `account` |  | Process the pending reconciliation of entries marked (i.e. uring imports).  The entries can be marked using the string `I*` as matching number where `*` can be anything. Once all the entries using identical numbers are posted, this function proceeds to do the real matching. |
| `_get_matched_move_ids` | preparation rule | self | `account` |  | Return a record set with both self.matched_debit_ids & self.matched_credit_ids |
| `_validate_analytic_distribution` | internal rule | self | `account` |  |  |
| `_create_analytic_lines` | internal rule | self | `account` |  | Create analytic items upon validation of an account.move.line having an analytic distribution. |
| `_prepare_analytic_lines` | preparation rule | self | `account`, `sale` |  | Note: This method is called only on the move.line that having an analytic distribution, and so that should create analytic entries. |
| `_prepare_analytic_distribution_line` | preparation rule | self, distribution, account_ids, distribution_on_each_plan | `account` |  | Prepare the values used to create() an account.analytic.line upon validation of an account.move.line having analytic tags with analytic distribution. |
| `_related_analytic_distribution` | internal rule | self | `account`, `purchase`, `sale` |  | Returns the analytic distribution set on the record which triggered the creation of this line. |
| `_update_analytic_distribution` | internal rule | self | `account` |  |  |
| `_round_analytic_distribution_line` | internal rule | self, analytic_lines_vals | `account` |  | Round the analytic lines amount, and cancel the rounding error. |
| `_get_installments_data` | preparation rule | self, payment_currency, payment_date, next_payment_date | `account` |  |  |
| `_get_integrity_hash_fields` | preparation rule | self | `account` |  |  |
| `_reconciled_lines` | internal rule | self | `account` |  |  |
| `_reconciled_by_number` | internal rule | self | `account` |  | Get the mapping of all the lines matched with the lines in self grouped by matching number. |
| `_filter_reconciled_by_number` | internal rule | self, mapping | `account` |  | Get all the the lines matched with the lines in self.  Uses a mapping built with `_reconciled_by_number` to avoid multiple calls to the database. |
| `_all_reconciled_lines` | internal rule | self | `account` |  | Get all the the lines matched with the lines in self. |
| `_get_attachment_domains` | preparation rule | self | `account`, `hr_expense` |  |  |
| `_get_attachment_by_record` | preparation rule | self, id_model2attachments, move_line | `account`, `hr_expense` | model |  |
| `_get_tax_exigible_domain` | preparation rule | self | `account` | model | Returns a domain to be used to identify the move lines that are allowed to be taken into account in the tax report. |
| `_get_invoiced_qty_per_product` | preparation rule | self | `account`, `mrp_account` |  |  |
| `_get_lock_date_protected_fields` | preparation rule | self | `account` |  | Returns the names of the fields that should be protected by the accounting fiscal year and tax lock dates |
| `get_import_templates` | operation | self | `account` | model |  |
| `_prepare_edi_vals_to_export` | preparation rule | self | `account` |  | The purpose of this helper is the same as '_prepare_edi_vals_to_export' but for a single invoice line. This includes the computation of the tax details for each invoice line or the management of the discount. Indeed, in some EDI, we need to provide extra values depending the discount such as: - the discount as an amount instead of a percentage. - the price_unit but after subtraction of the discount.  :return: A python dict containing default pre-processed values. |
| `_get_journal_items_full_name` | preparation rule | self, name, display_name | `account` |  |  |
| `_check_edi_line_tax_required` | validation | self | `account` |  |  |
| `_get_aml_values` | preparation rule | self, **kwargs | `account` |  |  |
| `_filter_aml_lot_valuation` | internal rule | self | `account` |  | Method used to filter the aml taken into account when computing the invoiced lot value in get_invoiced_lot_values Intended to be overriden in localization. |
| `_get_child_lines` | preparation rule | self | `account`, `l10n_gcc_invoice` |  | Return a tax-wise summary of account move lines linked to section. Groups lines by their tax IDs and computes subtotal and total for each group. |
| `get_section_subtotal` | operation | self | `account` |  |  |
| `get_section_total` | operation | self | `account` |  |  |
| `get_column_to_exclude_for_colspan_calculation` | operation | self, taxes | `account`, `l10n_ar` |  |  |
| `get_parent_section_line` | operation | self | `account` |  |  |
| `_get_section_lines` | preparation rule | self | `account` |  |  |
| `_is_line_in_section` | internal rule | self, line | `account` |  | Return whether the line is a direct or indirect child of the section. |
| `open_reconcile_view` | operation | self | `account` |  |  |
| `action_open_business_doc` | user action | self | `account` |  |  |
| `action_automatic_entry` | user action | self, default_action | `account` |  |  |
| `action_add_from_catalog` | user action | self | `account` |  | Will open the catalog view |
| `_get_product_catalog_lines_data` | preparation rule | self, **kwargs | `account` |  | Return information about account_move_line in `self`. If `self` is empty, this method returns only the default value(s) needed for the product catalog. In this case, the quantity that equals 0. Otherwise, it returns a quantity and a price based on the product of the move line(s) and whether the product is read-only or not. A product is considered read-only if the order is considered read-only or if `self` contains multiple records. Note: This method cannot be called with multiple records that have different products linked.  :param products: Recordset of `product.product`. :param dict kwargs:  |
| `_conditional_add_to_compute` | internal rule | self, fname, condition | `account` |  |  |
| `_copy_data_extend_business_fields` | internal rule | self, values | `account`, `purchase`, `sale` |  |  |
| `_get_downpayment_lines` | preparation rule | self | `account`, `pos_sale`, `sale` |  | Return the downpayment move lines associated with the move line. This method is overridden in the sale order module. |
| `_get_discount_lines` | preparation rule | self | `account`, `pos_discount`, `pos_loyalty`, `sale_loyalty`, `sale` |  | Return the discount move lines associated with the move line. |
| `_compute_need_vehicle` | computation | self | `account_fleet` |  |  |
| `_prepare_fleet_log_service` | preparation rule | self | `account_fleet` |  |  |
| `_compute_sale_line_warn_msg` | computation | self | `sale` | depends: `product_id.sale_line_warn_msg` |  |
| `_sale_can_be_reinvoice` | internal rule | self | `sale_expense`, `sale_stock`, `sale` |  | determine if the generated analytic line should be reinvoiced or not. For Vendor Bill flow, if the product has a 'erinvoice policy' and is a cost, then we will find the SO on which reinvoice the AAL |
| `_sale_create_reinvoice_sale_line` | internal rule | self | `sale_expense`, `sale` |  |  |
| `_sale_determine_order` | internal rule | self | `project_sale_expense`, `sale_expense`, `sale_project`, `sale` |  | Get the mapping of move.line with the sale.order record on which its analytic entries should be reinvoiced :return a dict where key is the move line id, and value is sale.order record (or None). |
| `_sale_prepare_sale_line_values` | internal rule | self, order, price | `sale_expense_margin`, `sale_expense`, `sale` |  | Generate the sale.line creation value from the current move line |
| `_sale_get_invoice_price` | internal rule | self, order | `sale` |  | Based on the current move line, compute the price to reinvoice the analytic line that is going to be created (so the price of the sale line). |
| `_eligible_for_stock_account` | internal rule | self | `repair`, `stock_account` |  |  |
| `_get_gross_unit_price` | preparation rule | self | `stock_account` |  |  |
| `_get_cogs_value` | preparation rule | self | `point_of_sale`, `stock_account` |  | Get the COGS price unit in the product's default unit of measure. |
| `_get_stock_moves` | preparation rule | self | `mrp_subcontracting_purchase`, `purchase_stock`, `sale_stock`, `stock_account` |  |  |
| `_get_cogs_qty` | preparation rule | self | `sale_stock`, `stock_account` |  |  |
| `_get_posted_cogs_value` | preparation rule | self | `sale_stock`, `stock_account` |  |  |
| `_get_lines_from_original_invoice` | preparation rule | self | `sale_stock`, `stock_account` |  |  |
| `_get_sale_stock_move` | preparation rule | self | `sale_mrp`, `sale_stock` |  |  |
| `_compute_tax_amount` | computation | self | `l10n_gcc_invoice`, `l10n_sa_edi` | depends: `price_subtotal`, `price_total` |  |
| `_compute_l10n_gcc_line_name` | computation | self | `l10n_gcc_invoice` | depends: `name` |  |
| `_l10n_gcc_get_section_total` | internal rule | self | `l10n_gcc_invoice` |  |  |
| `_l10n_gcc_get_section_tax_amount` | internal rule | self | `l10n_gcc_invoice` |  |  |
| `_auto_init` | lifecycle override | self | `l10n_gr_edi`, `l10n_hr_edi`, `l10n_latam_invoice_document` |  | Create all compute-stored fields here to avoid MemoryError when initializing on large databases. |
| `_l10n_ar_prices_and_taxes` | internal rule | self | `l10n_ar` |  |  |
| `_l10n_cl_prices_and_taxes` | internal rule | self | `l10n_cl` |  | this method is preserved here to allow compatibility with old templates, Nevertheless it will be deprecated in future versions, since it had been replaced by the method _l10n_cl_get_line_amounts, which is the same method used to calculate the values for the XML (DTE) file |
| `_l10n_cl_get_line_amounts` | internal rule | self | `l10n_cl` |  | This method is used to calculate the amount and taxes of the lines required in the Chilean localization electronic documents. |
| `_prepare_line_values_for_purchase` | preparation rule | self | `purchase` |  |  |
| `_compute_purchase_line_warn_msg` | computation | self | `purchase` | depends: `product_id.purchase_line_warn_msg` |  |
| `_l10n_es_tbai_is_ignored` | internal rule | self | `l10n_es_edi_tbai` |  |  |
| `_compute_l10n_gr_edi_detail_type` | computation | self | `l10n_gr_edi` | depends: `move_id.l10n_gr_edi_inv_type` |  |
| `_compute_l10n_gr_edi_available_cls_category` | computation | self | `l10n_gr_edi` | depends: `move_id.l10n_gr_edi_inv_type`, `move_id.l10n_gr_edi_correlation_id`, `l10n_gr_edi_detail_type` |  |
| `_compute_l10n_gr_edi_available_cls_type` | computation | self | `l10n_gr_edi` | depends: `l10n_gr_edi_cls_category`, `move_id.l10n_gr_edi_correlation_id` |  |
| `_l10n_gr_edi_get_preferred_classification_id` | internal rule | self, with_category | `l10n_gr_edi` |  |  |
| `_compute_l10n_gr_edi_cls_category` | computation | self | `l10n_gr_edi` | depends: `move_id.l10n_gr_edi_inv_type`, `l10n_gr_edi_available_cls_category`, `product_id` |  |
| `_compute_l10n_gr_edi_cls_type` | computation | self | `l10n_gr_edi` | depends: `move_id.l10n_gr_edi_inv_type`, `l10n_gr_edi_available_cls_type`, `product_id` |  |
| `_compute_l10n_gr_edi_cls_vat` | computation | self | `l10n_gr_edi` | depends: `l10n_gr_edi_available_cls_vat` |  |
| `_compute_l10n_gr_edi_need_exemption_category` | computation | self | `l10n_gr_edi` | depends: `tax_ids` |  |
| `_compute_l10n_gr_edi_tax_exemption_category` | computation | self | `l10n_gr_edi` | depends: `tax_ids` |  |
| `_compute_l10n_hr_product_id_kpd` | computation | self | `l10n_hr_edi` | depends: `product_id` | Copy KPD code from product template when a product is selected for the line. |
| `_l10n_id_coretax_build_invoice_line_vals` | internal rule | self, vals | `l10n_id_efaktur_coretax` |  | Fill in the vals['lines'] with some information regarding each invoice line |
| `_compute_l10n_in_withhold_tax_amount` | computation | self | `l10n_in` | depends: `tax_ids` |  |
| `_compute_l10n_in_hsn_code` | computation | self | `l10n_in` | depends: `product_id`, `product_id.l10n_in_hsn_code` |  |
| `_l10n_in_check_invalid_hsn_code` | internal rule | self | `l10n_in` |  |  |
| `_get_l10n_in_tax_tag_ids` | preparation rule | self | `l10n_in` |  |  |
| `_get_l10n_in_gstr_section` | preparation rule | self, tax_tags_dict | `l10n_in_pos`, `l10n_in` |  |  |
| `_set_l10n_in_gstr_section` | internal rule | self, tax_tags_dict | `l10n_in` |  |  |
| `_l10n_in_is_global_discount` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_check_einvoice_validation` | internal rule | self | `l10n_in_edi` |  |  |
| `_get_price_unit_val_dif_and_relevant_qty` | preparation rule | self | `mrp_subcontracting_purchase`, `purchase_stock` |  |  |
| `_compute_l10n_my_edi_classification_code` | computation | self | `l10n_my_edi` | depends: `product_id.product_tmpl_id` | Default to the product classification if any |
| `_apply_retention_tax_filter` | internal rule | self, tax_values | `l10n_sa_edi` |  |  |
| `_is_global_discount_line` | internal rule | self | `l10n_sa_edi` |  | Any line that has a negative amount and is not linked to a down-payment is considered as a global discount line. These can be created either manually, or through a promotions program. |
| `_compute_l10n_tr_ctsp_number` | computation | self | `l10n_tr_nilvera_einvoice_extended` | depends: `product_id.l10n_tr_ctsp_number` |  |
| `_check_l10n_tr_ctsp_number` | validation | self | `l10n_tr_nilvera_einvoice_extended` | constrains: `l10n_tr_ctsp_number` |  |
| `_onchange_product_id_landed_costs` | on change | self | `stock_landed_costs` | onchange: `product_id` |  |
| `_onchange_is_landed_costs_line` | on change | self | `stock_landed_costs` | onchange: `is_landed_costs_line` |  |
| `_get_so_mapping_domain` | preparation rule | self | `sale_project` |  |  |
| `_get_so_mapping_from_project` | preparation rule | self | `sale_project` |  | Get the mapping of move.line with the sale.order record on which its analytic entries should be reinvoiced. A sale.order matches a move.line if the sale.order's project contains all the same analytic accounts as the ones in the distribution of the move.line. :return a dict where key is the move line id, and value is sale.order record (or None). |
| `_get_so_mapping_from_expense` | preparation rule | self | `sale_expense` |  |  |
| `_timesheet_domain_get_invoiced_lines` | internal rule | self, sale_line_delivery | `sale_timesheet` | model | Get the domain for the timesheet to link to the created invoice :param sale_line_delivery: recordset of sale.order.line to invoice :return a normalized domain |

## Validation and error messages (36)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_constrains_account_id_journal_id` | UserError | The account %(name)s (%(code)s) is archived. | `account` |
| `_check_constrains_account_id_journal_id` | UserError | The account selected on your journal entry forces to provide a secondary currency. You should remove the secondary currency on the account. | `account` |
| `_check_off_balance` | UserError | If you want to use "Off-Balance Sheet" accounts, all the accounts of the journal entry must be of this type | `account` |
| `_check_off_balance` | UserError | You cannot use taxes on lines with an Off-Balance account | `account` |
| `_check_off_balance` | UserError | Lines from "Off-Balance Sheet" accounts cannot be reconciled | `account` |
| `_check_payable_receivable` | UserError | Account %s is of payable type, but is used in a sale operation. | `account` |
| `_check_payable_receivable` | UserError | Any journal item on a receivable account must have a due date and vice versa. | `account` |
| `_check_payable_receivable` | UserError | Account %s is of receivable type, but is used in a purchase operation. | `account` |
| `_check_payable_receivable` | UserError | Any journal item on a payable account must have a due date and vice versa. | `account` |
| `_check_tax_lock_date` | UserError | The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: %(lock_date_info)s. | `account` |
| `_check_reconciliation` | UserError | You cannot do this modification on a reconciled journal entry. You can just change some non legal fields or you must unreconcile first. Journal Entry (id): %(entry)s (%(id)s) | `account` |
| `_check_caba_non_caba_shared_tags` | ValidationError | Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag. | `account` |
| `_constrains_matching_number` | ValidationError | A temporary number can not be used in a real matching | `account` |
| `_constrains_deductible_amount` | ValidationError | Only vendor bills allow for deductibility of product/services. | `account` |
| `_constrains_deductible_amount` | ValidationError | The deductibility must be a value between 0 and 100. | `account` |
| `write` | UserError | You cannot use an archived account. | `account` |
| `write` | UserError | You cannot edit the following fields: %(fields)s. The following entries are already hashed: %(entries)s | `account` |
| `write` | UserError | You cannot modify the taxes related to a posted journal item, you should reset the journal entry to draft to do so. | `account` |
| `_unlink_except_posted` | UserError | You can't delete a posted journal item. Don’t play games with your accounting records; reset the journal entry to draft before deleting it. | `account` |
| `_prevent_automatic_line_deletion` | ValidationError | You cannot delete a tax line as it would impact the tax report | `account` |
| `_prevent_automatic_line_deletion` | ValidationError | You cannot delete a payable/receivable line as it would not be consistent with the payment terms | `account` |
| `_except_hashed_entry_lines` | UserError | You cannot delete journal items belonging to a locked journal entry. | `account` |
| `_check_amls_exigibility_for_reconciliation` | UserError | You are trying to reconcile some entries that are already reconciled. | `account` |
| `_check_amls_exigibility_for_reconciliation` | UserError | You can not reconcile cancelled entries. | `account` |
| `_check_amls_exigibility_for_reconciliation` | UserError | Entries are not from the same account: %s | `account` |
| `_check_amls_exigibility_for_reconciliation` | UserError | Entries don't belong to the same company: %s | `account` |
| `_check_amls_exigibility_for_reconciliation` | UserError | Account %s does not allow reconciliation. First change the configuration of this account to allow it. | `account` |
| `_create_exchange_difference_moves` | UserError | You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. | `account` |
| `_create_exchange_difference_moves` | UserError | You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. | `account` |
| `_create_exchange_difference_moves` | UserError | You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. | `account` |
| `_validate_analytic_distribution` | ValidationError | msg | `account` |
| `_sale_create_reinvoice_sale_line` | UserError | The Sales Order %(order)s to be reinvoiced must be validated before registering expenses. | `sale` |
| `_sale_create_reinvoice_sale_line` | UserError | The Sales Order %(order)s to be reinvoiced is cancelled. You cannot register an expense on a cancelled Sales Order. | `sale` |
| `_sale_create_reinvoice_sale_line` | UserError | The Sales Order %(order)s to be reinvoiced is currently locked. You cannot register an expense on a locked Sales Order. | `sale` |
| `_l10n_id_coretax_build_invoice_line_vals` | ValidationError | Price for line '%s' cannot be a negative amount. Please check again. | `l10n_id_efaktur_coretax` |
| `_check_l10n_tr_ctsp_number` | ValidationError | CTSP Number must be 12 digits or fewer. | `l10n_tr_nilvera_einvoice_extended` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `base.group_portal` | no | yes | no | no | `account` |
| `hr_expense.group_hr_expense_team_approver` | no | yes | no | no | `hr_expense` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `group_purchase_user` | yes | yes | yes | no | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Entry lines | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| All Journal Items | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Portal Invoice Lines | `[(4, ref('base.group_portal'))]` | `[('parent_state', 'not in', ('cancel', 'draft')), ('move_id.move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('move_id.partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |
| Readonly Move Line | `[(4, ref('account.group_account_readonly'))]` | `[(1, '=', 1)]` | True | False | False | False |
| Readonly Move Line | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Expense Team Approver Account Move Line | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `[('expense_id', '!=', False)]` | True | True | True | True |
| Invoice Line POS User | `[(4, ref('group_pos_user'))]` | `[('move_id.pos_order_ids', '!=', False)]` | True | True | True | True |
| Purchase User Account Move Line | `[(4, ref('purchase.group_purchase_user'))]` | `[('move_id.move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]` | True | True | True | True |
| Personal Invoice Lines | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_id.invoice_user_id', '=', user.id), ('move_id.invoice_user_id', '=', False)]` | True | True | True | True |
| All Invoice Lines | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund'))]` | True | True | True | True |

## Views (21)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_move_line_form` | form |  | `company_id`, `parent_state`, `name`, `partner_id`, `account_id`, `debit`, `credit`, `balance`, `quantity`, `date`, `date_maturity`, `tax_line_id`, `tax_ids`, `full_reconcile_id`, `matched_debit_ids`, `matched_credit_ids`, `currency_id`, `display_type`, `amount_currency`, `product_id`, `analytic_distribution`, `date`, `analytic_line_ids`, `move_id`, `statement_line_id` | `-> View partially reconciled entries` |  | `account` |
| `account.account_move_line_view_kanban` | kanban |  | `company_currency_id`, `account_id`, `date_maturity`, `name`, `partner_id`, `tax_ids`, `debit`, `credit` |  |  | `account` |
| `account.account_move_line_view_kanban_mobile` | xpath | `account_move_line_view_kanban` |  |  |  | `account` |
| `account.view_move_line_pivot` | pivot |  | `journal_id`, `date`, `balance` |  |  | `account` |
| `account.view_move_line_tree` | list |  | `move_id`, `invoice_date`, `date`, `company_id`, `company_id`, `journal_id`, `move_name`, `account_id`, `partner_id`, `ref`, `product_id`, `name`, `analytic_distribution`, `tax_ids`, `amount_currency`, `currency_id`, `debit`, `credit`, `tax_tag_ids`, `discount_date`, `discount_amount_currency`, `tax_line_id`, `date_maturity`, `balance`, `matching_number`, `amount_residual`, `amount_residual_currency`, `move_type`, `parent_state`, `account_type`, `statement_line_id`, `company_currency_id`, `is_same_currency`, `is_account_reconcile`, `sequence`, `parent_state`, `deductible_amount` | `edit`, `Post`, `Review`, `Edit` |  | `account` |
| `account.view_move_line_tree_grouped_sales_purchases` | field | `account.view_move_line_tree` | `date` |  |  | `account` |
| `account.view_move_line_tree_grouped_bank_cash` | field | `account.view_move_line_tree` | `date` |  |  | `account` |
| `account.view_move_line_tree_grouped_misc` | field | `account.view_move_line_tree` | `date` |  |  | `account` |
| `account.view_move_line_tree_grouped_general` | field | `account.view_move_line_tree` | `account_id` |  |  | `account` |
| `account.view_move_line_tree_grouped_partner` | field | `account.view_move_line_tree` | `partner_id` |  |  | `account` |
| `account.view_move_line_tax_audit_tree` | field | `account.view_move_line_tree` | `matching_number`, `tax_line_id`, `tax_base_amount` |  |  | `account` |
| `account.account_move_line_graph_date` | graph |  | `date`, `balance` |  |  | `account` |
| `account.view_account_move_line_filter` | search |  | `name`, `name`, `ref`, `invoice_date`, `date`, `date_maturity`, `discount_date`, `balance`, `account_id`, `account_type`, `partner_id`, `journal_id`, `journal_group_id`, `product_id`, `product_category_id`, `move_id`, `tax_ids`, `tax_line_id`, `reconcile_model_id`, `account_root_id` |  | `Unposted`, `Posted`, `Not Secured`, `To Review`, `Unreconciled`, `With residual`, `Sales`, `Purchases`, `Bank`, `Cash`, `Credit`, `Miscellaneous`, `Journals`, `Payable`, `Receivable`, `Non Trade Payable`, `Non Trade Receivable`, `P&L Accounts`, `No Bank Transaction`, `Date`, `Invoice Date`, `Report Dates`, `Report Dates`, `Analytic Accounts`, `Journal Entry`, `Account`, `Partner`, `Journal`, `Date`, `Invoice Date`, `Taxes`, `Tax Grid`, `Matching` | `account` |
| `account.view_move_line_payment_tree` | list |  | `move_id`, `invoice_date`, `date`, `date_maturity`, `discount_date`, `payment_date`, `company_id`, `company_id`, `journal_id`, `move_name`, `partner_id`, `ref`, `name`, `discount_amount_currency`, `amount_residual`, `amount_residual_currency`, `currency_id`, `company_currency_id`, `move_type`, `is_same_currency`, `is_account_reconcile`, `parent_state` | `Pay`, `edit` |  | `account` |
| `account.view_account_move_line_payment_filter` | search |  | `name`, `name`, `move_id`, `ref`, `payment_date`, `partner_id`, `journal_id`, `currency_id`, `company_currency_id` |  | `Posted`, `Invoices`, `Credit Notes`, `Bills`, `Refunds`, `Invoice Date`, `Next Payment Date`, `Overdue`, `Early Discount`, `Report Dates`, `Report Dates`, `Currency`, `Partner`, `Journal`, `Invoice Date`, `Next Payment Date` | `account` |
| `account_debit_note.view_account_move_line_filter_debit` | filter | `account.view_account_move_line_filter` |  |  | `unreconciled`, `Debit Note` | `account_debit_note` |
| `account_fleet.view_move_line_tree_fleet` | xpath | `account.view_move_line_tree` | `vehicle_id` |  |  | `account_fleet` |
| `l10n_in.view_move_line_list_l10n_in_withholding` | list |  | `product_id`, `name`, `account_id`, `tax_ids`, `l10n_in_tds_tcs_section_id`, `price_total` |  |  | `l10n_in` |
| `l10n_in.view_move_line_tree_hsn_l10n_in` | list |  | `move_id`, `product_id`, `name`, `l10n_in_hsn_code`, `tax_ids`, `discount` |  |  | `l10n_in` |
| `l10n_in.view_move_line_tree_l10n_in` | xpath | `account.view_move_line_tree` | `l10n_in_gstr_section` |  |  | `l10n_in` |
| `l10n_latam_invoice_document.view_account_move_line_filter` | separator | `account.view_account_move_line_filter` | `l10n_latam_document_type_id` |  |  | `l10n_latam_invoice_document` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_move_line_select` | Journal Items |  |  | `{'search_default_account_id': [active_id], 'search_default_posted': 1}` |  | `account` |
| `account.action_account_moves_all_a` | Journal Items | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'journal_type':'general', 'search_default_group_by_move': 1, 'search_default_posted':1, 'create':0}` |  | `account` |
| `account.action_account_moves_all_grouped_matching` | Journal Items | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'journal_type':'general', 'search_default_posted':1, 'expand':'1'}` |  | `account` |
| `account.action_account_moves_journal_sales` | Sales | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'journal_type':'sales', 'search_default_group_by_move': 1, 'search_default_posted':1, 'search_default_sales':1, 'expand': 1}` |  | `account` |
| `account.action_account_moves_journal_purchase` | Purchases | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'journal_type':'purchase', 'search_default_group_by_move': 1, 'search_default_posted':1, 'search_default_purchases':1, 'expand': 1}` |  | `account` |
| `account.action_account_moves_journal_bank_cash` | Bank and Cash | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'journal_type':'bank', 'search_default_group_by_move': 1, 'search_default_posted':1, 'search_default_bank':1, 'search_default_cash':1, 'expand': 1}` |  | `account` |
| `account.action_account_moves_journal_misc` | Miscellaneous | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'journal_type':'general', 'search_default_group_by_move': 1, 'search_default_posted':1, 'search_default_misc_filter':1, 'expand': 1}` |  | `account` |
| `account.action_account_moves_ledger_partner` | Partner Ledger | list,pivot,graph | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'journal_type':'general', 'search_default_group_by_partner': 1, 'search_default_posted':1, 'search_default_trade_payable':1, 'search_default_trade_receivable':1, 'search_default_unreconciled':1}` |  | `account` |
| `account.action_account_moves_all_tree` | Journal Items |  | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` | `{'search_default_partner_id': [active_id], 'default_partner_id': active_id, 'search_default_posted':1}` |  | `account` |
| `account.action_account_moves_all` | Journal Items | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note')), ('parent_state', '!=', 'cancel')]` | `{'journal_type':'general', 'search_default_posted':1}` |  | `account` |
| `account.action_amounts_to_settle` | Amounts to Settle | list | `[('parent_state', '=', 'posted'), ('date_maturity', '!=', False), ('amount_residual', '!=', 0), ('account_id.reconcile', '=', True)]` |  |  | `account` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `account.action_account_unreconcile` | Unreconcile | code |  | yes |
| `account.action_automatic_entry_change_account` | Move to Account | code |  | yes |
| `account.action_automatic_entry_change_period` | Change Period | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/account.move.line.json`; views: `../../../schemas/interfaces/views/account.move.line.json`.
