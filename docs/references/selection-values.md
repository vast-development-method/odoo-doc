# Selection values

Every field whose value comes from a fixed set: 1107 fields carrying 6334 values in total, as they exist once every capability package has contributed.

The value is stored and is part of any external contract, so it is reproduced exactly. The label is presentation.

## `account.account`

**`account_type`** — Type

| Value | Label |
|---|---|
| `asset_receivable` | Receivable |
| `asset_cash` | Bank and Cash |
| `asset_current` | Current Assets |
| `asset_non_current` | Non-current Assets |
| `asset_prepayments` | Prepayments |
| `asset_fixed` | Fixed Assets |
| `liability_payable` | Payable |
| `liability_credit_card` | Credit Card |
| `liability_current` | Current Liabilities |
| `liability_non_current` | Non-current Liabilities |
| `equity` | Equity |
| `equity_unaffected` | Current Year Earnings |
| `income` | Income |
| `income_other` | Other Income |
| `expense` | Expenses |
| `expense_other` | Other Expenses |
| `expense_depreciation` | Depreciation |
| `expense_direct_cost` | Cost of Revenue |
| `off_balance` | Off-Balance Sheet |

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`internal_group`** — Internal Group (not stored)

| Value | Label |
|---|---|
| `equity` | Equity |
| `asset` | Asset |
| `liability` | Liability |
| `income` | Income |
| `expense` | Expense |
| `off` | Off Balance |

## `account.account.tag`

**`applicability`** — Applicability

| Value | Label |
|---|---|
| `accounts` | Accounts |
| `taxes` | Taxes |
| `products` | Products |

## `account.analytic.applicability`

**`applicability`** — Applicability

| Value | Label |
|---|---|
| `optional` | Optional |
| `mandatory` | Mandatory |
| `unavailable` | Unavailable |

**`business_domain`** — Domain

| Value | Label |
|---|---|
| `general` | Miscellaneous |
| `invoice` | Invoice |
| `bill` | Vendor Bill |
| `expense` | Expense |
| `purchase_order` | Purchase Order |
| `timesheet` | Timesheet |
| `manufacturing_order` | Manufacturing Order |
| `stock_picking` | Stock Picking |
| `sale_order` | Sale Order |

## `account.analytic.line`

**`analytic_profitability`** — Profitability (not stored)

| Value | Label |
|---|---|
| `uncategorized` | Uncategorized |
| `revenue` | Revenue |
| `loss` | Loss |

**`category`** — Category

| Value | Label |
|---|---|
| `other` | Other |
| `invoice` | Customer Invoice |
| `vendor_bill` | Vendor Bill |
| `manufacturing_order` | Manufacturing Order |
| `picking_entry` | Inventory Transfer |

**`timesheet_invoice_type`** — Billable Type

| Value | Label |
|---|---|
| `billable_time` | Billed on Timesheets |
| `billable_fixed` | Billed at a Fixed price |
| `billable_milestones` | Billed on Milestones |
| `billable_manual` | Billed Manually |
| `non_billable` | Non-Billable |
| `timesheet_revenues` | Timesheet Revenues |
| `service_revenues` | Service Revenues |
| `other_revenues` | Other revenues |
| `other_costs` | Other costs |

## `account.analytic.plan`

**`default_applicability`** — Default Applicability

| Value | Label |
|---|---|
| `optional` | Optional |
| `mandatory` | Mandatory |
| `unavailable` | Unavailable |

## `account.automatic.entry.wizard`

**`account_type`** — Account Type

| Value | Label |
|---|---|
| `income` | Revenue |
| `expense` | Expense |

**`action`** — Action

| Value | Label |
|---|---|
| `change_period` | Change Period |
| `change_account` | Change Account |

## `account.cash.rounding`

**`rounding_method`** — Rounding Method

| Value | Label |
|---|---|
| `UP` | Up |
| `DOWN` | Down |
| `HALF-UP` | Nearest |

**`strategy`** — Rounding Strategy

| Value | Label |
|---|---|
| `biggest_tax` | Modify tax amount |
| `add_invoice_line` | Add a rounding line |

## `account.debit.note`

**`l10n_sa_reason`** — ZATCA Reason

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | Cancellation or suspension of the supplies after its occurrence either wholly or partially |
| `BR-KSA-17-reason-2` | In case of essential change or amendment in the supply, which leads to the change of the VAT due |
| `BR-KSA-17-reason-3` | Amendment of the supply value which is pre-agreed upon between the supplier and consumer |
| `BR-KSA-17-reason-4` | In case of goods or services refund |
| `BR-KSA-17-reason-5` | In case of change in Seller's or Buyer's information |

## `account.edi.document`

**`blocking_level`** — Blocking Level

| Value | Label |
|---|---|
| `info` | Info |
| `warning` | Warning |
| `error` | Error |

**`state`** — State

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `to_cancel` | To Cancel |
| `cancelled` | Cancelled |

## `account.fiscal.position`

**`foreign_vat_header_mode`** — Foreign Vat Header Mode (not stored)

| Value | Label |
|---|---|
| `templates_found` | Templates Found |
| `no_template` | No Template |

**`l10n_br_fp_type`** — Interstate Fiscal Position Type

| Value | Label |
|---|---|
| `internal` | Internal |
| `ss_nnm` | South/Southeast selling to North/Northeast/Midwest |
| `interstate` | Other interstate |

## `account.invoice.report`

**`move_type`** — Move Type

| Value | Label |
|---|---|
| `out_invoice` | Customer Invoice |
| `in_invoice` | Vendor Bill |
| `out_refund` | Customer Credit Note |
| `in_refund` | Vendor Credit Note |

**`payment_state`** — Payment Status

| Value | Label |
|---|---|
| `not_paid` | Not Paid |
| `in_payment` | In Payment |
| `paid` | Paid |
| `partial` | Partially Paid |
| `reversed` | Reversed |
| `blocked` | Blocked |
| `invoicing_legacy` | Invoicing App Legacy |

**`state`** — Invoice Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `posted` | Open |
| `cancel` | Cancelled |

## `account.journal`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`invoice_reference_model`** — Communication Standard

| Value | Label |
|---|---|
| `system` | Full Reference (INV/2024/00001) |
| `euro` | European (RF83INV202400001) |
| `number` | Numbers only (202400001) |
| `be` | Belgium (+++000/2024/00182+++) |
| `ch` | Switzerland (12 34560 00103 88500 1000 19188) |
| `fi` | Finnish Standard Reference (2024000068) |
| `fi_rf` | Finnish Creditor Reference (RF) (RF952024000071) |
| `no` | Norway (000001024000083) |
| `se_ocr2` | Sweden OCR Level 1 & 2 (1255) |
| `se_ocr3` | Sweden OCR Level 3 (12658) |
| `se_ocr4` | Sweden OCR Level 4 (001271) |
| `si` | Slovenian 01 (SI01 25-1235-8403) |
| `dk_fik_71` | Denmark FIK Number (+71) |
| `dk_fik_75` | Denmark FIK Number (+75) |

**`invoice_reference_type`** — Communication Type

| Value | Label |
|---|---|
| `partner` | Based on Customer |
| `invoice` | Based on Invoice |

**`type`** — Type

| Value | Label |
|---|---|
| `sale` | Sales |
| `purchase` | Purchase |
| `cash` | Cash |
| `bank` | Bank |
| `credit` | Credit Card |
| `general` | Miscellaneous |

## `account.lock_exception`

**`lock_date_field`** — Lock Date Field

| Value | Label |
|---|---|
| `fiscalyear_lock_date` | Global Lock Date |
| `tax_lock_date` | Tax Return Lock Date |
| `sale_lock_date` | Sales Lock Date |
| `purchase_lock_date` | Purchase Lock Date |

**`state`** — State (not stored)

| Value | Label |
|---|---|
| `active` | Active |
| `revoked` | Revoked |
| `expired` | Expired |

## `account.merge.wizard.line`

**`display_type`** — Display Type

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `account` | Account |

## `account.move`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`auto_post`** — Auto-post

| Value | Label |
|---|---|
| `no` | No |
| `at_date` | At Date |
| `monthly` | Monthly |
| `quarterly` | Quarterly |
| `yearly` | Yearly |

**`edi_blocking_level`** — Edi Blocking Level (not stored)

| Value | Label |
|---|---|
| `info` | Info |
| `warning` | Warning |
| `error` | Error |

**`edi_state`** — Electronic invoicing

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `to_cancel` | To Cancel |
| `cancelled` | Cancelled |

**`l10n_bg_exemption_reason`** — Exemption reason (BG)

| Value | Label |
|---|---|
| `01` | 01 - A delivery under Part 1 of Appendix 2 of LVAT |
| `02` | 02 - A delivery under Part 2 of Appendix 2 of LVAT |
| `03` | 03 - Import under Appendix 3 of VAT act |

**`l10n_es_edi_facturae_reason_code`** — Spanish Facturae EDI Reason Code

| Value | Label |
|---|---|
| `01` | Invoice number |
| `02` | Invoice serial number |
| `03` | Issue date |
| `04` | Name and surnames/Corporate name - Issuer (Sender) |
| `05` | Name and surnames/Corporate name - Receiver |
| `06` | Issuer's Tax Identification Number |
| `07` | Receiver's Tax Identification Number |
| `08` | Issuer's address |
| `09` | Receiver's address |
| `10` | Item line |
| `11` | Applicable Tax Rate |
| `12` | Applicable Tax Amount |
| `13` | Applicable Date/Period |
| `14` | Invoice Class |
| `15` | Legal literals |
| `16` | Taxable Base |
| `80` | Calculation of tax outputs |
| `81` | Calculation of tax inputs |
| `82` | Taxable Base modified due to return of packages and packaging materials |
| `83` | Taxable Base modified due to discounts and rebates |
| `84` | Taxable Base modified due to firm court ruling or administrative decision |
| `85` | Taxable Base modified due to unpaid outputs where there is a judgement opening insolvency proceedings |

**`l10n_es_edi_verifactu_refund_reason`** — Veri*Factu Refund Reason

| Value | Label |
|---|---|
| `R1` | R1: Art 80.1 and 80.2 and error of law |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Rest |
| `R5` | R5: Corrective invoices concerning simplified invoices |

**`l10n_es_edi_verifactu_state`** — Veri*Factu Status

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |
| `cancelled` | Cancelled |

**`l10n_es_payment_means`** — Payment Means

| Value | Label |
|---|---|
| `01` | In cash |
| `02` | Direct debit |
| `03` | Receipt |
| `04` | Credit transfer |
| `05` | Accepted bill of exchange |
| `06` | Documentary credit |
| `07` | Contract award |
| `08` | Bill of exchange |
| `09` | Transferable promissory note |
| `10` | Non transferable promissory note |
| `11` | Cheque |
| `12` | Open account reimbursement |
| `13` | Special payment |
| `14` | Set-off by reciprocal credits |
| `15` | Payment by postgiro |
| `16` | Certified cheque |
| `17` | Banker’s draft |
| `18` | Cash on delivery |
| `19` | Payment by card |

**`l10n_es_tbai_refund_reason`** — Invoice Refund Reason Code (TicketBai)

| Value | Label |
|---|---|
| `R1` | R1: Art. 80.1, 80.2, 80.6 and rights founded error |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Art. 80 - other |
| `R5` | R5: Factura rectificativa en facturas simplificadas |

**`l10n_es_tbai_state`** — TicketBAI status (not stored)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `cancelled` | Cancelled |

**`l10n_fr_pdp_flow_10_operation_type`** — L10N Fr Pdp Flow 10 Operation Type

| Value | Label |
|---|---|
| `sale` | Sale |
| `purchase` | Purchase |

**`l10n_fr_pdp_flow_10_report_type`** — L10N Fr Pdp Flow 10 Report Type

| Value | Label |
|---|---|
| `transaction` | Transaction |
| `payment` | Payment |

**`l10n_fr_pdp_status`** — E-Reporting Status

| Value | Label |
|---|---|
| `out_of_scope` | Out of scope |
| `pending` | Pending |
| `ready` | Ready |
| `error` | Error |
| `sent` | Sent |
| `completed` | Completed |

**`l10n_gr_edi_inv_type`** — myDATA Invoice Type

| Value | Label |
|---|---|
| `1.1` | 1.1 - Sales Invoice |
| `1.2` | 1.2 - Sales Invoice/Intra-community Supplies |
| `1.3` | 1.3 - Sales Invoice/Third Country Supplies |
| `1.4` | 1.4 - Sales Invoice/Sale on Behalf of Third Parties |
| `1.5` | 1.5 - Sales Invoice/Clearance of Sales on Behalf of Third Parties – Fees from Sales on Behalf of Third Parties |
| `1.6` | 1.6 - Sales Invoice/Supplemental Accounting Source Document |
| `2.1` | 2.1 - Service Rendered Invoice |
| `2.2` | 2.2 - Intra-community Service Rendered Invoice |
| `2.3` | 2.3 - Third Country Service Rendered Invoice |
| `2.4` | 2.4 - Service Rendered Invoice/Supplemental Accounting Source Document |
| `3.1` | 3.1 - Proof of Expenditure (non-liable Issuer) |
| `3.2` | 3.2 - Proof of Expenditure (denial of issuance by liable Issuer) |
| `5.1` | 5.1 - Credit Invoice/Associated |
| `5.2` | 5.2 - Credit Invoice/Non-Associated |
| `6.1` | 6.1 - Self-Delivery Record |
| `6.2` | 6.2 - Self-Supply Record |
| `7.1` | 7.1 - Contract – Income |
| `8.1` | 8.1 - Rents – Income |
| `8.2` | 8.2 - Special Record – Accommodation Tax Collection/Payment Receipt |
| `11.1` | 11.1 - Retail Sales Receipt |
| `11.2` | 11.2 - Service Rendered Receipt |
| `11.3` | 11.3 - Simplified Invoice |
| `11.4` | 11.4 - Retail Sales Credit Note |
| `11.5` | 11.5 - Retail Sales Receipt on Behalf of Third Parties |
| `13.1` | 13.1 - Expenses – Domestic/Foreign Retail Transaction Purchases |
| `13.2` | 13.2 - Domestic/Foreign Retail Transaction Provision |
| `13.3` | 13.3 - Shared Utility Bills |
| `13.4` | 13.4 - Subscriptions |
| `13.30` | 13.30 - Self-Declared Entity Accounting Source Documents (Dynamic) |
| `13.31` | 13.31 - Domestic/Foreign Retail Sales Credit Note |
| `14.1` | 14.1 - Invoice/Intra-community Acquisitions |
| `14.2` | 14.2 - Invoice/Third Country Acquisitions |
| `14.3` | 14.3 - Invoice/Intra-community Services Receipt |
| `14.4` | 14.4 - Invoice/Third Country Services Receipt |
| `14.5` | 14.5 - EFKA |
| `14.30` | 14.30 - Self-Declared Entity Accounting Source Documents (Dynamic) |
| `14.31` | 14.31 - Domestic/Foreign Credit Note |
| `15.1` | 15.1 - Contract-Expense |
| `16.1` | 16.1 - Rent-Expense |
| `17.1` | 17.1 - Payroll |
| `17.2` | 17.2 - Amortisations |
| `17.3` | 17.3 - Other Income Adjustment/Regularisation Entries – Accounting Base |
| `17.4` | 17.4 - Other Income Adjustment/Regularisation Entries – Tax Base |
| `17.5` | 17.5 - Other Expense Adjustment/Regularisation Entries – Accounting Base |
| `17.6` | 17.6 - Other Expense Adjustment/Regularisation Entries – Tax Base |

**`l10n_gr_edi_payment_method`** — Payment Method

| Value | Label |
|---|---|
| `1` | 1 - Domestic Payments Account Number |
| `2` | 2 - Foreign Payments Account Number |
| `3` | 3 - Cash |
| `4` | 4 - Check |
| `5` | 5 - On credit |
| `6` | 6 - Web Banking |
| `7` | 7 - POS / e-POS |

**`l10n_gr_edi_state`** — myDATA Status

| Value | Label |
|---|---|
| `invoice_sent` | Invoice sent |
| `bill_fetched` | Expense classification ready to send |
| `bill_sent` | Expense classification sent |
| `invoice_pending` | Invoice submission pending |

**`l10n_hr_process_type`** — Business Process Type

| Value | Label |
|---|---|
| `P1` | P1: Issuing invoices for deliveries of goods and services according to purchase orders, based on contracts |
| `P2` | P2: Periodic invoicing for deliveries of goods and services based on contracts |
| `P3` | P3: Issuing invoices for delivery according to an independent purchase order |
| `P4` | P4: Prepayment (advance payment) |
| `P5` | P5: Payment on the spot (Sport payment) |
| `P6` | P6: Payment before delivery, based on purchase order |
| `P7` | P7: Issuing invoices with references to the delivery note |
| `P8` | P8: Issuing invoices with references to the shipping and receipt notes |
| `P9` | P9: Credits or invoices with negative amounts, issued for various reasons, including empty returns packaging |
| `P10` | P10: Issuing a corrective invoice (reversal/correction of invoice) |
| `P11` | P11: Issuing partial and final invoices |
| `P12` | P12: Self-issuance of invoice |
| `P99` | P99: Customer-defined process |

**`l10n_hu_edi_state`** — NAV 3.0 status

| Value | Label |
|---|---|
| `sent` | Sent, waiting for response |
| `send_timeout` | Timeout when sending |
| `confirmed` | Confirmed |
| `confirmed_warning` | Confirmed with warnings |
| `rejected` | Rejected |
| `cancel_sent` | Cancellation request sent |
| `cancel_timeout` | Timeout when requesting cancellation |
| `cancel_pending` | Cancellation request pending |
| `cancelled` | Cancelled |

**`l10n_hu_payment_mode`** — Payment mode

| Value | Label |
|---|---|
| `TRANSFER` | Transfer |
| `CASH` | Cash |
| `CARD` | Credit/debit card |
| `VOUCHER` | Voucher |
| `OTHER` | Other |

**`l10n_id_coretax_add_info_07`** — L10N Id Coretax Add Info 07

| Value | Label |
|---|---|
| `TD.00501` | 1 - untuk Kawasan Bebas |
| `TD.00502` | 2 - untuk Tempat Penimbunan Berikat |
| `TD.00503` | 3 - untuk Hibah dan Bantuan Luar Negeri |
| `TD.00504` | 4 - untuk Avtur |
| `TD.00505` | 5 - untuk Lainnya |
| `TD.00506` | 6 - untuk Kontraktor Perjanjian Karya Pengusahaan Pertambangan Batubara Generasi I |
| `TD.00507` | 7 - untuk Penyerahan bahan bakar minyak untuk Kapal Angkutan Laut Luar Negeri |
| `TD.00508` | 8 - untuk Penyerahan jasa kena pajak terkait alat angkutan tertentu |
| `TD.00509` | 9 - untuk Penyerahan BKP Tertentu di KEK |
| `TD.00510` | 10 - untuk BKP tertentu yang bersifat strategis berupa anode slime |
| `TD.00511` | 11 - untuk Penyerahan alat angkutan tertentu dan/atau Jasa Kena Pajak terkait alat angkutan tertentu |
| `TD.00512` | 12 - untuk Penyerahan kepada Kontraktor Kerja Sama Migas yang mengikuti ketentuan Peraturan Pemerintah Nomor 27 Tahun 2017 |
| `TD.00513` | 13 - Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2025 |
| `TD.00514` | 14 - Penyerahan Jasa Sewa Ruangan atau Bangunan Kepada Pedagang Eceran yang Ditanggung Pemerintah Tahun Anggaran 2021 |
| `TD.00515` | 15 - Penyerahan Barang dan Jasa Dalam Rangka Penanganan Pandemi COVID-19 (PMK 239/PMK. 03/2020) |
| `TD.00516` | 16 - Insentif PMK-103/PMK.010/2021 berupa PPN atas Penyerahan Rumah Tapak dan Unit Hunian Rumah Susun yang Ditanggung Pemerintah Tahun Anggaran 2021 |
| `TD.00517` | 17 - Kawasan Ekonomi Khusus PP nomor 40 Tahun 2021 |
| `TD.00518` | 18 - Kawasan Bebas PP nomor 41 Tahun 2021 |
| `TD.00519` | 19 - Penyerahan Rumah Tapak dan Unit Hunian Rumah Susun yang Ditanggung Pemerintah Tahun Anggaran 2022 |
| `TD.00520` | 20 - PPN Ditanggung Pemerintah dalam rangka Penanganan Pandemi Corona Virus |
| `TD.00521` | 21 - Penyerahan kepada Kontraktor Kerja Sama Migas yang mengikuti ketentuan Peraturan Pemerintah Nomor 53 Tahun 2017 |
| `TD.00522` | 22 - BKP strategis tertentu dalam bentuk anode slime dan emas butiran |
| `TD.00523` | 23 - untuk penyerahan kertas koran dan/atau majalah |
| `TD.00524` | 24 - PPN Ditanggung Pemerintah |
| `TD.00525` | 25 - BKP dan JKP tertentu |
| `TD.00526` | 26 - Penyerahan BKP dan JKP di Ibu Kota Negara baru |
| `TD.00527` | 27 - Penyerahan kendaraan listrik berbasis baterai |
| `TD.00528` | 28 - Insentif Tambahan Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2025 |
| `TD.00529` | 29 - PPN atas Penyerahan Hewan Khusus Tertentu Berupa Kuda serta Perlengkapan Pendukungnya Pemerintah Tahun Anggaran 2025 |
| `TD.00530` | 30 - PPN atas Penyerahan Bekal Khusus Operasi Tertentu Yang Ditanggung Pemerintah Tahun Anggaran 2025 |
| `TD.00531` | 31 - Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2026 |

**`l10n_id_coretax_add_info_08`** — L10N Id Coretax Add Info 08

| Value | Label |
|---|---|
| `TD.00501` | 1 - untuk BKP dan JKP Tertentu |
| `TD.00502` | 2 - untuk BKP Tertentu yang Bersifat Strategis |
| `TD.00503` | 3 - untuk Jasa Kebandarudaraan |
| `TD.00504` | 4 - untuk Lainnya |
| `TD.00505` | 5 - untuk BKP Tertentu yang Bersifat Strategis sesuai PP Nomor 81 Tahun 2015 |
| `TD.00506` | 6 - untuk Penyerahan Jasa Kepelabuhan Tertentu untuk kegiatan angkutan laut Luar Negeri |
| `TD.00507` | 7 - untuk Penyerahan Air Bersih |
| `TD.00508` | 8 - Penyerahan BKP tertentu yang bersifat strategis berdasarkan PP 48 Tahun 2020 |
| `TD.00509` | 9 - Penyerahan kepada Perwakilan Negara Asing dan Badan Internasional serta Pejabatnya |
| `TD.00510` | 10 - BKP dan JKP tertentu |

**`l10n_id_coretax_facility_info_07`** — L10N Id Coretax Facility Info 07

| Value | Label |
|---|---|
| `TD.01101` | 1 - Pajak Pertambahan Nilai Tidak Dipungut berdasarkan PP Nomor 10 Tahun 2012 |
| `TD.01102` | 2 - Pajak Pertambahan Nilai atau Pajak Pertambahan Nilai dan Pajak Penjualan atas Barang Mewah tidak dipungut |
| `TD.01103` | 3 - Pajak Pertambahan Nilai dan Pajak Penjualan atas Barang Mewah Tidak Dipungut |
| `TD.01104` | 4 - Pajak Pertambahan Nilai Tidak Dipungut Sesuai PP Nomor 71 Tahun 2012 |
| `TD.01105` | 5 - (Tidak ada Cap) |
| `TD.01106` | 6 - PPN dan/atau PPnBM tidak dipungut berdasarkan PMK No. 194/PMK.03/2012 |
| `TD.01107` | 7 - PPN Tidak Dipungut Berdasarkan PP Nomor 15 Tahun 2015 |
| `TD.01108` | 8 - PPN Tidak Dipungut Berdasarkan PP Nomor 69 Tahun 2015 |
| `TD.01109` | 9 - PPN Tidak Dipungut Berdasarkan PP Nomor 96 Tahun 2015 |
| `TD.01110` | 10 - PPN Tidak Dipungut Berdasarkan PP Nomor 106 Tahun 2015 |
| `TD.01111` | 11 - PPN Tidak Dipungut Sesuai PP Nomor 50 Tahun 2019 |
| `TD.01112` | 12 - PPN atau PPN dan PPnBM Tidak Dipungut Sesuai Dengan PP Nomor 27 Tahun 2017 |
| `TD.01113` | 13 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 13 TAHUN 2025 |
| `TD.01114` | 14 - PPN DITANGGUNG PEMERINTAH EKS PMK 102/PMK.010/2021 |
| `TD.01115` | 15 - PPN DITANGGUNG PEMERINTAH EKS PMK 239/PMK.03/2020 |
| `TD.01116` | 16 - Insentif PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 103/PMK.010/2021 |
| `TD.01117` | 17 - PAJAK PERTAMBAHAN NILAI TIDAK DIPUNGUT BERDASARKAN PP NOMOR 40 TAHUN 2021 |
| `TD.01118` | 18 - PAJAK PERTAMBAHAN NILAI TIDAK DIPUNGUT BERDASARKAN PP NOMOR 41 TAHUN 2021 |
| `TD.01119` | 19 - PPN DITANGGUNG PEMERINTAH EKS PMK 6/PMK.010/2022 |
| `TD.01120` | 20 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 226/PMK.03/2021 |
| `TD.01121` | 21 - PPN ATAU PPN DAN PPnBM TIDAK DIPUNGUT SESUAI DENGAN PP NOMOR 53 TAHUN 2017 |
| `TD.01122` | 22 - PPN tidak dipungut berdasarkan PP Nomor 70 Tahun 2021 |
| `TD.01123` | 23 - PPN ditanggung Pemerintah Ex PMK-125/PMK.01/2020 |
| `TD.01124` | 24 - (Tidak ada Cap) |
| `TD.01125` | 25 - PPN tidak dipungut berdasarkan PP Nomor 49 Tahun 2022 |
| `TD.01126` | 26 - PPN tidak dipungut berdasarkan PP Nomor 12 Tahun 2023 |
| `TD.01127` | 27 - PPN Ditanggung Pemerintah berdasarkan PMK Nomor 12 Tahun 2025 |
| `TD.01128` | 28 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 60 TAHUN 2025 |
| `TD.01129` | 29 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 61 TAHUN 2025 |
| `TD.01130` | 30 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 44 TAHUN 2025 |
| `TD.01131` | 31 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 90 TAHUN 2025 |

**`l10n_id_coretax_facility_info_08`** — L10N Id Coretax Facility Info 08

| Value | Label |
|---|---|
| `TD.01101` | 1 - PPN Dibebaskan Sesuai PP Nomor 146 Tahun 2000 Sebagaimana Telah Diubah Dengan PP Nomor 38 Tahun 2003 |
| `TD.01102` | 2 - PPN Dibebaskan Sesuai PP Nomor 12 Tahun 2001 Sebagaimana Telah Beberapa Kali Diubah Terakhir Dengan PP Nomor 31 Tahun 2007 |
| `TD.01103` | 3 - PPN dibebaskan berdasarkan Peraturan Pemerintah Nomor 28 Tahun 2009 |
| `TD.01104` | 4 - (Tidak ada cap) |
| `TD.01105` | 5 - PPN Dibebaskan Sesuai Dengan PP Nomor 81 Tahun 2015 |
| `TD.01106` | 6 - PPN Dibebaskan Berdasarkan PP Nomor 74 Tahun 2015 |
| `TD.01107` | 7 - (tanpa cap) |
| `TD.01108` | 8 - PPN DIBEBASKAN SESUAI PP NOMOR 81 TAHUN 2015 SEBAGAIMANA TELAH DIUBAH DENGAN PP 48 TAHUN 2020 |
| `TD.01109` | 9 - PPN DIBEBASKAN BERDASARKAN PP NOMOR 47 TAHUN 2020 |
| `TD.01110` | 10 - PPN Dibebaskan berdasarkan PP Nomor 49 Tahun 2022 |

**`l10n_id_kode_transaksi`** — Kode Transaksi

| Value | Label |
|---|---|
| `01` | 01 To the Parties that is not VAT Collector (Regular Customers) |
| `02` | 02 To the Treasurer |
| `03` | 03 To other VAT Collectors other than the Treasurer |
| `04` | 04 Other Value of VAT Imposition Base |
| `05` | 05 Specified Amount (Article 9A Paragraph (1) VAT Law) |
| `06` | 06 to individuals holding foreign passports |
| `07` | 07 Deliveries that the VAT is not Collected |
| `08` | 08 Deliveries that the VAT is Exempted |
| `09` | 09 Deliveries of Assets (Article 16D of VAT Law) |
| `10` | 10 Other deliveries |

**`l10n_in_edi_cancel_reason`** — E-Invoice(IN) Cancel Reason

| Value | Label |
|---|---|
| `1` | Duplicate |
| `2` | Data Entry Mistake |
| `3` | Order Cancelled |
| `4` | Others |

**`l10n_in_edi_status`** — India E-Invoice Status

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `cancelled` | Cancelled |

**`l10n_in_gst_treatment`** — GST Treatment

| Value | Label |
|---|---|
| `regular` | Registered Business - Regular |
| `composition` | Registered Business - Composition |
| `unregistered` | Unregistered Business |
| `consumer` | Consumer |
| `overseas` | Overseas |
| `special_economic_zone` | Special Economic Zone |
| `deemed_export` | Deemed Export |
| `uin_holders` | UIN Holders |

**`l10n_it_edi_state`** — SDI State

| Value | Label |
|---|---|
| `being_sent` | Being Sent To SdI |
| `requires_user_signature` | Requires user signature |
| `processing` | SdI Processing |
| `rejected` | SdI Rejected |
| `forwarded` | SdI Accepted, Forwarded to Partner |
| `forward_failed` | SdI Accepted, Forward to Partner Failed |
| `forward_attempt` | SdI Accepted, Forwarding to Partner |
| `accepted_by_pa_partner` | SdI Accepted, Accepted by the PA Partner |
| `rejected_by_pa_partner` | SdI Accepted, Rejected by the PA Partner |
| `accepted_by_pa_partner_after_expiry` | SdI Accepted, PA Partner Expired Terms |

**`l10n_it_origin_document_type`** — Origin Document Type

| Value | Label |
|---|---|
| `purchase_order` | Purchase Order |
| `contract` | Contract |
| `agreement` | Agreement |

**`l10n_it_payment_method`** — L10N It Payment Method

| Value | Label |
|---|---|
| `MP01` | MP01 - Cash |
| `MP02` | MP02 - Check |
| `MP03` | MP03 - Cashier's check |
| `MP04` | MP04 - Cash at the Treasury |
| `MP05` | MP05 - Wire transfer |
| `MP06` | MP06 - Promissory note |
| `MP07` | MP07 - Bank slip |
| `MP08` | MP08 - Payment card |
| `MP09` | MP09 - RID |
| `MP10` | MP10 - RID users |
| `MP11` | MP11 - Fast RID |
| `MP12` | MP12 - RIBA |
| `MP13` | MP13 - MAV |
| `MP14` | MP14 - Treasury receipt |
| `MP15` | MP15 - Transfer of special accounting accounts |
| `MP16` | MP16 - Bank direct debit |
| `MP17` | MP17 - Postal domiciliation |
| `MP18` | MP18 - Postal account slip |
| `MP19` | MP19 - SEPA Direct Debit |
| `MP20` | MP20 - SEPA Direct Debit CORE |
| `MP21` | MP21 - SEPA Direct Debit B2B |
| `MP22` | MP22 - Withholding from sums already collected |
| `MP23` | MP23 - PagoPA |

**`l10n_jo_edi_invoice_type`** — Invoice Type

| Value | Label |
|---|---|
| `local` | Local |
| `export` | Export |
| `development` | Development Area |
| `transit` | Transit |
| `foreign` | Foreign Trade |
| `freezone` | Free Zone Transfer |

**`l10n_jo_edi_state`** — JoFotara State

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `demo` | Sent (Demo) |

**`l10n_my_edi_state`** — MyInvois State

| Value | Label |
|---|---|
| `in_progress` | Validation In Progress |
| `valid` | Valid |
| `rejected` | Rejected |
| `invalid` | Invalid |
| `cancelled` | Cancelled |

**`l10n_pl_edi_status`** — KSeF Status

| Value | Label |
|---|---|
| `sent` | Sent (In Progress) |
| `accepted` | Accepted |
| `rejected` | Rejected |
| `fetch_ready` | Fetch Ready |
| `fetched` | Fetched |
| `fetch_failed` | Fetch Failed |

**`l10n_ro_edi_state`** — E-Factura Status

| Value | Label |
|---|---|
| `invoice_not_indexed` | Not indexed |
| `invoice_sent` | Sent |
| `invoice_refused` | Refused |
| `invoice_validated` | Validated |

**`l10n_rs_edi_state`** — Serbia E-Invoice state

| Value | Label |
|---|---|
| `sent` | Sent |
| `sending_failed` | Error |

**`l10n_rs_tax_date_obligations_code`** — Tax Date Obligations

| Value | Label |
|---|---|
| `35` | By Delivery Date |
| `3` | By Issuance Date |
| `432` | By Billing System |

**`l10n_sa_reason`** — ZATCA Reason

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | Cancellation or suspension of the supplies after its occurrence either wholly or partially |
| `BR-KSA-17-reason-2` | In case of essential change or amendment in the supply, which leads to the change of the VAT due |
| `BR-KSA-17-reason-3` | Amendment of the supply value which is pre-agreed upon between the supplier and consumer |
| `BR-KSA-17-reason-4` | In case of goods or services refund |
| `BR-KSA-17-reason-5` | In case of change in Seller's or Buyer's information |

**`l10n_tr_gib_invoice_scenario`** — Invoice Scenario

| Value | Label |
|---|---|
| `TEMELFATURA` | Basic |
| `KAMU` | Public Sector |

**`l10n_tr_gib_invoice_type`** — GIB Invoice Type

| Value | Label |
|---|---|
| `SATIS` | Sales |
| `TEVKIFAT` | Withholding |
| `IHRACKAYITLI` | Registered for Export |
| `ISTISNA` | Tax Exempt |

**`l10n_tr_nilvera_send_status`** — Nilvera Status

| Value | Label |
|---|---|
| `error` | Error |
| `not_sent` | Not sent |
| `sent` | Sent and waiting response |
| `succeed` | Successful |
| `waiting` | Waiting |
| `unknown` | Unknown |

**`l10n_tr_shipping_type`** — Shipping Method

| Value | Label |
|---|---|
| `1` | Sea Transportation |
| `2` | Railway Transportation |
| `3` | Road Transportation |
| `4` | Air Transportation |
| `5` | Post |
| `6` | Combined Transportation |
| `7` | Fixed Transportation |
| `8` | Domestic Water Transportation |
| `9` | Invalid Transportation Method |

**`l10n_tw_edi_allowance_notify_way`** — Allowance Notify Way

| Value | Label |
|---|---|
| `email` | Email |
| `phone` | Phone |

**`l10n_tw_edi_carrier_type`** — Carrier Type

| Value | Label |
|---|---|
| `1` | ECpay e-invoice carrier |
| `2` | Citizen Digital Certificate |
| `3` | Mobile Barcode |
| `4` | EasyCard |
| `5` | iPass |

**`l10n_tw_edi_clearance_mark`** — Clearance Mark

| Value | Label |
|---|---|
| `1` | NOT via the customs |
| `2` | Via the customs |

**`l10n_tw_edi_invoice_type`** — Ecpay Invoice Type

| Value | Label |
|---|---|
| `07` | General Invoice |
| `08` | Special Invoice |

**`l10n_tw_edi_refund_agreement_type`** — Refund invoice Agreement Type

| Value | Label |
|---|---|
| `offline` | Offline Agreement |
| `online` | Online Agreement |

**`l10n_tw_edi_refund_state`** — Refund State

| Value | Label |
|---|---|
| `to_be_agreed` | To be agreed |
| `agreed` | Agreed |
| `disagreed` | Disagreed |

**`l10n_tw_edi_state`** — Invoice Status

| Value | Label |
|---|---|
| `invoiced` | Invoiced |
| `valid` | Valid |
| `invalid` | Invalid |

**`l10n_tw_edi_zero_tax_rate_reason`** — Zero Tax Rate Reason

| Value | Label |
|---|---|
| `71` | 71: No.1 export goods |
| `72` | 72: No.2 Services related to export sales, or services provided domestically but used abroad |
| `73` | 73: No.3 Duty-free shops established by law for the sale and transit or departure of passengers |
| `74` | 74: No.4 Sale of goods or services for operation by the operator of the FREE Trade Zone |
| `75` | 75: No.5 International transportation. However, foreign transport undertakings operating international transport business in Taiwan shall be limited to those whose countries shall give equal treatment to Taiwan's international transport undertakings or be exempt from similar taxes |
| `76` | 76: No.6 Ships, aircraft and distant-water fishing vessels for international transportation |
| `77` | 77: No.7 Goods or repair services used by ships, aircraft and distant-water fishing vessels for sale and international transport |
| `78` | 78: No.8 The bonded area operator sells goods that are not directly exported by the taxable area operator and the taxable area operator is not exported to the taxation area |
| `79` | 79: No.9 The bonded area operator sells the goods that the taxable area operator deposits into the bonded warehouse or logistics center managed by the free port area or customs administration for export |

**`l10n_vn_edi_adjustment_type`** — Adjustment type

| Value | Label |
|---|---|
| `1` | Money adjustment |
| `2` | Information adjustment |

**`l10n_vn_edi_invoice_state`** — Sinvoice Status

| Value | Label |
|---|---|
| `ready_to_send` | Ready to send |
| `sent` | Sent |
| `payment_state_to_update` | Payment status to update |
| `canceled` | Canceled |
| `adjusted` | Adjusted |
| `replaced` | Replaced |

**`move_sent_values`** — Sent (not stored)

| Value | Label |
|---|---|
| `sent` | Sent |
| `not_sent` | Not Sent |

**`move_type`** — Type

| Value | Label |
|---|---|
| `entry` | Journal Entry |
| `out_invoice` | Customer Invoice |
| `out_refund` | Customer Credit Note |
| `in_invoice` | Vendor Bill |
| `in_refund` | Vendor Credit Note |
| `out_receipt` | Sales Receipt |
| `in_receipt` | Purchase Receipt |

**`nemhandel_move_state`** — Nemhandel status

| Value | Label |
|---|---|
| `ready` | Ready to send |
| `to_send` | Queued |
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `BusinessAccept` | Approved |
| `BusinessReject` | Rejected |

**`payment_state`** — Payment Status

| Value | Label |
|---|---|
| `not_paid` | Not Paid |
| `in_payment` | In Payment |
| `paid` | Paid |
| `partial` | Partially Paid |
| `reversed` | Reversed |
| `blocked` | Blocked |
| `invoicing_legacy` | Invoicing App Legacy |

**`pdp_ppf_lifecycle_state`** — PPF Lifeycle Status

| Value | Label |
|---|---|
| `in_progress` | In Progress |
| `sent` | Sent |
| `done` | Done |
| `error` | Error |

**`pdp_ppf_move_state`** — PPF Invoice Status

| Value | Label |
|---|---|
| `in_progress` | In Progress |
| `sent` | Sent |
| `done` | Done |
| `error` | Error |

**`peppol_move_state`** — E-Invoicing Status

| Value | Label |
|---|---|
| `ready` | Ready to send |
| `to_send` | Queued |
| `skipped` | Skipped |
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `AB` | Received |
| `AP` | Approved |
| `RE` | Rejected |
| `PD` | With Payments |
| `submitted` | Submitted |
| `made_available` | Made Available |
| `refused` | Refused |
| `cancelled` | Cancelled |
| `sent` | Sent |
| `suspended` | Suspended |
| `completed` | Completed |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `posted` | Posted |
| `cancel` | Cancelled |

**`status_in_payment`** — Status In Payment (not stored)

| Value | Label |
|---|---|
| `not_paid` | Not Paid |
| `in_payment` | In Payment |
| `paid` | Paid |
| `partial` | Partially Paid |
| `reversed` | Reversed |
| `blocked` | Blocked |
| `invoicing_legacy` | Invoicing App Legacy |
| `draft` | Draft |
| `posted` | Posted |
| `sent` | Sent |
| `cancel` | Cancelled |

## `account.move.line`

**`display_type`** — Display Type

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

**`l10n_gr_edi_cls_category`** — myDATA Category

| Value | Label |
|---|---|
| `category1_1` | 1.1 - Commodity Sale Income |
| `category1_2` | 1.2 - Product Sale Income |
| `category1_3` | 1.3 - Provision of Services Income |
| `category1_4` | 1.4 - Sale of Fixed Assets Income |
| `category1_5` | 1.5 - Other Income/Profits |
| `category1_6` | 1.6 - Self-Deliveries/Self-Supplies |
| `category1_7` | 1.7 - Income on behalf of Third Parties |
| `category1_8` | 1.8 - Past fiscal years income |
| `category1_9` | 1.9 - Future fiscal years income |
| `category1_10` | 1.10 - Other Income Adjustment/Regularisation Entries |
| `category1_95` | 1.95 - Other Income-related Information |
| `category2_1` | 2.1 - Commodity Purchases |
| `category2_2` | 2.2 - Raw and Adjuvant Material Purchases |
| `category2_3` | 2.3 - Services Receipt |
| `category2_4` | 2.4 - General Expenses Subject to VAT Deduction |
| `category2_5` | 2.5 - General Expenses Not Subject to VAT Deduction |
| `category2_6` | 2.6 - Personnel Fees and Benefits |
| `category2_7` | 2.7 - Fixed Asset Purchases |
| `category2_8` | 2.8 - Fixed Asset Amortisations |
| `category2_9` | 2.9 - Expenses on behalf of Third Parties |
| `category2_10` | 2.10 - Past fiscal years expenses |
| `category2_11` | 2.11 - Future fiscal years expenses |
| `category2_12` | 2.12 - Other Expense Adjustment/Regularisation Entries |
| `category2_13` | 2.13 - Stock at Period Start |
| `category2_14` | 2.14 - Stock at Period End |
| `category2_95` | 2.95 - Other Expense-related Information |

**`l10n_gr_edi_cls_type`** — myDATA Type

| Value | Label |
|---|---|
| `E3_106` | E3_106 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Commodities |
| `E3_205` | E3_205 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials |
| `E3_210` | E3_210 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress |
| `E3_305` | E3_305 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials |
| `E3_310` | E3_310 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress |
| `E3_318` | E3_318 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Production expenses |
| `E3_561_001` | E3_561_001 - Wholesale Sales of Goods and Services – for Traders |
| `E3_561_002` | E3_561_002 - Wholesale Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000) |
| `E3_561_003` | E3_561_003 - Retail Sales of Goods and Services – Private Clientele |
| `E3_561_004` | E3_561_004 - Retail Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000) |
| `E3_561_005` | E3_561_005 - Intra-Community Foreign Sales of Goods and Services |
| `E3_561_006` | E3_561_006 - Third Country Foreign Sales of Goods and Services |
| `E3_561_007` | E3_561_007 - Other Sales of Goods and Services |
| `E3_562` | E3_562 - Other Ordinary Income |
| `E3_563` | E3_563 - Credit Interest and Related Income |
| `E3_564` | E3_564 - Credit Exchange Differences |
| `E3_565` | E3_565 - Income from Participations |
| `E3_566` | E3_566 - Profits from Disposing Non-Current Assets |
| `E3_567` | E3_567 - Profits from the Reversal of Provisions and Impairments |
| `E3_568` | E3_568 - Profits from Measurement at Fair Value |
| `E3_570` | E3_570 - Extraordinary income and profits |
| `E3_595` | E3_595 - Self-Production Expenses |
| `E3_596` | E3_596 - Subsidies - Grants |
| `E3_597` | E3_597 - Subsidies – Grants for Investment Purposes – Expense Coverage |
| `E3_880_001` | E3_880_001 - Wholesale Sales of Fixed Assets |
| `E3_880_002` | E3_880_002 - Retail Sales of Fixed Assets |
| `E3_880_003` | E3_880_003 - Intra-Community Foreign Sales of Fixed Assets |
| `E3_880_004` | E3_880_004 - Third Country Foreign Sales of Fixed Assets |
| `E3_881_001` | E3_881_001 - Wholesale Sales on behalf of Third Parties |
| `E3_881_002` | E3_881_002 - Retail Sales on behalf of Third Parties |
| `E3_881_003` | E3_881_003 - Intra-Community Foreign Sales on behalf of Third Parties |
| `E3_881_004` | E3_881_004 - Third Country Foreign Sales on behalf of Third Parties |
| `E3_598_001` | E3_598_001 - Sales of goods belonging to excise duty |
| `E3_598_003` | E3_598_003 - Sales on behalf of farmers through an agricultural cooperative e.t.c. |
| `E3_101` | E3_101 - Commodities at Period Start |
| `E3_102_001` | E3_102_001 - Fiscal Year Commodity Purchases (net amount)/Wholesale |
| `E3_102_002` | E3_102_002 - Fiscal Year Commodity Purchases (net amount)/Retail |
| `E3_102_003` | E3_102_003 - Fiscal Year Commodity Purchases (net amount)/Goods under article 39a paragraph 5 of the VAT Code (Law 2859/2000) |
| `E3_102_004` | E3_102_004 - Fiscal Year Commodity Purchases (net amount)/Foreign, Intra-Community |
| `E3_102_005` | E3_102_005 - Fiscal Year Commodity Purchases (net amount)/Foreign, Third Countries |
| `E3_102_006` | E3_102_006 - Fiscal Year Commodity Purchases (net amount)/Others |
| `E3_104` | E3_104 - Commodities at Period End |
| `E3_201` | E3_201 - Raw and Other Materials at Period Start/Production |
| `E3_202_001` | E3_202_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale |
| `E3_202_002` | E3_202_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail |
| `E3_202_003` | E3_202_003 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Intra-Community |
| `E3_202_004` | E3_202_004 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Third Countries |
| `E3_202_005` | E3_202_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others |
| `E3_204` | E3_204 - Raw and Other Material Stock at Period End/Production |
| `E3_207` | E3_207 - Products and Production in Progress at Period Start/Production |
| `E3_209` | E3_209 - Products and Production in Progress at Period End/Production |
| `E3_301` | E3_301 - Raw and Other Material at Period Start/Agricultural |
| `E3_302_001` | E3_302_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale |
| `E3_302_002` | E3_302_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail |
| `E3_302_003` | E3_302_003 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Intra-Community |
| `E3_302_004` | E3_302_004 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Third Countries |
| `E3_302_005` | E3_302_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others |
| `E3_304` | E3_304 - Raw and Other Material Stock at Period End/Agricultural |
| `E3_307` | E3_307 - Products and Production in Progress at Period Start/ Agricultural |
| `E3_309` | E3_309 - Products and Production in Progress at Period End/ Agricultural |
| `E3_312` | E3_312 - Stock at Period Start (Animals-Plants) |
| `E3_313_001` | E3_313_001 - Animal-Plant Purchases (net amount)/Wholesale |
| `E3_313_002` | E3_313_002 - Animal-Plant Purchases (net amount)/Retail |
| `E3_313_003` | E3_313_003 - Animal-Plant Purchases (net amount)/ Foreign, Intra-Community |
| `E3_313_004` | E3_313_004 - Animal-Plant Purchases (net amount)/ Foreign, Third Countries |
| `E3_313_005` | E3_313_005 - Animal-Plant Purchases/Others |
| `E3_315` | E3_315 - Stock at Period End (Animals-Plants)/Agricultural |
| `E3_581_001` | E3_581_001 - Employee Benefits/Gross Earnings |
| `E3_581_002` | E3_581_002 - Employee Benefits/Employer Contributions |
| `E3_581_003` | E3_581_003 - Employee Benefits/Other Benefits |
| `E3_582` | E3_582 - Asset Measurement Damages |
| `E3_583` | E3_583 - Debit Exchange Differences |
| `E3_584` | E3_584 - Damages from Disposing-Withdrawing Non-Current Assets |
| `E3_585_001` | E3_585_001 - Foreign/Domestic Management Fees |
| `E3_585_002` | E3_585_002 - Expenditures from Linked Enterprises |
| `E3_585_003` | E3_585_003 - Expenditures from Non-Cooperative States or Privileged Tax Regimes |
| `E3_585_004` | E3_585_004 - Expenditures for Information Day-Events |
| `E3_585_005` | E3_585_005 - Reception and Hospitality Expenses |
| `E3_585_006` | E3_585_006 - Travel expenses |
| `E3_585_007` | E3_585_007 - Self-Employed Social Security Contributions |
| `E3_585_008` | E3_585_008 - Commission Agent Expenses and Fees on behalf of Farmers |
| `E3_585_009` | E3_585_009 - Other Fees for Domestic Services |
| `E3_585_010` | E3_585_010 - Other Fees for Foreign Services |
| `E3_585_011` | E3_585_011 - Energy |
| `E3_585_012` | E3_585_012 - Water |
| `E3_585_013` | E3_585_013 - Telecommunications |
| `E3_585_014` | E3_585_014 - Rents |
| `E3_585_015` | E3_585_015 - Advertisement and promotion |
| `E3_585_016` | E3_585_016 - Other expenses |
| `E3_586` | E3_586 - Debit interests and related expenses |
| `E3_587` | E3_587 - Amortisations |
| `E3_588` | E3_588 - Extraordinary expenses, damages and fines |
| `E3_589` | E3_589 - Provisions (except for Personnel Provisions) |
| `E3_882_001` | E3_882_001 - Fiscal Year Tangible Asset Purchases/Wholesale |
| `E3_882_002` | E3_882_002 - Fiscal Year Tangible Asset Purchases/Retail |
| `E3_882_003` | E3_882_003 - Fiscal Year Tangible Asset Purchases/ Intra-Community Foreign |
| `E3_882_004` | E3_882_004 - Fiscal Year Tangible Asset Purchases/ Third Country Foreign |
| `E3_883_001` | E3_883_001 - Fiscal Year Intangible Asset Purchases/Wholesale |
| `E3_883_002` | E3_883_002 - Fiscal Year Intangible Asset Purchases/Retail |
| `E3_883_003` | E3_883_003 - Fiscal Year Intangible Asset Purchases/ Intra-Community Foreign |
| `E3_883_004` | E3_883_004 - Fiscal Year Intangible Asset Purchases/ Third Country Foreign |
| `E3_103` | E3_103 - Impairment of goods |
| `E3_203` | E3_203 - Impairment of raw materials and supplies |
| `E3_303` | E3_303 - Impairment of raw materials and supplies |
| `E3_208` | E3_208 - Impairment of products and production in progress |
| `E3_308` | E3_308 - Impairment of products and production in progress |
| `E3_314` | E3_314 - Impairment of animals-plants - goods |
| `XE3_106` | E3_106 - Own production of fixed assets – Self Deliveries – Inventory Disasters |
| `XE3_205` | E3_205 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_305` | E3_305 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_210` | E3_210 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_310` | E3_310 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_318` | E3_318 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `E3_598_002` | E3_598_002 - Purchases of goods falling into excise duty |

**`l10n_gr_edi_cls_vat`** — VAT Classification

| Value | Label |
|---|---|
| `VAT_361` | VAT_361 - Domestic Purchases & Expenditures |
| `VAT_362` | VAT_362 - Purchases & Imports of Investment Goods (Fixed Assets) |
| `VAT_363` | VAT_363 - Other Imports except for Investment Goods (Fixed Assets) |
| `VAT_364` | VAT_364 - Intra-Community Goods Acquisitions |
| `VAT_365` | VAT_365 - Intra-Community Services Receipts per article 14.2.a |
| `VAT_366` | VAT_366 - Other Recipient Actions |

**`l10n_gr_edi_detail_type`** — Detail Type

| Value | Label |
|---|---|
| `1` | 1 |
| `2` | 2 |

**`l10n_gr_edi_tax_exemption_category`** — Tax Exemption Category

| Value | Label |
|---|---|
| `1` | 1 - Without VAT - article 3 of the VAT code |
| `2` | 2 - Without VAT - article 5 of the VAT code |
| `3` | 3 - Without VAT - article 13 of the VAT code |
| `4` | 4 - Without VAT - article 14 of the VAT code |
| `5` | 5 - Without VAT - article 16 of the VAT code |
| `6` | 6 - Without VAT - article 19 of the VAT code |
| `7` | 7 - Without VAT - article 22 of the VAT code |
| `8` | 8 - Without VAT - article 24 of the VAT code |
| `9` | 9 - Without VAT - article 25 of the VAT code |
| `10` | 10 - Without VAT - article 26 of the VAT code |
| `11` | 11 - Without VAT - article 27 of the VAT code |
| `12` | 12 - Without VAT - article 27 - Seagoing Vessels of the VAT code |
| `13` | 13 - Without VAT - article 27.1.γ - Seagoing Vessels of the VAT code |
| `14` | 14 - Without VAT - article 28 of the VAT code |
| `15` | 15 - Without VAT - article 39 of the VAT code |
| `16` | 16 - Without VAT - article 39a of the VAT code |
| `17` | 17 - Without VAT - article 40 of the VAT code |
| `18` | 18 - Without VAT - article 41 of the VAT code |
| `19` | 19 - Without VAT - article 47 of the VAT code |
| `20` | 20 - VAT included - article 43 of the VAT code |
| `21` | 21 - VAT included - article 44 of the VAT code |
| `22` | 22 - VAT included - article 45 of the VAT code |
| `23` | 23 - VAT included - article 46 of the VAT code |
| `24` | 24 - Without VAT - article 6 of the VAT code |
| `25` | 25 - Without VAT - ΠΟΛ.1029 / 1995 |
| `26` | 26 - Without VAT - ΠΟΛ.1167 / 2015 |
| `27` | 27 - Without VAT – Other VAT exceptions |
| `28` | 28 - Without VAT - Article 24 (b)(1) of the VAT Code(Tax Free) |
| `29` | 29 - Without VAT - Article 47 b of the VAT Code(OSS non - EU scheme) |
| `30` | 30 - Without VAT - Article 47 c of the VAT Code(OSS EU scheme) |
| `31` | 31 - Excluding VAT - Article 47 d of the VAT Code(IOSS) |

**`l10n_in_gstr_section`** — GSTR Section

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

**`l10n_my_edi_classification_code`** — Malaysian classification code

| Value | Label |
|---|---|
| `001` | (001) Breastfeeding equipment  |
| `002` | (002) Child care centres and kindergartens fees |
| `003` | (003) Computer, smartphone or tablet |
| `004` | (004) Consolidated e-Invoice  |
| `005` | (005) Construction materials (as specified under Fourth Schedule of the Lembaga Pembangunan Industri Pembinaan Malaysia Act 1994) |
| `006` | (006) Disbursement |
| `007` | (007) Donation |
| `008` | (008) -Commerce - e-Invoice to buyer / purchaser |
| `009` | (009) e-Commerce - Self-billed e-Invoice to seller, logistics, etc.  |
| `010` | (010) Education fees |
| `011` | (011) Goods on consignment (Consignor) |
| `012` | (012) Goods on consignment (Consignee) |
| `013` | (013) Gym membership |
| `014` | (014) Insurance - Education and medical benefits |
| `015` | (015) Insurance - Takaful or life insurance |
| `016` | (016) Interest and financing expenses |
| `017` | (017) Internet subscription |
| `018` | (018) Land and building |
| `019` | (019) Medical examination for learning disabilities and early intervention or rehabilitation treatments of learning disabilities |
| `020` | (020) Medical examination or vaccination expenses |
| `021` | (021) Medical expenses for serious diseases |
| `022` | (022) Others |
| `023` | (023) Petroleum operations (as defined in Petroleum (Income Tax) Act 1967) |
| `024` | (024) Private retirement scheme or deferred annuity scheme |
| `025` | (025) Motor vehicle |
| `026` | (026) Subscription of books / journals / magazines / newspapers / other similar publications |
| `027` | (027) Reimbursement |
| `028` | (028) Rental of motor vehicle |
| `029` | (029) EV charging facilities (Installation, rental, sale / purchase or subscription fees)  |
| `030` | (030) Repair and maintenance |
| `031` | (031) Research and development |
| `032` | (032) Foreign income |
| `033` | (033) Self-billed - Betting and gaming |
| `034` | (034) Self-billed - Importation of goods |
| `035` | (035) Self-billed - Importation of services |
| `036` | (036) Self-billed - Others |
| `037` | (037) Self-billed - Monetary payment to agents, dealers or distributors |
| `038` | (038) Fees related to sports equipment, facility rentals, competition registration, and training imposed by registered sports organizations under the Sports Development Act 1997 |
| `039` | (039) Supporting equipment for disabled person |
| `040` | (040) Voluntary contribution to approved provident fund  |
| `041` | (041) Dental examination or treatment |
| `042` | (042) Fertility treatment |
| `043` | (043) Treatment and home care nursing, daycare centres and residential care centers |
| `044` | (044) Vouchers, gift cards, loyalty points, etc |
| `045` | (045) Self-billed - Non-monetary payment to agents, dealers or distributors |

## `account.move.reversal`

**`l10n_es_edi_verifactu_refund_reason`** — Veri*Factu Refund Reason

| Value | Label |
|---|---|
| `R1` | R1: Art 80.1 and 80.2 and error of law |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Rest |
| `R5` | R5: Corrective invoices concerning simplified invoices |

**`l10n_es_tbai_refund_reason`** — Invoice Refund Reason Code (TicketBai)

| Value | Label |
|---|---|
| `R1` | R1: Art. 80.1, 80.2, 80.6 and rights founded error |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Art. 80 - other |
| `R5` | R5: Factura rectificativa en facturas simplificadas |

**`l10n_sa_reason`** — ZATCA Reason

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | Cancellation or suspension of the supplies after its occurrence either wholly or partially |
| `BR-KSA-17-reason-2` | In case of essential change or amendment in the supply, which leads to the change of the VAT due |
| `BR-KSA-17-reason-3` | Amendment of the supply value which is pre-agreed upon between the supplier and consumer |
| `BR-KSA-17-reason-4` | In case of goods or services refund |
| `BR-KSA-17-reason-5` | In case of change in Seller's or Buyer's information |

**`l10n_tw_edi_allowance_notify_way`** — Allowance Notify Way

| Value | Label |
|---|---|
| `email` | Email |
| `phone` | Phone |

**`l10n_tw_edi_refund_agreement_type`** — Agreement Type

| Value | Label |
|---|---|
| `offline` | Offline |
| `online` | Online |

**`l10n_vn_edi_adjustment_type`** — Adjustment type

| Value | Label |
|---|---|
| `1` | Money adjustment |
| `2` | Information adjustment |

## `account.payment`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`partner_type`** — Partner Type

| Value | Label |
|---|---|
| `customer` | Customer |
| `supplier` | Vendor |

**`payment_type`** — Payment Type

| Value | Label |
|---|---|
| `outbound` | Send |
| `inbound` | Receive |

**`reconciled_invoices_type`** — Reconciled Invoices Type (not stored)

| Value | Label |
|---|---|
| `credit_note` | Credit Note |
| `invoice` | Invoice |

**`state`** — State

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_process` | In Process |
| `paid` | Paid |
| `canceled` | Canceled |
| `rejected` | Rejected |

## `account.payment.method`

**`payment_type`** — Payment Type

| Value | Label |
|---|---|
| `inbound` | Inbound |
| `outbound` | Outbound |

## `account.payment.method.line`

**`l10n_it_payment_method`** — Italian Payment Method

| Value | Label |
|---|---|
| `MP01` | MP01 - Cash |
| `MP02` | MP02 - Check |
| `MP03` | MP03 - Cashier's check |
| `MP04` | MP04 - Cash at the Treasury |
| `MP05` | MP05 - Wire transfer |
| `MP06` | MP06 - Promissory note |
| `MP07` | MP07 - Bank slip |
| `MP08` | MP08 - Payment card |
| `MP09` | MP09 - RID |
| `MP10` | MP10 - RID users |
| `MP11` | MP11 - Fast RID |
| `MP12` | MP12 - RIBA |
| `MP13` | MP13 - MAV |
| `MP14` | MP14 - Treasury receipt |
| `MP15` | MP15 - Transfer of special accounting accounts |
| `MP16` | MP16 - Bank direct debit |
| `MP17` | MP17 - Postal domiciliation |
| `MP18` | MP18 - Postal account slip |
| `MP19` | MP19 - SEPA Direct Debit |
| `MP20` | MP20 - SEPA Direct Debit CORE |
| `MP21` | MP21 - SEPA Direct Debit B2B |
| `MP22` | MP22 - Withholding from sums already collected |
| `MP23` | MP23 - PagoPA |

## `account.payment.register`

**`installments_mode`** — Installments Mode

| Value | Label |
|---|---|
| `next` | Next Installment |
| `overdue` | Overdue Amount |
| `before_date` | Before Next Payment Date |
| `full` | Full Amount |

**`partner_type`** — Partner Type

| Value | Label |
|---|---|
| `customer` | Customer |
| `supplier` | Vendor |

**`payment_difference_handling`** — Payment Difference Handling

| Value | Label |
|---|---|
| `open` | Keep open |
| `reconcile` | Mark as fully paid |

**`payment_type`** — Payment Type

| Value | Label |
|---|---|
| `outbound` | Send Money |
| `inbound` | Receive Money |

## `account.payment.register.withholding.line`

**`comodel_payment_type`** — Comodel Payment Type (not stored)

| Value | Label |
|---|---|
| `outbound` | Send Money |
| `inbound` | Receive Money |

**`placeholder_type`** — Placeholder Type

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

**`previous_placeholder_type`** — Previous Placeholder Type

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

## `account.payment.term`

**`early_pay_discount_computation`** — Cash Discount Tax Reduction

| Value | Label |
|---|---|
| `included` | On early payment |
| `excluded` | Never |
| `mixed` | Always (upon invoice) |

## `account.payment.term.line`

**`delay_type`** — Delay Type

| Value | Label |
|---|---|
| `days_after` | Days after invoice date |
| `days_after_end_of_month` | Days after end of month |
| `days_after_end_of_next_month` | Days after end of next month |
| `days_end_of_month_on_the` | Days end of month on the |

**`value`** — Value

| Value | Label |
|---|---|
| `percent` | Percent |
| `fixed` | Fixed |

## `account.payment.withholding.line`

**`comodel_payment_type`** — Comodel Payment Type (not stored)

| Value | Label |
|---|---|
| `outbound` | Send Money |
| `inbound` | Receive Money |

**`placeholder_type`** — Placeholder Type

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

**`previous_placeholder_type`** — Previous Placeholder Type

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

## `account.peppol.clarification`

**`list_identifier`** — List identifier

| Value | Label |
|---|---|
| `OPStatusReason` | OPStatusReason |
| `OPStatusAction` | OPStatusAction |

## `account.peppol.response`

**`pdp_flow_number`** — Flow Number

| Value | Label |
|---|---|
| `1` | Tax Extract |
| `2` | Status |
| `6` | Mandatory Status |
| `10` | Report |

**`pdp_ppf_state`** — PPF Status

| Value | Label |
|---|---|
| `sent` | Sent |
| `received` | received |
| `error` | Error |

**`pdp_ref_response_code`** — Original Response Code

| Value | Label |
|---|---|
| `AB` | Received |
| `AP` | Approved |
| `PD` | Payment |
| `RE` | Rejected |
| `submitted` | Submitted |
| `made_available` | Made Available |
| `refused` | Refused |
| `cancelled` | Cancelled |
| `sent` | Sent |
| `suspended` | Suspended |
| `completed` | Completed |

**`peppol_state`** — Peppol status

| Value | Label |
|---|---|
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `not_serviced` | Not Serviced |

**`response_code`** — Response Code

| Value | Label |
|---|---|
| `AB` | Acknowledgement |
| `IP` | In Process |
| `UQ` | Under query |
| `CA` | Conditionally accepted |
| `RE` | Rejection |
| `AP` | Approval |
| `PD` | Paid |
| `submitted` | Submitted |
| `made_available` | Made Available |
| `refused` | Refused |
| `cancelled` | Cancelled |
| `sent` | Sent |
| `suspended` | Suspended |
| `completed` | Completed |

## `account.reconcile.model`

**`match_amount`** — Amount

| Value | Label |
|---|---|
| `lower` | Is lower than or equal to |
| `greater` | Is greater than or equal to |
| `between` | Is between |

**`match_label`** — Label

| Value | Label |
|---|---|
| `contains` | Contains |
| `not_contains` | Not Contains |
| `match_regex` | Match Regex |

**`trigger`** — Trigger

| Value | Label |
|---|---|
| `manual` | Manual |
| `auto_reconcile` | Automated |

## `account.reconcile.model.line`

**`amount_type`** — Amount Type

| Value | Label |
|---|---|
| `fixed` | Fixed |
| `percentage` | Percentage of balance |
| `percentage_st_line` | Percentage of statement line |
| `regex` | From label |

## `account.report`

**`availability_condition`** — Availability

| Value | Label |
|---|---|
| `country` | Country Matches |
| `coa` | Chart of Accounts Matches |
| `always` | Always |

**`currency_translation`** — Currency Translation

| Value | Label |
|---|---|
| `current` | Use the most recent rate at the date of the report |
| `cta` | Use CTA |

**`default_opening_date_filter`** — Default Opening

| Value | Label |
|---|---|
| `this_year` | This Year |
| `this_quarter` | This Quarter |
| `this_month` | This Month |
| `today` | Today |
| `previous_month` | Last Month |
| `previous_quarter` | Last Quarter |
| `previous_year` | Last Year |
| `this_return_period` | This Return Period |
| `previous_return_period` | Last Return Period |

**`filter_account_type`** — Account Types

| Value | Label |
|---|---|
| `both` | Payable and receivable |
| `payable` | Payable |
| `receivable` | Receivable |
| `disabled` | Disabled |

**`filter_hide_0_lines`** — Hide lines at 0

| Value | Label |
|---|---|
| `by_default` | Enabled by Default |
| `optional` | Optional |
| `never` | Never |

**`filter_hierarchy`** — Account Groups

| Value | Label |
|---|---|
| `by_default` | Enabled by Default |
| `optional` | Optional |
| `never` | Never |

**`filter_multi_company`** — Multi-Company

| Value | Label |
|---|---|
| `selector` | Use Company Selector |
| `tax_units` | Use Tax Units |

**`integer_rounding`** — Integer Rounding

| Value | Label |
|---|---|
| `HALF-UP` | Nearest |
| `UP` | Up |
| `DOWN` | Down |

## `account.report.column`

**`figure_type`** — Figure Type

| Value | Label |
|---|---|
| `monetary` | Monetary |
| `percentage` | Percentage |
| `integer` | Integer |
| `float` | Float |
| `date` | Date |
| `datetime` | Datetime |
| `boolean` | Boolean |
| `string` | String |

## `account.report.expression`

**`date_scope`** — Date Scope

| Value | Label |
|---|---|
| `from_beginning` | From the very start |
| `from_fiscalyear` | From the start of the fiscal year |
| `to_beginning_of_fiscalyear` | At the beginning of the fiscal year |
| `to_beginning_of_period` | At the beginning of the period |
| `strict_range` | Strictly on the given dates |
| `previous_return_period` | From previous return period |

**`engine`** — Computation Engine

| Value | Label |
|---|---|
| `domain` | record filter |
| `tax_tags` | Tax Tags |
| `aggregation` | Aggregate Other Formulas |
| `account_codes` | Prefix of Account Codes |
| `external` | External Value |
| `custom` | Custom Python Function |

**`figure_type`** — Figure Type

| Value | Label |
|---|---|
| `monetary` | Monetary |
| `percentage` | Percentage |
| `integer` | Integer |
| `float` | Float |
| `date` | Date |
| `datetime` | Datetime |
| `boolean` | Boolean |
| `string` | String |

## `account.report.line`

**`horizontal_split_side`** — Horizontal Split Side

| Value | Label |
|---|---|
| `left` | Left |
| `right` | Right |

## `account.resequence.wizard`

**`ordering`** — Ordering

| Value | Label |
|---|---|
| `keep` | Keep current order |
| `date` | Reorder by accounting date |

## `account.sale.closing`

**`frequency`** — Closing Type

| Value | Label |
|---|---|
| `daily` | Daily |
| `monthly` | Monthly |
| `annually` | Annual |

## `account.tax`

**`amount_type`** — Tax Computation

| Value | Label |
|---|---|
| `group` | Group of Taxes |
| `fixed` | Fixed |
| `percent` | Percentage |
| `division` | Percentage Tax Included |
| `code` | Custom Formula |

**`l10n_ar_tax_type`** — WTH Tax

| Value | Label |
|---|---|
| `earnings` | Earnings |
| `earnings_scale` | Earnings Scale |
| `iibb_untaxed` | IIBB Untaxed |
| `iibb_total` | IIBB Total Amount |

**`l10n_ar_type_tax_use`** — Argentina Tax Type (not stored)

| Value | Label |
|---|---|
| `sale` | Sales |
| `purchase` | Purchases |
| `none` | Other |
| `supplier` | Vendor Payment Withholding |
| `customer` | Customer Payment Withholding |

**`l10n_ar_withholding_payment_type`** — Argentina Withholding Payment Type

| Value | Label |
|---|---|
| `supplier` | Vendor Payment |
| `customer` | Customer Payment |

**`l10n_ee_kmd_inf_code`** — KMD INF Code

| Value | Label |
|---|---|
| `1` | Sale KMS §41/42 |
| `2` | Sale KMS §41^1 |
| `11` | Purchase KMS §29(4)/30/32 |
| `12` | Purchase KMS §41^1 |

**`l10n_eg_eta_code`** — ETA Code (Egypt)

| Value | Label |
|---|---|
| `t1_v001` | T1 - V001 - Export |
| `t1_v002` | T1 - V002 - Export to free areas and other areas |
| `t1_v003` | T1 - V003 - Exempted good or service |
| `t1_v004` | T1 - V004 - A non-taxable good or service |
| `t1_v005` | T1 - V005 - Exemptions for diplomats, consulates and embassies |
| `t1_v006` | T1 - V006 - Defence and National security Exemptions |
| `t1_v007` | T1 - V007 - Agreements exemptions |
| `t1_v008` | T1 - V008 - Special Exemption and other reasons |
| `t1_v009` | T1 - V009 - General Item sales |
| `t1_v010` | T1 - V010 - Other Rates |
| `t2_tbl01` | T2 - Tbl01 - Table tax (percentage) |
| `t3_tbl02` | T3 - Tbl02 - Table tax (Fixed Amount) |
| `t4_w001` | T4 - W001 - Contracting |
| `t4_w002` | T4 - W002 - Supplies |
| `t4_w003` | T4 - W003 - Purchases |
| `t4_w004` | T4 - W004 - Services |
| `t4_w005` | T4 - W005 - Sums paid by the cooperative societies for car transportation to their members |
| `t4_w006` | T4 - W006 - Commission agency & brokerage |
| `t4_w007` | T4 - W007 - Discounts & grants & additional exceptional incentives (smoke, cement companies) |
| `t4_w008` | T4 - W008 - All discounts & grants & commissions (petroleum, telecommunications, and other) |
| `t4_w009` | T4 - W009 - Supporting export subsidies |
| `t4_w010` | T4 - W010 - Professional fees |
| `t4_w011` | T4 - W011 - Commission & brokerage _A_57 |
| `t4_w012` | T4 - W012 - Hospitals collecting from doctors |
| `t4_w013` | T4 - W013 - Royalties |
| `t4_w014` | T4 - W014 - Customs clearance |
| `t4_w015` | T4 - W015 - Exemption |
| `t4_w016` | T4 - W016 - advance payments |
| `t5_st01` | T5 - ST01 - Stamping tax (percentage) |
| `t6_st02` | T6 - ST02 - Stamping Tax (amount) |
| `t7_ent01` | T7 - Ent01 - Entertainment tax (rate) |
| `t7_ent02` | T7 - Ent02 - Entertainment tax (amount) |
| `t8_rd01` | T8 - RD01 - Resource development fee (rate) |
| `t8_rd02` | T8 - RD02 - Resource development fee (amount) |
| `t9_sc01` | T9 - SC01 - Service charges (rate) |
| `t9_sc02` | T9 - SC02 - Service charges (amount) |
| `t10_mn01` | T10 - Mn01 - Municipality Fees (rate) |
| `t10_mn02` | T10 - Mn02 - Municipality Fees (amount) |
| `t11_mi01` | T11 - MI01 - Medical insurance fee (rate) |
| `t11_mi02` | T11 - MI02 - Medical insurance fee (amount) |
| `t12_of01` | T12 - OF01 - Other fees (rate) |
| `t12_of02` | T12 - OF02 - Other fees (amount) |
| `t13_st03` | T13 - ST03 - Stamping tax (percentage) |
| `t14_st04` | T14 - ST04 - Stamping Tax (amount) |
| `t15_ent03` | T15 - Ent03 - Entertainment tax (rate) |
| `t15_ent04` | T15 - Ent04 - Entertainment tax (amount) |
| `t16_rd03` | T16 - RD03 - Resource development fee (rate) |
| `t16_rd04` | T16 - RD04 - Resource development fee (amount) |
| `t17_sc03` | T17 - SC03 - Service charges (rate) |
| `t17_sc04` | T17 - SC04 - Service charges (amount) |
| `t18_mn03` | T18 - Mn03 - Municipality Fees (rate) |
| `t18_mn04` | T18 - Mn04 - Municipality Fees (amount) |
| `t19_mi03` | T19 - MI03 - Medical insurance fee (rate) |
| `t19_mi04` | T19 - MI04 - Medical insurance fee (amount) |
| `t20_of03` | T20 - OF03 - Other fees (rate) |
| `t20_of04` | T20 - OF04 - Other fees (amount) |

**`l10n_es_applicability`** — Applicability (Spain)

| Value | Label |
|---|---|
| `01` | VAT |
| `02` | IPSI |
| `03` | IGIC |

**`l10n_es_edi_facturae_tax_type`** — Spanish Facturae EDI Tax Type

| Value | Label |
|---|---|
| `01` | Value-Added Tax |
| `02` | Taxes on production, services and imports in Ceuta and Melilla |
| `03` | IGIC: Canaries General Indirect Tax |
| `04` | IRPF: Personal Income Tax |
| `05` | Other |
| `06` | ITPAJD: Tax on wealth transfers and stamp duty |
| `07` | IE: Excise duties and consumption taxes |
| `08` | RA: Customs duties |
| `09` | IGTECM: Sales tax in Ceuta and Melilla |
| `10` | IECDPCAC: Excise duties on oil derivates in Canaries |
| `11` | IIIMAB: Tax on premises that affect the environment in the Balearic Islands |
| `12` | ICIO: Tax on construction, installation and works |
| `13` | IMVDN: Local tax on unoccupied homes in Navarre |
| `14` | IMSN: Local tax on building plots in Navarre |
| `15` | IMGSN: Local sumptuary tax in Navarre |
| `16` | IMPN: Local tax on advertising in Navarre |
| `17` | REIVA: Special VAT for travel agencies |
| `18` | REIGIC: Special IGIC: for travel agencies |
| `19` | REIPSI: Special IPSI for travel agencies |
| `20` | IPS: Insurance premiums Tax |
| `21` | SWUA: Surcharge for Winding Up Activity |
| `22` | IVPEE: Tax on the value of electricity generation |
| `23` | Tax on the production of spent nuclear fuel and radioactive waste from the generation of nuclear electric power |
| `24` | Tax on the storage of spent nuclear energy and radioactive waste in centralised facilities |
| `25` | IDEC: Tax on bank deposits |
| `26` | Excise duty applied to manufactured tobacco in Canaries |
| `27` | IGFEI: Tax on Fluorinated Greenhouse Gases |
| `28` | IRNR: Non-resident Income Tax |
| `29` | Corporation Tax |

**`l10n_es_exempt_reason`** — Exempt Reason (Spain)

| Value | Label |
|---|---|
| `E1` | Art. 20 |
| `E2` | Art. 21 |
| `E3` | Art. 22 |
| `E4` | Art. 23 y 24 |
| `E5` | Art. 25 |
| `E6` | Otros |

**`l10n_es_type`** — Tax Type (Spain)

| Value | Label |
|---|---|
| `exento` | Exento |
| `sujeto` | Sujeto |
| `sujeto_agricultura` | Sujeto Agricultura |
| `sujeto_isp` | Sujeto ISP |
| `no_sujeto` | No Sujeto |
| `no_sujeto_loc` | No Sujeto por reglas de Localization |
| `no_deducible` | No Deducible |
| `retencion` | Retencion |
| `recargo` | Recargo de Equivalencia |
| `dua` | DUA |
| `ignore` | Ignore even the base amount |

**`l10n_gr_edi_default_tax_exemption_category`** — Default Tax Exemption Category

| Value | Label |
|---|---|
| `1` | 1 - Without VAT - article 3 of the VAT code |
| `2` | 2 - Without VAT - article 5 of the VAT code |
| `3` | 3 - Without VAT - article 13 of the VAT code |
| `4` | 4 - Without VAT - article 14 of the VAT code |
| `5` | 5 - Without VAT - article 16 of the VAT code |
| `6` | 6 - Without VAT - article 19 of the VAT code |
| `7` | 7 - Without VAT - article 22 of the VAT code |
| `8` | 8 - Without VAT - article 24 of the VAT code |
| `9` | 9 - Without VAT - article 25 of the VAT code |
| `10` | 10 - Without VAT - article 26 of the VAT code |
| `11` | 11 - Without VAT - article 27 of the VAT code |
| `12` | 12 - Without VAT - article 27 - Seagoing Vessels of the VAT code |
| `13` | 13 - Without VAT - article 27.1.γ - Seagoing Vessels of the VAT code |
| `14` | 14 - Without VAT - article 28 of the VAT code |
| `15` | 15 - Without VAT - article 39 of the VAT code |
| `16` | 16 - Without VAT - article 39a of the VAT code |
| `17` | 17 - Without VAT - article 40 of the VAT code |
| `18` | 18 - Without VAT - article 41 of the VAT code |
| `19` | 19 - Without VAT - article 47 of the VAT code |
| `20` | 20 - VAT included - article 43 of the VAT code |
| `21` | 21 - VAT included - article 44 of the VAT code |
| `22` | 22 - VAT included - article 45 of the VAT code |
| `23` | 23 - VAT included - article 46 of the VAT code |
| `24` | 24 - Without VAT - article 6 of the VAT code |
| `25` | 25 - Without VAT - ΠΟΛ.1029 / 1995 |
| `26` | 26 - Without VAT - ΠΟΛ.1167 / 2015 |
| `27` | 27 - Without VAT – Other VAT exceptions |
| `28` | 28 - Without VAT - Article 24 (b)(1) of the VAT Code(Tax Free) |
| `29` | 29 - Without VAT - Article 47 b of the VAT Code(OSS non - EU scheme) |
| `30` | 30 - Without VAT - Article 47 c of the VAT Code(OSS EU scheme) |
| `31` | 31 - Excluding VAT - Article 47 d of the VAT Code(IOSS) |

**`l10n_hu_tax_type`** — NAV VAT Tax Type

| Value | Label |
|---|---|
| `VAT` | Normal VAT (percent based) |
| `AAM` | AAM - Personal tax exemption |
| `TAM` | TAM - tax-exempt activity or tax-exempt due to being in public interest or special in nature |
| `KBAET` | KBAET - intra-Community exempt supply, without new means of transport |
| `KBAUK` | KBAUK - tax-exempt, intra-Community sales of new means of transport |
| `EAM` | EAM - tax-exempt, extra-Community sales of goods (export of goods to a non-EU country) |
| `NAM` | NAM - tax-exempt on other grounds related to international transactions |
| `ATK` | ATK - Outside the scope of VAT |
| `EUFAD37` | EUFAD37 - Based on section 37 of the VAT Act, a reverse charge transaction carried out in another Member State |
| `EUFADE` | EUFADE - Reverse charge transaction carried out in another Member State, not subject to Section 37 of the VAT Act |
| `EUE` | EUE - Non-reverse charge transaction performed in another Member State |
| `HO` | HO - Transaction in a third country |
| `DOMESTIC_REVERSE` | DOMESTIC_REVERSE - Domestic reverse-charge regime |
| `TRAVEL_AGENCY` | TRAVEL_AGENCY - Profit-margin based regime for travel agencies |
| `SECOND_HAND` | SECOND_HAND - Profit-margin based regime for second-hand sales |
| `ARTWORK` | ARTWORK - Profit-margin based regime for artwork sales |
| `ANTIQUES` | ANTIQUES - Profit-margin based regime for antique sales |
| `REFUNDABLE_VAT` | REFUNDABLE_VAT - VAT incurred under sections 11 or 14, without an agreement from the beneficiary to reimburse VAT |
| `NONREFUNDABLE_VAT` | NONREFUNDABLE_VAT - VAT incurred under sections 11 or 14, with an agreement from the beneficiary to reimburse VAT |
| `NO_VAT` | VAT not applicable pursuant to section 17 of the VAT Act |

**`l10n_in_gst_tax_type`** — L10N In Gst Tax Type (not stored)

| Value | Label |
|---|---|
| `igst` | igst |
| `cgst` | cgst |
| `sgst` | sgst |
| `cess` | cess |

**`l10n_in_tax_type`** — Indian Tax Type

| Value | Label |
|---|---|
| `gst` | GST |
| `tcs` | TCS |
| `tds_sale` | TDS Sale |
| `tds_purchase` | TDS Purchase |
| `nil_rated` | Nil Rated |
| `exempt` | Exempt |
| `non_gst` | Non-GST |

**`l10n_it_exempt_reason`** — Exoneration

| Value | Label |
|---|---|
| `N1` | [N1] Escluse ex art. 15 |
| `N2.1` | [N2.1] Non soggette ad IVA ai sensi degli artt. Da 7 a 7-septies del DPR 633/72 |
| `N2.2` | [N2.2] Non soggette - altri casi |
| `N3.1` | [N3.1] Non imponibili - esportazioni |
| `N3.2` | [N3.2] Non imponibili - cessioni intracomunitarie |
| `N3.3` | [N3.3] Non imponibili - cessioni verso San Marino |
| `N3.4` | [N3.4] Non imponibili - operazioni assimilate alle cessioni all'esportazione |
| `N3.5` | [N3.5] Non imponibili - a seguito di dichiarazioni d'intento |
| `N3.6` | [N3.6] Non imponibili - altre operazioni che non concorrono alla formazione del plafond |
| `N4` | [N4] Esenti |
| `N5` | [N5] Regime del margine / IVA non esposta in fattura |
| `N6.1` | [N6.1] Inversione contabile - cessione di rottami e altri materiali di recupero |
| `N6.2` | [N6.2] Inversione contabile - cessione di oro e argento puro |
| `N6.3` | [N6.3] Inversione contabile - subappalto nel settore edile |
| `N6.4` | [N6.4] Inversione contabile - cessione di fabbricati |
| `N6.5` | [N6.5] Inversione contabile - cessione di telefoni cellulari |
| `N6.6` | [N6.6] Inversione contabile - cessione di prodotti elettronici |
| `N6.7` | [N6.7] Inversione contabile - prestazioni comparto edile esettori connessi |
| `N6.8` | [N6.8] Inversione contabile - operazioni settore energetico |
| `N6.9` | [N6.9] Inversione contabile - altri casi |
| `N7` | [N7] IVA assolta in altro stato UE (prestazione di servizi di telecomunicazioni, tele-radiodiffusione ed elettronici ex art. 7-octies, comma 1 lett. a, b, art. 74-sexies DPR 633/72) |

**`l10n_it_pension_fund_type`** — Pension fund type (Italy)

| Value | Label |
|---|---|
| `TC01` | [TC01] National pension fund for lawyers and solicitors |
| `TC02` | [TC02] Pension fund for accountants with a degree |
| `TC03` | [TC03] Pension fund for surveyors |
| `TC04` | [TC04] National pension fund for associated engineers and architects |
| `TC05` | [TC05] National pension fund for notaries |
| `TC06` | [TC06] Pension fund for accountants without a degree and commercial experts |
| `TC07` | [TC07] ENASARCO pension fund for sales agents |
| `TC08` | [TC08] ENPACL pension fund for labor consultants |
| `TC09` | [TC09] ENPAM pension fund for doctors |
| `TC10` | [TC10] ENPAF pension fund for chemists |
| `TC11` | [TC11] ENPAV pension fund for veterinaries |
| `TC12` | [TC12] ENPAIA pension fund for people working in agriculture |
| `TC13` | [TC13] Pension fund for employees in delivery and marine agencies |
| `TC14` | [TC14] INPGI pension fund for journalists |
| `TC15` | [TC15] ONAOSI fund for sanitary orphans |
| `TC16` | [TC16] CASAGIT Additional pension fund for journalists |
| `TC17` | [TC17] EPPI pension fund for industrial experts |
| `TC18` | [TC18] EPAP pension fund |
| `TC19` | [TC19] ENPAB national pension fund for biologists |
| `TC20` | [TC20] ENPAPI national pension fund for nurses |
| `TC21` | [TC21] ENPAP national pension fund for psychologists |
| `TC22` | [TC22] INPS national pension fund |

**`l10n_it_withholding_reason`** — Withholding tax reason (Italy)

| Value | Label |
|---|---|
| `A` | [A] Autonomous work in the fields of art or profession |
| `B` | [B] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science |
| `C` | [C] Income from work as part of association groups or other cooperation determined by contracts |
| `D` | [D] Income as partner or founder of a corporation |
| `E` | [E] Income from client-related bill protests made by town secretaries |
| `G` | [G] Compensation for the end of a professional sport career |
| `H` | [H] Compensation for the end of a societary career (excluded those earned before 31.12.2003) and already taxed |
| `I` | [I] Compensation for the end of a notary career |
| `K` | [K] Civil service checks, ref art. 16 D.lgs. n.40 6/03/2017 |
| `L` | [L] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science, but not made by the author/inventor |
| `L1` | [L1] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science, from someone who actively bought the use rights |
| `M` | [M] Autonomous work which isn't part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow |
| `M1` | [M1] Incomes due for an obligation to act, not to act, or to allow |
| `M2` | [M2] Autonomous work which isn't part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow - that require being registered to the "Gestione separata" |
| `N` | [N] Compensation for travel, expenses, prizes, or other compensations for amateur sport activities |
| `O` | [O] Autonomous work which isn't part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow - that do not require being registered to the "Gestione separata" |
| `O1` | [O1] Incomes due for an obligation to act, not to act, or to allow - that do not require being registered to the "Gestione Separata" |
| `P` | [P] Compensation for people residing abroad for continuous use or concession of industrial machinery, commercial or scientific tools that are on the Italian soil |
| `Q` | [Q] Provisions for exclusive agents or sales representatives' work |
| `R` | [R] Provisions for non-exclusive agents or sales representatives' work |
| `S` | [S] Provisions for commissioner work |
| `T` | [T] Provisions for mediator work |
| `U` | [U] Provisions for procurer work |
| `V` | [V] Provisions for door-to-door sales persons and newspaper selling in kiosks |
| `V1` | [V1] Income from unusual commercial activities (such as provisions for occasional work or sales representative, mediator, procurer) |
| `V2` | [V2] Income from unusual work activities from door-to-door sales representatives |
| `W` | [W] Income from tenders subject to Art. 25-ter of Presidential Decree 600/1973 |
| `X` | [X] Income from 2014 for foreign companies or institutions subject to law art. 26-quater, c. 1, lett. a) and b) D.P.R. 600/1973 |
| `Y` | [Y] Income from 1.01.2005 to 26.07.2005 from companies or institutions not included in the description above |
| `Z` | [Z] Deprecated |
| `ZO` | [ZO] Other reason |

**`l10n_it_withholding_type`** — Withholding tax type (Italy)

| Value | Label |
|---|---|
| `RT01` | [RT01] Withholding for persons |
| `RT02` | [RT02] Withholding for personal businesses |
| `RT03` | [RT03] INPS Pension fund contribution |
| `RT04` | [RT04] ENASARCO pension fund contribution |
| `RT05` | [RT05] ENPAM pension fund contribution |
| `RT06` | [RT06] Other pension fund contribution |

**`l10n_mx_factor_type`** — Factor Type

| Value | Label |
|---|---|
| `Tasa` | Tasa |
| `Cuota` | Cuota |
| `Exento` | Exento |

**`l10n_mx_tax_type`** — SAT Tax Type

| Value | Label |
|---|---|
| `isr` | ISR |
| `iva` | IVA |
| `ieps` | IEPS |
| `local` | Local |

**`l10n_my_tax_type`** — Malaysian Tax Type

| Value | Label |
|---|---|
| `01` | Sales Tax |
| `02` | Service Tax |
| `03` | Tourism Tax |
| `04` | High-Value Goods Tax |
| `05` | Sales Tax on Low Value Goods |
| `06` | Not Applicable |
| `E` | Tax exemption (where applicable) |

**`l10n_pe_edi_isc_type`** — ISC Type

| Value | Label |
|---|---|
| `01` | System to value |
| `02` | Application of the Fixed Amount |
| `03` | Retail Price System |

**`l10n_pe_edi_tax_code`** — Code

| Value | Label |
|---|---|
| `1000` | IGV - General Sales Tax |
| `1016` | IVAP - Tax on Sale Paddy Rice |
| `2000` | ISC - Selective Excise Tax |
| `7152` | ICBPER - Plastic bag tax |
| `9995` | EXP - Exportation |
| `9996` | GRA - Free |
| `9997` | EXO - Exonerated |
| `9998` | INA - Unaffected |
| `9999` | OTHERS - Other taxes |

**`l10n_pe_edi_unece_category`** — UNECE Code

| Value | Label |
|---|---|
| `E` | Exempt from tax |
| `G` | Free export item, tax not charged |
| `O` | Services outside scope of tax |
| `S` | Standard rate |
| `Z` | Zero rated goods |

**`l10n_sa_exemption_reason_code`** — Exemption Reason Code

| Value | Label |
|---|---|
| `VATEX-SA-29` | VATEX-SA-29 Financial services mentioned in Article 29 of the VAT Regulations. |
| `VATEX-SA-29-7` | VATEX-SA-29-7 Life insurance services mentioned in Article 29 of the VAT Regulations. |
| `VATEX-SA-30` | VATEX-SA-30 Real estate transactions mentioned in Article 30 of the VAT Regulations. |
| `VATEX-SA-32` | VATEX-SA-32 Export of goods. |
| `VATEX-SA-33` | VATEX-SA-33 Export of Services. |
| `VATEX-SA-34-1` | VATEX-SA-34-1 The international transport of Goods. |
| `VATEX-SA-34-2` | VATEX-SA-34-2 The international transport of Passengers. |
| `VATEX-SA-34-3` | VATEX-SA-34-3 Services directly connected and incidental to a Supply of international passenger transport. |
| `VATEX-SA-34-4` | VATEX-SA-34-4 Supply of a qualifying means of transport. |
| `VATEX-SA-34-5` | VATEX-SA-34-5 Any services relating to Goods or passenger transportation, as defined in article twenty five of these Regulations. |
| `VATEX-SA-35` | VATEX-SA-35 Medicines and medical equipment. |
| `VATEX-SA-36` | VATEX-SA-36 Qualifying metals. |
| `VATEX-SA-EDU` | VATEX-SA-EDU Private education to citizen. |
| `VATEX-SA-HEA` | VATEX-SA-HEA Private healthcare to citizen. |
| `VATEX-SA-OOS` | VATEX-SA-OOS Not subject to VAT. |

**`l10n_tw_edi_special_tax_type`** — Ecpay Special Tax Type

| Value | Label |
|---|---|
| `1` | Saloons and tea rooms, coffee shops and bars offering companionship services: Tax rate is 25% |
| `2` | Night clubs or restaurants providing entertaining show programs: Tax rate is 15% |
| `3` | Banking businesses, insurance businesses, trust investment businesses, securities businesses, futures businesses, commercial paper businesses and pawn-broking businesses: Tax rate is 2% |
| `4` | The sales amounts from reinsurance premiums shall be taxed at 1% |
| `5` | Banking businesses, insurance businesses, trust investment businesses, securities businesses, futures businesses, commercial paper businesses and pawn-broking businesses: Tax rate is 5% |
| `6` | Core business revenues from the banking and insurance business of the banking and insurance industries (Applicable to sales after July 2014): Tax rate is 5% |
| `7` | Core business revenues from the banking and insurance business of the banking and insurance industries (Applicable to sales after June 2014): Tax rate is 5% |
| `8` | Duty free or non-output data |

**`l10n_tw_edi_tax_type`** — Ecpay Tax Type

| Value | Label |
|---|---|
| `1` | Taxable |
| `2` | Zero tax rate |
| `3` | Duty free |
| `4` | Taxable (special tax rate) |

**`l10n_uy_tax_category`** — Tax Category

| Value | Label |
|---|---|
| `vat` | VAT |

**`price_include_override`** — Included in Price

| Value | Label |
|---|---|
| `tax_included` | Tax Included |
| `tax_excluded` | Tax Excluded |

**`tax_exigibility`** — Tax Exigibility

| Value | Label |
|---|---|
| `on_invoice` | Based on Invoice |
| `on_payment` | Based on Payment |

**`tax_scope`** — Tax Scope

| Value | Label |
|---|---|
| `service` | Services |
| `consu` | Goods |
| `merch` | Merchandise |
| `invest` | Investment |

**`type_tax_use`** — Tax Type

| Value | Label |
|---|---|
| `sale` | Sales |
| `purchase` | Purchases |
| `none` | None |

**`ubl_cii_tax_category_code`** — Tax Category Code

| Value | Label |
|---|---|
| `AE` | AE - Vat Reverse Charge |
| `E` | E - Exempt from Tax |
| `S` | S - Standard rate |
| `Z` | Z - Zero rated goods |
| `G` | G - Free export item, VAT not charged |
| `O` | O - Services outside scope of tax |
| `K` | K - VAT exempt for EEA intra-community supply of goods and services |
| `L` | L - Canary Islands general indirect tax |
| `M` | M - Tax for production, services and importation in Ceuta and Melilla |
| `B` | B - Transferred (VAT), In Italy |
| `SR` | SG - Local supply of goods and services |
| `SRCA-S` | SG - Customer accounting supply made by the supplier |
| `SRCA-C` | SG - Customer accounting supply made by the customer on supplier’s behalf |
| `SROVR-RS` | SG - Supply of remote services accountable by the electronic marketplace under the Overseas Vendor Registration Regime |
| `SROVR-LVG` | SG - Supply of low-value goods accountable by the redeliverer or electronic marketplace on behalf of third-party suppliers |
| `SRRC` | SG - Reverse charge regime for Business-to-Business (“B2B”) supplies of imported services |
| `SRLVG` | SG - Own supply of low-value goods |
| `ZR` | SG - Supplies involving goods for export/ provision of international services |
| `ES33` | SG - Specific categories of exempt supplies listed under regulation 33 of the GST (General) Regulations |
| `ESN33` | SG - Exempt supplies other than those listed under regulation 33 of the GST (General) Regulations |
| `DS` | SG - Supplies required to be reported pursuant to the GST legislation |
| `OS` | SG - Supplies outside the scope of the GST Act |
| `NG` | SG - Supplies from a company which is not registered for GST |
| `NA` | SG - Taxable supplies where GST need not be charged |

**`ubl_cii_tax_exemption_reason_code`** — Tax Exemption Reason Code

| Value | Label |
|---|---|
| `VATEX-EU-79-C` | VATEX-EU-79-C - Exempt based on article 79, point c of Council Directive 2006/112/EC |
| `VATEX-EU-132` | VATEX-EU-132 - Exempt based on article 132 of Council Directive 2006/112/EC |
| `VATEX-EU-132-1A` | VATEX-EU-132-1A - Exempt based on article 132, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1B` | VATEX-EU-132-1B - Exempt based on article 132, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1C` | VATEX-EU-132-1C - Exempt based on article 132, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1D` | VATEX-EU-132-1D - Exempt based on article 132, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1E` | VATEX-EU-132-1E - Exempt based on article 132, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1F` | VATEX-EU-132-1F - Exempt based on article 132, section 1 (f) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1G` | VATEX-EU-132-1G - Exempt based on article 132, section 1 (g) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1H` | VATEX-EU-132-1H - Exempt based on article 132, section 1 (h) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1I` | VATEX-EU-132-1I - Exempt based on article 132, section 1 (i) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1J` | VATEX-EU-132-1J - Exempt based on article 132, section 1 (j) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1K` | VATEX-EU-132-1K - Exempt based on article 132, section 1 (k) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1L` | VATEX-EU-132-1L - Exempt based on article 132, section 1 (l) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1M` | VATEX-EU-132-1M - Exempt based on article 132, section 1 (m) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1N` | VATEX-EU-132-1N - Exempt based on article 132, section 1 (n) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1O` | VATEX-EU-132-1O - Exempt based on article 132, section 1 (o) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1P` | VATEX-EU-132-1P - Exempt based on article 132, section 1 (p) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1Q` | VATEX-EU-132-1Q - Exempt based on article 132, section 1 (q) of Council Directive 2006/112/EC |
| `VATEX-EU-135-1` | VATEX-EU-135-1 - Exempt based on article 135, section 1 of Council Directive 2006/112/EC |
| `VATEX-EU-143` | VATEX-EU-143 - Exempt based on article 143 of Council Directive 2006/112/EC |
| `VATEX-EU-143-1A` | VATEX-EU-143-1A - Exempt based on article 143, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1B` | VATEX-EU-143-1B - Exempt based on article 143, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1C` | VATEX-EU-143-1C - Exempt based on article 143, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1D` | VATEX-EU-143-1D - Exempt based on article 143, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1E` | VATEX-EU-143-1E - Exempt based on article 143, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1F` | VATEX-EU-143-1F - Exempt based on article 143, section 1 (f) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1FA` | VATEX-EU-143-1FA - Exempt based on article 143, section 1 (fa) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1G` | VATEX-EU-143-1G - Exempt based on article 143, section 1 (g) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1H` | VATEX-EU-143-1H - Exempt based on article 143, section 1 (h) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1I` | VATEX-EU-143-1I - Exempt based on article 143, section 1 (i) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1J` | VATEX-EU-143-1J - Exempt based on article 143, section 1 (j) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1K` | VATEX-EU-143-1K - Exempt based on article 143, section 1 (k) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1L` | VATEX-EU-143-1L - Exempt based on article 143, section 1 (l) of Council Directive 2006/112/EC |
| `VATEX-EU-144` | VATEX-EU-144 - Exempt based on article 144 of Council Directive 2006/112/EC |
| `VATEX-EU-146-1E` | VATEX-EU-146-1E - Exempt based on article 146 section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-148` | VATEX-EU-148 - Exempt based on article 148 of Council Directive 2006/112/EC |
| `VATEX-EU-148-A` | VATEX-EU-148-A - Exempt based on article 148, section (a) of Council Directive 2006/112/EC |
| `VATEX-EU-148-B` | VATEX-EU-148-B - Exempt based on article 148, section (b) of Council Directive 2006/112/EC |
| `VATEX-EU-148-C` | VATEX-EU-148-C - Exempt based on article 148, section (c) of Council Directive 2006/112/EC |
| `VATEX-EU-148-D` | VATEX-EU-148-D - Exempt based on article 148, section (d) of Council Directive 2006/112/EC |
| `VATEX-EU-148-E` | VATEX-EU-148-E - Exempt based on article 148, section (e) of Council Directive 2006/112/EC |
| `VATEX-EU-148-F` | VATEX-EU-148-F - Exempt based on article 148, section (f) of Council Directive 2006/112/EC |
| `VATEX-EU-148-G` | VATEX-EU-148-G - Exempt based on article 148, section (g) of Council Directive 2006/112/EC |
| `VATEX-EU-151` | VATEX-EU-151 - Exempt based on article 151 of Council Directive 2006/112/EC |
| `VATEX-EU-151-1A` | VATEX-EU-151-1A - Exempt based on article 151, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1AA` | VATEX-EU-151-1AA - Exempt based on article 151, section 1 (aa) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1B` | VATEX-EU-151-1B - Exempt based on article 151, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1C` | VATEX-EU-151-1C - Exempt based on article 151, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1D` | VATEX-EU-151-1D - Exempt based on article 151, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1E` | VATEX-EU-151-1E - Exempt based on article 151, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-153` | VATEX-EU-153 - Exempt based on article 153 of Council Directive 2006/112/EC |
| `VATEX-EU-159` | VATEX-EU-159 - Exempt based on article 159 of Council Directive 2006/112/EC |
| `VATEX-EU-309` | VATEX-EU-309 - Exempt based on article 309 of Council Directive 2006/112/EC |
| `VATEX-EU-AE` | VATEX-EU-AE - Reverse charge |
| `VATEX-EU-D` | VATEX-EU-D - Intra-Community acquisition from second hand means of transport |
| `VATEX-EU-F` | VATEX-EU-F - Intra-Community acquisition of second hand goods |
| `VATEX-EU-G` | VATEX-EU-G - Export outside the EU |
| `VATEX-EU-I` | VATEX-EU-I - Intra-Community acquisition of works of art |
| `VATEX-EU-IC` | VATEX-EU-IC - Intra-Community supply |
| `VATEX-EU-O` | VATEX-EU-O - Not subject to VAT |
| `VATEX-EU-J` | VATEX-EU-J - Intra-Community acquisition of collectors items and antiques |
| `VATEX-FR-FRANCHISE` | VATEX-FR-FRANCHISE - France domestic VAT franchise in base |
| `VATEX-FR-CNWVAT` | VATEX-FR-CNWVAT - France domestic Credit Notes without VAT, due to supplier forfeit of VAT for discount |
| `VATEX-FR-CGI261-1` | VATEX-FR-CGI261-1 - Exempt based on 1 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-2` | VATEX-FR-CGI261-2 - Exempt based on 2 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-3` | VATEX-FR-CGI261-3 - Exempt based on 3 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-4` | VATEX-FR-CGI261-4 - Exempt based on 4 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-5` | VATEX-FR-CGI261-5 - Exempt based on 5 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-7` | VATEX-FR-CGI261-7 - Exempt based on 7 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-8` | VATEX-FR-CGI261-8 - Exempt based on 8 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261A` | VATEX-FR-CGI261A - Exempt based on article 261 A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261B` | VATEX-FR-CGI261B - Exempt based on article 261 B of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-1` | VATEX-FR-CGI261C-1 - Exempt based on 1° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-2` | VATEX-FR-CGI261C-2 - Exempt based on 2° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-3` | VATEX-FR-CGI261C-3 - Exempt based on 3° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-1` | VATEX-FR-CGI261D-1 - Exempt based on 1° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-1BIS` | VATEX-FR-CGI261D-1BIS - Exempt based on 1°bis of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-2` | VATEX-FR-CGI261D-2 - Exempt based on 2° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-3` | VATEX-FR-CGI261D-3 - Exempt based on 3° of article 261 D of the Code Général des Impôts (CGI ; General tax code) Exonération de TVA - Article 261 D-3° du Code Général des Impôts |
| `VATEX-FR-CGI261D-4` | VATEX-FR-CGI261D-4 - Exempt based on 4° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261E-1` | VATEX-FR-CGI261E-1 - Exempt based on 1° of article 261 E of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261E-2` | VATEX-FR-CGI261E-2 - Exempt based on 2° of article 261 E of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI277A` | VATEX-FR-CGI277A - Exempt based on article 277 A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI275` | VATEX-FR-CGI275 - Exempt based on article 275 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-298SEXDECIESA` | VATEX-FR-298SEXDECIESA - Exempt based on article 298 sexdecies A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI295` | VATEX-FR-CGI295 - Exempt based on article 295 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-AE` | VATEX-FR-AE - Exempt based on 2 of article 283 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-F` | VATEX-FR-F - Second-hand sales |
| `VATEX-FR-I` | VATEX-FR-I - Sales of works of art |
| `VATEX-FR-J` | VATEX-FR-J - Sales of antiques |

## `account.tax.group`

**`l10n_ar_tribute_afip_code`** — Tribute ARCA Code

| Value | Label |
|---|---|
| `01` | 01 - National Taxes |
| `02` | 02 - Provincial Taxes |
| `03` | 03 - Municipal Taxes |
| `04` | 04 - Internal Taxes |
| `06` | 06 - VAT perception |
| `07` | 07 - IIBB perception |
| `08` | 08 - Municipal Taxes Perceptions |
| `09` | 09 - Other Perceptions |
| `99` | 99 - Others |

**`l10n_ar_vat_afip_code`** — VAT ARCA Code

| Value | Label |
|---|---|
| `0` | Not Applicable |
| `1` | Untaxed |
| `2` | Exempt |
| `3` | 0% |
| `4` | 10.5% |
| `5` | 21% |
| `6` | 27% |
| `8` | 5% |
| `9` | 2,5% |

**`l10n_ec_type`** — Type Ecuadorian Tax

| Value | Label |
|---|---|
| `vat05` | VAT 5% |
| `vat08` | VAT 8% |
| `vat12` | VAT 12% |
| `vat13` | VAT 13% |
| `vat14` | VAT 14% |
| `vat15` | VAT 15% |
| `zero_vat` | VAT 0% |
| `not_charged_vat` | VAT Not Charged |
| `exempt_vat` | VAT Exempt |
| `ice` | Special Consumptions Tax (ICE) |
| `irbpnr` | Plastic Bottles (IRBPNR) |
| `withhold_vat_sale` | VAT Withhold on Sales |
| `withhold_vat_purchase` | VAT Withhold on Purchases |
| `withhold_income_sale` | Profit Withhold on Sales |
| `withhold_income_purchase` | Profit Withhold on Purchases |
| `outflows_tax` | Exchange Outflows |
| `other` | Others |

## `account.tax.repartition.line`

**`document_type`** — Related to

| Value | Label |
|---|---|
| `invoice` | Invoice |
| `refund` | Refund |

**`repartition_type`** — Based On

| Value | Label |
|---|---|
| `base` | Base |
| `tax` | of tax |

## `account.withholding.line`

**`comodel_payment_type`** — Comodel Payment Type (not stored)

| Value | Label |
|---|---|
| `outbound` | Send Money |
| `inbound` | Receive Money |

**`placeholder_type`** — Placeholder Type

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

**`previous_placeholder_type`** — Previous Placeholder Type

| Value | Label |
|---|---|
| `given_by_sequence` | Given By the Sequence |
| `given_by_name` | Given By the Name |
| `not_defined` | Not defined |

## `account_edi_proxy_client.user`

**`edi_mode`** — EDI operating mode

| Value | Label |
|---|---|
| `prod` | Production mode |
| `test` | Test mode |
| `demo` | Demo mode |

**`proxy_type`** — Proxy Type

| Value | Label |
|---|---|
| `peppol` | PEPPOL |
| `nemhandel` | Nemhandel |
| `l10n_it_edi` | Italian EDI |
| `l10n_my_edi` | Malaysian EDI |
| `pdp` | Approved Platform |
| `l10n_gr_edi` | Greek EDI |

## `auth.totp.rate.limit.log`

**`limit_type`** — Limit Type

| Value | Label |
|---|---|
| `send_email` | Send Email |
| `code_check` | Code Checking |

## `barcode.nomenclature`

**`upc_ean_conv`** — UPC/EAN Conversion

| Value | Label |
|---|---|
| `none` | Never |
| `ean2upc` | EAN-13 to UPC-A |
| `upc2ean` | UPC-A to EAN-13 |
| `always` | Always |

## `barcode.rule`

**`encoding`** — Encoding

| Value | Label |
|---|---|
| `any` | Any |
| `ean13` | EAN-13 |
| `ean8` | EAN-8 |
| `upca` | UPC-A |
| `gs1-128` | GS1-128 |

**`gs1_content_type`** — GS1 Content Type

| Value | Label |
|---|---|
| `date` | Date |
| `measure` | Measure |
| `identifier` | Numeric Identifier |
| `alpha` | Alpha-Numeric Name |

**`type`** — Type

| Value | Label |
|---|---|
| `alias` | Alias |
| `product` | Unit Product |
| `quantity` | Quantity |
| `weight` | Weighted Product |
| `location` | Location |
| `location_dest` | Destination location |
| `lot` | Lot |
| `package` | Package |
| `use_date` | Best before Date |
| `expiration_date` | Expiration Date |
| `package_type` | Package Type |
| `pack_date` | Pack Date |
| `price` | Priced Product |
| `discount` | Discounted Product |
| `client` | Client |
| `cashier` | Cashier |
| `coupon` | Coupon |

## `base.automation`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`trg_date_range_mode`** — Delay mode

| Value | Label |
|---|---|
| `after` | After |
| `before` | Before |

**`trg_date_range_type`** — Delay unit

| Value | Label |
|---|---|
| `minutes` | Minutes |
| `hour` | Hours |
| `day` | Days |
| `month` | Months |

**`trigger`** — Trigger

| Value | Label |
|---|---|
| `on_stage_set` | Stage is set to |
| `on_user_set` | User is set |
| `on_tag_set` | Tag is added |
| `on_state_set` | State is set to |
| `on_priority_set` | Priority is set to |
| `on_archive` | On archived |
| `on_unarchive` | On unarchived |
| `on_create` | On create |
| `on_create_or_write` | On create and edit |
| `on_write` | On update |
| `on_unlink` | On deletion |
| `on_change` | On UI change |
| `on_time` | Based on date field |
| `on_time_created` | After creation |
| `on_time_updated` | After last update |
| `on_message_received` | On incoming message |
| `on_message_sent` | On outgoing message |
| `on_webhook` | On webhook |

## `base.enable.profiling.wizard`

**`duration`** — Enable profiling for

| Value | Label |
|---|---|
| `minutes_5` | 5 Minutes |
| `hours_1` | 1 Hour |
| `days_1` | 1 Day |
| `months_1` | 1 Month |

## `base.import.module`

**`state`** — Status

| Value | Label |
|---|---|
| `init` | init |
| `done` | done |

## `base.language.export`

**`export_type`** — Export Type

| Value | Label |
|---|---|
| `module` | Module |
| `model` | Model |

**`format`** — File Format

| Value | Label |
|---|---|
| `csv` | CSV File |
| `po` | PO File |
| `tgz` | TGZ Archive |

**`state`** — State

| Value | Label |
|---|---|
| `choose` | choose |
| `get` | get |

## `base.module.update`

**`state`** — Status

| Value | Label |
|---|---|
| `init` | init |
| `done` | done |

## `base.partner.merge.automatic.wizard`

**`state`** — State

| Value | Label |
|---|---|
| `option` | Option |
| `selection` | Selection |
| `finished` | Finished |

## `calendar.alarm`

**`alarm_type`** — Type

| Value | Label |
|---|---|
| `notification` | Notification |
| `email` | Email |
| `sms` | SMS Text Message |

**`interval`** — Unit

| Value | Label |
|---|---|
| `minutes` | Minutes |
| `hours` | Hours |
| `days` | Days |

## `calendar.attendee`

**`availability`** — Available/Busy

| Value | Label |
|---|---|
| `free` | Available |
| `busy` | Busy |

**`state`** — Status

| Value | Label |
|---|---|
| `accepted` | Yes |
| `declined` | No |
| `tentative` | Maybe |
| `needsAction` | Needs Action |

## `calendar.event`

**`byday`** — By day (not stored)

| Value | Label |
|---|---|
| `1` | First |
| `2` | Second |
| `3` | Third |
| `4` | Fourth |
| `-1` | Last |

**`effective_privacy`** — Effective Privacy (not stored)

| Value | Label |
|---|---|
| `public` | Public |
| `private` | Private |
| `confidential` | Only internal users |

**`end_type`** — Recurrence Termination (not stored)

| Value | Label |
|---|---|
| `count` | Number of repetitions |
| `end_date` | End date |
| `forever` | Forever |

**`month_by`** — Option (not stored)

| Value | Label |
|---|---|
| `date` | Date of month |
| `day` | Day of month |

**`privacy`** — Privacy

| Value | Label |
|---|---|
| `public` | Public |
| `private` | Private |
| `confidential` | Only internal users |

**`recurrence_update`** — Recurrence Update (not stored)

| Value | Label |
|---|---|
| `self_only` | This event |
| `future_events` | This and following events |
| `all_events` | All events |

**`rrule_type`** — Recurrence (not stored)

| Value | Label |
|---|---|
| `daily` | Days |
| `weekly` | Weeks |
| `monthly` | Months |
| `yearly` | Years |

**`rrule_type_ui`** — Repeat (not stored)

| Value | Label |
|---|---|
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `yearly` | Yearly |
| `custom` | Custom |

**`show_as`** — Show as

| Value | Label |
|---|---|
| `free` | Available |
| `busy` | Busy |

**`videocall_source`** — Videocall Source (not stored)

| Value | Label |
|---|---|
| `discuss` | Discuss |
| `custom` | Custom |
| `google_meet` | Google Meet |

**`weekday`** — Weekday (not stored)

| Value | Label |
|---|---|
| `MON` | Monday |
| `TUE` | Tuesday |
| `WED` | Wednesday |
| `THU` | Thursday |
| `FRI` | Friday |
| `SAT` | Saturday |
| `SUN` | Sunday |

## `calendar.popover.delete.wizard`

**`delete`** — Delete

| Value | Label |
|---|---|
| `one` | Delete this event |
| `next` | Delete this and following events |
| `all` | Delete all the events |

## `calendar.provider.config`

**`external_calendar_provider`** — Choose an external calendar to configure

| Value | Label |
|---|---|
| `google` | Google |
| `microsoft` | Outlook |

## `calendar.recurrence`

**`byday`** — By day

| Value | Label |
|---|---|
| `1` | First |
| `2` | Second |
| `3` | Third |
| `4` | Fourth |
| `-1` | Last |

**`end_type`** — End Type

| Value | Label |
|---|---|
| `count` | Number of repetitions |
| `end_date` | End date |
| `forever` | Forever |

**`month_by`** — Month By

| Value | Label |
|---|---|
| `date` | Date of month |
| `day` | Day of month |

**`rrule_type`** — Rrule Type

| Value | Label |
|---|---|
| `daily` | Days |
| `weekly` | Weeks |
| `monthly` | Months |
| `yearly` | Years |

**`weekday`** — Weekday

| Value | Label |
|---|---|
| `MON` | Monday |
| `TUE` | Tuesday |
| `WED` | Wednesday |
| `THU` | Thursday |
| `FRI` | Friday |
| `SAT` | Saturday |
| `SUN` | Sunday |

## `card.campaign`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

## `card.card`

**`share_status`** — Share Status

| Value | Label |
|---|---|
| `shared` | Shared |
| `visited` | Visited |

## `certificate.certificate`

**`content_format`** — Original certificate format

| Value | Label |
|---|---|
| `der` | DER |
| `pem` | PEM |
| `pkcs12` | PKCS12 |

**`scope`** — Certificate scope

| Value | Label |
|---|---|
| `general` | General |
| `facturae` | Facturae |
| `sii` | SII |
| `tbai` | TBAI |
| `verifactu` | Veri*Factu |

## `chatbot.script`

**`first_step_warning`** — First Step Warning (not stored)

| Value | Label |
|---|---|
| `first_step_operator` | First Step Operator |
| `first_step_invalid` | First Step Invalid |

## `chatbot.script.step`

**`step_type`** — Step Type

| Value | Label |
|---|---|
| `text` | Text |
| `question_selection` | Question |
| `question_email` | Email |
| `question_phone` | Phone |
| `forward_operator` | Forward to Operator |
| `free_input_single` | Free Input |
| `free_input_multi` | Free Input (Multi-Line) |
| `create_lead` | Create Lead |
| `create_lead_and_forward` | Create Lead & Forward |

## `crm.activity.report`

**`lead_type`** — Type

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

**`won_status`** — Is Won

| Value | Label |
|---|---|
| `won` | Won |
| `lost` | Lost |
| `pending` | Pending |

## `crm.iap.lead.mining.request`

**`contact_filter_type`** — Filter on

| Value | Label |
|---|---|
| `role` | Role |
| `seniority` | Seniority |

**`error_type`** — Error Type

| Value | Label |
|---|---|
| `credits` | Insufficient Credits |
| `no_result` | No Result |

**`lead_type`** — Type

| Value | Label |
|---|---|
| `lead` | Leads |
| `opportunity` | Opportunities |

**`search_type`** — Target

| Value | Label |
|---|---|
| `companies` | Companies |
| `people` | Companies and their Contacts |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `error` | Error |
| `done` | Done |

## `crm.lead`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`email_state`** — Email Quality

| Value | Label |
|---|---|
| `correct` | Correct |
| `incorrect` | Incorrect |

**`phone_state`** — Phone Quality

| Value | Label |
|---|---|
| `correct` | Correct |
| `incorrect` | Incorrect |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Low |
| `1` | Medium |
| `2` | High |
| `3` | Very High |

**`type`** — Type

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

**`won_status`** — Won/Lost

| Value | Label |
|---|---|
| `won` | Won |
| `lost` | Lost |
| `pending` | Pending |

## `crm.lead.forward.to.partner`

**`forward_type`** — Forward selected leads to

| Value | Label |
|---|---|
| `single` | a single partner: manual selection of partner |
| `assigned` | several partners: automatic assignment, using GPS coordinates and partner's grades |

## `crm.lead2opportunity.partner`

**`action`** — Related Customer

| Value | Label |
|---|---|
| `create` | Create a new customer |
| `exist` | Link to an existing customer |

**`name`** — Conversion Action

| Value | Label |
|---|---|
| `convert` | Convert to opportunity |
| `merge` | Merge with existing opportunities |

## `crm.lead2opportunity.partner.mass`

**`action`** — Related Customer

| Value | Label |
|---|---|
| `create` | Create a new customer |
| `exist` | Link to an existing customer |
| `each_exist_or_create` | Use existing partner or create |

**`name`** — Conversion Action

| Value | Label |
|---|---|
| `convert` | Convert to opportunity |
| `merge` | Merge with existing opportunities |

## `crm.quotation.partner`

**`action`** — Quotation Customer

| Value | Label |
|---|---|
| `create` | Create a new customer |
| `exist` | Link to an existing customer |
| `nothing` | Do not link to a customer |

## `crm.reveal.rule`

**`contact_filter_type`** — Filter On

| Value | Label |
|---|---|
| `role` | Role |
| `seniority` | Seniority |

**`lead_for`** — Data Tracking

| Value | Label |
|---|---|
| `companies` | Companies |
| `people` | Companies and their Contacts |

**`lead_type`** — Type

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Low |
| `1` | Medium |
| `2` | High |
| `3` | Very High |

## `crm.reveal.view`

**`reveal_state`** — State

| Value | Label |
|---|---|
| `to_process` | To Process |
| `not_found` | Not Found |

## `data_recycle.model`

**`notify_frequency_period`** — Notify Frequency Period

| Value | Label |
|---|---|
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

**`recycle_action`** — Recycle Action

| Value | Label |
|---|---|
| `archive` | Archive |
| `unlink` | Delete |

**`recycle_mode`** — Recycle Mode

| Value | Label |
|---|---|
| `manual` | Manual |
| `automatic` | Automatic |

**`time_field_delta_unit`** — Delta Unit

| Value | Label |
|---|---|
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |
| `years` | Years |

## `delivery.carrier`

**`delivery_type`** — Provider

| Value | Label |
|---|---|
| `base_on_rule` | Based on Rules |
| `fixed` | Fixed Price |
| `gelato` | Gelato |
| `in_store` | Pick up in store |

**`gelato_shipping_service_type`** — Gelato Shipping Service Type

| Value | Label |
|---|---|
| `normal` | Standard Delivery |
| `express` | Express Delivery |

**`integration_level`** — Integration Level

| Value | Label |
|---|---|
| `rate` | Get Rate |
| `rate_and_ship` | Get Rate and Create Shipment |

**`invoice_policy`** — Invoicing Policy

| Value | Label |
|---|---|
| `estimated` | Estimated cost |
| `real` | Real cost |

## `delivery.price.rule`

**`operator`** — Operator

| Value | Label |
|---|---|
| `==` | = |
| `<=` | <= |
| `<` | < |
| `>=` | >= |
| `>` | > |

**`variable`** — Variable

| Value | Label |
|---|---|
| `weight` | Weight |
| `volume` | Volume |
| `wv` | Weight * Volume |
| `price` | Price |
| `quantity` | Quantity |

**`variable_factor`** — Variable Factor

| Value | Label |
|---|---|
| `weight` | Weight |
| `volume` | Volume |
| `wv` | Weight * Volume |
| `price` | Price |
| `quantity` | Quantity |

## `digest.digest`

**`periodicity`** — Periodicity

| Value | Label |
|---|---|
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `quarterly` | Quarterly |

**`state`** — Status

| Value | Label |
|---|---|
| `activated` | Activated |
| `deactivated` | Deactivated |

## `discuss.channel`

**`channel_type`** — Channel Type

| Value | Label |
|---|---|
| `chat` | Chat |
| `channel` | Channel |
| `group` | Group |
| `livechat` | Livechat Conversation |

**`default_display_mode`** — Default Display Mode

| Value | Label |
|---|---|
| `video_full_screen` | Full screen video |

**`livechat_failure`** — Live Chat Session Failure

| Value | Label |
|---|---|
| `no_answer` | Never Answered |
| `no_agent` | No one Available |
| `no_failure` | No Failure |

**`livechat_outcome`** — Livechat Outcome

| Value | Label |
|---|---|
| `no_answer` | Never Answered |
| `no_agent` | No one Available |
| `no_failure` | Success |
| `escalated` | Escalated |

**`livechat_status`** — Livechat Status

| Value | Label |
|---|---|
| `in_progress` | In progress |
| `waiting` | Waiting for customer |
| `need_help` | Looking for help |

**`livechat_week_day`** — Day of the Week

| Value | Label |
|---|---|
| `0` | Monday |
| `1` | Tuesday |
| `2` | Wednesday |
| `3` | Thursday |
| `4` | Friday |
| `5` | Saturday |
| `6` | Sunday |

**`rating_avg_text`** — Rating Avg Text (not stored)

| Value | Label |
|---|---|
| `top` | Happy |
| `ok` | Neutral |
| `ko` | Unhappy |
| `none` | Not Rated yet |

## `discuss.channel.member`

**`custom_notifications`** — Customized Notifications

| Value | Label |
|---|---|
| `all` | All Messages |
| `mentions` | Mentions Only |
| `no_notif` | Nothing |

**`livechat_member_type`** — Livechat Member Type (not stored)

| Value | Label |
|---|---|
| `agent` | Agent |
| `visitor` | Visitor |
| `bot` | Chatbot |

## `event.booth`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`state`** — Status

| Value | Label |
|---|---|
| `available` | Available |
| `unavailable` | Unavailable |

## `event.event`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`badge_format`** — Badge Dimension

| Value | Label |
|---|---|
| `A4_french_fold` | A4 foldable |
| `A6` | A6 |
| `four_per_sheet` | 4 per sheet |

**`kanban_state`** — Kanban State

| Value | Label |
|---|---|
| `normal` | In Progress |
| `done` | Ready for Next Stage |
| `blocked` | Blocked |
| `cancel` | Cancelled |

**`website_visibility`** — Website Visibility

| Value | Label |
|---|---|
| `public` | Public |
| `link` | Via a Link |
| `logged_users` | Logged Users |

## `event.lead.rule`

**`lead_creation_basis`** — Create

| Value | Label |
|---|---|
| `attendee` | Per Attendee |
| `order` | Per Order |

**`lead_creation_trigger`** — When

| Value | Label |
|---|---|
| `create` | Attendees are created |
| `confirm` | Attendees are registered |
| `done` | Attendees attended |

**`lead_type`** — Lead Type

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

## `event.mail`

**`interval_type`** — Trigger 

| Value | Label |
|---|---|
| `after_sub` | After each registration |
| `before_event` | Before the event starts |
| `after_event_start` | After the event started |
| `after_event` | After the event ended |
| `before_event_end` | Before the event ends |

**`interval_unit`** — Unit

| Value | Label |
|---|---|
| `now` | Immediately |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

**`mail_state`** — Global communication Status (not stored)

| Value | Label |
|---|---|
| `running` | Running |
| `scheduled` | Scheduled |
| `sent` | Sent |
| `error` | Error |
| `cancelled` | Cancelled |

**`notification_type`** — Send (not stored)

| Value | Label |
|---|---|
| `mail` | Mail |
| `sms` | SMS |

## `event.question`

**`question_type`** — Question Type

| Value | Label |
|---|---|
| `simple_choice` | Selection |
| `text_box` | Text Input |
| `name` | Name |
| `email` | Email |
| `phone` | Phone |
| `company_name` | Company |

## `event.registration`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`sale_status`** — Sale Status

| Value | Label |
|---|---|
| `to_pay` | Not Sold |
| `sold` | Sold |
| `free` | Free |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Unconfirmed |
| `open` | Registered |
| `done` | Attended |
| `cancel` | Cancelled |

## `event.sale.report`

**`event_registration_state`** — Registration Status

| Value | Label |
|---|---|
| `draft` | Unconfirmed |
| `cancel` | Cancelled |
| `open` | Confirmed |
| `done` | Attended |

**`sale_order_state`** — Sale Order Status

| Value | Label |
|---|---|
| `draft` | Quotation |
| `sent` | Quotation Sent |
| `sale` | Sales Order |
| `cancel` | Cancelled |

**`sale_status`** — Payment Status

| Value | Label |
|---|---|
| `to_pay` | Not Sold |
| `sold` | Sold |
| `free` | Free |

## `event.sponsor`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`exhibitor_type`** — Sponsor Type

| Value | Label |
|---|---|
| `sponsor` | Footer Logo Only |
| `exhibitor` | Exhibitor |
| `online` | Online Exhibitor |

## `event.sponsor.type`

**`display_ribbon_style`** — Ribbon Style

| Value | Label |
|---|---|
| `no_ribbon` | No Ribbon |
| `Gold` | Gold |
| `Silver` | Silver |
| `Bronze` | Bronze |

## `event.track`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`kanban_state`** — Kanban State

| Value | Label |
|---|---|
| `normal` | Grey |
| `done` | Green |
| `blocked` | Red |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Low |
| `1` | Medium |
| `2` | High |
| `3` | Highest |

## `event.type.mail`

**`interval_type`** — Trigger

| Value | Label |
|---|---|
| `after_sub` | After each registration |
| `before_event` | Before the event starts |
| `after_event_start` | After the event started |
| `after_event` | After the event ended |
| `before_event_end` | Before the event ends |

**`interval_unit`** — Unit

| Value | Label |
|---|---|
| `now` | Immediately |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

**`notification_type`** — Send (not stored)

| Value | Label |
|---|---|
| `mail` | Mail |
| `sms` | SMS |

## `fetchmail.server`

**`server_type`** — Server Type

| Value | Label |
|---|---|
| `imap` | IMAP Server |
| `pop` | POP Server |
| `local` | Local Server |
| `gmail` | Gmail OAuth Authentication |
| `outlook` | Outlook OAuth Authentication |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Not Confirmed |
| `done` | Confirmed |

## `fleet.service.type`

**`category`** — Category

| Value | Label |
|---|---|
| `contract` | Contract |
| `service` | Service |

## `fleet.vehicle`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`co2_emission_unit`** — Co2 Emission Unit

| Value | Label |
|---|---|
| `g/km` | g/km |
| `g/mi` | g/mi |

**`contract_state`** — Last Contract State (not stored)

| Value | Label |
|---|---|
| `futur` | Incoming |
| `open` | In Progress |
| `expired` | Expired |
| `closed` | Closed |

**`frame_type`** — Bike Frame Type

| Value | Label |
|---|---|
| `diamant` | Diamant |
| `trapez` | Trapez |
| `wave` | Wave |

**`fuel_type`** — Fuel Type

| Value | Label |
|---|---|
| `diesel` | Diesel |
| `gasoline` | Gasoline |
| `full_hybrid` | Full Hybrid |
| `plug_in_hybrid_diesel` | Plug-in Hybrid Diesel |
| `plug_in_hybrid_gasoline` | Plug-in Hybrid Gasoline |
| `cng` | CNG |
| `lpg` | LPG |
| `hydrogen` | Hydrogen |
| `electric` | Electric |

**`odometer_unit`** — Odometer Unit

| Value | Label |
|---|---|
| `kilometers` | km |
| `miles` | mi |

**`power_unit`** — Power Unit

| Value | Label |
|---|---|
| `power` | kW |
| `horsepower` | Horsepower |

**`range_unit`** — Range Unit

| Value | Label |
|---|---|
| `km` | km |
| `mi` | mi |

**`service_activity`** — Service Activity (not stored)

| Value | Label |
|---|---|
| `none` | None |
| `overdue` | Overdue |
| `today` | Today |

**`transmission`** — Transmission

| Value | Label |
|---|---|
| `manual` | Manual |
| `automatic` | Automatic |

## `fleet.vehicle.cost.report`

**`cost_type`** — Cost Type

| Value | Label |
|---|---|
| `contract` | Contract |
| `service` | Service |

**`vehicle_type`** — Vehicle Type

| Value | Label |
|---|---|
| `car` | Car |
| `bike` | Bike |

## `fleet.vehicle.log.contract`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`cost_frequency`** — Recurring Cost Frequency

| Value | Label |
|---|---|
| `no` | No |
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `yearly` | Yearly |

**`state`** — Status

| Value | Label |
|---|---|
| `futur` | New |
| `open` | Running |
| `expired` | Expired |
| `closed` | Cancelled |

## `fleet.vehicle.log.services`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`state`** — Stage

| Value | Label |
|---|---|
| `new` | New |
| `running` | Running |
| `done` | Done |
| `cancelled` | Cancelled |

## `fleet.vehicle.model`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`co2_emission_unit`** — Co2 Emission Unit (not stored)

| Value | Label |
|---|---|
| `g/km` | g/km |
| `g/mi` | g/mi |

**`default_fuel_type`** — Fuel Type

| Value | Label |
|---|---|
| `diesel` | Diesel |
| `gasoline` | Gasoline |
| `full_hybrid` | Full Hybrid |
| `plug_in_hybrid_diesel` | Plug-in Hybrid Diesel |
| `plug_in_hybrid_gasoline` | Plug-in Hybrid Gasoline |
| `cng` | CNG |
| `lpg` | LPG |
| `hydrogen` | Hydrogen |
| `electric` | Electric |

**`drive_type`** — Drive Type

| Value | Label |
|---|---|
| `fwd` | Front-Wheel Drive (FWD) |
| `awd` | All-Wheel Drive (AWD) |
| `rwd` | Rear-Wheel Drive (RWD) |
| `4wd` | Four-Wheel Drive (4WD) |

**`power_unit`** — Power Unit

| Value | Label |
|---|---|
| `power` | kW |
| `horsepower` | Horsepower (hp) |

**`range_unit`** — Range Unit

| Value | Label |
|---|---|
| `km` | km |
| `mi` | mi |

**`transmission`** — Transmission

| Value | Label |
|---|---|
| `manual` | Manual |
| `automatic` | Automatic |

**`vehicle_type`** — Vehicle Type

| Value | Label |
|---|---|
| `car` | Car |
| `bike` | Bike |

## `forum.forum`

**`default_order`** — Default

| Value | Label |
|---|---|
| `create_date desc` | Newest |
| `last_activity_date desc` | Last Updated |
| `vote_count desc` | Most Voted |
| `relevancy desc` | Relevance |
| `child_count desc` | Answered |

**`mode`** — Mode

| Value | Label |
|---|---|
| `questions` | Questions (1 answer) |
| `discussions` | Discussions (multiple answers) |

**`privacy`** — Privacy

| Value | Label |
|---|---|
| `public` | Public |
| `connected` | Signed In |
| `private` | Some users |

## `forum.post`

**`state`** — Status

| Value | Label |
|---|---|
| `active` | Active |
| `pending` | Waiting Validation |
| `close` | Closed |
| `offensive` | Offensive |
| `flagged` | Flagged |

## `forum.post.reason`

**`reason_type`** — Reason Type

| Value | Label |
|---|---|
| `basic` | Basic |
| `offensive` | Offensive |

## `forum.post.vote`

**`vote`** — Vote

| Value | Label |
|---|---|
| `1` | 1 |
| `-1` | -1 |
| `0` | 0 |

## `gamification.badge`

**`level`** — Forum Badge Level

| Value | Label |
|---|---|
| `bronze` | Bronze |
| `silver` | Silver |
| `gold` | Gold |

**`rule_auth`** — Allowance to Grant

| Value | Label |
|---|---|
| `everyone` | Everyone |
| `users` | A selected list of users |
| `having` | People having some badges |
| `nobody` | No one, assigned through challenges |

## `gamification.challenge`

**`challenge_category`** — Appears in

| Value | Label |
|---|---|
| `hr` | Human Resources / Engagement |
| `other` | Settings / Gamification Tools |
| `certification` | Certifications |
| `forum` | Website / Forum |
| `slides` | Website / Slides |

**`period`** — Periodicity

| Value | Label |
|---|---|
| `once` | Non recurring |
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `yearly` | Yearly |

**`report_message_frequency`** — Report Frequency

| Value | Label |
|---|---|
| `never` | Never |
| `onchange` | On change |
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `yearly` | Yearly |

**`state`** — State

| Value | Label |
|---|---|
| `draft` | Draft |
| `inprogress` | In Progress |
| `done` | Done |

**`visibility_mode`** — Display Mode

| Value | Label |
|---|---|
| `personal` | Individual Goals |
| `ranking` | Leader Board (Group Ranking) |

## `gamification.goal`

**`state`** — State

| Value | Label |
|---|---|
| `draft` | Draft |
| `inprogress` | In progress |
| `reached` | Reached |
| `failed` | Failed |
| `canceled` | Cancelled |

## `gamification.goal.definition`

**`computation_mode`** — Computation Mode

| Value | Label |
|---|---|
| `manually` | Recorded manually |
| `count` | Automatic: number of records |
| `sum` | Automatic: sum on a field |
| `python` | Automatic: execute a specific Python code |

**`condition`** — Goal Performance

| Value | Label |
|---|---|
| `higher` | The higher the better |
| `lower` | The lower the better |

**`display_mode`** — Displayed as

| Value | Label |
|---|---|
| `progress` | Progressive (using numerical values) |
| `boolean` | Exclusive (done or not-done) |

## `google.calendar.account.reset`

**`delete_policy`** — User's Existing Events

| Value | Label |
|---|---|
| `dont_delete` | Leave them untouched |
| `delete_google` | Delete from the current Google Calendar account |
| `delete_system` | Delete from the system |
| `delete_both` | Delete from both |

**`sync_policy`** — Next Synchronization

| Value | Label |
|---|---|
| `new` | Synchronize only new events |
| `all` | Synchronize all existing events |

## `hr.applicant`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`application_status`** — Application Status (not stored)

| Value | Label |
|---|---|
| `ongoing` | Ongoing |
| `hired` | Hired |
| `refused` | Refused |
| `archived` | Archived |

**`kanban_state`** — Kanban State

| Value | Label |
|---|---|
| `normal` | In Progress |
| `done` | Ready for Next Stage |
| `waiting` | Waiting |
| `blocked` | Blocked |

**`priority`** — Evaluation

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Good |
| `2` | Very Good |
| `3` | Excellent |

## `hr.attendance`

**`in_mode`** — Mode

| Value | Label |
|---|---|
| `kiosk` | Kiosk |
| `systray` | Systray |
| `manual` | Manual |
| `technical` | Technical |

**`out_mode`** — Out Mode

| Value | Label |
|---|---|
| `kiosk` | Kiosk |
| `systray` | Systray |
| `manual` | Manual |
| `technical` | Technical |
| `auto_check_out` | Automatic Check-Out |

**`overtime_status`** — Overtime Status

| Value | Label |
|---|---|
| `to_approve` | To Approve |
| `approved` | Approved |
| `refused` | Refused |

## `hr.attendance.overtime.line`

**`status`** — Status

| Value | Label |
|---|---|
| `to_approve` | To Approve |
| `approved` | Approved |
| `refused` | Refused |

## `hr.attendance.overtime.rule`

**`base_off`** — Based Off

| Value | Label |
|---|---|
| `quantity` | Quantity |
| `timing` | Timing |

**`quantity_period`** — Quantity Period

| Value | Label |
|---|---|
| `day` | Day |
| `week` | Week |

**`timing_type`** — Timing Type

| Value | Label |
|---|---|
| `work_days` | On any working day |
| `non_work_days` | On any non-working day |
| `leave` | When employee is off |
| `schedule` | Outside of a specific schedule |

## `hr.attendance.overtime.ruleset`

**`rate_combination_mode`** — Rate Combination Mode

| Value | Label |
|---|---|
| `max` | Maximum Rate |
| `sum` | Sum of all rates |

## `hr.department`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

## `hr.employee`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`attendance_state`** — Attendance Status (not stored)

| Value | Label |
|---|---|
| `checked_out` | Checked out |
| `checked_in` | Checked in |

**`current_leave_state`** — Current Time Off Status (not stored)

| Value | Label |
|---|---|
| `confirm` | Waiting Approval |
| `refuse` | Refused |
| `validate1` | Waiting Second Approval |
| `validate` | Approved |
| `cancel` | Cancelled |

**`hr_icon_display`** — Hr Icon Display (not stored)

| Value | Label |
|---|---|
| `presence_present` | Present |
| `presence_out_of_working_hour` | Off-Hours |
| `presence_absent` | Absent |
| `presence_archive` | Archived |
| `presence_undetermined` | Undetermined |
| `presence_holiday_absent` | On leave |
| `presence_holiday_present` | Present but on leave |
| `presence_home` | At Home |
| `presence_office` | At Office |
| `presence_other` | At Other |

**`hr_presence_state`** — Hr Presence State (not stored)

| Value | Label |
|---|---|
| `present` | Present |
| `absent` | Absent |
| `archive` | Archived |
| `out_of_working_hour` | Off-Hours |

**`hr_presence_state_display`** — Hr Presence State Display

| Value | Label |
|---|---|
| `out_of_working_hour` | Off-Hours |
| `present` | Present |
| `absent` | Absent |

**`work_location_type`** — Work Location Type (not stored)

| Value | Label |
|---|---|
| `home` | Home |
| `office` | Office |
| `other` | Other |

## `hr.employee.public`

**`hr_presence_state`** — Hr Presence State (not stored)

| Value | Label |
|---|---|
| `present` | Present |
| `absent` | Absent |
| `archive` | Archived |
| `out_of_working_hour` | Off-Hours |

**`hr_presence_state_display`** — Hr Presence State Display

| Value | Label |
|---|---|
| `out_of_working_hour` | Off-Hours |
| `present` | Present |
| `absent` | Absent |

## `hr.expense`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`approval_state`** — Approval State

| Value | Label |
|---|---|
| `submitted` | Submitted |
| `approved` | Approved |
| `refused` | Refused |

**`payment_mode`** — Paid By

| Value | Label |
|---|---|
| `own_account` | Employee (to reimburse) |
| `company_account` | Company |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `submitted` | Submitted |
| `approved` | Approved |
| `posted` | Posted |
| `in_payment` | In Payment |
| `paid` | Paid |
| `refused` | Refused |

## `hr.expense.split`

**`approval_state`** — Approval State

| Value | Label |
|---|---|
| `submitted` | Submitted |
| `approved` | Approved |
| `refused` | Refused |

## `hr.holidays.summary.employee`

**`holiday_type`** — Select Time Off Type

| Value | Label |
|---|---|
| `Approved` | Approved |
| `Confirmed` | Confirmed |
| `both` | Both Approved and Confirmed |

## `hr.job`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

## `hr.leave`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`request_date_from_period`** — Date Period Start

| Value | Label |
|---|---|
| `am` | Morning |
| `pm` | Afternoon |

**`request_date_to_period`** — Date Period End

| Value | Label |
|---|---|
| `am` | Morning |
| `pm` | Afternoon |

**`state`** — Status

| Value | Label |
|---|---|
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |
| `cancel` | Cancelled |

## `hr.leave.accrual.level`

**`accrual_validity_type`** — Accrual Validity Type

| Value | Label |
|---|---|
| `day` | Days |
| `month` | Months |

**`action_with_unused_accruals`** — Action With Unused Accruals

| Value | Label |
|---|---|
| `lost` | Lost |
| `all` | Carried over |

**`added_value_type`** — Added Value Type

| Value | Label |
|---|---|
| `day` | Day(s) |
| `hour` | Hour(s) |

**`carryover_options`** — Carryover Options

| Value | Label |
|---|---|
| `unlimited` | Unlimited |
| `limited` | Up to |

**`first_month`** — First Month

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |

**`frequency`** — Frequency

| Value | Label |
|---|---|
| `hourly` | Hourly |
| `daily` | Daily |
| `weekly` | Weekly |
| `bimonthly` | Twice a month |
| `monthly` | Monthly |
| `biyearly` | Twice a year |
| `yearly` | Yearly |
| `worked_hours` | Per Hour Worked |

**`milestone_date`** — Milestone Date

| Value | Label |
|---|---|
| `creation` | At allocation creation |
| `after` | After |

**`second_month`** — Second Month

| Value | Label |
|---|---|
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

**`start_type`** — Start Type

| Value | Label |
|---|---|
| `day` | Days |
| `month` | Months |
| `year` | Years |

**`week_day`** — Allocation on

| Value | Label |
|---|---|
| `0` | Monday |
| `1` | Tuesday |
| `2` | Wednesday |
| `3` | Thursday |
| `4` | Friday |
| `5` | Saturday |
| `6` | Sunday |

**`yearly_month`** — Yearly Month

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

## `hr.leave.accrual.plan`

**`accrued_gain_time`** — Accrued Gain Time

| Value | Label |
|---|---|
| `start` | At the start of the accrual period |
| `end` | At the end of the accrual period |

**`added_value_type`** — Added Value Type

| Value | Label |
|---|---|
| `day` | Days |
| `hour` | Hours |

**`carryover_date`** — Carry-Over Time

| Value | Label |
|---|---|
| `year_start` | At the start of the year |
| `allocation` | At the allocation date |
| `other` | Custom date |

**`carryover_month`** — Carryover Month

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

**`transition_mode`** — Transition Mode

| Value | Label |
|---|---|
| `immediately` | Immediately |
| `end_of_accrual` | After this accrual's period |

## `hr.leave.allocation`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`allocation_type`** — Allocation Type

| Value | Label |
|---|---|
| `regular` | Regular Allocation |
| `accrual` | Accrual Allocation |

**`state`** — Status

| Value | Label |
|---|---|
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

**`type_request_unit`** — Type Request Unit (not stored)

| Value | Label |
|---|---|
| `hour` | Hours |
| `half_day` | Half-Day |
| `day` | Day |

## `hr.leave.allocation.generate.multi.wizard`

**`allocation_mode`** — Allocation Mode

| Value | Label |
|---|---|
| `employee` | By Employee |
| `company` | By Company |
| `department` | By Department |
| `category` | By Employee Tag |

**`allocation_type`** — Allocation Type

| Value | Label |
|---|---|
| `regular` | Regular Allocation |
| `accrual` | Based on Accrual Plan |

## `hr.leave.employee.type.report`

**`holiday_status`** — Holiday Status

| Value | Label |
|---|---|
| `taken` | Taken |
| `left` | Left |
| `planned` | Planned |

**`state`** — Status

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

## `hr.leave.generate.multi.wizard`

**`allocation_mode`** — Allocation Mode

| Value | Label |
|---|---|
| `employee` | By Employee |
| `company` | By Company |
| `department` | By Department |
| `category` | By Employee Tag |

## `hr.leave.report`

**`leave_type`** — Request Type

| Value | Label |
|---|---|
| `allocation` | Allocation |
| `request` | Time Off |

**`state`** — Status

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

## `hr.leave.report.calendar`

**`state`** — State

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

## `hr.leave.type`

**`allocation_validation_type`** — Approval

| Value | Label |
|---|---|
| `no_validation` | None needed |
| `hr` | By Time Off Officer |
| `manager` | By Employee's Approver |
| `both` | By Employee's Approver and Time Off Officer |

**`leave_validation_type`** — Time Off Validation

| Value | Label |
|---|---|
| `no_validation` | None needed |
| `hr` | By Time Off Officer |
| `manager` | By Employee's Approver |
| `both` | By Employee's Approver and Time Off Officer |

**`request_unit`** — Duration Type

| Value | Label |
|---|---|
| `day` | Day |
| `half_day` | Half-Day |
| `hour` | Hours |

**`time_type`** — Kind of Time Off

| Value | Label |
|---|---|
| `other` | Worked Time |
| `leave` | Absence |

## `hr.resume.line`

**`course_type`** — Course Type

| Value | Label |
|---|---|
| `external` | External |
| `onsite` | Onsite |
| `elearning` | eLearning |

**`expiration_status`** — Expiration Status

| Value | Label |
|---|---|
| `expired` | Expired |
| `expiring` | Expiring |
| `valid` | Valid |

## `hr.version`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`distance_home_work_unit`** — Home-Work Distance unit

| Value | Label |
|---|---|
| `kilometers` | km |
| `miles` | mi |

**`employee_type`** — Employee Type

| Value | Label |
|---|---|
| `employee` | Employee |
| `worker` | Worker |
| `student` | Student |
| `trainee` | Trainee |
| `contractor` | Contractor |
| `freelance` | Freelancer |

**`sex`** — Gender

| Value | Label |
|---|---|
| `male` | Male |
| `female` | Female |
| `other` | Other |

**`work_entry_source`** — Work Entry Source

| Value | Label |
|---|---|
| `calendar` | Working Schedule |

## `hr.work.entry`

**`state`** — State

| Value | Label |
|---|---|
| `draft` | New |
| `conflict` | In Conflict |
| `validated` | In Payslip |
| `cancelled` | Cancelled |

## `hr.work.location`

**`location_type`** — Cover Image

| Value | Label |
|---|---|
| `home` | Home |
| `office` | Office |
| `other` | Other |

## `html_editor.converter.test`

**`selection_str`** — Lorsqu'un pancake prend l'avion à destination de Toronto et qu'il fait une escale technique à St Claude, on dit:

| Value | Label |
|---|---|
| `A` | Qu'il n'est pas arrivé à Toronto |
| `B` | Qu'il était supposé arriver à Toronto |
| `C` | Qu'est-ce qu'il fout ce maudit pancake, tabernacle ? |
| `D` | La réponse D |

## `iap.account`

**`state`** — State

| Value | Label |
|---|---|
| `banned` | Banned |
| `registered` | Registered |
| `unregistered` | Unregistered |

## `im_livechat.channel`

**`max_sessions_mode`** — Sessions per Operator

| Value | Label |
|---|---|
| `unlimited` | Unlimited |
| `limited` | Limited |

## `im_livechat.channel.member.history`

**`help_status`** — Help Status

| Value | Label |
|---|---|
| `requested` | Help Requested |
| `provided` | Help Provided |

**`livechat_member_type`** — Livechat Member Type

| Value | Label |
|---|---|
| `agent` | Agent |
| `visitor` | Visitor |
| `bot` | Chatbot |

## `im_livechat.channel.rule`

**`action`** — Live Chat Button

| Value | Label |
|---|---|
| `display_button` | Show |
| `display_button_and_text` | Show with notification |
| `auto_popup` | Open automatically |
| `hide_button` | Hide |

**`chatbot_enabled_condition`** — Enable ChatBot

| Value | Label |
|---|---|
| `always` | Always |
| `only_if_no_operator` | Only when no operator is available |
| `only_if_operator` | Only when an operator is available |

## `im_livechat.report.channel`

**`day_number`** — Day of the Week

| Value | Label |
|---|---|
| `0` | Sunday |
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |

**`session_outcome`** — Session Outcome

| Value | Label |
|---|---|
| `no_answer` | Never Answered |
| `no_agent` | No one Available |
| `no_failure` | Success |
| `escalated` | Escalated |

## `ir.actions.act_url`

**`binding_type`** — Binding Type

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

**`target`** — Action Target

| Value | Label |
|---|---|
| `new` | New Window |
| `self` | This Window |
| `download` | Download |

## `ir.actions.act_window`

**`binding_type`** — Binding Type

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

**`target`** — Target Window

| Value | Label |
|---|---|
| `current` | Current Window |
| `new` | New Window |
| `fullscreen` | Full Screen |
| `main` | Main action of Current Window |

## `ir.actions.act_window.view`

**`view_mode`** — View Type

| Value | Label |
|---|---|
| `list` | List |
| `form` | Form |
| `graph` | Graph |
| `pivot` | Pivot |
| `calendar` | Calendar |
| `kanban` | Kanban |
| `hierarchy` | Hierarchy |
| `activity` | Activity |

## `ir.actions.act_window_close`

**`binding_type`** — Binding Type

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

## `ir.actions.actions`

**`binding_type`** — Binding Type

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

## `ir.actions.client`

**`binding_type`** — Binding Type

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

**`target`** — Target Window

| Value | Label |
|---|---|
| `current` | Current Window |
| `new` | New Window |
| `fullscreen` | Full Screen |
| `main` | Main action of Current Window |

## `ir.actions.report`

**`binding_type`** — Binding Type

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

**`report_type`** — Report Type

| Value | Label |
|---|---|
| `qweb-html` | HTML |
| `qweb-pdf` | PDF |
| `qweb-text` | Text |

## `ir.actions.server`

**`activity_date_deadline_range_type`** — Due type

| Value | Label |
|---|---|
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`activity_user_type`** — User Type

| Value | Label |
|---|---|
| `specific` | Specific User |
| `generic` | Dynamic User (based on record) |

**`binding_type`** — Binding Type

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

**`evaluation_type`** — Value Type

| Value | Label |
|---|---|
| `value` | Update |
| `sequence` | Sequence |
| `equation` | Compute |

**`followers_type`** — Followers Type

| Value | Label |
|---|---|
| `specific` | Specific Followers |
| `generic` | Dynamic Followers |

**`mail_post_method`** — Send Email As

| Value | Label |
|---|---|
| `email` | Email |
| `comment` | Message |
| `note` | Note |

**`sms_method`** — Send SMS As

| Value | Label |
|---|---|
| `sms` | SMS (without note) |
| `comment` | SMS (with note) |
| `note` | Note only |

**`state`** — Type

| Value | Label |
|---|---|
| `object_write` | Update Record |
| `object_create` | Create Record |
| `object_copy` | Duplicate Record |
| `next_activity` | Create Activity |
| `mail_post` | Send Email |
| `sms` | Send SMS |
| `followers` | Add Followers |
| `remove_followers` | Remove Followers |
| `code` | Execute Code |
| `webhook` | Send Webhook Notification |
| `multi` | Multi Actions |

**`update_boolean_value`** — Boolean Value

| Value | Label |
|---|---|
| `true` | Yes (True) |
| `false` | No (False) |

**`update_m2m_operation`** — Many2many Operations

| Value | Label |
|---|---|
| `add` | Adding |
| `remove` | Removing |
| `set` | Setting it to |
| `clear` | Clearing it |

**`usage`** — Usage

| Value | Label |
|---|---|
| `ir_actions_server` | Server Action |
| `ir_cron` | Scheduled Action |
| `base_automation` | Automation Rule |

**`value_field_to_show`** — Value Field To Show (not stored)

| Value | Label |
|---|---|
| `value` | value |
| `html_value` | html_value |
| `sequence_id` | sequence_id |
| `resource_ref` | reference |
| `update_boolean_value` | update_boolean_value |
| `selection_value` | selection_value |

## `ir.actions.todo`

**`state`** — Status

| Value | Label |
|---|---|
| `open` | To Do |
| `done` | Done |

## `ir.asset`

**`directive`** — Directive

| Value | Label |
|---|---|
| `append` | Append |
| `prepend` | Prepend |
| `after` | After |
| `before` | Before |
| `remove` | Remove |
| `replace` | Replace |
| `include` | Include |

## `ir.attachment`

**`type`** — Type

| Value | Label |
|---|---|
| `url` | URL |
| `binary` | File |
| `cloud_storage` | Cloud Storage |

## `ir.cron`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`interval_type`** — Interval Unit

| Value | Label |
|---|---|
| `minutes` | Minutes |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

## `ir.logging`

**`type`** — Type

| Value | Label |
|---|---|
| `client` | Client |
| `server` | Server |

## `ir.mail_server`

**`smtp_authentication`** — Authenticate with

| Value | Label |
|---|---|
| `login` | Username |
| `certificate` | SSL Certificate |
| `cli` | Command Line Interface |
| `gmail` | Gmail OAuth Authentication |
| `outlook` | Outlook OAuth Authentication |

**`smtp_encryption`** — Connection Encryption

| Value | Label |
|---|---|
| `none` | None |
| `starttls_strict` | TLS (STARTTLS), encryption and validation |
| `starttls` | TLS (STARTTLS), encryption only |
| `ssl_strict` | SSL/TLS, encryption and validation |
| `ssl` | SSL/TLS, encryption only |

## `ir.model`

**`state`** — Type

| Value | Label |
|---|---|
| `manual` | Custom Object |
| `base` | Base Object |

## `ir.model.fields`

**`on_delete`** — On Delete

| Value | Label |
|---|---|
| `cascade` | Cascade |
| `set null` | Set NULL |
| `restrict` | Restrict |

**`state`** — Type

| Value | Label |
|---|---|
| `manual` | Custom Field |
| `base` | Base Field |

**`translate`** — Translatable

| Value | Label |
|---|---|
| `standard` | Translate as a whole |
| `html_translate` | Translate HTML terms |
| `xml_translate` | Translate XML terms |

**`ttype`** — Field Type

| Value | Label |
|---|---|
| `binary` | binary |
| `boolean` | boolean |
| `char` | char |
| `date` | date |
| `datetime` | datetime |
| `float` | float |
| `html` | html |
| `integer` | integer |
| `json` | json |
| `many2many` | many2many |
| `many2one` | many2one |
| `many2one_reference` | many2one_reference |
| `monetary` | monetary |
| `one2many` | one2many |
| `properties` | properties |
| `properties_definition` | properties_definition |
| `reference` | reference |
| `selection` | selection |
| `text` | text |
| `serialized` | serialized |

## `ir.module.module`

**`license`** — License

| Value | Label |
|---|---|
| `GPL-2` | GPL Version 2 |
| `GPL-2 or any later version` | GPL-2 or later version |
| `GPL-3` | GPL Version 3 |
| `GPL-3 or any later version` | GPL-3 or later version |
| `AGPL-3` | Affero GPL-3 |
| `LGPL-3` | LGPL Version 3 |
| `Other OSI approved licence` | Other OSI Approved License |
| `OEEL-1` | the enterprise edition Edition License v1.0 |
| `OPL-1` | Proprietary License, version 1.0 |
| `Other proprietary` | Other Proprietary |

**`module_type`** — Module Type

| Value | Label |
|---|---|
| `official` | Official Apps |
| `industries` | Industries |

**`state`** — Status

| Value | Label |
|---|---|
| `uninstallable` | Uninstallable |
| `uninstalled` | Not Installed |
| `installed` | Installed |
| `to upgrade` | To be upgraded |
| `to remove` | To be removed |
| `to install` | To be installed |

## `ir.module.module.dependency`

**`state`** — Status (not stored)

| Value | Label |
|---|---|
| `uninstallable` | Uninstallable |
| `uninstalled` | Not Installed |
| `installed` | Installed |
| `to upgrade` | To be upgraded |
| `to remove` | To be removed |
| `to install` | To be installed |
| `unknown` | Unknown |

## `ir.module.module.exclusion`

**`state`** — Status (not stored)

| Value | Label |
|---|---|
| `uninstallable` | Uninstallable |
| `uninstalled` | Not Installed |
| `installed` | Installed |
| `to upgrade` | To be upgraded |
| `to remove` | To be removed |
| `to install` | To be installed |
| `unknown` | Unknown |

## `ir.sequence`

**`implementation`** — Implementation

| Value | Label |
|---|---|
| `standard` | Standard |
| `no_gap` | No gap |

## `ir.ui.view`

**`mode`** — View inheritance mode

| Value | Label |
|---|---|
| `primary` | Base view |
| `extension` | Extension View |

**`type`** — View Type

| Value | Label |
|---|---|
| `list` | List |
| `form` | Form |
| `graph` | Graph |
| `pivot` | Pivot |
| `calendar` | Calendar |
| `kanban` | Kanban |
| `search` | Search |
| `qweb` | QWeb |
| `hierarchy` | Hierarchy |
| `activity` | Activity |

**`visibility`** — Visibility

| Value | Label |
|---|---|
| `` | Public |
| `connected` | Signed In |
| `restricted_group` | Restricted Group |
| `password` | With Password |

## `l10n.fr.pdp.reports.flow`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`operation_type`** — Operation Type

| Value | Label |
|---|---|
| `sale` | Sales |
| `purchase` | Acquisitions |

**`period_status`** — Period Status (not stored)

| Value | Label |
|---|---|
| `open` | Open |
| `grace` | Grace |
| `closed` | Closed |

**`report_type`** — Report Type

| Value | Label |
|---|---|
| `transaction` | Transaction |
| `payment` | Payment |

**`state`** — Status

| Value | Label |
|---|---|
| `ready` | Ready |
| `error` | Error |
| `sent` | Sent |
| `completed` | Completed |

**`transmission_type`** — Transmission Type (not stored)

| Value | Label |
|---|---|
| `initial` | Initial |
| `rectificative` | Rectificative |

## `l10n.in.ewaybill`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`blocking_level`** — Blocking Level

| Value | Label |
|---|---|
| `warning` | Warning |
| `error` | Error |

**`cancel_reason`** — Cancel reason

| Value | Label |
|---|---|
| `1` | Duplicate |
| `2` | Data Entry Mistake |
| `3` | Order Cancelled |
| `4` | Others |

**`mode`** — Transportation Mode

| Value | Label |
|---|---|
| `1` | By Road |
| `2` | Rail |
| `3` | Air |
| `4` | Ship or Ship Cum Road/Rail |

**`state`** — Status

| Value | Label |
|---|---|
| `pending` | Pending |
| `generated` | Generated |
| `cancel` | Cancelled |
| `challan` | Challan |

**`supply_type`** — Supply Type (not stored)

| Value | Label |
|---|---|
| `O` | Outward |
| `I` | Inward |

**`vehicle_type`** — Vehicle Type

| Value | Label |
|---|---|
| `R` | Regular |
| `O` | Over Dimensional Cargo |

## `l10n.in.ewaybill.cancel`

**`cancel_reason`** — Cancel Reason

| Value | Label |
|---|---|
| `1` | Duplicate |
| `2` | Data Entry Mistake |
| `3` | Order Cancelled |
| `4` | Others |

## `l10n.in.ewaybill.type`

**`allowed_supply_type`** — Allowed for supply type

| Value | Label |
|---|---|
| `both` | Incoming and Outgoing |
| `out` | Outgoing |
| `in` | Incoming |

## `l10n_es_edi_tbai.document`

**`state`** — status

| Value | Label |
|---|---|
| `to_send` | To Send |
| `accepted` | Accepted |
| `rejected` | Rejected |

## `l10n_es_edi_verifactu.document`

**`document_type`** — Document Type

| Value | Label |
|---|---|
| `submission` | Submission |
| `cancellation` | Cancellation |

**`state`** — Status

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |

## `l10n_fr.fec.export.wizard`

**`export_type`** — Export Type

| Value | Label |
|---|---|
| `official` | Official FEC report (posted entries only) |
| `nonofficial` | Non-official FEC report (posted and unposted entries) |

## `l10n_gr_edi.document`

**`provider_pdf_state`** — Final PDF Status

| Value | Label |
|---|---|
| `pending` | Pending |
| `sent` | Sent |
| `error` | Failed |

**`state`** — myDATA Status

| Value | Label |
|---|---|
| `invoice_sent` | Invoice sent |
| `invoice_error` | Invoice send failed |
| `bill_fetched` | Expense classification ready to send |
| `bill_sent` | Expense classification sent |
| `bill_error` | Expense classification send failed |
| `invoice_pending` | Invoice submission pending |

## `l10n_gr_edi.preferred_classification`

**`l10n_gr_edi_cls_category`** — myDATA Category

| Value | Label |
|---|---|
| `category1_1` | 1.1 - Commodity Sale Income |
| `category1_2` | 1.2 - Product Sale Income |
| `category1_3` | 1.3 - Provision of Services Income |
| `category1_4` | 1.4 - Sale of Fixed Assets Income |
| `category1_5` | 1.5 - Other Income/Profits |
| `category1_6` | 1.6 - Self-Deliveries/Self-Supplies |
| `category1_7` | 1.7 - Income on behalf of Third Parties |
| `category1_8` | 1.8 - Past fiscal years income |
| `category1_9` | 1.9 - Future fiscal years income |
| `category1_10` | 1.10 - Other Income Adjustment/Regularisation Entries |
| `category1_95` | 1.95 - Other Income-related Information |
| `category2_1` | 2.1 - Commodity Purchases |
| `category2_2` | 2.2 - Raw and Adjuvant Material Purchases |
| `category2_3` | 2.3 - Services Receipt |
| `category2_4` | 2.4 - General Expenses Subject to VAT Deduction |
| `category2_5` | 2.5 - General Expenses Not Subject to VAT Deduction |
| `category2_6` | 2.6 - Personnel Fees and Benefits |
| `category2_7` | 2.7 - Fixed Asset Purchases |
| `category2_8` | 2.8 - Fixed Asset Amortisations |
| `category2_9` | 2.9 - Expenses on behalf of Third Parties |
| `category2_10` | 2.10 - Past fiscal years expenses |
| `category2_11` | 2.11 - Future fiscal years expenses |
| `category2_12` | 2.12 - Other Expense Adjustment/Regularisation Entries |
| `category2_13` | 2.13 - Stock at Period Start |
| `category2_14` | 2.14 - Stock at Period End |
| `category2_95` | 2.95 - Other Expense-related Information |

**`l10n_gr_edi_cls_type`** — myDATA Type

| Value | Label |
|---|---|
| `E3_106` | E3_106 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Commodities |
| `E3_205` | E3_205 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials |
| `E3_210` | E3_210 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress |
| `E3_305` | E3_305 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials |
| `E3_310` | E3_310 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress |
| `E3_318` | E3_318 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Production expenses |
| `E3_561_001` | E3_561_001 - Wholesale Sales of Goods and Services – for Traders |
| `E3_561_002` | E3_561_002 - Wholesale Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000) |
| `E3_561_003` | E3_561_003 - Retail Sales of Goods and Services – Private Clientele |
| `E3_561_004` | E3_561_004 - Retail Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000) |
| `E3_561_005` | E3_561_005 - Intra-Community Foreign Sales of Goods and Services |
| `E3_561_006` | E3_561_006 - Third Country Foreign Sales of Goods and Services |
| `E3_561_007` | E3_561_007 - Other Sales of Goods and Services |
| `E3_562` | E3_562 - Other Ordinary Income |
| `E3_563` | E3_563 - Credit Interest and Related Income |
| `E3_564` | E3_564 - Credit Exchange Differences |
| `E3_565` | E3_565 - Income from Participations |
| `E3_566` | E3_566 - Profits from Disposing Non-Current Assets |
| `E3_567` | E3_567 - Profits from the Reversal of Provisions and Impairments |
| `E3_568` | E3_568 - Profits from Measurement at Fair Value |
| `E3_570` | E3_570 - Extraordinary income and profits |
| `E3_595` | E3_595 - Self-Production Expenses |
| `E3_596` | E3_596 - Subsidies - Grants |
| `E3_597` | E3_597 - Subsidies – Grants for Investment Purposes – Expense Coverage |
| `E3_880_001` | E3_880_001 - Wholesale Sales of Fixed Assets |
| `E3_880_002` | E3_880_002 - Retail Sales of Fixed Assets |
| `E3_880_003` | E3_880_003 - Intra-Community Foreign Sales of Fixed Assets |
| `E3_880_004` | E3_880_004 - Third Country Foreign Sales of Fixed Assets |
| `E3_881_001` | E3_881_001 - Wholesale Sales on behalf of Third Parties |
| `E3_881_002` | E3_881_002 - Retail Sales on behalf of Third Parties |
| `E3_881_003` | E3_881_003 - Intra-Community Foreign Sales on behalf of Third Parties |
| `E3_881_004` | E3_881_004 - Third Country Foreign Sales on behalf of Third Parties |
| `E3_598_001` | E3_598_001 - Sales of goods belonging to excise duty |
| `E3_598_003` | E3_598_003 - Sales on behalf of farmers through an agricultural cooperative e.t.c. |
| `E3_101` | E3_101 - Commodities at Period Start |
| `E3_102_001` | E3_102_001 - Fiscal Year Commodity Purchases (net amount)/Wholesale |
| `E3_102_002` | E3_102_002 - Fiscal Year Commodity Purchases (net amount)/Retail |
| `E3_102_003` | E3_102_003 - Fiscal Year Commodity Purchases (net amount)/Goods under article 39a paragraph 5 of the VAT Code (Law 2859/2000) |
| `E3_102_004` | E3_102_004 - Fiscal Year Commodity Purchases (net amount)/Foreign, Intra-Community |
| `E3_102_005` | E3_102_005 - Fiscal Year Commodity Purchases (net amount)/Foreign, Third Countries |
| `E3_102_006` | E3_102_006 - Fiscal Year Commodity Purchases (net amount)/Others |
| `E3_104` | E3_104 - Commodities at Period End |
| `E3_201` | E3_201 - Raw and Other Materials at Period Start/Production |
| `E3_202_001` | E3_202_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale |
| `E3_202_002` | E3_202_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail |
| `E3_202_003` | E3_202_003 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Intra-Community |
| `E3_202_004` | E3_202_004 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Third Countries |
| `E3_202_005` | E3_202_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others |
| `E3_204` | E3_204 - Raw and Other Material Stock at Period End/Production |
| `E3_207` | E3_207 - Products and Production in Progress at Period Start/Production |
| `E3_209` | E3_209 - Products and Production in Progress at Period End/Production |
| `E3_301` | E3_301 - Raw and Other Material at Period Start/Agricultural |
| `E3_302_001` | E3_302_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale |
| `E3_302_002` | E3_302_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail |
| `E3_302_003` | E3_302_003 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Intra-Community |
| `E3_302_004` | E3_302_004 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Third Countries |
| `E3_302_005` | E3_302_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others |
| `E3_304` | E3_304 - Raw and Other Material Stock at Period End/Agricultural |
| `E3_307` | E3_307 - Products and Production in Progress at Period Start/ Agricultural |
| `E3_309` | E3_309 - Products and Production in Progress at Period End/ Agricultural |
| `E3_312` | E3_312 - Stock at Period Start (Animals-Plants) |
| `E3_313_001` | E3_313_001 - Animal-Plant Purchases (net amount)/Wholesale |
| `E3_313_002` | E3_313_002 - Animal-Plant Purchases (net amount)/Retail |
| `E3_313_003` | E3_313_003 - Animal-Plant Purchases (net amount)/ Foreign, Intra-Community |
| `E3_313_004` | E3_313_004 - Animal-Plant Purchases (net amount)/ Foreign, Third Countries |
| `E3_313_005` | E3_313_005 - Animal-Plant Purchases/Others |
| `E3_315` | E3_315 - Stock at Period End (Animals-Plants)/Agricultural |
| `E3_581_001` | E3_581_001 - Employee Benefits/Gross Earnings |
| `E3_581_002` | E3_581_002 - Employee Benefits/Employer Contributions |
| `E3_581_003` | E3_581_003 - Employee Benefits/Other Benefits |
| `E3_582` | E3_582 - Asset Measurement Damages |
| `E3_583` | E3_583 - Debit Exchange Differences |
| `E3_584` | E3_584 - Damages from Disposing-Withdrawing Non-Current Assets |
| `E3_585_001` | E3_585_001 - Foreign/Domestic Management Fees |
| `E3_585_002` | E3_585_002 - Expenditures from Linked Enterprises |
| `E3_585_003` | E3_585_003 - Expenditures from Non-Cooperative States or Privileged Tax Regimes |
| `E3_585_004` | E3_585_004 - Expenditures for Information Day-Events |
| `E3_585_005` | E3_585_005 - Reception and Hospitality Expenses |
| `E3_585_006` | E3_585_006 - Travel expenses |
| `E3_585_007` | E3_585_007 - Self-Employed Social Security Contributions |
| `E3_585_008` | E3_585_008 - Commission Agent Expenses and Fees on behalf of Farmers |
| `E3_585_009` | E3_585_009 - Other Fees for Domestic Services |
| `E3_585_010` | E3_585_010 - Other Fees for Foreign Services |
| `E3_585_011` | E3_585_011 - Energy |
| `E3_585_012` | E3_585_012 - Water |
| `E3_585_013` | E3_585_013 - Telecommunications |
| `E3_585_014` | E3_585_014 - Rents |
| `E3_585_015` | E3_585_015 - Advertisement and promotion |
| `E3_585_016` | E3_585_016 - Other expenses |
| `E3_586` | E3_586 - Debit interests and related expenses |
| `E3_587` | E3_587 - Amortisations |
| `E3_588` | E3_588 - Extraordinary expenses, damages and fines |
| `E3_589` | E3_589 - Provisions (except for Personnel Provisions) |
| `E3_882_001` | E3_882_001 - Fiscal Year Tangible Asset Purchases/Wholesale |
| `E3_882_002` | E3_882_002 - Fiscal Year Tangible Asset Purchases/Retail |
| `E3_882_003` | E3_882_003 - Fiscal Year Tangible Asset Purchases/ Intra-Community Foreign |
| `E3_882_004` | E3_882_004 - Fiscal Year Tangible Asset Purchases/ Third Country Foreign |
| `E3_883_001` | E3_883_001 - Fiscal Year Intangible Asset Purchases/Wholesale |
| `E3_883_002` | E3_883_002 - Fiscal Year Intangible Asset Purchases/Retail |
| `E3_883_003` | E3_883_003 - Fiscal Year Intangible Asset Purchases/ Intra-Community Foreign |
| `E3_883_004` | E3_883_004 - Fiscal Year Intangible Asset Purchases/ Third Country Foreign |
| `E3_103` | E3_103 - Impairment of goods |
| `E3_203` | E3_203 - Impairment of raw materials and supplies |
| `E3_303` | E3_303 - Impairment of raw materials and supplies |
| `E3_208` | E3_208 - Impairment of products and production in progress |
| `E3_308` | E3_308 - Impairment of products and production in progress |
| `E3_314` | E3_314 - Impairment of animals-plants - goods |
| `XE3_106` | E3_106 - Own production of fixed assets – Self Deliveries – Inventory Disasters |
| `XE3_205` | E3_205 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_305` | E3_305 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_210` | E3_210 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_310` | E3_310 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `XE3_318` | E3_318 - Own production of fixed assets - Self Deliveries – Inventory Disasters |
| `E3_598_002` | E3_598_002 - Purchases of goods falling into excise duty |

**`l10n_gr_edi_inv_type`** — Invoice Type

| Value | Label |
|---|---|
| `1.1` | 1.1 - Sales Invoice |
| `1.2` | 1.2 - Sales Invoice/Intra-community Supplies |
| `1.3` | 1.3 - Sales Invoice/Third Country Supplies |
| `1.4` | 1.4 - Sales Invoice/Sale on Behalf of Third Parties |
| `1.5` | 1.5 - Sales Invoice/Clearance of Sales on Behalf of Third Parties – Fees from Sales on Behalf of Third Parties |
| `1.6` | 1.6 - Sales Invoice/Supplemental Accounting Source Document |
| `2.1` | 2.1 - Service Rendered Invoice |
| `2.2` | 2.2 - Intra-community Service Rendered Invoice |
| `2.3` | 2.3 - Third Country Service Rendered Invoice |
| `2.4` | 2.4 - Service Rendered Invoice/Supplemental Accounting Source Document |
| `3.1` | 3.1 - Proof of Expenditure (non-liable Issuer) |
| `3.2` | 3.2 - Proof of Expenditure (denial of issuance by liable Issuer) |
| `5.1` | 5.1 - Credit Invoice/Associated |
| `5.2` | 5.2 - Credit Invoice/Non-Associated |
| `6.1` | 6.1 - Self-Delivery Record |
| `6.2` | 6.2 - Self-Supply Record |
| `7.1` | 7.1 - Contract – Income |
| `8.1` | 8.1 - Rents – Income |
| `8.2` | 8.2 - Special Record – Accommodation Tax Collection/Payment Receipt |
| `11.1` | 11.1 - Retail Sales Receipt |
| `11.2` | 11.2 - Service Rendered Receipt |
| `11.3` | 11.3 - Simplified Invoice |
| `11.4` | 11.4 - Retail Sales Credit Note |
| `11.5` | 11.5 - Retail Sales Receipt on Behalf of Third Parties |
| `13.1` | 13.1 - Expenses – Domestic/Foreign Retail Transaction Purchases |
| `13.2` | 13.2 - Domestic/Foreign Retail Transaction Provision |
| `13.3` | 13.3 - Shared Utility Bills |
| `13.4` | 13.4 - Subscriptions |
| `13.30` | 13.30 - Self-Declared Entity Accounting Source Documents (Dynamic) |
| `13.31` | 13.31 - Domestic/Foreign Retail Sales Credit Note |
| `14.1` | 14.1 - Invoice/Intra-community Acquisitions |
| `14.2` | 14.2 - Invoice/Third Country Acquisitions |
| `14.3` | 14.3 - Invoice/Intra-community Services Receipt |
| `14.4` | 14.4 - Invoice/Third Country Services Receipt |
| `14.5` | 14.5 - EFKA |
| `14.30` | 14.30 - Self-Declared Entity Accounting Source Documents (Dynamic) |
| `14.31` | 14.31 - Domestic/Foreign Credit Note |
| `15.1` | 15.1 - Contract-Expense |
| `16.1` | 16.1 - Rent-Expense |
| `17.1` | 17.1 - Payroll |
| `17.2` | 17.2 - Amortisations |
| `17.3` | 17.3 - Other Income Adjustment/Regularisation Entries – Accounting Base |
| `17.4` | 17.4 - Other Income Adjustment/Regularisation Entries – Tax Base |
| `17.5` | 17.5 - Other Expense Adjustment/Regularisation Entries – Accounting Base |
| `17.6` | 17.6 - Other Expense Adjustment/Regularisation Entries – Tax Base |

## `l10n_hr_edi.addendum`

**`business_document_status`** — Business document status

| Value | Label |
|---|---|
| `0` | APPROVED |
| `1` | REJECTED |
| `2` | PAYMENT_FULFILLED |
| `3` | PAYMENT_PARTIALLY_FULLFILLED |
| `4` | RECEIVING_CONFIRMED |
| `99` | RECEIVED |
| `None` | None |

**`fiscalization_channel_type`** — Delivery channel type

| Value | Label |
|---|---|
| `0` | Delivered via EDI |
| `1` | Not delivered via EDI |

**`fiscalization_status`** — Fiscalization status

| Value | Label |
|---|---|
| `0` | Successful |
| `1` | Unsuccessful |
| `2` | Pending |

**`mer_document_status`** — MojEracun document status

| Value | Label |
|---|---|
| `20` | In validation |
| `30` | Sent |
| `40` | Delivered |
| `45` | Canceled |
| `50` | Unsuccessful |
| `70` | Delivered (eReporting) |

**`payment_method_type`** — Payment Method Type

| Value | Label |
|---|---|
| `T` | Transakcijski račun |
| `O` | Obračunsko plaćanje |
| `Z` | Ostalo |

## `l10n_hr_edi.mojeracun_reject_wizard`

**`rejection_type`** — Rejection reason type

| Value | Label |
|---|---|
| `N` | 'N' - Data discrepancy that does not affect tax calculation |
| `U` | 'U' - Data discrepancy that affects tax calculation |
| `O` | 'O' - Other |

## `l10n_hu_edi.cancellation`

**`code`** — Annulment Code

| Value | Label |
|---|---|
| `ERRATIC_DATA` | ERRATIC_DATA - Erroneous data |
| `ERRATIC_INVOICE_NUMBER` | ERRATIC_INVOICE_NUMBER - Erroneous invoice number |
| `ERRATIC_INVOICE_ISSUE_DATE` | ERRATIC_INVOICE_ISSUE_DATE - Erroneous issue date |

## `l10n_hu_edi.tax_audit_export`

**`selection_mode`** — Selection mode

| Value | Label |
|---|---|
| `date` | By date |
| `name` | By serial number |

## `l10n_id_efaktur_coretax.document`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

## `l10n_in.pan.entity`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`msme_type`** — MSME/Udyam Registration Type

| Value | Label |
|---|---|
| `micro` | Micro |
| `small` | Small |
| `medium` | Medium |

**`tds_deduction`** — TDS Deduction

| Value | Label |
|---|---|
| `normal` | Normal |
| `lower` | Lower |
| `higher` | Higher |
| `no` | No |

**`type`** — Type

| Value | Label |
|---|---|
| `a` | Association of Persons |
| `b` | Body of Individuals |
| `c` | Company |
| `f` | Firms |
| `g` | Government |
| `h` | Hindu Undivided Family |
| `j` | Artificial Judicial Person |
| `l` | Local Authority |
| `p` | Individual |
| `t` | Association of Persons for a Trust |
| `k` | Krish (Trust Krish) |

## `l10n_in.section.alert`

**`aggregate_period`** — Aggregate Period

| Value | Label |
|---|---|
| `monthly` | Monthly |
| `fiscal_yearly` | Financial Yearly |

**`consider_amount`** — Consider

| Value | Label |
|---|---|
| `untaxed_amount` | Untaxed Amount |
| `total_amount` | Total Amount |

**`tax_source_type`** — Tax Source Type

| Value | Label |
|---|---|
| `tds` | TDS |
| `tcs` | TCS |

## `l10n_in.withhold.wizard`

**`tds_deduction`** — TDS Deduction (not stored)

| Value | Label |
|---|---|
| `normal` | Normal Deduction |
| `lower` | Lower Deduction |
| `higher` | Higher Deduction |
| `no` | No Deduction |

## `l10n_in_edi.cancel`

**`cancel_reason`** — Cancel Reason

| Value | Label |
|---|---|
| `1` | Duplicate |
| `2` | Data Entry Mistake |
| `3` | Order Cancelled |
| `4` | Others |

## `l10n_it.document.type`

**`type`** — Type

| Value | Label |
|---|---|
| `sale` | Sale |
| `purchase` | Purchase |

## `l10n_it_edi_doi.declaration_of_intent`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`state`** — State

| Value | Label |
|---|---|
| `draft` | Draft |
| `active` | Active |
| `revoked` | Revoked |
| `terminated` | Terminated |

## `l10n_ke.item.code`

**`tax_rate`** — Tax Rate

| Value | Label |
|---|---|
| `C` | Zero Rated |
| `E` | Exempted |
| `B` | Taxable at 8% |

## `l10n_latam.check`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`issue_state`** — Issue State

| Value | Label |
|---|---|
| `handed` | Handed |
| `debited` | Debited |
| `voided` | Voided |

## `l10n_latam.document.type`

**`internal_type`** — Internal Type

| Value | Label |
|---|---|
| `invoice` | Invoices |
| `invoice_in` | Purchase Invoices |
| `debit_note` | Debit Notes |
| `credit_note` | Credit Notes |
| `all` | All Documents |
| `receipt_invoice` | Receipt Invoice |
| `stock_picking` | Stock Delivery |
| `purchase_liquidation` | Purchase Liquidation |
| `withhold` | Withhold |

**`purchase_aliquots`** — Purchase Aliquots

| Value | Label |
|---|---|
| `not_zero` | Not Zero |
| `zero` | Zero |

## `l10n_pl.bank.account.verification`

**`verification_status`** — Verification Status

| Value | Label |
|---|---|
| `valid` | Valid |
| `invalid` | Invalid |
| `incomplete_partner` | Incomplete partner |
| `not_found_partner` | Partner not found |
| `error` | An error occurred during check with Government API |

## `l10n_ro_edi.document`

**`state`** — E-Factura Status

| Value | Label |
|---|---|
| `invoice_sent` | Sent |
| `invoice_refused` | Error |
| `invoice_validated` | Validated |
| `stock_sent` | Sent |
| `stock_sending_failed` | Error |
| `stock_validated` | Validated |

## `l10n_tr.nilvera.trailer.plate`

**`plate_number_type`** — Plate Number

| Value | Label |
|---|---|
| `vehicle` | Vehicle |
| `trailer` | Plate |

## `l10n_tr_nilvera_einvoice_extended.account.tax.code`

**`code_type`** — Code Type

| Value | Label |
|---|---|
| `withholding` | Withholding |
| `exception` | Exception |
| `export_exception` | Export Exception |
| `export_registration` | Export Registration |

## `l10n_tw_edi.invoice.print`

**`print_format_b2b`** — Print Format (B2B)

| Value | Label |
|---|---|
| `1` | A4 printing  |
| `2` | A5 printing  |

**`print_format_b2c`** — Print Format (B2C)

| Value | Label |
|---|---|
| `1` | single-sided printing |
| `2` | double-sided printing |
| `3` | printing with thermal paper |

## `l10n_vn_edi_viettel.sinvoice.template`

**`template_invoice_type`** — Template Invoice Type

| Value | Label |
|---|---|
| `1` | 1 - Value-added invoice |
| `2` | 2 - Sales invoice |
| `3` | 3 - Public assets sales |
| `4` | 4 - National reserve sales |
| `5` | 5 - Invoice for national reserve sales |
| `6` | 6 - Warehouse release note |

## `lot.label.layout`

**`label_quantity`** — Quantity to print

| Value | Label |
|---|---|
| `lots` | One per lot/SN |
| `units` | One per unit |

**`print_format`** — Format

| Value | Label |
|---|---|
| `4x12` | 4 x 12 |
| `zpl` | ZPL Labels |

## `loyalty.generate.wizard`

**`mode`** — For

| Value | Label |
|---|---|
| `anonymous` | Anonymous Customers |
| `selected` | Selected Customers |

## `loyalty.mail`

**`trigger`** — When

| Value | Label |
|---|---|
| `create` | At Creation |
| `points_reach` | When Reaching |

## `loyalty.program`

**`applies_on`** — Applies On

| Value | Label |
|---|---|
| `current` | Current order |
| `future` | Future orders |
| `both` | Current & Future orders |

**`program_type`** — Program Type

| Value | Label |
|---|---|
| `coupons` | Coupons |
| `gift_card` | Gift Card |
| `loyalty` | Loyalty Cards |
| `promotion` | Promotions |
| `ewallet` | eWallet |
| `promo_code` | Discount Code |
| `buy_x_get_y` | Buy X Get Y |
| `next_order_coupons` | Next Order Coupons |

**`trigger`** — Trigger

| Value | Label |
|---|---|
| `auto` | Automatic |
| `with_code` | Use a code |

## `loyalty.reward`

**`discount_applicability`** — Discount Applicability

| Value | Label |
|---|---|
| `order` | Order |
| `cheapest` | Cheapest Product |
| `specific` | Specific Products |

**`reward_type`** — Reward Type

| Value | Label |
|---|---|
| `product` | Free Product |
| `discount` | Discount |
| `shipping` | Free Shipping |

## `loyalty.rule`

**`minimum_amount_tax_mode`** — Minimum Amount Tax Mode

| Value | Label |
|---|---|
| `incl` | tax included |
| `excl` | tax excluded |

**`mode`** — Application

| Value | Label |
|---|---|
| `auto` | Automatic |
| `with_code` | With a promotion code |

## `lunch.alert`

**`mode`** — Display

| Value | Label |
|---|---|
| `alert` | Alert in app |
| `chat` | Chat notification |

**`notification_moment`** — Notification Moment

| Value | Label |
|---|---|
| `am` | AM |
| `pm` | PM |

**`recipients`** — Recipients

| Value | Label |
|---|---|
| `everyone` | Everyone |
| `last_week` | Employee who ordered last week |
| `last_month` | Employee who ordered last month |
| `last_year` | Employee who ordered last year |

## `lunch.order`

**`state`** — Status

| Value | Label |
|---|---|
| `new` | To Order |
| `ordered` | Ordered |
| `sent` | Sent |
| `confirmed` | Received |
| `cancelled` | Cancelled |

## `lunch.supplier`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`delivery`** — Delivery

| Value | Label |
|---|---|
| `delivery` | Delivery |
| `no_delivery` | No Delivery |

**`moment`** — Moment

| Value | Label |
|---|---|
| `am` | AM |
| `pm` | PM |

**`send_by`** — Send Order By

| Value | Label |
|---|---|
| `phone` | Phone |
| `mail` | Email |

**`topping_quantity_1`** — Extra 1 Quantity

| Value | Label |
|---|---|
| `0_more` | None or More |
| `1_more` | One or More |
| `1` | Only One |

**`topping_quantity_2`** — Extra 2 Quantity

| Value | Label |
|---|---|
| `0_more` | None or More |
| `1_more` | One or More |
| `1` | Only One |

**`topping_quantity_3`** — Extra 3 Quantity

| Value | Label |
|---|---|
| `0_more` | None or More |
| `1_more` | One or More |
| `1` | Only One |

## `mail.activity`

**`state`** — State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |
| `done` | Done |

## `mail.activity.mixin`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

## `mail.activity.plan.template`

**`delay_from`** — Trigger

| Value | Label |
|---|---|
| `before_plan_date` | Before Plan Date |
| `after_plan_date` | After Plan Date |

**`delay_unit`** — Delay units

| Value | Label |
|---|---|
| `days` | days |
| `weeks` | weeks |
| `months` | months |

**`responsible_type`** — Assignment

| Value | Label |
|---|---|
| `on_demand` | Ask at launch |
| `other` | Default user |
| `coach` | Coach |
| `manager` | Manager |
| `employee` | Employee |
| `fleet_manager` | Fleet Manager |

## `mail.activity.type`

**`category`** — Action

| Value | Label |
|---|---|
| `default` | None |
| `upload_file` | Upload Document |
| `phonecall` | Phonecall |
| `meeting` | Meeting |

**`chaining_type`** — Chaining Type

| Value | Label |
|---|---|
| `suggest` | Suggest Next Activity |
| `trigger` | Trigger Next Activity |

**`decoration_type`** — Decoration Type

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`delay_from`** — Delay Type

| Value | Label |
|---|---|
| `current_date` | after previous activity completion date |
| `previous_activity` | after previous activity deadline |

**`delay_unit`** — Delay units

| Value | Label |
|---|---|
| `days` | days |
| `weeks` | weeks |
| `months` | months |

## `mail.alias`

**`alias_contact`** — Alias Contact Security

| Value | Label |
|---|---|
| `everyone` | Everyone |
| `partners` | Authenticated Partners |
| `followers` | Followers only |
| `employees` | Authenticated Employees |

**`alias_status`** — Alias Status

| Value | Label |
|---|---|
| `not_tested` | Not Tested |
| `valid` | Valid |
| `invalid` | Invalid |

## `mail.compose.message`

**`composition_comment_option`** — Comment Options

| Value | Label |
|---|---|
| `reply_all` | Reply-All |
| `forward` | Forward |

**`composition_mode`** — Composition mode

| Value | Label |
|---|---|
| `comment` | Post on a document |
| `mass_mail` | Email Mass Mailing |

**`message_type`** — Type

| Value | Label |
|---|---|
| `auto_comment` | Automated Targeted Notification |
| `comment` | Comment |
| `notification` | System notification |

**`reply_to_mode`** — Replies (not stored)

| Value | Label |
|---|---|
| `update` | Store email and replies in the chatter of each record |
| `new` | Collect replies on a specific email address |

## `mail.followers.edit`

**`operation`** — Operation

| Value | Label |
|---|---|
| `add` | Add |
| `remove` | Remove |

## `mail.group`

**`access_mode`** — Privacy

| Value | Label |
|---|---|
| `public` | Everyone |
| `members` | Members only |
| `groups` | Selected group of users |

## `mail.group.message`

**`author_moderation`** — Author Moderation Status (not stored)

| Value | Label |
|---|---|
| `ban` | Banned |
| `allow` | Whitelisted |

**`moderation_status`** — Status

| Value | Label |
|---|---|
| `pending_moderation` | Pending Moderation |
| `accepted` | Accepted |
| `rejected` | Rejected |

## `mail.group.message.reject`

**`action`** — Action

| Value | Label |
|---|---|
| `reject` | Reject |
| `ban` | Ban |

## `mail.group.moderation`

**`status`** — Status

| Value | Label |
|---|---|
| `allow` | Always Allow |
| `ban` | Permanent Ban |

## `mail.ice.server`

**`server_type`** — Type

| Value | Label |
|---|---|
| `stun` | stun: |
| `turn` | turn: |

## `mail.mail`

**`failure_type`** — Failure type

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `mail_spam` | Detected As Spam |
| `mail_email_invalid` | Invalid email address |
| `mail_email_missing` | Missing email |
| `mail_from_invalid` | Invalid from address |
| `mail_from_missing` | Missing from address |
| `mail_smtp` | Connection failed (outgoing mail server problem) |
| `mail_bl` | Blacklisted Address |
| `mail_optout` | Opted Out |
| `mail_dup` | Duplicated Email |

**`state`** — Status

| Value | Label |
|---|---|
| `outgoing` | Outgoing |
| `sent` | Sent |
| `received` | Received |
| `exception` | Delivery Failed |
| `cancel` | Cancelled |

## `mail.message`

**`message_type`** — Type

| Value | Label |
|---|---|
| `email` | Incoming Email |
| `comment` | Comment |
| `email_outgoing` | Outgoing Email |
| `notification` | System notification |
| `auto_comment` | Automated Targeted Notification |
| `out_of_office` | Out-of-office Message |
| `user_notification` | User Specific Notification |
| `sms` | SMS |
| `snailmail` | Snailmail |

## `mail.notification`

**`failure_type`** — Failure type

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `mail_bounce` | Bounce |
| `mail_spam` | Detected As Spam |
| `mail_email_invalid` | Invalid email address |
| `mail_email_missing` | Missing email address |
| `mail_from_invalid` | Invalid from address |
| `mail_from_missing` | Missing from address |
| `mail_smtp` | Connection failed (outgoing mail server problem) |
| `mail_bl` | Blacklisted Address |
| `mail_optout` | Opted Out |
| `mail_dup` | Duplicated Email |
| `sms_number_missing` | Missing Number |
| `sms_number_format` | Wrong Number Format |
| `sms_credit` | Insufficient Credit |
| `sms_country_not_supported` | Country Not Supported |
| `sms_registration_needed` | Country-specific Registration Required |
| `sms_server` | Server Error |
| `sms_acc` | Unregistered Account |
| `sms_expired` | Expired |
| `sms_invalid_destination` | Invalid Destination |
| `sms_not_allowed` | Not Allowed |
| `sms_not_delivered` | Not Delivered |
| `sms_rejected` | Rejected |
| `sn_credit` | Snailmail Credit Error |
| `sn_trial` | Snailmail Trial Error |
| `sn_price` | Snailmail No Price Available |
| `sn_fields` | Snailmail Missing Required Fields |
| `sn_format` | Snailmail Format Error |
| `sn_error` | Snailmail Unknown Error |
| `twilio_authentication` | Authentication Error" |
| `twilio_callback` | Incorrect callback URL |
| `twilio_from_missing` | Missing From Number |
| `twilio_from_to` | From / To identic |

**`notification_status`** — Status

| Value | Label |
|---|---|
| `ready` | Ready to Send |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `bounce` | Bounced |
| `exception` | Exception |
| `canceled` | Cancelled |

**`notification_type`** — Notification Type

| Value | Label |
|---|---|
| `inbox` | Inbox |
| `email` | Email |
| `sms` | SMS |
| `snail` | Snailmail |

## `mail.presence`

**`status`** — IM Status

| Value | Label |
|---|---|
| `online` | Online |
| `away` | Away |
| `offline` | Offline |

## `mail.scheduled.message`

**`composition_comment_option`** — Comment Options

| Value | Label |
|---|---|
| `reply_all` | Reply-All |
| `forward` | Forward |

## `mail.template`

**`template_category`** — Template Category (not stored)

| Value | Label |
|---|---|
| `base_template` | Base Template |
| `hidden_template` | Hidden Template |
| `custom_template` | Custom Template |

## `mailing.list.merge`

**`merge_options`** — Merge Option

| Value | Label |
|---|---|
| `new` | Merge into a new mailing list |
| `existing` | Merge into an existing mailing list |

## `mailing.mailing`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`mailing_type`** — Mailing Type

| Value | Label |
|---|---|
| `mail` | Email |
| `sms` | SMS |

**`reply_to_mode`** — Reply-To Mode

| Value | Label |
|---|---|
| `update` | Recipient Followers |
| `new` | Specified Email Address |

**`schedule_type`** — Schedule

| Value | Label |
|---|---|
| `now` | Send now |
| `scheduled` | Send on |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_queue` | In Queue |
| `sending` | Sending |
| `done` | Sent |

## `mailing.trace`

**`failure_type`** — Failure type

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `mail_bounce` | Bounce |
| `mail_spam` | Detected As Spam |
| `mail_email_invalid` | Invalid email address |
| `mail_email_missing` | Missing email address |
| `mail_from_invalid` | Invalid from address |
| `mail_from_missing` | Missing from address |
| `mail_smtp` | Connection failed (outgoing mail server problem) |
| `mail_bl` | Blacklisted Address |
| `mail_dup` | Duplicated Email |
| `mail_optout` | Opted Out |
| `sms_number_missing` | Missing Number |
| `sms_number_format` | Wrong Number Format |
| `sms_credit` | Insufficient Credit |
| `sms_country_not_supported` | Country Not Supported |
| `sms_registration_needed` | Country-specific Registration Required |
| `sms_server` | Server Error |
| `sms_acc` | Unregistered Account |
| `sms_blacklist` | Blacklisted |
| `sms_duplicate` | Duplicate |
| `sms_optout` | Opted Out |
| `sms_expired` | Expired |
| `sms_invalid_destination` | Invalid Destination |
| `sms_not_allowed` | Not Allowed |
| `sms_not_delivered` | Not Delivered |
| `sms_rejected` | Rejected |
| `twilio_authentication` | Authentication Error" |
| `twilio_callback` | Incorrect callback URL |
| `twilio_from_missing` | Missing From Number |
| `twilio_from_to` | From / To identic |

**`trace_status`** — Status

| Value | Label |
|---|---|
| `outgoing` | Outgoing |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `open` | Opened |
| `reply` | Replied |
| `bounce` | Bounced |
| `error` | Exception |
| `cancel` | Cancelled |

**`trace_type`** — Type

| Value | Label |
|---|---|
| `mail` | Email |
| `sms` | SMS |

## `mailing.trace.report`

**`mailing_type`** — Type

| Value | Label |
|---|---|
| `mail` | Mail |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `test` | Tested |
| `done` | Sent |

## `maintenance.equipment`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`equipment_assign_to`** — Used By

| Value | Label |
|---|---|
| `department` | Department |
| `employee` | Employee |
| `other` | Other |

## `maintenance.request`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`instruction_type`** — Instruction

| Value | Label |
|---|---|
| `pdf` | PDF |
| `google_slide` | Google Slide |
| `text` | Text |

**`kanban_state`** — Kanban State

| Value | Label |
|---|---|
| `normal` | In Progress |
| `blocked` | Blocked |
| `done` | Ready for next stage |

**`maintenance_type`** — Maintenance Type

| Value | Label |
|---|---|
| `corrective` | Corrective |
| `preventive` | Preventive |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Very Low |
| `1` | Low |
| `2` | Normal |
| `3` | High |

**`repeat_type`** — Until

| Value | Label |
|---|---|
| `forever` | Forever |
| `until` | Until |

**`repeat_unit`** — Repeat Unit

| Value | Label |
|---|---|
| `day` | Days |
| `week` | Weeks |
| `month` | Months |
| `year` | Years |

## `microsoft.calendar.account.reset`

**`delete_policy`** — User's Existing Events

| Value | Label |
|---|---|
| `dont_delete` | Leave them untouched |
| `delete_microsoft` | Delete from the current Microsoft Calendar account |
| `delete_system` | Delete from the system |
| `delete_both` | Delete from both |

**`sync_policy`** — Next Synchronization

| Value | Label |
|---|---|
| `new` | Synchronize only new events |
| `all` | Synchronize all existing events |

## `mrp.bom`

**`consumption`** — Flexible Consumption

| Value | Label |
|---|---|
| `flexible` | Allowed |
| `warning` | Allowed with warning |
| `strict` | Blocked |

**`ready_to_produce`** — Manufacturing Readiness

| Value | Label |
|---|---|
| `all_available` |  When all components are available |
| `asap` | When components for 1st operation are available |

**`type`** — BoM Type

| Value | Label |
|---|---|
| `normal` | Manufacture this product |
| `phantom` | Kit |
| `subcontract` | Subcontracting |

## `mrp.consumption.warning`

**`consumption`** — Consumption (not stored)

| Value | Label |
|---|---|
| `flexible` | Allowed |
| `warning` | Allowed with warning |
| `strict` | Blocked |

## `mrp.production`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`components_availability_state`** — Components Availability State (not stored)

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |
| `unavailable` | Not Available |

**`consumption`** — Consumption

| Value | Label |
|---|---|
| `flexible` | Allowed |
| `warning` | Allowed with warning |
| `strict` | Blocked |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

**`reservation_state`** — MO Readiness

| Value | Label |
|---|---|
| `confirmed` | Waiting |
| `assigned` | Ready |
| `waiting` | Waiting Another Operation |

**`search_date_category`** — Date Category (not stored)

| Value | Label |
|---|---|
| `before` | Before |
| `yesterday` | Yesterday |
| `today` | Today |
| `day_1` | Tomorrow |
| `day_2` | The day after tomorrow |
| `after` | After |

**`state`** — State

| Value | Label |
|---|---|
| `draft` | Draft |
| `confirmed` | Confirmed |
| `progress` | In Progress |
| `to_close` | To Close |
| `done` | Done |
| `cancel` | Cancelled |

## `mrp.routing.workcenter`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`cost_mode`** — Cost based on

| Value | Label |
|---|---|
| `actual` | Actual time |
| `estimated` | Theorical time |

**`time_mode`** — Duration Computation

| Value | Label |
|---|---|
| `manual` | Fixed |
| `auto` | Computed |

## `mrp.unbuild`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Done |

## `mrp.workcenter`

**`working_state`** — Workcenter Status

| Value | Label |
|---|---|
| `normal` | Normal |
| `blocked` | Blocked |
| `done` | In Progress |

## `mrp.workcenter.productivity.loss.type`

**`loss_type`** — Category

| Value | Label |
|---|---|
| `availability` | Availability |
| `performance` | Performance |
| `quality` | Quality |
| `productive` | Productive |

## `mrp.workorder`

**`cost_mode`** — Cost Mode

| Value | Label |
|---|---|
| `actual` | Actual |
| `estimated` | Estimated |

**`state`** — Status

| Value | Label |
|---|---|
| `blocked` | Blocked |
| `ready` | To Do |
| `progress` | In Progress |
| `done` | Finished |
| `cancel` | Cancelled |

## `myinvois.consolidate.invoice.wizard`

**`consolidation_type`** — Consolidation Type

| Value | Label |
|---|---|
| `invoice` | Invoice |
| `pos` | PoS Order |

## `myinvois.document`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`myinvois_state`** — MyInvois State

| Value | Label |
|---|---|
| `in_progress` | Validation In Progress |
| `valid` | Valid |
| `rejected` | Rejected |
| `invalid` | Invalid |
| `cancelled` | Cancelled |

## `nemhandel.registration`

**`edi_mode`** — EDI mode (not stored)

| Value | Label |
|---|---|
| `demo` | Demo |
| `test` | Test |
| `prod` | Live |

## `nemhandel.response`

**`nemhandel_state`** — Nemhandel status

| Value | Label |
|---|---|
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `not_serviced` | Not Serviced |

**`response_code`** — Response Code

| Value | Label |
|---|---|
| `BusinessAccept` | Approval |
| `BusinessReject` | Rejection |

## `onboarding.onboarding`

**`current_onboarding_state`** — Completion State (not stored)

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

## `onboarding.onboarding.step`

**`current_step_state`** — Completion State (not stored)

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

## `onboarding.progress`

**`onboarding_state`** — Onboarding progress

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

## `onboarding.progress.step`

**`step_state`** — Onboarding Step Progress

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

## `payment.method`

**`support_manual_capture`** — Manual Capture

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Full & Partial |

**`support_refund`** — Refund

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Full & Partial |

## `payment.provider`

**`asiapay_brand`** — Asiapay Brand

| Value | Label |
|---|---|
| `paydollar` | PayDollar |
| `pesopay` | PesoPay |
| `siampay` | SiamPay |
| `bimopay` | BimoPay |

**`asiapay_secure_hash_function`** — AsiaPay Secure Hash Function

| Value | Label |
|---|---|
| `sha1` | SHA1 |
| `sha256` | SHA256 |
| `sha512` | SHA512 |

**`code`** — Code

| Value | Label |
|---|---|
| `none` | No Provider Set |
| `adyen` | Adyen |
| `aps` | Amazon Payment Services |
| `asiapay` | AsiaPay |
| `authorize` | Authorize.Net |
| `buckaroo` | Buckaroo |
| `custom` | Custom |
| `demo` | Demo |
| `dpo` | DPO |
| `ecpay` | ECPay |
| `flutterwave` | Flutterwave |
| `iyzico` | Iyzico |
| `mercado_pago` | Mercado Pago |
| `mollie` | Mollie |
| `nuvei` | Nuvei |
| `paymob` | Paymob |
| `paypal` | PayPal |
| `payu` | PayU |
| `razorpay` | Razorpay |
| `redsys` | Redsys |
| `stripe` | Stripe |
| `toss_payments` | Toss Payments |
| `worldline` | Worldline |
| `xendit` | Xendit |

**`custom_mode`** — Custom Mode

| Value | Label |
|---|---|
| `wire_transfer` | Wire Transfer |
| `cash_on_delivery` | Cash On Delivery |
| `on_site` | Pay on site |

**`so_reference_type`** — Communication

| Value | Label |
|---|---|
| `so_name` | Based on Document Reference |
| `partner` | Based on Customer ID |

**`state`** — State

| Value | Label |
|---|---|
| `disabled` | Disabled |
| `enabled` | Enabled |
| `test` | Test Mode |

**`support_manual_capture`** — Manual Capture Supported (not stored)

| Value | Label |
|---|---|
| `full_only` | Full Only |
| `partial` | Partial |

**`support_refund`** — Refund (not stored)

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Full & Partial |

## `payment.refund.wizard`

**`support_refund`** — Refund (not stored)

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Partial |

## `payment.token`

**`demo_simulated_state`** — Simulated State

| Value | Label |
|---|---|
| `pending` | Pending |
| `done` | Confirmed |
| `cancel` | Canceled |
| `error` | Error |

## `payment.transaction`

**`operation`** — Operation

| Value | Label |
|---|---|
| `online_redirect` | Online payment with redirection |
| `online_direct` | Online direct payment |
| `online_token` | Online payment by token |
| `validation` | Validation of the payment method |
| `offline` | Offline payment by token |
| `refund` | Refund |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `pending` | Pending |
| `authorized` | Authorized |
| `done` | Confirmed |
| `cancel` | Canceled |
| `error` | Error |

## `pdp.registration`

**`edi_mode`** — EDI mode (not stored)

| Value | Label |
|---|---|
| `demo` | Demo |
| `test` | Test |
| `prod` | Live |

## `pdp.response.wizard`

**`reason_code`** — Reason Code

| Value | Label |
|---|---|
| `TX_TVA_ERR` | Incorrect VAT rate |
| `MONTANTTOTAL_ERR` | Incorrect Total Amount |
| `CALCUL_ERR` | Billing calculation error |
| `NON_CONFORME` | Legal information missing |
| `DEST_ERR` | Wrong recipient |
| `TRANSAC_INC` | Unknown transaction |
| `EMMET_INC` | Unknown sender |
| `CONTRAT_TERM` | Contract completed |
| `DOUBLE_FACT` | Duplicate Invoice |
| `CMD_ERR` | Order number is incorrect or missing |
| `ADR_ERR` | Incorrect electronic billing address |
| `REF_CT_ABSENT` | Contract reference required to process the missing invoice |
| `JUSTIF_ABS` | Missing or Insufficient Supporting Documentation |
| `COORD_BANC_ERR` | Bank Account Information Error |
| `SIRET_ERR` | Incorrect or missing SIRET number |
| `CODE_ROUTAGE_ERR` | Missing or incorrect CODE_ROUTAGE |
| `REF_ERR` | Incorrect reference |

**`status`** — Status

| Value | Label |
|---|---|
| `PD` | Paid |
| `cancelled` | Cancelled |
| `suspended` | Suspended |
| `refused` | Refused |
| `AP` | Approved |
| `in_hand` | In Hand |
| `completed` | Completed |

## `peppol.registration`

**`edi_mode`** — EDI mode (not stored)

| Value | Label |
|---|---|
| `demo` | Demo |
| `test` | Test |
| `prod` | Live |

**`use_parent_connection_selection`** — Use Parent Connection Selection

| Value | Label |
|---|---|
| `use_parent` | Send from parent company |
| `use_self` | Register this company on peppol |

## `picking.label.type`

**`label_type`** — Labels to print

| Value | Label |
|---|---|
| `products` | Product Labels |
| `lots` | Lot/SN Labels |

## `portal.wizard.user`

**`email_state`** — Status (not stored)

| Value | Label |
|---|---|
| `ok` | Valid |
| `ko` | Invalid |
| `exist` | Already Registered |

## `pos.config`

**`default_screen`** — Default Screen

| Value | Label |
|---|---|
| `tables` | Tables |
| `register` | Register |

**`iface_tax_included`** — Tax Display

| Value | Label |
|---|---|
| `subtotal` | Tax-Excluded Price |
| `total` | Tax-Included Price |

**`picking_policy`** — Shipping Policy

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

**`self_ordering_mode`** — Self Ordering Mode

| Value | Label |
|---|---|
| `nothing` | Disable |
| `consultation` | QR menu |
| `mobile` | QR menu + Ordering |
| `kiosk` | Kiosk |

**`self_ordering_service_mode`** — Self Ordering Service Mode

| Value | Label |
|---|---|
| `counter` | Pickup zone |
| `table` | Table |

**`status`** — Status (not stored)

| Value | Label |
|---|---|
| `inactive` | Inactive |
| `active` | Active |

## `pos.order`

**`invoice_status`** — Invoice Status (not stored)

| Value | Label |
|---|---|
| `invoiced` | Fully Invoiced |
| `to_invoice` | To Invoice |

**`l10n_es_edi_verifactu_refund_reason`** — Veri*Factu Refund Reason

| Value | Label |
|---|---|
| `R1` | R1: Art 80.1 and 80.2 and error of law |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Rest |
| `R5` | R5: Corrective invoices concerning simplified invoices |

**`l10n_es_edi_verifactu_state`** — Veri*Factu Status

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |
| `cancelled` | Cancelled |

**`l10n_es_tbai_refund_reason`** — Invoice Refund Reason Code (TicketBai)

| Value | Label |
|---|---|
| `R1` | R1: Art. 80.1, 80.2, 80.6 and rights founded error |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Art. 80 - other |
| `R5` | R5: Factura rectificativa en facturas simplificadas |

**`l10n_es_tbai_state`** — TicketBAI status (not stored)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |

**`l10n_jo_edi_pos_state`** — JoFotara State

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `demo` | Sent (Demo) |

**`l10n_sa_reason`** — ZATCA Reason

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | Cancellation or suspension of the supplies after its occurrence either wholly or partially |
| `BR-KSA-17-reason-2` | In case of essential change or amendment in the supply, which leads to the change of the VAT due |
| `BR-KSA-17-reason-3` | Amendment of the supply value which is pre-agreed upon between the supplier and consumer |
| `BR-KSA-17-reason-4` | In case of goods or services refund |
| `BR-KSA-17-reason-5` | In case of change in Seller's or Buyer's information |

**`l10n_tw_edi_carrier_type`** — Carrier Type

| Value | Label |
|---|---|
| `1` | ECpay e-invoice carrier |
| `2` | Citizen Digital Certificate |
| `3` | Mobile Barcode |
| `4` | EasyCard |
| `5` | iPass |

**`source`** — Origin

| Value | Label |
|---|---|
| `pos` | Point of Sale |
| `mobile` | Self-Order Mobile |
| `kiosk` | Self-Order Kiosk |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | New |
| `cancel` | Cancelled |
| `paid` | Paid |
| `done` | Posted |

## `pos.order.line`

**`price_type`** — Price Type

| Value | Label |
|---|---|
| `original` | Original |
| `manual` | Manual |
| `automatic` | Automatic |

## `pos.payment.method`

**`dpopay_payment_mode`** — Dpopay Payment Mode

| Value | Label |
|---|---|
| `card` | Card |
| `momo` | Mobile Money |

**`pine_labs_allowed_payment_mode`** — Pine Labs Allowed Payment Modes

| Value | Label |
|---|---|
| `all` | All |
| `card` | Card |
| `upi` | Upi |

**`qfpay_payment_type`** — QFPay Payment Type

| Value | Label |
|---|---|
| `card_payment` | Visa/Mastercard |
| `wx` | WeChat Pay |
| `alipay` | Alipay |
| `payme` | PayMe |
| `union` | UnionPay QuickPass |
| `fps` | FPS |
| `octopus` | Octopus |
| `unionpay_card` | Unionpay Card |
| `amex_card` | American Express Card |

**`razorpay_allowed_payment_modes`** — Razorpay Allowed Payment Modes

| Value | Label |
|---|---|
| `all` | All |
| `card` | Card |
| `upi` | UPI |
| `bharatqr` | BHARATQR |

**`safaricom_payment_type`** — Payment Type

| Value | Label |
|---|---|
| `mpesa_express` | M-PESA Express |
| `lipa_na_mpesa` | Lipa na M-PESA |

**`type`** — Type (not stored)

| Value | Label |
|---|---|
| `cash` | Cash |
| `bank` | Bank |
| `pay_later` | Customer Account |
| `online` | Online |

## `pos.preset`

**`identification`** — Identification

| Value | Label |
|---|---|
| `none` | Not required |
| `address` | Address |
| `name` | Name |

**`service_at`** — Service at

| Value | Label |
|---|---|
| `counter` | Pickup zone |
| `table` | Table |
| `delivery` | Delivery |

## `pos.printer`

**`printer_type`** — Printer Type

| Value | Label |
|---|---|
| `iot` | Use a printer connected to the IoT Box |
| `epson_epos` | Use an Epson printer |

## `pos.session`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`state`** — Status

| Value | Label |
|---|---|
| `opening_control` | Opening Control |
| `opened` | In Progress |
| `closing_control` | Closing Control |
| `closed` | Closed & Posted |

## `pos_self_order.custom_link`

**`style`** — Style

| Value | Label |
|---|---|
| `primary` | Primary |
| `secondary` | Secondary |
| `success` | Success |
| `warning` | Warning |
| `danger` | Danger |
| `info` | Info |
| `light` | Light |
| `dark` | Dark |

## `product.attribute`

**`create_variant`** — Variant Creation

| Value | Label |
|---|---|
| `always` | Instantly |
| `dynamic` | Dynamically |
| `no_variant` | Never |

**`display_type`** — Display Type

| Value | Label |
|---|---|
| `radio` | Radio |
| `pills` | Pills |
| `select` | Select |
| `color` | Color |
| `multi` | Multi-checkbox |
| `image` | Image |

**`preview_variants`** — On Product Cards

| Value | Label |
|---|---|
| `visible` | Visible |
| `hidden` | Hidden |
| `hover` | Hover |

**`visibility`** — Visibility

| Value | Label |
|---|---|
| `visible` | Visible |
| `hidden` | Hidden |

## `product.category`

**`packaging_reserve_method`** — Reserve Packagings

| Value | Label |
|---|---|
| `full` | Reserve Only Full Packagings |
| `partial` | Reserve Partial Packagings |

**`property_cost_method`** — Costing Method

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

**`property_valuation`** — Inventory Valuation

| Value | Label |
|---|---|
| `periodic` | Periodic (at closing) |
| `real_time` | Perpetual (at invoicing) |

## `product.document`

**`attached_on_mrp`** — MRP : Visible at

| Value | Label |
|---|---|
| `hidden` | Hidden |
| `bom` | Bill of Materials |

**`attached_on_sale`** — Sale : Visible at

| Value | Label |
|---|---|
| `hidden` | Hidden |
| `quotation` | On quote |
| `sale_order` | On confirmed order |
| `inside` | Inside quote pdf |

## `product.feed`

**`target`** — Target

| Value | Label |
|---|---|
| `gmc` | Google Merchant Center |

## `product.label.layout`

**`move_quantity`** — Quantity to print

| Value | Label |
|---|---|
| `move` | Operation Quantities |
| `custom` | Custom |

**`print_format`** — Format

| Value | Label |
|---|---|
| `dymo` | Dymo |
| `2x7xprice` | 2 x 7 with price |
| `4x7xprice` | 4 x 7 with price |
| `4x12` | 4 x 12 |
| `4x12xprice` | 4 x 12 with price |
| `zpl` | ZPL Labels |
| `zplxprice` | ZPL Labels with price |

**`zpl_template`** — ZPL Template

| Value | Label |
|---|---|
| `normal` | Normal (2.25" x 1.25") |
| `small` | Small (1.25" x 1.00") |
| `alternative` | Alternative (2.00" x 1.00") |
| `jewelry` | Jewelry (2.20" x 0.50") |

## `product.margin`

**`invoice_state`** — Invoice State

| Value | Label |
|---|---|
| `paid` | Paid |
| `open_paid` | Open and Paid |
| `draft_open_paid` | Draft, Open and Paid |

## `product.pricelist`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

## `product.pricelist.item`

**`applied_on`** — Apply On

| Value | Label |
|---|---|
| `3_global` | All Products |
| `2_product_category` | Product Category |
| `1_product` | Product |
| `0_product_variant` | Product Variant |

**`base`** — Based on

| Value | Label |
|---|---|
| `list_price` | Sales Price |
| `standard_price` | Cost |
| `pricelist` | Other Pricelist |

**`compute_price`** — Compute Price

| Value | Label |
|---|---|
| `percentage` | Discount |
| `formula` | Formula |
| `fixed` | Fixed Price |

**`display_applied_on`** — Display Applied On

| Value | Label |
|---|---|
| `1_product` | Product |
| `2_product_category` | Category |

## `product.product`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`invoice_state`** — Invoice State (not stored)

| Value | Label |
|---|---|
| `paid` | Paid |
| `open_paid` | Open and Paid |
| `draft_open_paid` | Draft, Open and Paid |

## `product.ribbon`

**`assign`** — Assign

| Value | Label |
|---|---|
| `manual` | Manually |
| `sale` | On Sale |
| `new` | When New |
| `out_of_stock` | when out of stock |

**`position`** — Position

| Value | Label |
|---|---|
| `left` | Left |
| `right` | Right |

**`style`** — Style

| Value | Label |
|---|---|
| `ribbon` | Ribbon |
| `tag` | Badge |

## `product.template`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`cost_method`** — Cost Method (not stored)

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

**`expense_policy`** — Re-Invoice Costs

| Value | Label |
|---|---|
| `no` | No |
| `cost` | At cost |
| `sales_price` | Sales price |

**`invoice_policy`** — Invoicing Policy

| Value | Label |
|---|---|
| `order` | Ordered quantities |
| `delivery` | Delivered quantities |

**`l10n_hu_product_code_type`** — Product Code Type

| Value | Label |
|---|---|
| `VTSZ` | VTSZ - Customs Code |
| `SZJ` | SZJ - Service Registry Code |
| `TESZOR` | TESZOR - CPA 2.1 Code |
| `KN` | KN - Combined Nomenclature Code |
| `AHK` | AHK - e-TKO Excise Duty Code |
| `KT` | KT - Environmental Product Code |
| `CSK` | CSK - Packaging Catalogue Code |
| `EJ` | EJ - Building Registry Number |
| `OTHER` | Other |

**`l10n_my_edi_classification_code`** — Malaysian classification code

| Value | Label |
|---|---|
| `001` | (001) Breastfeeding equipment  |
| `002` | (002) Child care centres and kindergartens fees |
| `003` | (003) Computer, smartphone or tablet |
| `004` | (004) Consolidated e-Invoice  |
| `005` | (005) Construction materials (as specified under Fourth Schedule of the Lembaga Pembangunan Industri Pembinaan Malaysia Act 1994) |
| `006` | (006) Disbursement |
| `007` | (007) Donation |
| `008` | (008) -Commerce - e-Invoice to buyer / purchaser |
| `009` | (009) e-Commerce - Self-billed e-Invoice to seller, logistics, etc.  |
| `010` | (010) Education fees |
| `011` | (011) Goods on consignment (Consignor) |
| `012` | (012) Goods on consignment (Consignee) |
| `013` | (013) Gym membership |
| `014` | (014) Insurance - Education and medical benefits |
| `015` | (015) Insurance - Takaful or life insurance |
| `016` | (016) Interest and financing expenses |
| `017` | (017) Internet subscription |
| `018` | (018) Land and building |
| `019` | (019) Medical examination for learning disabilities and early intervention or rehabilitation treatments of learning disabilities |
| `020` | (020) Medical examination or vaccination expenses |
| `021` | (021) Medical expenses for serious diseases |
| `022` | (022) Others |
| `023` | (023) Petroleum operations (as defined in Petroleum (Income Tax) Act 1967) |
| `024` | (024) Private retirement scheme or deferred annuity scheme |
| `025` | (025) Motor vehicle |
| `026` | (026) Subscription of books / journals / magazines / newspapers / other similar publications |
| `027` | (027) Reimbursement |
| `028` | (028) Rental of motor vehicle |
| `029` | (029) EV charging facilities (Installation, rental, sale / purchase or subscription fees)  |
| `030` | (030) Repair and maintenance |
| `031` | (031) Research and development |
| `032` | (032) Foreign income |
| `033` | (033) Self-billed - Betting and gaming |
| `034` | (034) Self-billed - Importation of goods |
| `035` | (035) Self-billed - Importation of services |
| `036` | (036) Self-billed - Others |
| `037` | (037) Self-billed - Monetary payment to agents, dealers or distributors |
| `038` | (038) Fees related to sports equipment, facility rentals, competition registration, and training imposed by registered sports organizations under the Sports Development Act 1997 |
| `039` | (039) Supporting equipment for disabled person |
| `040` | (040) Voluntary contribution to approved provident fund  |
| `041` | (041) Dental examination or treatment |
| `042` | (042) Fertility treatment |
| `043` | (043) Treatment and home care nursing, daycare centres and residential care centers |
| `044` | (044) Vouchers, gift cards, loyalty points, etc |
| `045` | (045) Self-billed - Non-monetary payment to agents, dealers or distributors |

**`l10n_pl_vat_gtu`** — GTU Codes

| Value | Label |
|---|---|
| `GTU_01` | GTU_01 - Alcoholic beverages |
| `GTU_02` | GTU_02 - Goods referred to under Art. 103 sec 5aa |
| `GTU_03` | GTU_03 - Fuel oil for excise duty, lubricating oils and other oils |
| `GTU_04` | GTU_04 - Tobacco products, tobacco, e-liquid |
| `GTU_05` | GTU_05 - Wastes |
| `GTU_06` | GTU_06 - Electronic devices, their parts and materials |
| `GTU_07` | GTU_07 - Vehicles and vehicle parts |
| `GTU_08` | GTU_08 - Precious metals and base metals |
| `GTU_09` | GTU_09 - Medicament and medical devices, medicinal products |
| `GTU_10` | GTU_10 - Buildings, structures and land |
| `GTU_11` | GTU_11 - Services related to the greenhouse gas emission allowance trading |
| `GTU_12` | GTU_12 - Intangible services |
| `GTU_13` | GTU_13 - Transport services and warehouse management services |

**`product_add_mode`** — Add product mode

| Value | Label |
|---|---|
| `configurator` | Product Configurator |
| `matrix` | Order Grid Entry |

**`purchase_method`** — Control Policy

| Value | Label |
|---|---|
| `purchase` | On ordered quantities |
| `receive` | On received quantities |

**`rating_avg_text`** — Rating Avg Text (not stored)

| Value | Label |
|---|---|
| `top` | Happy |
| `ok` | Neutral |
| `ko` | Unhappy |
| `none` | Not Rated yet |

**`service_tracking`** — Create on Order

| Value | Label |
|---|---|
| `no` | Nothing |
| `event` | Event Registration |
| `partnership` | Membership / Partnership |
| `repair` | Repair Order |
| `event_booth` | Event Booth |
| `task_global_project` | Task |
| `task_in_project` | Project & Task |
| `project_only` | Project |
| `course` | Course Access |

**`service_type`** — Track Service

| Value | Label |
|---|---|
| `manual` | Manually set quantities on order |
| `milestones` | Project Milestones |
| `timesheet` | Timesheets on project (one fare per SO/Project) |

**`split_method_landed_cost`** — Default Split Method

| Value | Label |
|---|---|
| `equal` | Equal |
| `by_quantity` | By Quantity |
| `by_current_cost_price` | By Current Cost |
| `by_weight` | By Weight |
| `by_volume` | By Volume |

**`tracking`** — Tracking

| Value | Label |
|---|---|
| `serial` | By Unique Serial Number |
| `lot` | By Lots |
| `none` | By Quantity |

**`type`** — Product Type

| Value | Label |
|---|---|
| `consu` | Goods |
| `service` | Service |
| `combo` | Combo |

**`valuation`** — Valuation (not stored)

| Value | Label |
|---|---|
| `periodic` | Periodic (at closing) |
| `real_time` | Perpetual (at invoicing) |

## `project.project`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`billing_type`** — Billing Type

| Value | Label |
|---|---|
| `not_billable` | not billable |
| `manually` | billed manually |

**`last_update_status`** — Last Update Status

| Value | Label |
|---|---|
| `on_track` | On Track |
| `at_risk` | At Risk |
| `off_track` | Off Track |
| `on_hold` | On Hold |
| `to_define` | Set Status |
| `done` | Complete |

**`pricing_type`** — Pricing (not stored)

| Value | Label |
|---|---|
| `task_rate` | Task rate |
| `fixed_rate` | Project rate |
| `employee_rate` | Employee rate |

**`privacy_visibility`** — Visibility

| Value | Label |
|---|---|
| `followers` | Invited internal users |
| `invited_users` | Invited internal and portal users |
| `employees` | All internal users |
| `portal` | All internal users and invited portal users |

## `project.share.collaborator.wizard`

**`access_mode`** — Access Mode

| Value | Label |
|---|---|
| `read` | Read |
| `edit_limited` | Edit with limited access |
| `edit` | Edit |

## `project.task`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Low priority |
| `1` | Medium priority |
| `2` | High priority |
| `3` | Urgent |

**`rating_avg_text`** — Rating Avg Text (not stored)

| Value | Label |
|---|---|
| `top` | Happy |
| `ok` | Neutral |
| `ko` | Unhappy |
| `none` | Not Rated yet |

**`repeat_type`** — Until (not stored)

| Value | Label |
|---|---|
| `forever` | Forever |
| `until` | Until |

**`repeat_unit`** — Repeat Unit (not stored)

| Value | Label |
|---|---|
| `day` | Days |
| `week` | Weeks |
| `month` | Months |
| `year` | Years |

**`state`** — State

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `02_changes_requested` | Changes Requested |
| `03_approved` | Approved |
| `1_done` | Done |
| `1_canceled` | Cancelled |
| `04_waiting_normal` | Waiting |

## `project.task.burndown.chart.report`

**`is_closed`** — Closing Stage

| Value | Label |
|---|---|
| `closed` | Closed tasks |
| `open` | Open tasks |

**`state`** — State

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `1_done` | Done |
| `04_waiting_normal` | Waiting |
| `03_approved` | Approved |
| `1_canceled` | Cancelled |
| `02_changes_requested` | Changes Requested |

## `project.task.recurrence`

**`repeat_type`** — Until

| Value | Label |
|---|---|
| `forever` | Forever |
| `until` | Until |

**`repeat_unit`** — Repeat Unit

| Value | Label |
|---|---|
| `day` | Days |
| `week` | Weeks |
| `month` | Months |
| `year` | Years |

## `project.task.type`

**`rating_status`** — Customer Ratings Status

| Value | Label |
|---|---|
| `stage` | when reaching this stage |
| `periodic` | on a periodic basis |

**`rating_status_period`** — Rating Frequency

| Value | Label |
|---|---|
| `daily` | Daily |
| `weekly` | Weekly |
| `bimonthly` | Twice a Month |
| `monthly` | Once a Month |
| `quarterly` | Quarterly |
| `yearly` | Yearly |

## `project.update`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`status`** — Status

| Value | Label |
|---|---|
| `on_track` | On Track |
| `at_risk` | At Risk |
| `off_track` | Off Track |
| `on_hold` | On Hold |
| `done` | Complete |

## `purchase.order`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`invoice_status`** — Billing Status

| Value | Label |
|---|---|
| `no` | Nothing to Bill |
| `to invoice` | Waiting Bills |
| `invoiced` | Fully Billed |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

**`receipt_status`** — Receipt Status

| Value | Label |
|---|---|
| `pending` | Not Received |
| `partial` | Partially Received |
| `full` | Fully Received |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | RFQ |
| `sent` | RFQ Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

## `purchase.order.line`

**`display_type`** — Display Type

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |

**`qty_received_method`** — Received Qty Method

| Value | Label |
|---|---|
| `manual` | Manual |
| `stock_moves` | Stock Moves |

## `purchase.report`

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft RFQ |
| `sent` | RFQ Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

## `purchase.requisition`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`requisition_type`** — Agreement Type

| Value | Label |
|---|---|
| `blanket_order` | Blanket Order |
| `purchase_template` | Purchase Template |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `confirmed` | Confirmed |
| `done` | Closed |
| `cancel` | Cancelled |

## `quotation.document`

**`document_type`** — Document Type

| Value | Label |
|---|---|
| `header` | Header |
| `footer` | Footer |

## `rating.mixin`

**`rating_avg_text`** — Rating Avg Text (not stored)

| Value | Label |
|---|---|
| `top` | Happy |
| `ok` | Neutral |
| `ko` | Unhappy |
| `none` | Not Rated yet |

## `rating.rating`

**`rating_text`** — Rating

| Value | Label |
|---|---|
| `top` | Happy |
| `ok` | Neutral |
| `ko` | Unhappy |
| `none` | Not Rated yet |

## `repair.order`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`parts_availability_state`** — Parts Availability State (not stored)

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

**`search_date_category`** — Date Category (not stored)

| Value | Label |
|---|---|
| `before` | Before |
| `yesterday` | Yesterday |
| `today` | Today |
| `day_1` | Tomorrow |
| `day_2` | The day after tomorrow |
| `after` | After |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | New |
| `confirmed` | Confirmed |
| `under_repair` | Under Repair |
| `done` | Repaired |
| `cancel` | Cancelled |

## `report.paperformat`

**`format`** — Paper size

| Value | Label |
|---|---|
| `A0` | A0  5   841 x 1189 mm |
| `A1` | A1  6   594 x 841 mm |
| `A2` | A2  7   420 x 594 mm |
| `A3` | A3  8   297 x 420 mm |
| `A4` | A4  0   210 x 297 mm, 8.26 x 11.69 inches |
| `A5` | A5  9   148 x 210 mm |
| `A6` | A6  10  105 x 148 mm |
| `A7` | A7  11  74 x 105 mm |
| `A8` | A8  12  52 x 74 mm |
| `A9` | A9  13  37 x 52 mm |
| `B0` | B0  14  1000 x 1414 mm |
| `B1` | B1  15  707 x 1000 mm |
| `B2` | B2  17  500 x 707 mm |
| `B3` | B3  18  353 x 500 mm |
| `B4` | B4  19  250 x 353 mm |
| `B5` | B5  1   176 x 250 mm, 6.93 x 9.84 inches |
| `B6` | B6  20  125 x 176 mm |
| `B7` | B7  21  88 x 125 mm |
| `B8` | B8  22  62 x 88 mm |
| `B9` | B9  23  33 x 62 mm |
| `B10` | B10    16  31 x 44 mm |
| `C5E` | C5E 24  163 x 229 mm |
| `Comm10E` | Comm10E 25  105 x 241 mm, U.S. Common 10 Envelope |
| `DLE` | DLE 26 110 x 220 mm |
| `Executive` | Executive 4   7.5 x 10 inches, 190.5 x 254 mm |
| `Folio` | Folio 27  210 x 330 mm |
| `Ledger` | Ledger  28  431.8 x 279.4 mm |
| `Legal` | Legal    3   8.5 x 14 inches, 215.9 x 355.6 mm |
| `Letter` | Letter 2 8.5 x 11 inches, 215.9 x 279.4 mm |
| `Tabloid` | Tabloid 29 279.4 x 431.8 mm |
| `custom` | Custom |

**`orientation`** — Orientation

| Value | Label |
|---|---|
| `Landscape` | Landscape |
| `Portrait` | Portrait |

## `report.pos.order`

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | New |
| `paid` | Paid |
| `done` | Posted |
| `cancel` | Cancelled |

## `report.project.task.user`

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Low priority |
| `1` | Medium priority |
| `2` | High priority |
| `3` | Urgent |

**`state`** — State

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `1_done` | Done |
| `04_waiting_normal` | Waiting |
| `03_approved` | Approved |
| `1_canceled` | Cancelled |
| `02_changes_requested` | Changes Requested |

## `report.stock.quantity`

**`state`** — State

| Value | Label |
|---|---|
| `forecast` | Forecasted Stock |
| `in` | Forecasted Receipts |
| `out` | Forecasted Deliveries |

## `res.company`

**`account_check_printing_layout`** — Check Layout

| Value | Label |
|---|---|
| `disabled` | None |

**`account_peppol_proxy_state`** — PEPPOL status

| Value | Label |
|---|---|
| `not_registered` | Not registered |
| `sender` | Can send but not receive |
| `smp_registration` | Can send, pending registration to receive |
| `receiver` | Can send and receive |
| `rejected` | Rejected |

**`account_price_include`** — Default Sales Price Include

| Value | Label |
|---|---|
| `tax_included` | Tax Included |
| `tax_excluded` | Tax Excluded |

**`annual_inventory_month`** — Annual Inventory Month

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

**`attendance_barcode_source`** — Barcode Source

| Value | Label |
|---|---|
| `scanner` | Scanner |
| `front` | Front Camera |
| `back` | Back Camera |

**`attendance_kiosk_mode`** — Attendance Mode

| Value | Label |
|---|---|
| `barcode` | Barcode / RFID |
| `barcode_manual` | Barcode / RFID and Manual Selection |
| `manual` | Manual Selection |

**`attendance_overtime_validation`** — Extra Hours Validation

| Value | Label |
|---|---|
| `no_validation` | Automatically Approved |
| `by_manager` | Approved by Manager |

**`cost_method`** — Cost Method

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

**`fiscalyear_last_month`** — Fiscalyear Last Month

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

**`font`** — Font

| Value | Label |
|---|---|
| `Lato` | Lato |
| `Roboto` | Roboto |
| `Open_Sans` | Open Sans |
| `Montserrat` | Montserrat |
| `Oswald` | Oswald |
| `Raleway` | Raleway |
| `Tajawal` | Tajawal |
| `Fira_Mono` | Fira Mono |

**`inventory_period`** — Inventory Period

| Value | Label |
|---|---|
| `manual` | Manual |
| `daily` | Daily |
| `monthly` | Monthly |

**`inventory_valuation`** — Valuation

| Value | Label |
|---|---|
| `periodic` | Periodic (at closing) |
| `real_time` | Perpetual (at invoicing) |

**`l10n_dk_nemhandel_proxy_state`** — Nemhandel status

| Value | Label |
|---|---|
| `not_registered` | Not registered |
| `in_verification` | In verification |
| `receiver` | Can send and receive |
| `rejected` | Rejected |

**`l10n_es_edi_verifactu_special_vat_regime`** — Veri*Factu VAT Regime

| Value | Label |
|---|---|
| `simplified` | Simplified Regime |
| `reagyp` | REAGYP (Special Regime for Agriculture, Livestock and Fisheries) |
| `recargo` | Recargo de Equivalencia |

**`l10n_es_sii_tax_agency`** — Tax Agency for SII

| Value | Label |
|---|---|
| `aeat` | Agencia Tributaria española |
| `gipuzkoa` | Hacienda Foral de Gipuzkoa |
| `bizkaia` | Hacienda Foral de Bizkaia |
| `navarra` | Hacienda Foral de Navarra |

**`l10n_es_tbai_tax_agency`** — Tax Agency for TBAI

| Value | Label |
|---|---|
| `araba` | Hacienda Foral de Araba |
| `bizkaia` | Hacienda Foral de Bizkaia |
| `gipuzkoa` | Hacienda Foral de Gipuzkoa |

**`l10n_fr_pdp_periodicity`** — Flow 10 Report Periodicity

| Value | Label |
|---|---|
| `normal_monthly` | Real Monthly Normal Regime |
| `normal_quarterly` | Real Normal Quarterly Regime |
| `simplified_monthly` | Simplified VAT Regime (Monthly) |
| `simplified_bimonthly` | Franchised VAT Regime (Bimonthly) |

**`l10n_hr_mer_connection_mode`** — MojEracun Operating mode

| Value | Label |
|---|---|
| `prod` | Production |
| `test` | Test |
| `demo` | Demo |

**`l10n_hr_mer_connection_state`** — MojEracun connection status

| Value | Label |
|---|---|
| `inactive` | Inactive |
| `active` | Active |

**`l10n_hu_edi_server_mode`** — Server Mode

| Value | Label |
|---|---|
| `production` | Production |
| `test` | Test |
| `demo` | Demo |

**`l10n_hu_tax_regime`** — NAV Tax Regime

| Value | Label |
|---|---|
| `ie` | Individual Exemption |
| `ca` | Cash Accounting |
| `sb` | Small Business |

**`l10n_in_hsn_code_digit`** — HSN Code Digit

| Value | Label |
|---|---|
| `4` | 4 Digits (turnover < 5 CR.) |
| `6` | 6 Digits (turnover > 5 CR.) |
| `8` | 8 Digits |

**`l10n_it_eco_index_liquidation_state`** — Liquidation state

| Value | Label |
|---|---|
| `LS` | The company is in a state of liquidation |
| `LN` | The company is not in a state of liquidation |

**`l10n_it_eco_index_sole_shareholder`** — Shareholder

| Value | Label |
|---|---|
| `NO` | Not a limited liability company |
| `SU` | Socio unico |
| `SM` | Più soci |

**`l10n_it_tax_system`** — Tax System

| Value | Label |
|---|---|
| `RF01` | [RF01] Ordinario |
| `RF02` | [RF02] Contribuenti minimi (art.1, c.96-117, L. 244/07) |
| `RF04` | [RF04] Agricoltura e attività connesse e pesca (artt.34 e 34-bis, DPR 633/72) |
| `RF05` | [RF05] Vendita sali e tabacchi (art.74, c.1, DPR. 633/72) |
| `RF06` | [RF06] Commercio fiammiferi (art.74, c.1, DPR  633/72) |
| `RF07` | [RF07] Editoria (art.74, c.1, DPR  633/72) |
| `RF08` | [RF08] Gestione servizi telefonia pubblica (art.74, c.1, DPR 633/72) |
| `RF09` | [RF09] Rivendita documenti di trasporto pubblico e di sosta (art.74, c.1, DPR  633/72) |
| `RF10` | [RF10] Intrattenimenti, giochi e altre attività di cui alla tariffa allegata al DPR 640/72 (art.74, c.6, DPR 633/72) |
| `RF11` | [RF11] Agenzie viaggi e turismo (art.74-ter, DPR 633/72) |
| `RF12` | [RF12] Agriturismo (art.5, c.2, L. 413/91) |
| `RF13` | [RF13] Vendite a domicilio (art.25-bis, c.6, DPR  600/73) |
| `RF14` | [RF14] Rivendita beni usati, oggetti d’arte, d’antiquariato o da collezione (art.36, DL 41/95) |
| `RF15` | [RF15] Agenzie di vendite all’asta di oggetti d’arte, antiquariato o da collezione (art.40-bis, DL 41/95) |
| `RF16` | [RF16] IVA per cassa P.A. (art.6, c.5, DPR 633/72) |
| `RF17` | [RF17] IVA per cassa (art. 32-bis, DL 83/2012) |
| `RF18` | [RF18] Altro |
| `RF19` | [RF19] Regime forfettario (art.1, c.54-89, L. 190/2014) |

**`l10n_jo_edi_taxpayer_type`** — JoFotara Taxpayer Type

| Value | Label |
|---|---|
| `income` | Unregistered in the sales tax |
| `sales` | Registered in the sales tax |
| `special` | Registered in the special sales tax |

**`l10n_my_edi_mode`** — L10N My Edi Mode

| Value | Label |
|---|---|
| `test` | Pre-Production |
| `prod` | Production |

**`l10n_sa_api_mode`** — L10N Sa Api Mode

| Value | Label |
|---|---|
| `sandbox` | Sandbox |
| `preprod` | Simulation (Pre-Production) |
| `prod` | Production |

**`layout_background`** — Layout Background

| Value | Label |
|---|---|
| `Blank` | Blank |
| `Demo logo` | Demo logo |
| `Custom` | Custom |

**`pdp_kyc_status`** — Pdp Kyc Status

| Value | Label |
|---|---|
| `processing` | Processing |
| `success` | Success |
| `fail` | Fail |

**`po_double_validation`** — Levels of Approvals

| Value | Label |
|---|---|
| `one_step` | Confirm purchase orders in one step |
| `two_step` | Get 2 levels of approvals to confirm a purchase order |

**`po_lock`** — Purchase Order Modification

| Value | Label |
|---|---|
| `edit` | Allow to edit purchase orders |
| `lock` | Confirmed purchase orders are not editable |

**`point_of_sale_ticket_portal_url_display_mode`** — Print

| Value | Label |
|---|---|
| `qr_code` | QR code |
| `url` | URL |
| `qr_code_and_url` | QR code + URL |

**`point_of_sale_update_stock_quantities`** — Update quantities in stock

| Value | Label |
|---|---|
| `closing` | At the session closing |
| `real` | In real time |

**`quick_edit_mode`** — Quick encoding

| Value | Label |
|---|---|
| `out_invoices` | Customer Invoices |
| `in_invoices` | Vendor Bills |
| `out_and_in_invoices` | Customer Invoices and Vendor Bills |

**`sale_onboarding_payment_method`** — Sale onboarding selected payment method

| Value | Label |
|---|---|
| `digital_signature` | Sign online |
| `paypal` | PayPal |
| `stripe` | Stripe |
| `other` | Pay with another payment provider |
| `manual` | Manual Payment |

**`sms_provider`** — SMS Provider

| Value | Label |
|---|---|
| `iap` | Send via the system |
| `twilio` | Send via Twilio |

**`stock_confirmation_type`** — Stock Confirmation Type

| Value | Label |
|---|---|
| `sms` | SMS |

**`tax_calculation_rounding_method`** — Tax Calculation Rounding Method

| Value | Label |
|---|---|
| `round_globally` | Round per Tax |
| `round_per_line` | Round per Line |

**`terms_type`** — Terms & Conditions format

| Value | Label |
|---|---|
| `plain` | Add a Note |
| `html` | Add a link to a Web Page |

## `res.config.settings`

**`account_on_checkout`** — Customer Accounts (not stored)

| Value | Label |
|---|---|
| `optional` | Optional |
| `disabled` | Disabled |
| `mandatory` | Mandatory |

**`auth_signup_uninvited`** — Customer Account (not stored)

| Value | Label |
|---|---|
| `b2b` | On invitation |
| `b2c` | Free sign up |

**`auth_totp_policy`** — Two-factor authentication enforcing policy

| Value | Label |
|---|---|
| `employee_required` | Employees only |
| `all_required` | All users |

**`cloud_storage_provider`** — Cloud Storage Provider for new attachments

| Value | Label |
|---|---|
| `azure` | Azure Cloud Storage |
| `google` | Google Cloud Storage |

**`crm_auto_assignment_action`** — Auto Assignment Action

| Value | Label |
|---|---|
| `manual` | Manually |
| `auto` | Repeatedly |

**`crm_auto_assignment_interval_type`** — Auto Assignment Interval Unit

| Value | Label |
|---|---|
| `minutes` | Minutes |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |

**`default_invoice_policy`** — Invoicing Policy

| Value | Label |
|---|---|
| `order` | Invoice what is ordered |
| `delivery` | Invoice what is delivered |

**`default_picking_policy`** — Picking Policy

| Value | Label |
|---|---|
| `direct` | Ship products as soon as available, with back orders |
| `one` | Ship all products at once |

**`l10n_in_gsp`** — GSP (not stored)

| Value | Label |
|---|---|
| `bvm` | BVM IT Consulting |
| `tera` | Tera Software (Deprecated) |

**`lead_enrich_auto`** — Enrich lead automatically

| Value | Label |
|---|---|
| `manual` | Enrich leads on demand only |
| `auto` | Enrich all leads automatically |

**`onboarding_payment_module`** — Onboarding Payment Module (not stored)

| Value | Label |
|---|---|
| `mercado_pago` | Mercado Pago |
| `razorpay` | Razorpay |
| `stripe` | Stripe |

**`peppol_participation_role`** — Peppol Participation Role (not stored)

| Value | Label |
|---|---|
| `sending_and_receiving` | Sending & Receiving |
| `sending_only` | Sending Only |

**`product_volume_volume_in_cubic_feet`** — Volume unit of measure

| Value | Label |
|---|---|
| `0` | Cubic Meters (m³) |
| `1` | Cubic Feet (ft³) |

**`product_weight_in_lbs`** — Weight unit of measure

| Value | Label |
|---|---|
| `0` | Kilograms (kg) |
| `1` | Pounds (lb) |

**`timesheet_encode_method`** — Encoding Method (not stored)

| Value | Label |
|---|---|
| `hours` | Hours / Minutes |
| `days` | Days / Half-Days |

## `res.country`

**`name_position`** — Customer Name Position

| Value | Label |
|---|---|
| `before` | Before Address |
| `after` | After Address |

## `res.currency`

**`position`** — Symbol Position

| Value | Label |
|---|---|
| `after` | After Amount |
| `before` | Before Amount |

## `res.device`

**`device_type`** — Device Type

| Value | Label |
|---|---|
| `computer` | Computer |
| `mobile` | Mobile |

## `res.device.log`

**`device_type`** — Device Type

| Value | Label |
|---|---|
| `computer` | Computer |
| `mobile` | Mobile |

## `res.groups`

**`lock_timeout_2fa_selection`** — Lock Timeout 2Fa Selection (not stored)

| Value | Label |
|---|---|
| `without_2fa` | Logout |
| `with_2fa` | Logout with two-factor authentication |

**`lock_timeout_delay_unit`** — Lock Timeout Delay Unit (not stored)

| Value | Label |
|---|---|
| `minutes` | minutes |
| `hours` | hours |
| `days` | days |

**`lock_timeout_inactivity_2fa_selection`** — Lock Timeout Inactivity 2Fa Selection (not stored)

| Value | Label |
|---|---|
| `without_2fa` | Screen lock |
| `with_2fa` | Screen lock with two-factor authentication |

**`lock_timeout_inactivity_delay_unit`** — Lock Timeout Inactivity Delay Unit (not stored)

| Value | Label |
|---|---|
| `minutes` | minutes |
| `hours` | hours |
| `days` | days |

## `res.lang`

**`direction`** — Direction

| Value | Label |
|---|---|
| `ltr` | Left-to-Right |
| `rtl` | Right-to-Left |

**`grouping`** — Separator Format

| Value | Label |
|---|---|
| `[3,0]` | International Grouping |
| `[3,2,0]` | Indian Grouping |

**`time_format`** — Time Format

| Value | Label |
|---|---|
| `%H:%M:%S` | 13:00:00 |
| `%I:%M:%S %p` |  1:00:00 PM |

**`week_start`** — First Day of Week

| Value | Label |
|---|---|
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |
| `7` | Sunday |

## `res.partner`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`autopost_bills`** — Auto-post bills

| Value | Label |
|---|---|
| `always` | Always |
| `ask` | Ask after 3 validations without edits |
| `never` | Never |

**`company_type`** — Company Type (not stored)

| Value | Label |
|---|---|
| `person` | Person |
| `company` | Company |

**`group_on`** — Week Day

| Value | Label |
|---|---|
| `default` | Expected Date |
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |
| `7` | Sunday |

**`group_rfq`** — Group RFQ

| Value | Label |
|---|---|
| `default` | On Order |
| `day` | Daily |
| `week` | Weekly |
| `all` | Always |

**`invoice_edi_format`** — eInvoice format (not stored)

| Value | Label |
|---|---|
| `facturx` | France (FacturX) |
| `ubl_bis3` | EU Standard (Peppol Bis 3.0) |
| `zugferd` | Germany (ZUGFeRD) |
| `xrechnung` | Germany (XRechnung) |
| `nlcius` | Netherlands (NLCIUS) |
| `ubl_a_nz` | Australia (BIS Billing 3.0 A-NZ) |
| `ubl_sg` | Singapore (BIS Billing 3.0 SG) |
| `pint_anz` | Australia (Peppol Pint AU) |
| `pint_jp` | Japan (Peppol PINT JP) |
| `pint_my` | Malaysia (Peppol PINT MY) |
| `pint_sg` | Singapore (Peppol PINT SG) |
| `ubl_tr` | Türkiye (UBL TR 1.2) |
| `tw_ecpay` | ECPay |
| `oioubl_21` | OIOUBL 2.1 |
| `oioubl_201` | Denmark (Oioubl) |
| `es_facturae` | Spain (FacturaE) |
| `ubl_hr` | CIUS HR |
| `it_edi_xml` | Italy (Factura PA) |
| `fa3_pl` | Polish FA3 |
| `ciusro` | Romania (CIUS RO) |
| `vn_sinvoice` | Vietnam (SInvoice) |
| `ubl_21_fr` | France E-Invoicing (UBL 2.1) |

**`invoice_sending_method`** — Invoice sending

| Value | Label |
|---|---|
| `manual` | Manual |
| `email` | by Email |
| `snailmail` | by Post |
| `peppol` | by Peppol |
| `nemhandel` | By Nemhandel |
| `mojeracun` | by MojEracun |

**`l10n_ar_gross_income_type`** — Gross Income Type

| Value | Label |
|---|---|
| `multilateral` | Multilateral |
| `local` | Local |
| `exempt` | Exempt |

**`l10n_cl_sii_taxpayer_type`** — Taxpayer Type

| Value | Label |
|---|---|
| `1` | VAT Affected (1st Category) |
| `2` | Fees Receipt Issuer (2nd category) |
| `3` | End Consumer |
| `4` | Foreigner |

**`l10n_id_buyer_document_type`** — Document Type

| Value | Label |
|---|---|
| `TIN` | TIN |
| `NIK` | NIK |
| `Passport` | Passport |
| `Other` | Others |

**`l10n_id_kode_transaksi`** — Invoice Transaction Code

| Value | Label |
|---|---|
| `01` | 01 To the Parties that is not VAT Collector (Regular Customers) |
| `02` | 02 To the Treasurer |
| `03` | 03 To other VAT Collectors other than the Treasurer |
| `04` | 04 Other Value of VAT Imposition Base |
| `05` | 05 Specified Amount (Article 9A Paragraph (1) VAT Law) |
| `06` | 06 to individuals holding foreign passports |
| `07` | 07 Deliveries that the VAT is not Collected |
| `08` | 08 Deliveries that the VAT is Exempted |
| `09` | 09 Deliveries of Assets (Article 16D of VAT Law) |
| `10` | 10 Other deliveries |

**`l10n_in_gst_treatment`** — GST Treatment

| Value | Label |
|---|---|
| `regular` | Registered Business - Regular |
| `composition` | Registered Business - Composition |
| `unregistered` | Unregistered Business |
| `consumer` | Consumer |
| `overseas` | Overseas |
| `special_economic_zone` | Special Economic Zone |
| `deemed_export` | Deemed Export |
| `uin_holders` | UIN Holders |

**`l10n_my_identification_type`** — ID Type

| Value | Label |
|---|---|
| `NRIC` | MyKad/MyTentera/MyPR/MyKAS |
| `BRN` | Business Registration Number |
| `PASSPORT` | Passport |
| `ARMY` | Army |

**`l10n_my_tin_validation_state`** — Tin Validation State

| Value | Label |
|---|---|
| `valid` | Valid |
| `invalid` | Invalid |

**`l10n_sa_edi_additional_identification_scheme`** — Identification Scheme

| Value | Label |
|---|---|
| `TIN` | Tax Identification Number |
| `CRN` | Commercial Registration Number |
| `MOM` | Momra License |
| `MLS` | MLSD License |
| `700` | 700 Number |
| `SAG` | Sagia License |
| `NAT` | National ID |
| `GCC` | GCC ID |
| `IQA` | Iqama Number |
| `PAS` | Passport ID |
| `OTH` | Other ID |

**`l10n_tr_nilvera_customer_status`** — Nilvera Status

| Value | Label |
|---|---|
| `not_checked` | Not Verified |
| `earchive` | E-Archive |
| `einvoice` | E-Invoice |

**`nemhandel_identifier_type`** — Nemhandel Endpoint Type

| Value | Label |
|---|---|
| `0088` | EAN/GLN |
| `0184` | CVR |
| `9918` | IBAN |
| `0198` | SE |

**`nemhandel_verification_state`** — Nemhandel endpoint verification

| Value | Label |
|---|---|
| `not_verified` | Not verified yet |
| `not_valid` | Not on Nemhandel |
| `valid` | Valid |

**`pdp_verification_display_state`** — E-Invoicing State (not stored)

| Value | Label |
|---|---|
| `not_verified` | Not verified yet |
| `pdp_not_valid` | Partner is not in the annuaire |
| `pdp_not_valid_format` | Partner cannot receive format |
| `pdp_valid` | Partner is in the annuaire |
| `peppol_not_valid` | Partner is not on Peppol |
| `peppol_not_valid_format` | Partner cannot receive format |
| `peppol_valid` | Partner is on Peppol |

**`peppol_eas`** — Peppol e-address (EAS)

| Value | Label |
|---|---|
| `9923` | Albania VAT |
| `9922` | Andorra VAT |
| `0151` | Australia ABN |
| `9914` | Austria UID |
| `9915` | Austria VOKZ |
| `0208` | Belgian Company Registry |
| `9925` | Belgian VAT |
| `9924` | Bosnia and Herzegovina VAT |
| `9926` | Bulgaria VAT |
| `9934` | Croatia VAT |
| `9928` | Cyprus VAT |
| `9929` | Czech Republic VAT |
| `0096` | Denmark P |
| `0184` | Denmark CVR |
| `0198` | Denmark SE |
| `0191` | Estonia Company code |
| `9931` | Estonia VAT |
| `0037` | Finland LY-tunnus |
| `0216` | Finland OVT code |
| `0213` | Finland VAT |
| `0002` | France SIRENE |
| `0009` | France SIRET |
| `9957` | France VAT |
| `0225` | France FRCTC Electronic Address |
| `0240` | France Register of legal persons |
| `0246` | German Electronic Business Address |
| `0204` | Germany Leitweg-ID |
| `9930` | Germany VAT |
| `9933` | Greece VAT |
| `9910` | Hungary VAT |
| `0196` | Iceland Kennitala |
| `9935` | Ireland VAT |
| `0211` | Italia Partita IVA |
| `0097` | Italia FTI |
| `0188` | Japan SST |
| `0221` | Japan IIN |
| `0218` | Latvia Unified registration number |
| `9939` | Latvia VAT |
| `9936` | Liechtenstein VAT |
| `0200` | Lithuania JAK |
| `9937` | Lithuania VAT |
| `9938` | Luxembourg VAT |
| `9942` | Macedonia VAT |
| `0230` | Malaysia |
| `9943` | Malta VAT |
| `9940` | Monaco VAT |
| `9941` | Montenegro VAT |
| `0106` | Netherlands KvK |
| `0190` | Netherlands OIN |
| `9944` | Netherlands VAT |
| `0244` | Nigeria Tax Identification |
| `0192` | Norway Org.nr. |
| `9945` | Poland VAT |
| `9946` | Portugal VAT |
| `9947` | Romania VAT |
| `9948` | Serbia VAT |
| `0195` | Singapore UEN |
| `0245` | SK Tax identification number (DIČ) |
| `9949` | Slovenia VAT |
| `9950` | Slovakia VAT |
| `9920` | Spain VAT |
| `0007` | Sweden Org.nr. |
| `9955` | Sweden VAT |
| `9927` | Swiss VAT |
| `0183` | Swiss UIDB |
| `9952` | Turkey VAT |
| `0235` | UAE Tax Identification Number (TIN) |
| `9932` | United Kingdom VAT |
| `9959` | USA EIN |
| `0060` | DUNS Number |
| `0088` | EAN Location Code |
| `0130` | Directorates of the European Commission |
| `0135` | SIA Object Identifiers |
| `0142` | SECETI Object Identifiers |
| `0193` | UBL.BE party identifier |
| `0199` | Legal Entity Identifier (LEI) |
| `0201` | Codice Univoco Unità Organizzativa iPA |
| `0202` | Indirizzo di Posta Elettronica Certificata |
| `0209` | GS1 identification keys |
| `0210` | Codice Fiscale |
| `9913` | Business Registers Network |
| `9918` | S.W.I.F.T |
| `9919` | Kennziffer des Unternehmensregisters |
| `9951` | San Marino VAT |
| `9953` | Vatican VAT |
| `AN` | O.F.T.P. (ODETTE File Transfer Protocol) |
| `AQ` | X.400 address for mail text |
| `AS` | AS2 exchange |
| `AU` | File Transfer Protocol |
| `EM` | Electronic mail |
| `odemo` | the demonstration data set ID |

**`peppol_verification_state`** — Peppol status

| Value | Label |
|---|---|
| `not_verified` | Unchecked |
| `not_valid` | Partner is not on Peppol |
| `not_valid_format` | Partner cannot receive format |
| `valid` | Partner is on Peppol |

**`trust`** — Degree of trust you have in this debtor

| Value | Label |
|---|---|
| `good` | Good Debtor |
| `normal` | Normal Debtor |
| `bad` | Bad Debtor |

**`type`** — Address Type

| Value | Label |
|---|---|
| `contact` | Contact |
| `invoice` | Invoice |
| `delivery` | Delivery |
| `facturae_ac` | FACe Center |
| `other` | Other |

**`tz`** — Timezone

| Value | Label |
|---|---|
| `Africa/Abidjan` | Africa/Abidjan |
| `Africa/Accra` | Africa/Accra |
| `Africa/Addis_Ababa` | Africa/Addis_Ababa |
| `Africa/Algiers` | Africa/Algiers |
| `Africa/Asmara` | Africa/Asmara |
| `Africa/Asmera` | Africa/Asmera |
| `Africa/Bamako` | Africa/Bamako |
| `Africa/Bangui` | Africa/Bangui |
| `Africa/Banjul` | Africa/Banjul |
| `Africa/Bissau` | Africa/Bissau |
| `Africa/Blantyre` | Africa/Blantyre |
| `Africa/Brazzaville` | Africa/Brazzaville |
| `Africa/Bujumbura` | Africa/Bujumbura |
| `Africa/Cairo` | Africa/Cairo |
| `Africa/Casablanca` | Africa/Casablanca |
| `Africa/Ceuta` | Africa/Ceuta |
| `Africa/Conakry` | Africa/Conakry |
| `Africa/Dakar` | Africa/Dakar |
| `Africa/Dar_es_Salaam` | Africa/Dar_es_Salaam |
| `Africa/Djibouti` | Africa/Djibouti |
| `Africa/Douala` | Africa/Douala |
| `Africa/El_Aaiun` | Africa/El_Aaiun |
| `Africa/Freetown` | Africa/Freetown |
| `Africa/Gaborone` | Africa/Gaborone |
| `Africa/Harare` | Africa/Harare |
| `Africa/Johannesburg` | Africa/Johannesburg |
| `Africa/Juba` | Africa/Juba |
| `Africa/Kampala` | Africa/Kampala |
| `Africa/Khartoum` | Africa/Khartoum |
| `Africa/Kigali` | Africa/Kigali |
| `Africa/Kinshasa` | Africa/Kinshasa |
| `Africa/Lagos` | Africa/Lagos |
| `Africa/Libreville` | Africa/Libreville |
| `Africa/Lome` | Africa/Lome |
| `Africa/Luanda` | Africa/Luanda |
| `Africa/Lubumbashi` | Africa/Lubumbashi |
| `Africa/Lusaka` | Africa/Lusaka |
| `Africa/Malabo` | Africa/Malabo |
| `Africa/Maputo` | Africa/Maputo |
| `Africa/Maseru` | Africa/Maseru |
| `Africa/Mbabane` | Africa/Mbabane |
| `Africa/Mogadishu` | Africa/Mogadishu |
| `Africa/Monrovia` | Africa/Monrovia |
| `Africa/Nairobi` | Africa/Nairobi |
| `Africa/Ndjamena` | Africa/Ndjamena |
| `Africa/Niamey` | Africa/Niamey |
| `Africa/Nouakchott` | Africa/Nouakchott |
| `Africa/Ouagadougou` | Africa/Ouagadougou |
| `Africa/Porto-Novo` | Africa/Porto-Novo |
| `Africa/Sao_Tome` | Africa/Sao_Tome |
| `Africa/Timbuktu` | Africa/Timbuktu |
| `Africa/Tripoli` | Africa/Tripoli |
| `Africa/Tunis` | Africa/Tunis |
| `Africa/Windhoek` | Africa/Windhoek |
| `America/Adak` | America/Adak |
| `America/Anchorage` | America/Anchorage |
| `America/Anguilla` | America/Anguilla |
| `America/Antigua` | America/Antigua |
| `America/Araguaina` | America/Araguaina |
| `America/Argentina/Buenos_Aires` | America/Argentina/Buenos_Aires |
| `America/Argentina/Catamarca` | America/Argentina/Catamarca |
| `America/Argentina/ComodRivadavia` | America/Argentina/ComodRivadavia |
| `America/Argentina/Cordoba` | America/Argentina/Cordoba |
| `America/Argentina/Jujuy` | America/Argentina/Jujuy |
| `America/Argentina/La_Rioja` | America/Argentina/La_Rioja |
| `America/Argentina/Mendoza` | America/Argentina/Mendoza |
| `America/Argentina/Rio_Gallegos` | America/Argentina/Rio_Gallegos |
| `America/Argentina/Salta` | America/Argentina/Salta |
| `America/Argentina/San_Juan` | America/Argentina/San_Juan |
| `America/Argentina/San_Luis` | America/Argentina/San_Luis |
| `America/Argentina/Tucuman` | America/Argentina/Tucuman |
| `America/Argentina/Ushuaia` | America/Argentina/Ushuaia |
| `America/Aruba` | America/Aruba |
| `America/Asuncion` | America/Asuncion |
| `America/Atikokan` | America/Atikokan |
| `America/Atka` | America/Atka |
| `America/Bahia` | America/Bahia |
| `America/Bahia_Banderas` | America/Bahia_Banderas |
| `America/Barbados` | America/Barbados |
| `America/Belem` | America/Belem |
| `America/Belize` | America/Belize |
| `America/Blanc-Sablon` | America/Blanc-Sablon |
| `America/Boa_Vista` | America/Boa_Vista |
| `America/Bogota` | America/Bogota |
| `America/Boise` | America/Boise |
| `America/Buenos_Aires` | America/Buenos_Aires |
| `America/Cambridge_Bay` | America/Cambridge_Bay |
| `America/Campo_Grande` | America/Campo_Grande |
| `America/Cancun` | America/Cancun |
| `America/Caracas` | America/Caracas |
| `America/Catamarca` | America/Catamarca |
| `America/Cayenne` | America/Cayenne |
| `America/Cayman` | America/Cayman |
| `America/Chicago` | America/Chicago |
| `America/Chihuahua` | America/Chihuahua |
| `America/Ciudad_Juarez` | America/Ciudad_Juarez |
| `America/Coral_Harbour` | America/Coral_Harbour |
| `America/Cordoba` | America/Cordoba |
| `America/Costa_Rica` | America/Costa_Rica |
| `America/Coyhaique` | America/Coyhaique |
| `America/Creston` | America/Creston |
| `America/Cuiaba` | America/Cuiaba |
| `America/Curacao` | America/Curacao |
| `America/Danmarkshavn` | America/Danmarkshavn |
| `America/Dawson` | America/Dawson |
| `America/Dawson_Creek` | America/Dawson_Creek |
| `America/Denver` | America/Denver |
| `America/Detroit` | America/Detroit |
| `America/Dominica` | America/Dominica |
| `America/Edmonton` | America/Edmonton |
| `America/Eirunepe` | America/Eirunepe |
| `America/El_Salvador` | America/El_Salvador |
| `America/Ensenada` | America/Ensenada |
| `America/Fort_Nelson` | America/Fort_Nelson |
| `America/Fort_Wayne` | America/Fort_Wayne |
| `America/Fortaleza` | America/Fortaleza |
| `America/Glace_Bay` | America/Glace_Bay |
| `America/Godthab` | America/Godthab |
| `America/Goose_Bay` | America/Goose_Bay |
| `America/Grand_Turk` | America/Grand_Turk |
| `America/Grenada` | America/Grenada |
| `America/Guadeloupe` | America/Guadeloupe |
| `America/Guatemala` | America/Guatemala |
| `America/Guayaquil` | America/Guayaquil |
| `America/Guyana` | America/Guyana |
| `America/Halifax` | America/Halifax |
| `America/Havana` | America/Havana |
| `America/Hermosillo` | America/Hermosillo |
| `America/Indiana/Indianapolis` | America/Indiana/Indianapolis |
| `America/Indiana/Knox` | America/Indiana/Knox |
| `America/Indiana/Marengo` | America/Indiana/Marengo |
| `America/Indiana/Petersburg` | America/Indiana/Petersburg |
| `America/Indiana/Tell_City` | America/Indiana/Tell_City |
| `America/Indiana/Vevay` | America/Indiana/Vevay |
| `America/Indiana/Vincennes` | America/Indiana/Vincennes |
| `America/Indiana/Winamac` | America/Indiana/Winamac |
| `America/Indianapolis` | America/Indianapolis |
| `America/Inuvik` | America/Inuvik |
| `America/Iqaluit` | America/Iqaluit |
| `America/Jamaica` | America/Jamaica |
| `America/Jujuy` | America/Jujuy |
| `America/Juneau` | America/Juneau |
| `America/Kentucky/Louisville` | America/Kentucky/Louisville |
| `America/Kentucky/Monticello` | America/Kentucky/Monticello |
| `America/Knox_IN` | America/Knox_IN |
| `America/Kralendijk` | America/Kralendijk |
| `America/La_Paz` | America/La_Paz |
| `America/Lima` | America/Lima |
| `America/Los_Angeles` | America/Los_Angeles |
| `America/Louisville` | America/Louisville |
| `America/Lower_Princes` | America/Lower_Princes |
| `America/Maceio` | America/Maceio |
| `America/Managua` | America/Managua |
| `America/Manaus` | America/Manaus |
| `America/Marigot` | America/Marigot |
| `America/Martinique` | America/Martinique |
| `America/Matamoros` | America/Matamoros |
| `America/Mazatlan` | America/Mazatlan |
| `America/Mendoza` | America/Mendoza |
| `America/Menominee` | America/Menominee |
| `America/Merida` | America/Merida |
| `America/Metlakatla` | America/Metlakatla |
| `America/Mexico_City` | America/Mexico_City |
| `America/Miquelon` | America/Miquelon |
| `America/Moncton` | America/Moncton |
| `America/Monterrey` | America/Monterrey |
| `America/Montevideo` | America/Montevideo |
| `America/Montreal` | America/Montreal |
| `America/Montserrat` | America/Montserrat |
| `America/Nassau` | America/Nassau |
| `America/New_York` | America/New_York |
| `America/Nipigon` | America/Nipigon |
| `America/Nome` | America/Nome |
| `America/Noronha` | America/Noronha |
| `America/North_Dakota/Beulah` | America/North_Dakota/Beulah |
| `America/North_Dakota/Center` | America/North_Dakota/Center |
| `America/North_Dakota/New_Salem` | America/North_Dakota/New_Salem |
| `America/Nuuk` | America/Nuuk |
| `America/Ojinaga` | America/Ojinaga |
| `America/Panama` | America/Panama |
| `America/Pangnirtung` | America/Pangnirtung |
| `America/Paramaribo` | America/Paramaribo |
| `America/Phoenix` | America/Phoenix |
| `America/Port-au-Prince` | America/Port-au-Prince |
| `America/Port_of_Spain` | America/Port_of_Spain |
| `America/Porto_Acre` | America/Porto_Acre |
| `America/Porto_Velho` | America/Porto_Velho |
| `America/Puerto_Rico` | America/Puerto_Rico |
| `America/Punta_Arenas` | America/Punta_Arenas |
| `America/Rainy_River` | America/Rainy_River |
| `America/Rankin_Inlet` | America/Rankin_Inlet |
| `America/Recife` | America/Recife |
| `America/Regina` | America/Regina |
| `America/Resolute` | America/Resolute |
| `America/Rio_Branco` | America/Rio_Branco |
| `America/Rosario` | America/Rosario |
| `America/Santa_Isabel` | America/Santa_Isabel |
| `America/Santarem` | America/Santarem |
| `America/Santiago` | America/Santiago |
| `America/Santo_Domingo` | America/Santo_Domingo |
| `America/Sao_Paulo` | America/Sao_Paulo |
| `America/Scoresbysund` | America/Scoresbysund |
| `America/Shiprock` | America/Shiprock |
| `America/Sitka` | America/Sitka |
| `America/St_Barthelemy` | America/St_Barthelemy |
| `America/St_Johns` | America/St_Johns |
| `America/St_Kitts` | America/St_Kitts |
| `America/St_Lucia` | America/St_Lucia |
| `America/St_Thomas` | America/St_Thomas |
| `America/St_Vincent` | America/St_Vincent |
| `America/Swift_Current` | America/Swift_Current |
| `America/Tegucigalpa` | America/Tegucigalpa |
| `America/Thule` | America/Thule |
| `America/Thunder_Bay` | America/Thunder_Bay |
| `America/Tijuana` | America/Tijuana |
| `America/Toronto` | America/Toronto |
| `America/Tortola` | America/Tortola |
| `America/Vancouver` | America/Vancouver |
| `America/Virgin` | America/Virgin |
| `America/Whitehorse` | America/Whitehorse |
| `America/Winnipeg` | America/Winnipeg |
| `America/Yakutat` | America/Yakutat |
| `America/Yellowknife` | America/Yellowknife |
| `Antarctica/Casey` | Antarctica/Casey |
| `Antarctica/Davis` | Antarctica/Davis |
| `Antarctica/DumontDUrville` | Antarctica/DumontDUrville |
| `Antarctica/Macquarie` | Antarctica/Macquarie |
| `Antarctica/Mawson` | Antarctica/Mawson |
| `Antarctica/McMurdo` | Antarctica/McMurdo |
| `Antarctica/Palmer` | Antarctica/Palmer |
| `Antarctica/Rothera` | Antarctica/Rothera |
| `Antarctica/South_Pole` | Antarctica/South_Pole |
| `Antarctica/Syowa` | Antarctica/Syowa |
| `Antarctica/Troll` | Antarctica/Troll |
| `Antarctica/Vostok` | Antarctica/Vostok |
| `Arctic/Longyearbyen` | Arctic/Longyearbyen |
| `Asia/Aden` | Asia/Aden |
| `Asia/Almaty` | Asia/Almaty |
| `Asia/Amman` | Asia/Amman |
| `Asia/Anadyr` | Asia/Anadyr |
| `Asia/Aqtau` | Asia/Aqtau |
| `Asia/Aqtobe` | Asia/Aqtobe |
| `Asia/Ashgabat` | Asia/Ashgabat |
| `Asia/Ashkhabad` | Asia/Ashkhabad |
| `Asia/Atyrau` | Asia/Atyrau |
| `Asia/Baghdad` | Asia/Baghdad |
| `Asia/Bahrain` | Asia/Bahrain |
| `Asia/Baku` | Asia/Baku |
| `Asia/Bangkok` | Asia/Bangkok |
| `Asia/Barnaul` | Asia/Barnaul |
| `Asia/Beirut` | Asia/Beirut |
| `Asia/Bishkek` | Asia/Bishkek |
| `Asia/Brunei` | Asia/Brunei |
| `Asia/Calcutta` | Asia/Calcutta |
| `Asia/Chita` | Asia/Chita |
| `Asia/Choibalsan` | Asia/Choibalsan |
| `Asia/Chongqing` | Asia/Chongqing |
| `Asia/Chungking` | Asia/Chungking |
| `Asia/Colombo` | Asia/Colombo |
| `Asia/Dacca` | Asia/Dacca |
| `Asia/Damascus` | Asia/Damascus |
| `Asia/Dhaka` | Asia/Dhaka |
| `Asia/Dili` | Asia/Dili |
| `Asia/Dubai` | Asia/Dubai |
| `Asia/Dushanbe` | Asia/Dushanbe |
| `Asia/Famagusta` | Asia/Famagusta |
| `Asia/Gaza` | Asia/Gaza |
| `Asia/Harbin` | Asia/Harbin |
| `Asia/Hebron` | Asia/Hebron |
| `Asia/Ho_Chi_Minh` | Asia/Ho_Chi_Minh |
| `Asia/Hong_Kong` | Asia/Hong_Kong |
| `Asia/Hovd` | Asia/Hovd |
| `Asia/Irkutsk` | Asia/Irkutsk |
| `Asia/Istanbul` | Asia/Istanbul |
| `Asia/Jakarta` | Asia/Jakarta |
| `Asia/Jayapura` | Asia/Jayapura |
| `Asia/Jerusalem` | Asia/Jerusalem |
| `Asia/Kabul` | Asia/Kabul |
| `Asia/Kamchatka` | Asia/Kamchatka |
| `Asia/Karachi` | Asia/Karachi |
| `Asia/Kashgar` | Asia/Kashgar |
| `Asia/Kathmandu` | Asia/Kathmandu |
| `Asia/Katmandu` | Asia/Katmandu |
| `Asia/Khandyga` | Asia/Khandyga |
| `Asia/Kolkata` | Asia/Kolkata |
| `Asia/Krasnoyarsk` | Asia/Krasnoyarsk |
| `Asia/Kuala_Lumpur` | Asia/Kuala_Lumpur |
| `Asia/Kuching` | Asia/Kuching |
| `Asia/Kuwait` | Asia/Kuwait |
| `Asia/Macao` | Asia/Macao |
| `Asia/Macau` | Asia/Macau |
| `Asia/Magadan` | Asia/Magadan |
| `Asia/Makassar` | Asia/Makassar |
| `Asia/Manila` | Asia/Manila |
| `Asia/Muscat` | Asia/Muscat |
| `Asia/Nicosia` | Asia/Nicosia |
| `Asia/Novokuznetsk` | Asia/Novokuznetsk |
| `Asia/Novosibirsk` | Asia/Novosibirsk |
| `Asia/Omsk` | Asia/Omsk |
| `Asia/Oral` | Asia/Oral |
| `Asia/Phnom_Penh` | Asia/Phnom_Penh |
| `Asia/Pontianak` | Asia/Pontianak |
| `Asia/Pyongyang` | Asia/Pyongyang |
| `Asia/Qatar` | Asia/Qatar |
| `Asia/Qostanay` | Asia/Qostanay |
| `Asia/Qyzylorda` | Asia/Qyzylorda |
| `Asia/Rangoon` | Asia/Rangoon |
| `Asia/Riyadh` | Asia/Riyadh |
| `Asia/Saigon` | Asia/Saigon |
| `Asia/Sakhalin` | Asia/Sakhalin |
| `Asia/Samarkand` | Asia/Samarkand |
| `Asia/Seoul` | Asia/Seoul |
| `Asia/Shanghai` | Asia/Shanghai |
| `Asia/Singapore` | Asia/Singapore |
| `Asia/Srednekolymsk` | Asia/Srednekolymsk |
| `Asia/Taipei` | Asia/Taipei |
| `Asia/Tashkent` | Asia/Tashkent |
| `Asia/Tbilisi` | Asia/Tbilisi |
| `Asia/Tehran` | Asia/Tehran |
| `Asia/Tel_Aviv` | Asia/Tel_Aviv |
| `Asia/Thimbu` | Asia/Thimbu |
| `Asia/Thimphu` | Asia/Thimphu |
| `Asia/Tokyo` | Asia/Tokyo |
| `Asia/Tomsk` | Asia/Tomsk |
| `Asia/Ujung_Pandang` | Asia/Ujung_Pandang |
| `Asia/Ulaanbaatar` | Asia/Ulaanbaatar |
| `Asia/Ulan_Bator` | Asia/Ulan_Bator |
| `Asia/Urumqi` | Asia/Urumqi |
| `Asia/Ust-Nera` | Asia/Ust-Nera |
| `Asia/Vientiane` | Asia/Vientiane |
| `Asia/Vladivostok` | Asia/Vladivostok |
| `Asia/Yakutsk` | Asia/Yakutsk |
| `Asia/Yangon` | Asia/Yangon |
| `Asia/Yekaterinburg` | Asia/Yekaterinburg |
| `Asia/Yerevan` | Asia/Yerevan |
| `Atlantic/Azores` | Atlantic/Azores |
| `Atlantic/Bermuda` | Atlantic/Bermuda |
| `Atlantic/Canary` | Atlantic/Canary |
| `Atlantic/Cape_Verde` | Atlantic/Cape_Verde |
| `Atlantic/Faeroe` | Atlantic/Faeroe |
| `Atlantic/Faroe` | Atlantic/Faroe |
| `Atlantic/Jan_Mayen` | Atlantic/Jan_Mayen |
| `Atlantic/Madeira` | Atlantic/Madeira |
| `Atlantic/Reykjavik` | Atlantic/Reykjavik |
| `Atlantic/South_Georgia` | Atlantic/South_Georgia |
| `Atlantic/St_Helena` | Atlantic/St_Helena |
| `Atlantic/Stanley` | Atlantic/Stanley |
| `Australia/ACT` | Australia/ACT |
| `Australia/Adelaide` | Australia/Adelaide |
| `Australia/Brisbane` | Australia/Brisbane |
| `Australia/Broken_Hill` | Australia/Broken_Hill |
| `Australia/Canberra` | Australia/Canberra |
| `Australia/Currie` | Australia/Currie |
| `Australia/Darwin` | Australia/Darwin |
| `Australia/Eucla` | Australia/Eucla |
| `Australia/Hobart` | Australia/Hobart |
| `Australia/LHI` | Australia/LHI |
| `Australia/Lindeman` | Australia/Lindeman |
| `Australia/Lord_Howe` | Australia/Lord_Howe |
| `Australia/Melbourne` | Australia/Melbourne |
| `Australia/NSW` | Australia/NSW |
| `Australia/North` | Australia/North |
| `Australia/Perth` | Australia/Perth |
| `Australia/Queensland` | Australia/Queensland |
| `Australia/South` | Australia/South |
| `Australia/Sydney` | Australia/Sydney |
| `Australia/Tasmania` | Australia/Tasmania |
| `Australia/Victoria` | Australia/Victoria |
| `Australia/West` | Australia/West |
| `Australia/Yancowinna` | Australia/Yancowinna |
| `Brazil/Acre` | Brazil/Acre |
| `Brazil/DeNoronha` | Brazil/DeNoronha |
| `Brazil/East` | Brazil/East |
| `Brazil/West` | Brazil/West |
| `CET` | CET |
| `CST6CDT` | CST6CDT |
| `Canada/Atlantic` | Canada/Atlantic |
| `Canada/Central` | Canada/Central |
| `Canada/Eastern` | Canada/Eastern |
| `Canada/Mountain` | Canada/Mountain |
| `Canada/Newfoundland` | Canada/Newfoundland |
| `Canada/Pacific` | Canada/Pacific |
| `Canada/Saskatchewan` | Canada/Saskatchewan |
| `Canada/Yukon` | Canada/Yukon |
| `Chile/Continental` | Chile/Continental |
| `Chile/EasterIsland` | Chile/EasterIsland |
| `Cuba` | Cuba |
| `EET` | EET |
| `EST` | EST |
| `EST5EDT` | EST5EDT |
| `Egypt` | Egypt |
| `Eire` | Eire |
| `Europe/Amsterdam` | Europe/Amsterdam |
| `Europe/Andorra` | Europe/Andorra |
| `Europe/Astrakhan` | Europe/Astrakhan |
| `Europe/Athens` | Europe/Athens |
| `Europe/Belfast` | Europe/Belfast |
| `Europe/Belgrade` | Europe/Belgrade |
| `Europe/Berlin` | Europe/Berlin |
| `Europe/Bratislava` | Europe/Bratislava |
| `Europe/Brussels` | Europe/Brussels |
| `Europe/Bucharest` | Europe/Bucharest |
| `Europe/Budapest` | Europe/Budapest |
| `Europe/Busingen` | Europe/Busingen |
| `Europe/Chisinau` | Europe/Chisinau |
| `Europe/Copenhagen` | Europe/Copenhagen |
| `Europe/Dublin` | Europe/Dublin |
| `Europe/Gibraltar` | Europe/Gibraltar |
| `Europe/Guernsey` | Europe/Guernsey |
| `Europe/Helsinki` | Europe/Helsinki |
| `Europe/Isle_of_Man` | Europe/Isle_of_Man |
| `Europe/Istanbul` | Europe/Istanbul |
| `Europe/Jersey` | Europe/Jersey |
| `Europe/Kaliningrad` | Europe/Kaliningrad |
| `Europe/Kiev` | Europe/Kiev |
| `Europe/Kirov` | Europe/Kirov |
| `Europe/Kyiv` | Europe/Kyiv |
| `Europe/Lisbon` | Europe/Lisbon |
| `Europe/Ljubljana` | Europe/Ljubljana |
| `Europe/London` | Europe/London |
| `Europe/Luxembourg` | Europe/Luxembourg |
| `Europe/Madrid` | Europe/Madrid |
| `Europe/Malta` | Europe/Malta |
| `Europe/Mariehamn` | Europe/Mariehamn |
| `Europe/Minsk` | Europe/Minsk |
| `Europe/Monaco` | Europe/Monaco |
| `Europe/Moscow` | Europe/Moscow |
| `Europe/Nicosia` | Europe/Nicosia |
| `Europe/Oslo` | Europe/Oslo |
| `Europe/Paris` | Europe/Paris |
| `Europe/Podgorica` | Europe/Podgorica |
| `Europe/Prague` | Europe/Prague |
| `Europe/Riga` | Europe/Riga |
| `Europe/Rome` | Europe/Rome |
| `Europe/Samara` | Europe/Samara |
| `Europe/San_Marino` | Europe/San_Marino |
| `Europe/Sarajevo` | Europe/Sarajevo |
| `Europe/Saratov` | Europe/Saratov |
| `Europe/Simferopol` | Europe/Simferopol |
| `Europe/Skopje` | Europe/Skopje |
| `Europe/Sofia` | Europe/Sofia |
| `Europe/Stockholm` | Europe/Stockholm |
| `Europe/Tallinn` | Europe/Tallinn |
| `Europe/Tirane` | Europe/Tirane |
| `Europe/Tiraspol` | Europe/Tiraspol |
| `Europe/Ulyanovsk` | Europe/Ulyanovsk |
| `Europe/Uzhgorod` | Europe/Uzhgorod |
| `Europe/Vaduz` | Europe/Vaduz |
| `Europe/Vatican` | Europe/Vatican |
| `Europe/Vienna` | Europe/Vienna |
| `Europe/Vilnius` | Europe/Vilnius |
| `Europe/Volgograd` | Europe/Volgograd |
| `Europe/Warsaw` | Europe/Warsaw |
| `Europe/Zagreb` | Europe/Zagreb |
| `Europe/Zaporozhye` | Europe/Zaporozhye |
| `Europe/Zurich` | Europe/Zurich |
| `GB` | GB |
| `GB-Eire` | GB-Eire |
| `GMT` | GMT |
| `GMT+0` | GMT+0 |
| `GMT-0` | GMT-0 |
| `GMT0` | GMT0 |
| `Greenwich` | Greenwich |
| `HST` | HST |
| `Hongkong` | Hongkong |
| `Iceland` | Iceland |
| `Indian/Antananarivo` | Indian/Antananarivo |
| `Indian/Chagos` | Indian/Chagos |
| `Indian/Christmas` | Indian/Christmas |
| `Indian/Cocos` | Indian/Cocos |
| `Indian/Comoro` | Indian/Comoro |
| `Indian/Kerguelen` | Indian/Kerguelen |
| `Indian/Mahe` | Indian/Mahe |
| `Indian/Maldives` | Indian/Maldives |
| `Indian/Mauritius` | Indian/Mauritius |
| `Indian/Mayotte` | Indian/Mayotte |
| `Indian/Reunion` | Indian/Reunion |
| `Iran` | Iran |
| `Israel` | Israel |
| `Jamaica` | Jamaica |
| `Japan` | Japan |
| `Kwajalein` | Kwajalein |
| `Libya` | Libya |
| `MET` | MET |
| `MST` | MST |
| `MST7MDT` | MST7MDT |
| `Mexico/BajaNorte` | Mexico/BajaNorte |
| `Mexico/BajaSur` | Mexico/BajaSur |
| `Mexico/General` | Mexico/General |
| `NZ` | NZ |
| `NZ-CHAT` | NZ-CHAT |
| `Navajo` | Navajo |
| `PRC` | PRC |
| `PST8PDT` | PST8PDT |
| `Pacific/Apia` | Pacific/Apia |
| `Pacific/Auckland` | Pacific/Auckland |
| `Pacific/Bougainville` | Pacific/Bougainville |
| `Pacific/Chatham` | Pacific/Chatham |
| `Pacific/Chuuk` | Pacific/Chuuk |
| `Pacific/Easter` | Pacific/Easter |
| `Pacific/Efate` | Pacific/Efate |
| `Pacific/Enderbury` | Pacific/Enderbury |
| `Pacific/Fakaofo` | Pacific/Fakaofo |
| `Pacific/Fiji` | Pacific/Fiji |
| `Pacific/Funafuti` | Pacific/Funafuti |
| `Pacific/Galapagos` | Pacific/Galapagos |
| `Pacific/Gambier` | Pacific/Gambier |
| `Pacific/Guadalcanal` | Pacific/Guadalcanal |
| `Pacific/Guam` | Pacific/Guam |
| `Pacific/Honolulu` | Pacific/Honolulu |
| `Pacific/Johnston` | Pacific/Johnston |
| `Pacific/Kanton` | Pacific/Kanton |
| `Pacific/Kiritimati` | Pacific/Kiritimati |
| `Pacific/Kosrae` | Pacific/Kosrae |
| `Pacific/Kwajalein` | Pacific/Kwajalein |
| `Pacific/Majuro` | Pacific/Majuro |
| `Pacific/Marquesas` | Pacific/Marquesas |
| `Pacific/Midway` | Pacific/Midway |
| `Pacific/Nauru` | Pacific/Nauru |
| `Pacific/Niue` | Pacific/Niue |
| `Pacific/Norfolk` | Pacific/Norfolk |
| `Pacific/Noumea` | Pacific/Noumea |
| `Pacific/Pago_Pago` | Pacific/Pago_Pago |
| `Pacific/Palau` | Pacific/Palau |
| `Pacific/Pitcairn` | Pacific/Pitcairn |
| `Pacific/Pohnpei` | Pacific/Pohnpei |
| `Pacific/Ponape` | Pacific/Ponape |
| `Pacific/Port_Moresby` | Pacific/Port_Moresby |
| `Pacific/Rarotonga` | Pacific/Rarotonga |
| `Pacific/Saipan` | Pacific/Saipan |
| `Pacific/Samoa` | Pacific/Samoa |
| `Pacific/Tahiti` | Pacific/Tahiti |
| `Pacific/Tarawa` | Pacific/Tarawa |
| `Pacific/Tongatapu` | Pacific/Tongatapu |
| `Pacific/Truk` | Pacific/Truk |
| `Pacific/Wake` | Pacific/Wake |
| `Pacific/Wallis` | Pacific/Wallis |
| `Pacific/Yap` | Pacific/Yap |
| `Poland` | Poland |
| `Portugal` | Portugal |
| `ROC` | ROC |
| `ROK` | ROK |
| `Singapore` | Singapore |
| `Turkey` | Turkey |
| `UCT` | UCT |
| `US/Alaska` | US/Alaska |
| `US/Aleutian` | US/Aleutian |
| `US/Arizona` | US/Arizona |
| `US/Central` | US/Central |
| `US/East-Indiana` | US/East-Indiana |
| `US/Eastern` | US/Eastern |
| `US/Hawaii` | US/Hawaii |
| `US/Indiana-Starke` | US/Indiana-Starke |
| `US/Michigan` | US/Michigan |
| `US/Mountain` | US/Mountain |
| `US/Pacific` | US/Pacific |
| `US/Samoa` | US/Samoa |
| `UTC` | UTC |
| `Universal` | Universal |
| `W-SU` | W-SU |
| `WET` | WET |
| `Zulu` | Zulu |
| `Etc/GMT` | Etc/GMT |
| `Etc/GMT+0` | Etc/GMT+0 |
| `Etc/GMT+1` | Etc/GMT+1 |
| `Etc/GMT+10` | Etc/GMT+10 |
| `Etc/GMT+11` | Etc/GMT+11 |
| `Etc/GMT+12` | Etc/GMT+12 |
| `Etc/GMT+2` | Etc/GMT+2 |
| `Etc/GMT+3` | Etc/GMT+3 |
| `Etc/GMT+4` | Etc/GMT+4 |
| `Etc/GMT+5` | Etc/GMT+5 |
| `Etc/GMT+6` | Etc/GMT+6 |
| `Etc/GMT+7` | Etc/GMT+7 |
| `Etc/GMT+8` | Etc/GMT+8 |
| `Etc/GMT+9` | Etc/GMT+9 |
| `Etc/GMT-0` | Etc/GMT-0 |
| `Etc/GMT-1` | Etc/GMT-1 |
| `Etc/GMT-10` | Etc/GMT-10 |
| `Etc/GMT-11` | Etc/GMT-11 |
| `Etc/GMT-12` | Etc/GMT-12 |
| `Etc/GMT-13` | Etc/GMT-13 |
| `Etc/GMT-14` | Etc/GMT-14 |
| `Etc/GMT-2` | Etc/GMT-2 |
| `Etc/GMT-3` | Etc/GMT-3 |
| `Etc/GMT-4` | Etc/GMT-4 |
| `Etc/GMT-5` | Etc/GMT-5 |
| `Etc/GMT-6` | Etc/GMT-6 |
| `Etc/GMT-7` | Etc/GMT-7 |
| `Etc/GMT-8` | Etc/GMT-8 |
| `Etc/GMT-9` | Etc/GMT-9 |
| `Etc/GMT0` | Etc/GMT0 |
| `Etc/Greenwich` | Etc/Greenwich |
| `Etc/UCT` | Etc/UCT |
| `Etc/UTC` | Etc/UTC |
| `Etc/Universal` | Etc/Universal |
| `Etc/Zulu` | Etc/Zulu |

## `res.partner.bank`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`l10n_us_bank_account_type`** — Bank Account Type

| Value | Label |
|---|---|
| `checking` | Checking |
| `savings` | Savings |

**`proxy_type`** — Proxy Type

| Value | Label |
|---|---|
| `none` | None |
| `id` | FPS ID |
| `ewallet_id` | Ewallet ID |
| `merchant_tax_id` | Merchant Tax ID |
| `email` | Email Address |
| `mobile` | Mobile Number |
| `bakong_id_solo` | Bakong Account ID (Solo Merchant) |
| `bakong_id_merchant` | Bakong Account ID (Corporate Merchant) |
| `uen` | UEN |
| `merchant_id` | Merchant ID |
| `payment_service` | Payment Service |
| `atm_card` | ATM Card Number |
| `bank_acc` | Bank Account |
| `br_cpf_cnpj` | CPF/CNPJ (BR) |
| `br_random` | Random Key (BR) |

## `res.users`

**`calendar_default_privacy`** — Calendar Default Privacy (not stored)

| Value | Label |
|---|---|
| `public` | Public by default |
| `private` | Private by default |
| `confidential` | Internal users only |

**`manual_im_status`** — IM status manually set by the user

| Value | Label |
|---|---|
| `away` | Away |
| `busy` | Do Not Disturb |
| `offline` | Offline |

**`notification_type`** — Notification

| Value | Label |
|---|---|
| `email` | By Emails |
| `inbox` | In the system |

**`system_robot_state`** — System Robot Status

| Value | Label |
|---|---|
| `not_initialized` | Not initialized |
| `onboarding_emoji` | Onboarding emoji |
| `onboarding_attachement` | Onboarding attachment |
| `onboarding_command` | Onboarding command |
| `onboarding_ping` | Onboarding ping |
| `onboarding_canned` | Onboarding canned |
| `idle` | Idle |
| `disabled` | Disabled |

**`outgoing_mail_server_type`** — Outgoing Mail Server Type (not stored)

| Value | Label |
|---|---|
| `default` | Default |
| `gmail` | Gmail |
| `outlook` | Outlook |

**`role`** — Role (not stored)

| Value | Label |
|---|---|
| `group_user` | User |
| `group_system` | Administrator |

**`state`** — Status (not stored)

| Value | Label |
|---|---|
| `new` | Invited |
| `active` | Confirmed |

## `res.users.deletion`

**`state`** — State

| Value | Label |
|---|---|
| `todo` | To Do |
| `done` | Done |
| `fail` | Failed |

## `res.users.identitycheck`

**`auth_method`** — Auth Method

| Value | Label |
|---|---|
| `password` | Password |
| `webauthn` | Passkey |

## `res.users.settings`

**`calendar_default_privacy`** — Calendar Default Privacy

| Value | Label |
|---|---|
| `public` | Public |
| `private` | Private |
| `confidential` | Only internal users |

**`channel_notifications`** — Channel Notifications

| Value | Label |
|---|---|
| `all` | All Messages |
| `no_notif` | Nothing |

## `reset.view.arch.wizard`

**`reset_mode`** — Reset Mode

| Value | Label |
|---|---|
| `soft` | Restore previous version (soft reset). |
| `hard` | Reset to file version (hard reset). |
| `other_view` | Reset to another view. |

## `resource.calendar`

**`schedule_type`** — Schedule Type

| Value | Label |
|---|---|
| `flexible` | Flexible |
| `fully_fixed` | Fully Fixed |

## `resource.calendar.attendance`

**`day_period`** — Day Period

| Value | Label |
|---|---|
| `morning` | Morning |
| `lunch` | Break |
| `afternoon` | Afternoon |
| `full_day` | Full Day |

**`dayofweek`** — Day of Week

| Value | Label |
|---|---|
| `0` | Monday |
| `1` | Tuesday |
| `2` | Wednesday |
| `3` | Thursday |
| `4` | Friday |
| `5` | Saturday |
| `6` | Sunday |

**`display_type`** — Display Type

| Value | Label |
|---|---|
| `line_section` | Section |

**`week_type`** — Week Number

| Value | Label |
|---|---|
| `1` | Second |
| `0` | First |

## `resource.calendar.leaves`

**`time_type`** — Time Type

| Value | Label |
|---|---|
| `leave` | Time Off |
| `other` | Other |

## `resource.resource`

**`resource_type`** — Type

| Value | Label |
|---|---|
| `user` | Human |
| `material` | Material |

## `restaurant.table`

**`shape`** — Shape

| Value | Label |
|---|---|
| `square` | Square |
| `round` | Round |

## `sale.advance.payment.inv`

**`advance_payment_method`** — Create Invoice

| Value | Label |
|---|---|
| `delivered` | Regular invoice |
| `percentage` | Down payment (percentage) |
| `fixed` | Down payment (fixed amount) |

## `sale.order`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`delivery_status`** — Delivery Status

| Value | Label |
|---|---|
| `pending` | Not Delivered |
| `started` | Started |
| `partial` | Partially Delivered |
| `full` | Fully Delivered |

**`invoice_status`** — Invoice Status

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

**`l10n_it_origin_document_type`** — Origin Document Type

| Value | Label |
|---|---|
| `purchase_order` | Purchase Order |
| `contract` | Contract |
| `agreement` | Agreement |

**`l10n_tw_edi_carrier_type`** — Carrier Type

| Value | Label |
|---|---|
| `1` | Member Account |
| `2` | Citizen Digital Certificate |
| `3` | Mobile Barcode |
| `4` | EasyCard |
| `5` | iPass |

**`picking_policy`** — Shipping Policy

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Quotation |
| `sent` | Quotation Sent |
| `sale` | Sales Order |
| `cancel` | Cancelled |

## `sale.order.discount`

**`discount_type`** — Discount Type

| Value | Label |
|---|---|
| `sol_discount` | On All Order Lines |
| `so_discount` | Global Discount |
| `amount` | Fixed Amount |

## `sale.order.line`

**`display_type`** — Display Type

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |

**`invoice_status`** — Invoice Status

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

**`qty_delivered_method`** — Method to update delivered qty

| Value | Label |
|---|---|
| `manual` | Manual |
| `analytic` | Analytic From Expenses |
| `stock_move` | Stock Moves |
| `milestones` | Milestones |
| `timesheet` | Timesheets |

## `sale.order.template.line`

**`display_type`** — Display Type

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |

## `sale.pdf.form.field`

**`document_type`** — Document Type

| Value | Label |
|---|---|
| `quotation_document` | Header/Footer |
| `product_document` | Product Document |

## `sale.report`

**`invoice_status`** — Order Invoice Status

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

**`line_invoice_status`** — Invoice Status

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Quotation |
| `sent` | Quotation Sent |
| `sale` | Sales Order |
| `cancel` | Cancelled |
| `paid` | Paid |
| `invoiced` | Invoiced |
| `done` | Posted |

## `slide.channel`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`channel_type`** — Course type

| Value | Label |
|---|---|
| `training` | Training |
| `documentation` | Documentation |

**`enroll`** — Enroll Policy

| Value | Label |
|---|---|
| `public` | Open |
| `invite` | On Invitation |
| `payment` | On payment |

**`promote_strategy`** — Featured Content

| Value | Label |
|---|---|
| `latest` | Latest Created |
| `most_voted` | Most Voted |
| `most_viewed` | Most Viewed |
| `specific` | Select Manually |
| `none` | None |

**`rating_avg_text`** — Rating Avg Text (not stored)

| Value | Label |
|---|---|
| `top` | Happy |
| `ok` | Neutral |
| `ko` | Unhappy |
| `none` | Not Rated yet |

**`visibility`** — Show Course To

| Value | Label |
|---|---|
| `public` | Everyone |
| `connected` | Signed In |
| `members` | Course Attendees |
| `link` | Anyone with the link |

## `slide.channel.partner`

**`member_status`** — Attendee Status

| Value | Label |
|---|---|
| `invited` | Invite Sent |
| `joined` | Joined |
| `ongoing` | Ongoing |
| `completed` | Finished |

## `slide.slide`

**`slide_category`** — Category

| Value | Label |
|---|---|
| `infographic` | Image |
| `article` | Article |
| `document` | Document |
| `video` | Video |
| `quiz` | Quiz |
| `certification` | Certification |

**`slide_type`** — Slide Type

| Value | Label |
|---|---|
| `image` | Image |
| `article` | Article |
| `quiz` | Quiz |
| `pdf` | PDF |
| `sheet` | Sheet (Excel, Google Sheet, ...) |
| `doc` | Document (Word, Google Doc, ...) |
| `slides` | Slides (PowerPoint, Google Slides, ...) |
| `youtube_video` | YouTube Video |
| `google_drive_video` | Google Drive Video |
| `vimeo_video` | Vimeo Video |
| `certification` | Certification |

**`source_type`** — Source Type

| Value | Label |
|---|---|
| `local_file` | Upload from Device |
| `external` | Retrieve from Google Drive |

**`video_source_type`** — Video Source (not stored)

| Value | Label |
|---|---|
| `youtube` | YouTube |
| `google_drive` | Google Drive |
| `vimeo` | Vimeo |

## `slide.slide.resource`

**`resource_type`** — Resource Type

| Value | Label |
|---|---|
| `file` | File |
| `url` | Link |

## `sms.composer`

**`composition_mode`** — Composition Mode

| Value | Label |
|---|---|
| `numbers` | Send to numbers |
| `comment` | Post on a document |
| `mass` | Send SMS in batch |

## `sms.sms`

**`failure_type`** — Failure Type

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `sms_number_missing` | Missing Number |
| `sms_number_format` | Wrong Number Format |
| `sms_country_not_supported` | Country Not Supported |
| `sms_registration_needed` | Country-specific Registration Required |
| `sms_credit` | Insufficient Credit |
| `sms_server` | Server Error |
| `sms_acc` | Unregistered Account |
| `sms_blacklist` | Blacklisted |
| `sms_duplicate` | Duplicate |
| `sms_optout` | Opted Out |
| `twilio_authentication` | Authentication Error" |
| `twilio_callback` | Incorrect callback URL |
| `twilio_from_missing` | Missing From Number |
| `twilio_from_to` | From / To identic |

**`state`** — SMS Status

| Value | Label |
|---|---|
| `outgoing` | In Queue |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `error` | Error |
| `canceled` | Cancelled |

## `snailmail.letter`

**`error_code`** — Error

| Value | Label |
|---|---|
| `MISSING_REQUIRED_FIELDS` | MISSING_REQUIRED_FIELDS |
| `CREDIT_ERROR` | CREDIT_ERROR |
| `TRIAL_ERROR` | TRIAL_ERROR |
| `NO_PRICE_AVAILABLE` | NO_PRICE_AVAILABLE |
| `FORMAT_ERROR` | FORMAT_ERROR |
| `UNKNOWN_ERROR` | UNKNOWN_ERROR |
| `ATTACHMENT_ERROR` | ATTACHMENT_ERROR |

**`state`** — Status

| Value | Label |
|---|---|
| `pending` | In Queue |
| `sent` | Sent |
| `error` | Error |
| `canceled` | Cancelled |

## `sparse_fields.test`

**`selection`** — Selection (not stored)

| Value | Label |
|---|---|
| `one` | One |
| `two` | Two |

## `stock.add.to.wave`

**`mode`** — Mode

| Value | Label |
|---|---|
| `existing` | an existing wave transfer |
| `new` | a new wave transfer |

## `stock.avco.report`

**`res_model_name`** — Resource Model Name

| Value | Label |
|---|---|
| `stock.move` | Stock Move |
| `product.value` | Product Value |

## `stock.landed.cost`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`state`** — State

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Posted |
| `cancel` | Cancelled |

**`target_model`** — Apply On

| Value | Label |
|---|---|
| `picking` | Transfers |
| `manufacturing` | Manufacturing Orders |

## `stock.landed.cost.lines`

**`split_method`** — Split Method

| Value | Label |
|---|---|
| `equal` | Equal |
| `by_quantity` | By Quantity |
| `by_current_cost_price` | By Current Cost |
| `by_weight` | By Weight |
| `by_volume` | By Volume |

## `stock.location`

**`usage`** — Location Type

| Value | Label |
|---|---|
| `supplier` | Vendor |
| `view` | Virtual |
| `internal` | Internal |
| `customer` | Customer |
| `inventory` | Inventory Loss |
| `production` | Production |
| `transit` | Transit |

## `stock.lot`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

## `stock.move`

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

**`procure_method`** — Supply Method

| Value | Label |
|---|---|
| `make_to_stock` | Default: Take From Stock |
| `make_to_order` | Advanced: Apply Procurement Rules |

**`repair_line_type`** — Type

| Value | Label |
|---|---|
| `add` | Add |
| `remove` | Remove |
| `recycle` | Recycle |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | New |
| `waiting` | Waiting Another Move |
| `confirmed` | Waiting |
| `partially_available` | Partially Available |
| `assigned` | Available |
| `done` | Done |
| `cancel` | Cancelled |

## `stock.orderpoint.snooze`

**`predefined_date`** — Snooze for

| Value | Label |
|---|---|
| `day` | 1 Day |
| `week` | 1 Week |
| `month` | 1 Month |
| `custom` | Custom |

## `stock.package.type`

**`package_carrier_type`** — Carrier

| Value | Label |
|---|---|
| `none` | No carrier integration |

**`package_use`** — Package Use

| Value | Label |
|---|---|
| `disposable` | Disposable Box |
| `reusable` | Reusable Box (totes) |

## `stock.picking`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`l10n_it_transport_method`** — Transport Method

| Value | Label |
|---|---|
| `sender` | Sender |
| `recipient` | Recipient |
| `courier` | Courier service |

**`l10n_it_transport_reason`** — Transport Reason

| Value | Label |
|---|---|
| `sale` | Sale |
| `outsourcing` | Outsourcing |
| `evaluation` | Evaluation |
| `gift` | Gift |
| `transfer` | Transfer |
| `substitution` | Returned goods |
| `attemped_sale` | Attempted Sale |
| `loaned_use` | Loaned for Use |
| `repair` | Repair |

**`l10n_ro_edi_stock_end_bcp`** — End Border Crossing Point

| Value | Label |
|---|---|
| `1` | Petea (HU) |
| `2` | Borș(HU) |
| `3` | Vărșand(HU) |
| `4` | Nădlac(HU) |
| `5` | Calafat (BG) |
| `6` | Bechet(BG) |
| `7` | Turnu Măgurele(BG) |
| `8` | Zimnicea(BG) |
| `9` | Giurgiu(BG) |
| `10` | Ostrov(BG) |
| `11` | Negru Vodă(BG) |
| `12` | Vama Veche(BG) |
| `13` | Călărași(BG) |
| `14` | Corabia(BG) |
| `15` | Oltenița(BG) |
| `16` | Carei  (HU) |
| `17` | Cenad (HU) |
| `18` | Episcopia Bihor (HU) |
| `19` | Salonta (HU) |
| `20` | Săcuieni (HU) |
| `21` | Turnu (HU) |
| `22` | Urziceni (HU) |
| `23` | Valea lui Mihai (HU) |
| `24` | Vladimirescu (HU) |
| `25` | Porțile de Fier 1 (RS) |
| `26` | Naidăș(RS) |
| `27` | Stamora Moravița(RS) |
| `28` | Jimbolia(RS) |
| `29` | Halmeu (UA) |
| `30` | Stânca Costești (MD) |
| `31` | Sculeni(MD) |
| `32` | Albița(MD) |
| `33` | Oancea(MD) |
| `34` | Galați Giurgiulești(MD) |
| `35` | Constanța Sud Agigea |
| `36` | Siret  (UA) |
| `37` | Nădlac 2 - A1 (HU) |
| `38` | Borș 2 - A3 (HU) |

**`l10n_ro_edi_stock_end_customs_office`** — End Customs Office

| Value | Label |
|---|---|
| `12801` | BVI Alba Iulia (ROBV0300) |
| `22801` | BVI Arad (ROTM0200) |
| `22901` | BVF Arad Aeroport (ROTM0230) |
| `22902` | BVF Zona Liberă Curtici (ROTM2300) |
| `32801` | BVI Pitești (ROCR7000) |
| `42801` | BVI Bacău (ROIS0600) |
| `42901` | BVF Bacău Aeroport (ROIS0620) |
| `52801` | BVI Oradea (ROCJ6570) |
| `52901` | BVF Oradea Aeroport (ROCJ6580) |
| `62801` | BVI Bistriţa-Năsăud (ROCJ0400) |
| `72801` | BVI Botoşani (ROIS1600) |
| `72901` | BVF Stanca Costeşti (ROIS1610) |
| `72902` | BVF Rădăuţi Prut (ROIS1620) |
| `82801` | BVI Braşov (ROBV0900) |
| `92901` | BVF Zona Liberă Brăila (ROGL0710) |
| `92902` | BVF Brăila (ROGL0700) |
| `102801` | BVI Buzău (ROGL1500) |
| `112801` | BVI Reșița (ROTM7600) |
| `112901` | BVF Naidăș (ROTM6100) |
| `122801` | BVI Cluj Napoca (ROCJ1800) |
| `122901` | BVF Cluj Napoca Aero (ROCJ1810) |
| `132901` | BVF Constanţa Sud Agigea (ROCT1900) |
| `132902` | BVF Mihail Kogălniceanu (ROCT5100) |
| `132903` | BVF Mangalia (ROCT5400) |
| `132904` | BVF Constanţa Port (ROCT1970) |
| `142801` | BVI Sfântu Gheorghe (ROBV7820) |
| `152801` | BVI Târgoviște (ROBU8600) |
| `162801` | BVI Craiova (ROCR2100) |
| `162901` | BVF Craiova Aeroport (ROCR2110) |
| `162902` | BVF Bechet (ROCR1720) |
| `162903` | BVF Calafat (ROCR1700) |
| `172901` | BVF Zona Liberă Galaţi (ROGL3810) |
| `172902` | BVF Giurgiuleşti (ROGL3850) |
| `172903` | BVF Oancea (ROGL3610) |
| `172904` | BVF Galaţi (ROGL3800) |
| `182801` | BVI Târgu Jiu (ROCR8810) |
| `192801` | BVI Miercurea Ciuc (ROBV5600) |
| `202801` | BVI Deva (ROTM8100) |
| `212801` | BVI Slobozia (ROCT8220) |
| `222901` | BVF Iaşi Aero (ROIS4660) |
| `222902` | BVF Sculeni (ROIS4990) |
| `222903` | BVF Iaşi (ROIS4650) |
| `232801` | BVI Antrepozite/Ilfov (ROBU1200) |
| `232901` | BVF Otopeni Călători (ROBU1030) |
| `242801` | BVI Baia Mare (ROCJ0500) |
| `242901` | BVF Aero Baia Mare (ROCJ0510) |
| `242902` | BVF Sighet (ROCJ8000) |
| `252901` | BVF Orşova (ROCR7280) |
| `252902` | BVF Porţile De Fier I (ROCR7270) |
| `252903` | BVF Porţile De Fier II (ROCR7200) |
| `252904` | BVF Drobeta Turnu Severin (ROCR9000) |
| `262801` | BVI Târgu Mureş (ROBV8800) |
| `262901` | BVF Târgu Mureş Aeroport (ROBV8820) |
| `272801` | BVI Piatra Neamţ (ROIS7400) |
| `282801` | BVI Corabia (ROCR2000) |
| `282802` | BVI Olt (ROCR8210) |
| `292801` | BVI Ploiești (ROBU7100) |
| `302801` | BVI Satu-Mare (ROCJ7810) |
| `302901` | BVF Halmeu (ROCJ4310) |
| `302902` | BVF Aeroport Satu Mare (ROCJ7830) |
| `312801` | BVI Zalău (ROCJ9700) |
| `322801` | BVI Sibiu (ROBV7900) |
| `322901` | BVF Sibiu Aeroport (ROBV7910) |
| `332801` | BVI Suceava (ROIS8230) |
| `332901` | BVF Dorneşti (ROIS2700) |
| `332902` | BVF Siret (ROIS8200) |
| `332903` | BVF Suceava Aero (ROIS8250) |
| `332904` | BVF Vicovu De Sus (ROIS9620) |
| `342801` | BVI Alexandria (ROCR0310) |
| `342901` | BVF Turnu Măgurele (ROCR9100) |
| `342902` | BVF Zimnicea (ROCR5800) |
| `352802` | BVI Timişoara Bază (ROTM8720) |
| `352901` | BVF Jimbolia (ROTM5010) |
| `352902` | BVF Moraviţa (ROTM5510) |
| `352903` | BVF Timişoara Aeroport (ROTM8730) |
| `362901` | BVF Sulina (ROCT8300) |
| `362902` | BVF Aeroport Delta Dunării Tulcea (ROGL8910) |
| `362903` | BVF Tulcea (ROGL8900) |
| `362904` | BVF Isaccea (ROGL8920) |
| `372801` | BVI Vaslui (ROIS9610) |
| `372901` | BVF Fălciu (-) |
| `372902` | BVF Albiţa (ROIS0100) |
| `382801` | BVI Râmnicu Vâlcea (ROCR7700) |
| `392801` | BVI Focșani (ROGL3600) |
| `402801` | BVI Bucureşti Poştă (ROBU1380) |
| `402802` | BVI Târguri și Expoziții (ROBU1400) |
| `402901` | BVF Băneasa (ROBU1040) |
| `512801` | BVI Călăraşi (ROCT1710) |
| `522801` | BVI Giurgiu (ROBU3910) |
| `522901` | BVF Zona Liberă Giurgiu (ROBU3980) |

**`l10n_ro_edi_stock_end_loc_type`** — End Location Type

| Value | Label |
|---|---|
| `location` | Location |
| `bcp` | Border Crossing Point |
| `customs` | Customs Office |

**`l10n_ro_edi_stock_operation_scope`** — Operation Scope

| Value | Label |
|---|---|
| `101` | Marketing |
| `201` | Output |
| `301` | Gratuities |
| `401` | Commercial equipment |
| `501` | Fixed assets |
| `601` | Own consumption |
| `703` | Delivery operations with installation |
| `704` | Transfer between managements |
| `705` | Goods made available to the customer |
| `801` | Financial/operational leasing |
| `802` | Goods under warranty |
| `901` | Exempt operations |
| `1001` | Investment in progress |
| `1101` | Donations, help |
| `9901` | Other |
| `9999` | Same with operation |

**`l10n_ro_edi_stock_operation_type`** — eTransport Operation Type

| Value | Label |
|---|---|
| `10` | Intra-community purchase |
| `12` | Operations in lohn system (EU) - input |
| `14` | Stocks available to the customer (Call-off stock) - entry |
| `20` | Intra-Community delivery |
| `22` | Operations in lohn system (EU) - exit |
| `24` | Stocks available to the customer (Call-off stock) - exit |
| `30` | Transport on the national territory |
| `40` | Import |
| `50` | Export |
| `60` | Intra-community transaction - Entry for storage/formation of new transport |
| `70` | Intra-community transaction - Exit after storage/formation of new transport |

**`l10n_ro_edi_stock_start_bcp`** — Start Border Crossing Point

| Value | Label |
|---|---|
| `1` | Petea (HU) |
| `2` | Borș(HU) |
| `3` | Vărșand(HU) |
| `4` | Nădlac(HU) |
| `5` | Calafat (BG) |
| `6` | Bechet(BG) |
| `7` | Turnu Măgurele(BG) |
| `8` | Zimnicea(BG) |
| `9` | Giurgiu(BG) |
| `10` | Ostrov(BG) |
| `11` | Negru Vodă(BG) |
| `12` | Vama Veche(BG) |
| `13` | Călărași(BG) |
| `14` | Corabia(BG) |
| `15` | Oltenița(BG) |
| `16` | Carei  (HU) |
| `17` | Cenad (HU) |
| `18` | Episcopia Bihor (HU) |
| `19` | Salonta (HU) |
| `20` | Săcuieni (HU) |
| `21` | Turnu (HU) |
| `22` | Urziceni (HU) |
| `23` | Valea lui Mihai (HU) |
| `24` | Vladimirescu (HU) |
| `25` | Porțile de Fier 1 (RS) |
| `26` | Naidăș(RS) |
| `27` | Stamora Moravița(RS) |
| `28` | Jimbolia(RS) |
| `29` | Halmeu (UA) |
| `30` | Stânca Costești (MD) |
| `31` | Sculeni(MD) |
| `32` | Albița(MD) |
| `33` | Oancea(MD) |
| `34` | Galați Giurgiulești(MD) |
| `35` | Constanța Sud Agigea |
| `36` | Siret  (UA) |
| `37` | Nădlac 2 - A1 (HU) |
| `38` | Borș 2 - A3 (HU) |

**`l10n_ro_edi_stock_start_customs_office`** — Start Customs Office

| Value | Label |
|---|---|
| `12801` | BVI Alba Iulia (ROBV0300) |
| `22801` | BVI Arad (ROTM0200) |
| `22901` | BVF Arad Aeroport (ROTM0230) |
| `22902` | BVF Zona Liberă Curtici (ROTM2300) |
| `32801` | BVI Pitești (ROCR7000) |
| `42801` | BVI Bacău (ROIS0600) |
| `42901` | BVF Bacău Aeroport (ROIS0620) |
| `52801` | BVI Oradea (ROCJ6570) |
| `52901` | BVF Oradea Aeroport (ROCJ6580) |
| `62801` | BVI Bistriţa-Năsăud (ROCJ0400) |
| `72801` | BVI Botoşani (ROIS1600) |
| `72901` | BVF Stanca Costeşti (ROIS1610) |
| `72902` | BVF Rădăuţi Prut (ROIS1620) |
| `82801` | BVI Braşov (ROBV0900) |
| `92901` | BVF Zona Liberă Brăila (ROGL0710) |
| `92902` | BVF Brăila (ROGL0700) |
| `102801` | BVI Buzău (ROGL1500) |
| `112801` | BVI Reșița (ROTM7600) |
| `112901` | BVF Naidăș (ROTM6100) |
| `122801` | BVI Cluj Napoca (ROCJ1800) |
| `122901` | BVF Cluj Napoca Aero (ROCJ1810) |
| `132901` | BVF Constanţa Sud Agigea (ROCT1900) |
| `132902` | BVF Mihail Kogălniceanu (ROCT5100) |
| `132903` | BVF Mangalia (ROCT5400) |
| `132904` | BVF Constanţa Port (ROCT1970) |
| `142801` | BVI Sfântu Gheorghe (ROBV7820) |
| `152801` | BVI Târgoviște (ROBU8600) |
| `162801` | BVI Craiova (ROCR2100) |
| `162901` | BVF Craiova Aeroport (ROCR2110) |
| `162902` | BVF Bechet (ROCR1720) |
| `162903` | BVF Calafat (ROCR1700) |
| `172901` | BVF Zona Liberă Galaţi (ROGL3810) |
| `172902` | BVF Giurgiuleşti (ROGL3850) |
| `172903` | BVF Oancea (ROGL3610) |
| `172904` | BVF Galaţi (ROGL3800) |
| `182801` | BVI Târgu Jiu (ROCR8810) |
| `192801` | BVI Miercurea Ciuc (ROBV5600) |
| `202801` | BVI Deva (ROTM8100) |
| `212801` | BVI Slobozia (ROCT8220) |
| `222901` | BVF Iaşi Aero (ROIS4660) |
| `222902` | BVF Sculeni (ROIS4990) |
| `222903` | BVF Iaşi (ROIS4650) |
| `232801` | BVI Antrepozite/Ilfov (ROBU1200) |
| `232901` | BVF Otopeni Călători (ROBU1030) |
| `242801` | BVI Baia Mare (ROCJ0500) |
| `242901` | BVF Aero Baia Mare (ROCJ0510) |
| `242902` | BVF Sighet (ROCJ8000) |
| `252901` | BVF Orşova (ROCR7280) |
| `252902` | BVF Porţile De Fier I (ROCR7270) |
| `252903` | BVF Porţile De Fier II (ROCR7200) |
| `252904` | BVF Drobeta Turnu Severin (ROCR9000) |
| `262801` | BVI Târgu Mureş (ROBV8800) |
| `262901` | BVF Târgu Mureş Aeroport (ROBV8820) |
| `272801` | BVI Piatra Neamţ (ROIS7400) |
| `282801` | BVI Corabia (ROCR2000) |
| `282802` | BVI Olt (ROCR8210) |
| `292801` | BVI Ploiești (ROBU7100) |
| `302801` | BVI Satu-Mare (ROCJ7810) |
| `302901` | BVF Halmeu (ROCJ4310) |
| `302902` | BVF Aeroport Satu Mare (ROCJ7830) |
| `312801` | BVI Zalău (ROCJ9700) |
| `322801` | BVI Sibiu (ROBV7900) |
| `322901` | BVF Sibiu Aeroport (ROBV7910) |
| `332801` | BVI Suceava (ROIS8230) |
| `332901` | BVF Dorneşti (ROIS2700) |
| `332902` | BVF Siret (ROIS8200) |
| `332903` | BVF Suceava Aero (ROIS8250) |
| `332904` | BVF Vicovu De Sus (ROIS9620) |
| `342801` | BVI Alexandria (ROCR0310) |
| `342901` | BVF Turnu Măgurele (ROCR9100) |
| `342902` | BVF Zimnicea (ROCR5800) |
| `352802` | BVI Timişoara Bază (ROTM8720) |
| `352901` | BVF Jimbolia (ROTM5010) |
| `352902` | BVF Moraviţa (ROTM5510) |
| `352903` | BVF Timişoara Aeroport (ROTM8730) |
| `362901` | BVF Sulina (ROCT8300) |
| `362902` | BVF Aeroport Delta Dunării Tulcea (ROGL8910) |
| `362903` | BVF Tulcea (ROGL8900) |
| `362904` | BVF Isaccea (ROGL8920) |
| `372801` | BVI Vaslui (ROIS9610) |
| `372901` | BVF Fălciu (-) |
| `372902` | BVF Albiţa (ROIS0100) |
| `382801` | BVI Râmnicu Vâlcea (ROCR7700) |
| `392801` | BVI Focșani (ROGL3600) |
| `402801` | BVI Bucureşti Poştă (ROBU1380) |
| `402802` | BVI Târguri și Expoziții (ROBU1400) |
| `402901` | BVF Băneasa (ROBU1040) |
| `512801` | BVI Călăraşi (ROCT1710) |
| `522801` | BVI Giurgiu (ROBU3910) |
| `522901` | BVF Zona Liberă Giurgiu (ROBU3980) |

**`l10n_ro_edi_stock_start_loc_type`** — Start Location Type

| Value | Label |
|---|---|
| `location` | Location |
| `bcp` | Border Crossing Point |
| `customs` | Customs Office |

**`l10n_ro_edi_stock_state`** — eTransport Status

| Value | Label |
|---|---|
| `stock_sent` | Sent |
| `stock_sending_failed` | Error |
| `stock_validated` | Validated |

**`l10n_tr_nilvera_dispatch_state`** — e-Dispatch State

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |

**`l10n_tr_nilvera_dispatch_type`** — Dispatch Type

| Value | Label |
|---|---|
| `SEVK` | Online |
| `MATBUDAN` | Pre-printed |

**`move_type`** — Shipping Policy

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

**`priority`** — Priority

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

**`products_availability_state`** — Products Availability State (not stored)

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |

**`search_date_category`** — Date Category (not stored)

| Value | Label |
|---|---|
| `before` | Before |
| `yesterday` | Yesterday |
| `today` | Today |
| `day_1` | Tomorrow |
| `day_2` | The day after tomorrow |
| `after` | After |

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `waiting` | Waiting Another Operation |
| `confirmed` | Waiting |
| `assigned` | Ready |
| `done` | Done |
| `cancel` | Cancelled |

## `stock.picking.batch`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`l10n_ro_edi_stock_end_bcp`** — End Border Crossing Point

| Value | Label |
|---|---|
| `1` | Petea (HU) |
| `2` | Borș(HU) |
| `3` | Vărșand(HU) |
| `4` | Nădlac(HU) |
| `5` | Calafat (BG) |
| `6` | Bechet(BG) |
| `7` | Turnu Măgurele(BG) |
| `8` | Zimnicea(BG) |
| `9` | Giurgiu(BG) |
| `10` | Ostrov(BG) |
| `11` | Negru Vodă(BG) |
| `12` | Vama Veche(BG) |
| `13` | Călărași(BG) |
| `14` | Corabia(BG) |
| `15` | Oltenița(BG) |
| `16` | Carei  (HU) |
| `17` | Cenad (HU) |
| `18` | Episcopia Bihor (HU) |
| `19` | Salonta (HU) |
| `20` | Săcuieni (HU) |
| `21` | Turnu (HU) |
| `22` | Urziceni (HU) |
| `23` | Valea lui Mihai (HU) |
| `24` | Vladimirescu (HU) |
| `25` | Porțile de Fier 1 (RS) |
| `26` | Naidăș(RS) |
| `27` | Stamora Moravița(RS) |
| `28` | Jimbolia(RS) |
| `29` | Halmeu (UA) |
| `30` | Stânca Costești (MD) |
| `31` | Sculeni(MD) |
| `32` | Albița(MD) |
| `33` | Oancea(MD) |
| `34` | Galați Giurgiulești(MD) |
| `35` | Constanța Sud Agigea |
| `36` | Siret  (UA) |
| `37` | Nădlac 2 - A1 (HU) |
| `38` | Borș 2 - A3 (HU) |

**`l10n_ro_edi_stock_end_customs_office`** — End Customs Office

| Value | Label |
|---|---|
| `12801` | BVI Alba Iulia (ROBV0300) |
| `22801` | BVI Arad (ROTM0200) |
| `22901` | BVF Arad Aeroport (ROTM0230) |
| `22902` | BVF Zona Liberă Curtici (ROTM2300) |
| `32801` | BVI Pitești (ROCR7000) |
| `42801` | BVI Bacău (ROIS0600) |
| `42901` | BVF Bacău Aeroport (ROIS0620) |
| `52801` | BVI Oradea (ROCJ6570) |
| `52901` | BVF Oradea Aeroport (ROCJ6580) |
| `62801` | BVI Bistriţa-Năsăud (ROCJ0400) |
| `72801` | BVI Botoşani (ROIS1600) |
| `72901` | BVF Stanca Costeşti (ROIS1610) |
| `72902` | BVF Rădăuţi Prut (ROIS1620) |
| `82801` | BVI Braşov (ROBV0900) |
| `92901` | BVF Zona Liberă Brăila (ROGL0710) |
| `92902` | BVF Brăila (ROGL0700) |
| `102801` | BVI Buzău (ROGL1500) |
| `112801` | BVI Reșița (ROTM7600) |
| `112901` | BVF Naidăș (ROTM6100) |
| `122801` | BVI Cluj Napoca (ROCJ1800) |
| `122901` | BVF Cluj Napoca Aero (ROCJ1810) |
| `132901` | BVF Constanţa Sud Agigea (ROCT1900) |
| `132902` | BVF Mihail Kogălniceanu (ROCT5100) |
| `132903` | BVF Mangalia (ROCT5400) |
| `132904` | BVF Constanţa Port (ROCT1970) |
| `142801` | BVI Sfântu Gheorghe (ROBV7820) |
| `152801` | BVI Târgoviște (ROBU8600) |
| `162801` | BVI Craiova (ROCR2100) |
| `162901` | BVF Craiova Aeroport (ROCR2110) |
| `162902` | BVF Bechet (ROCR1720) |
| `162903` | BVF Calafat (ROCR1700) |
| `172901` | BVF Zona Liberă Galaţi (ROGL3810) |
| `172902` | BVF Giurgiuleşti (ROGL3850) |
| `172903` | BVF Oancea (ROGL3610) |
| `172904` | BVF Galaţi (ROGL3800) |
| `182801` | BVI Târgu Jiu (ROCR8810) |
| `192801` | BVI Miercurea Ciuc (ROBV5600) |
| `202801` | BVI Deva (ROTM8100) |
| `212801` | BVI Slobozia (ROCT8220) |
| `222901` | BVF Iaşi Aero (ROIS4660) |
| `222902` | BVF Sculeni (ROIS4990) |
| `222903` | BVF Iaşi (ROIS4650) |
| `232801` | BVI Antrepozite/Ilfov (ROBU1200) |
| `232901` | BVF Otopeni Călători (ROBU1030) |
| `242801` | BVI Baia Mare (ROCJ0500) |
| `242901` | BVF Aero Baia Mare (ROCJ0510) |
| `242902` | BVF Sighet (ROCJ8000) |
| `252901` | BVF Orşova (ROCR7280) |
| `252902` | BVF Porţile De Fier I (ROCR7270) |
| `252903` | BVF Porţile De Fier II (ROCR7200) |
| `252904` | BVF Drobeta Turnu Severin (ROCR9000) |
| `262801` | BVI Târgu Mureş (ROBV8800) |
| `262901` | BVF Târgu Mureş Aeroport (ROBV8820) |
| `272801` | BVI Piatra Neamţ (ROIS7400) |
| `282801` | BVI Corabia (ROCR2000) |
| `282802` | BVI Olt (ROCR8210) |
| `292801` | BVI Ploiești (ROBU7100) |
| `302801` | BVI Satu-Mare (ROCJ7810) |
| `302901` | BVF Halmeu (ROCJ4310) |
| `302902` | BVF Aeroport Satu Mare (ROCJ7830) |
| `312801` | BVI Zalău (ROCJ9700) |
| `322801` | BVI Sibiu (ROBV7900) |
| `322901` | BVF Sibiu Aeroport (ROBV7910) |
| `332801` | BVI Suceava (ROIS8230) |
| `332901` | BVF Dorneşti (ROIS2700) |
| `332902` | BVF Siret (ROIS8200) |
| `332903` | BVF Suceava Aero (ROIS8250) |
| `332904` | BVF Vicovu De Sus (ROIS9620) |
| `342801` | BVI Alexandria (ROCR0310) |
| `342901` | BVF Turnu Măgurele (ROCR9100) |
| `342902` | BVF Zimnicea (ROCR5800) |
| `352802` | BVI Timişoara Bază (ROTM8720) |
| `352901` | BVF Jimbolia (ROTM5010) |
| `352902` | BVF Moraviţa (ROTM5510) |
| `352903` | BVF Timişoara Aeroport (ROTM8730) |
| `362901` | BVF Sulina (ROCT8300) |
| `362902` | BVF Aeroport Delta Dunării Tulcea (ROGL8910) |
| `362903` | BVF Tulcea (ROGL8900) |
| `362904` | BVF Isaccea (ROGL8920) |
| `372801` | BVI Vaslui (ROIS9610) |
| `372901` | BVF Fălciu (-) |
| `372902` | BVF Albiţa (ROIS0100) |
| `382801` | BVI Râmnicu Vâlcea (ROCR7700) |
| `392801` | BVI Focșani (ROGL3600) |
| `402801` | BVI Bucureşti Poştă (ROBU1380) |
| `402802` | BVI Târguri și Expoziții (ROBU1400) |
| `402901` | BVF Băneasa (ROBU1040) |
| `512801` | BVI Călăraşi (ROCT1710) |
| `522801` | BVI Giurgiu (ROBU3910) |
| `522901` | BVF Zona Liberă Giurgiu (ROBU3980) |

**`l10n_ro_edi_stock_end_loc_type`** — End Location Type

| Value | Label |
|---|---|
| `location` | Location |
| `bcp` | Border Crossing Point |
| `customs` | Customs Office |

**`l10n_ro_edi_stock_operation_scope`** — Operation Scope

| Value | Label |
|---|---|
| `101` | Marketing |
| `201` | Output |
| `301` | Gratuities |
| `401` | Commercial equipment |
| `501` | Fixed assets |
| `601` | Own consumption |
| `703` | Delivery operations with installation |
| `704` | Transfer between managements |
| `705` | Goods made available to the customer |
| `801` | Financial/operational leasing |
| `802` | Goods under warranty |
| `901` | Exempt operations |
| `1001` | Investment in progress |
| `1101` | Donations, help |
| `9901` | Other |
| `9999` | Same with operation |

**`l10n_ro_edi_stock_operation_type`** — eTransport Operation Type

| Value | Label |
|---|---|
| `10` | Intra-community purchase |
| `12` | Operations in lohn system (EU) - input |
| `14` | Stocks available to the customer (Call-off stock) - entry |
| `20` | Intra-Community delivery |
| `22` | Operations in lohn system (EU) - exit |
| `24` | Stocks available to the customer (Call-off stock) - exit |
| `30` | Transport on the national territory |
| `40` | Import |
| `50` | Export |
| `60` | Intra-community transaction - Entry for storage/formation of new transport |
| `70` | Intra-community transaction - Exit after storage/formation of new transport |

**`l10n_ro_edi_stock_start_bcp`** — Start Border Crossing Point

| Value | Label |
|---|---|
| `1` | Petea (HU) |
| `2` | Borș(HU) |
| `3` | Vărșand(HU) |
| `4` | Nădlac(HU) |
| `5` | Calafat (BG) |
| `6` | Bechet(BG) |
| `7` | Turnu Măgurele(BG) |
| `8` | Zimnicea(BG) |
| `9` | Giurgiu(BG) |
| `10` | Ostrov(BG) |
| `11` | Negru Vodă(BG) |
| `12` | Vama Veche(BG) |
| `13` | Călărași(BG) |
| `14` | Corabia(BG) |
| `15` | Oltenița(BG) |
| `16` | Carei  (HU) |
| `17` | Cenad (HU) |
| `18` | Episcopia Bihor (HU) |
| `19` | Salonta (HU) |
| `20` | Săcuieni (HU) |
| `21` | Turnu (HU) |
| `22` | Urziceni (HU) |
| `23` | Valea lui Mihai (HU) |
| `24` | Vladimirescu (HU) |
| `25` | Porțile de Fier 1 (RS) |
| `26` | Naidăș(RS) |
| `27` | Stamora Moravița(RS) |
| `28` | Jimbolia(RS) |
| `29` | Halmeu (UA) |
| `30` | Stânca Costești (MD) |
| `31` | Sculeni(MD) |
| `32` | Albița(MD) |
| `33` | Oancea(MD) |
| `34` | Galați Giurgiulești(MD) |
| `35` | Constanța Sud Agigea |
| `36` | Siret  (UA) |
| `37` | Nădlac 2 - A1 (HU) |
| `38` | Borș 2 - A3 (HU) |

**`l10n_ro_edi_stock_start_customs_office`** — Start Customs Office

| Value | Label |
|---|---|
| `12801` | BVI Alba Iulia (ROBV0300) |
| `22801` | BVI Arad (ROTM0200) |
| `22901` | BVF Arad Aeroport (ROTM0230) |
| `22902` | BVF Zona Liberă Curtici (ROTM2300) |
| `32801` | BVI Pitești (ROCR7000) |
| `42801` | BVI Bacău (ROIS0600) |
| `42901` | BVF Bacău Aeroport (ROIS0620) |
| `52801` | BVI Oradea (ROCJ6570) |
| `52901` | BVF Oradea Aeroport (ROCJ6580) |
| `62801` | BVI Bistriţa-Năsăud (ROCJ0400) |
| `72801` | BVI Botoşani (ROIS1600) |
| `72901` | BVF Stanca Costeşti (ROIS1610) |
| `72902` | BVF Rădăuţi Prut (ROIS1620) |
| `82801` | BVI Braşov (ROBV0900) |
| `92901` | BVF Zona Liberă Brăila (ROGL0710) |
| `92902` | BVF Brăila (ROGL0700) |
| `102801` | BVI Buzău (ROGL1500) |
| `112801` | BVI Reșița (ROTM7600) |
| `112901` | BVF Naidăș (ROTM6100) |
| `122801` | BVI Cluj Napoca (ROCJ1800) |
| `122901` | BVF Cluj Napoca Aero (ROCJ1810) |
| `132901` | BVF Constanţa Sud Agigea (ROCT1900) |
| `132902` | BVF Mihail Kogălniceanu (ROCT5100) |
| `132903` | BVF Mangalia (ROCT5400) |
| `132904` | BVF Constanţa Port (ROCT1970) |
| `142801` | BVI Sfântu Gheorghe (ROBV7820) |
| `152801` | BVI Târgoviște (ROBU8600) |
| `162801` | BVI Craiova (ROCR2100) |
| `162901` | BVF Craiova Aeroport (ROCR2110) |
| `162902` | BVF Bechet (ROCR1720) |
| `162903` | BVF Calafat (ROCR1700) |
| `172901` | BVF Zona Liberă Galaţi (ROGL3810) |
| `172902` | BVF Giurgiuleşti (ROGL3850) |
| `172903` | BVF Oancea (ROGL3610) |
| `172904` | BVF Galaţi (ROGL3800) |
| `182801` | BVI Târgu Jiu (ROCR8810) |
| `192801` | BVI Miercurea Ciuc (ROBV5600) |
| `202801` | BVI Deva (ROTM8100) |
| `212801` | BVI Slobozia (ROCT8220) |
| `222901` | BVF Iaşi Aero (ROIS4660) |
| `222902` | BVF Sculeni (ROIS4990) |
| `222903` | BVF Iaşi (ROIS4650) |
| `232801` | BVI Antrepozite/Ilfov (ROBU1200) |
| `232901` | BVF Otopeni Călători (ROBU1030) |
| `242801` | BVI Baia Mare (ROCJ0500) |
| `242901` | BVF Aero Baia Mare (ROCJ0510) |
| `242902` | BVF Sighet (ROCJ8000) |
| `252901` | BVF Orşova (ROCR7280) |
| `252902` | BVF Porţile De Fier I (ROCR7270) |
| `252903` | BVF Porţile De Fier II (ROCR7200) |
| `252904` | BVF Drobeta Turnu Severin (ROCR9000) |
| `262801` | BVI Târgu Mureş (ROBV8800) |
| `262901` | BVF Târgu Mureş Aeroport (ROBV8820) |
| `272801` | BVI Piatra Neamţ (ROIS7400) |
| `282801` | BVI Corabia (ROCR2000) |
| `282802` | BVI Olt (ROCR8210) |
| `292801` | BVI Ploiești (ROBU7100) |
| `302801` | BVI Satu-Mare (ROCJ7810) |
| `302901` | BVF Halmeu (ROCJ4310) |
| `302902` | BVF Aeroport Satu Mare (ROCJ7830) |
| `312801` | BVI Zalău (ROCJ9700) |
| `322801` | BVI Sibiu (ROBV7900) |
| `322901` | BVF Sibiu Aeroport (ROBV7910) |
| `332801` | BVI Suceava (ROIS8230) |
| `332901` | BVF Dorneşti (ROIS2700) |
| `332902` | BVF Siret (ROIS8200) |
| `332903` | BVF Suceava Aero (ROIS8250) |
| `332904` | BVF Vicovu De Sus (ROIS9620) |
| `342801` | BVI Alexandria (ROCR0310) |
| `342901` | BVF Turnu Măgurele (ROCR9100) |
| `342902` | BVF Zimnicea (ROCR5800) |
| `352802` | BVI Timişoara Bază (ROTM8720) |
| `352901` | BVF Jimbolia (ROTM5010) |
| `352902` | BVF Moraviţa (ROTM5510) |
| `352903` | BVF Timişoara Aeroport (ROTM8730) |
| `362901` | BVF Sulina (ROCT8300) |
| `362902` | BVF Aeroport Delta Dunării Tulcea (ROGL8910) |
| `362903` | BVF Tulcea (ROGL8900) |
| `362904` | BVF Isaccea (ROGL8920) |
| `372801` | BVI Vaslui (ROIS9610) |
| `372901` | BVF Fălciu (-) |
| `372902` | BVF Albiţa (ROIS0100) |
| `382801` | BVI Râmnicu Vâlcea (ROCR7700) |
| `392801` | BVI Focșani (ROGL3600) |
| `402801` | BVI Bucureşti Poştă (ROBU1380) |
| `402802` | BVI Târguri și Expoziții (ROBU1400) |
| `402901` | BVF Băneasa (ROBU1040) |
| `512801` | BVI Călăraşi (ROCT1710) |
| `522801` | BVI Giurgiu (ROBU3910) |
| `522901` | BVF Zona Liberă Giurgiu (ROBU3980) |

**`l10n_ro_edi_stock_start_loc_type`** — Start Location Type

| Value | Label |
|---|---|
| `location` | Location |
| `bcp` | Border Crossing Point |
| `customs` | Customs Office |

**`l10n_ro_edi_stock_state`** — eTransport Status

| Value | Label |
|---|---|
| `stock_sent` | Sent |
| `stock_sending_failed` | Error |
| `stock_validated` | Validated |

**`state`** — State

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_progress` | In progress |
| `done` | Done |
| `cancel` | Cancelled |

## `stock.picking.to.batch`

**`mode`** — Mode

| Value | Label |
|---|---|
| `existing` | an existing batch transfer |
| `new` | a new batch transfer |

## `stock.picking.type`

**`code`** — Type of Operation

| Value | Label |
|---|---|
| `incoming` | Receipt |
| `outgoing` | Delivery |
| `internal` | Internal Transfer |
| `mrp_operation` | Manufacturing |
| `repair_operation` | Repair |
| `dropship` | Dropship |

**`create_backorder`** — Create Backorder

| Value | Label |
|---|---|
| `ask` | Ask |
| `always` | Always |
| `never` | Never |

**`done_mrp_lot_label_to_print`** — Lot/SN Label to Print

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

**`generated_mrp_lot_label_to_print`** — Generated Lot/SN Label to Print

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

**`lot_label_format`** — Lot Label Format to auto-print

| Value | Label |
|---|---|
| `4x12_lots` | 4 x 12 - One per lot/SN |
| `4x12_units` | 4 x 12 - One per unit |
| `zpl_lots` | ZPL Labels - One per lot/SN |
| `zpl_units` | ZPL Labels - One per unit |

**`move_type`** — Shipping Policy

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

**`mrp_product_label_to_print`** — Product Label to Print

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

**`package_label_to_print`** — Package Label to Print

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

**`product_label_format`** — Product Label Format to auto-print

| Value | Label |
|---|---|
| `dymo` | Dymo |
| `2x7xprice` | 2 x 7 with price |
| `4x7xprice` | 4 x 7 with price |
| `4x12` | 4 x 12 |
| `4x12xprice` | 4 x 12 with price |
| `zpl` | ZPL Labels |
| `zplxprice` | ZPL Labels with price |

**`reservation_method`** — Reservation Method

| Value | Label |
|---|---|
| `at_confirm` | At Confirmation |
| `manual` | Manually |
| `by_date` | Before scheduled date |

## `stock.putaway.rule`

**`sublocation`** — Sublocation

| Value | Label |
|---|---|
| `no` | No |
| `last_used` | Last Used |
| `closest_location` | Closest Location |

## `stock.quant`

**`cost_method`** — Cost Method (not stored)

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

## `stock.replenishment.info`

**`based_on`** — Based on

| Value | Label |
|---|---|
| `one_week` | Last 7 days |
| `one_month` | Last 30 days |
| `three_months` | Last 3 months |
| `one_year` | Last 12 months |
| `last_year` | Same month last year |
| `last_year_2` | Next month last year |
| `last_year_3` | After next month last year |
| `last_year_quarter` | Last year quarter |

## `stock.rule`

**`action`** — Action

| Value | Label |
|---|---|
| `pull` | Pull From |
| `push` | Push To |
| `pull_push` | Pull & Push |
| `manufacture` | Manufacture |
| `buy` | Buy |

**`auto`** — Automatic Move

| Value | Label |
|---|---|
| `manual` | Manual Operation |
| `transparent` | Automatic No Step Added |

**`procure_method`** — Supply Method

| Value | Label |
|---|---|
| `make_to_stock` | Take From Stock |
| `make_to_order` | Trigger Another Rule |
| `mts_else_mto` | Take From Stock, if unavailable, Trigger Another Rule |

## `stock.scrap`

**`state`** — Status

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Done |

## `stock.storage.category`

**`allow_new_product`** — Allow New Product

| Value | Label |
|---|---|
| `empty` | If the location is empty |
| `same` | If all products are same |
| `mixed` | Allow mixed products |

## `stock.warehouse`

**`delivery_steps`** — Outgoing Shipments

| Value | Label |
|---|---|
| `ship_only` | Deliver (1 step) |
| `pick_ship` | Pick then Deliver (2 steps) |
| `pick_pack_ship` | Pick, Pack, then Deliver (3 steps) |

**`manufacture_steps`** — Manufacture

| Value | Label |
|---|---|
| `mrp_one_step` | Manufacture (1 step) |
| `pbm` | Pick components then manufacture (2 steps) |
| `pbm_sam` | Pick components, manufacture, then store products (3 steps) |

**`reception_steps`** — Incoming Shipments

| Value | Label |
|---|---|
| `one_step` | Receive and Store (1 step) |
| `two_steps` | Receive then Store (2 steps) |
| `three_steps` | Receive, Quality Control, then Store (3 steps) |

## `stock.warehouse.orderpoint`

**`trigger`** — Trigger

| Value | Label |
|---|---|
| `auto` | Auto |
| `manual` | Manual |

## `survey.invite`

**`existing_mode`** — Handle existing

| Value | Label |
|---|---|
| `new` | New invite |
| `resend` | Resend invite |

## `survey.question`

**`matrix_subtype`** — Matrix Type

| Value | Label |
|---|---|
| `simple` | One choice per row |
| `multiple` | Multiple choices per row |

**`question_type`** — Question Type

| Value | Label |
|---|---|
| `simple_choice` | Multiple choice: only one answer |
| `multiple_choice` | Multiple choice: multiple answers allowed |
| `text_box` | Multiple Lines Text Box |
| `char_box` | Single Line Text Box |
| `numerical_box` | Numerical Value |
| `scale` | Scale |
| `date` | Date |
| `datetime` | Datetime |
| `matrix` | Matrix |

## `survey.survey`

**`access_mode`** — Access Mode

| Value | Label |
|---|---|
| `public` | Anyone with the link |
| `token` | Invited people only |

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`certification_report_layout`** — Certification template

| Value | Label |
|---|---|
| `modern_purple` | Modern Purple |
| `modern_blue` | Modern Blue |
| `modern_gold` | Modern Gold |
| `classic_purple` | Classic Purple |
| `classic_blue` | Classic Blue |
| `classic_gold` | Classic Gold |

**`progression_mode`** — Display Progress as

| Value | Label |
|---|---|
| `percent` | Percentage left |
| `number` | Number |

**`questions_layout`** — Pagination

| Value | Label |
|---|---|
| `page_per_question` | One page per question |
| `page_per_section` | One page per section |
| `one_page` | One page with all the questions |

**`questions_selection`** — Question Selection

| Value | Label |
|---|---|
| `all` | All questions |
| `random` | Randomized per Section |

**`scoring_type`** — Scoring

| Value | Label |
|---|---|
| `no_scoring` | No scoring |
| `scoring_with_answers_after_page` | Scoring with answers after each page |
| `scoring_with_answers` | Scoring with answers at the end |
| `scoring_without_answers` | Scoring without answers |

**`session_state`** — Session State

| Value | Label |
|---|---|
| `ready` | Ready |
| `in_progress` | In Progress |

**`survey_type`** — Survey Type

| Value | Label |
|---|---|
| `survey` | Survey |
| `live_session` | Live session |
| `assessment` | Assessment |
| `custom` | Custom |
| `recruitment` | Recruitment |

## `survey.user_input`

**`activity_exception_decoration`** — Activity Exception Decoration (not stored)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

**`activity_state`** — Activity State (not stored)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

**`state`** — Status

| Value | Label |
|---|---|
| `new` | New |
| `in_progress` | In Progress |
| `done` | Completed |

## `survey.user_input.line`

**`answer_type`** — Answer Type

| Value | Label |
|---|---|
| `text_box` | Free Text |
| `char_box` | Text |
| `numerical_box` | Number |
| `scale` | Number |
| `date` | Date |
| `datetime` | Datetime |
| `suggestion` | Suggestion |

## `theme.ir.asset`

**`directive`** — Directive

| Value | Label |
|---|---|
| `append` | Append |
| `prepend` | Prepend |
| `after` | After |
| `before` | Before |
| `remove` | Remove |
| `replace` | Replace |
| `include` | Include |

## `theme.ir.ui.view`

**`mode`** — Mode

| Value | Label |
|---|---|
| `primary` | Base view |
| `extension` | Extension View |

## `timesheets.analysis.report`

**`timesheet_invoice_type`** — Billable Type

| Value | Label |
|---|---|
| `billable_time` | Billed on Timesheets |
| `billable_fixed` | Billed at a Fixed price |
| `billable_milestones` | Billed on Milestones |
| `billable_manual` | Billed Manually |
| `non_billable` | Non-Billable |
| `timesheet_revenues` | Timesheet Revenues |
| `service_revenues` | Service Revenues |
| `other_revenues` | Other revenues |
| `other_costs` | Other costs |

## `uom.uom`

**`l10n_es_edi_facturae_uom_code`** — Spanish EDI Units

| Value | Label |
|---|---|
| `01` | Units |
| `02` | Hours |
| `03` | Kilograms |
| `04` | Liters |
| `05` | Other |
| `06` | Boxes |
| `07` | Trays, one layer no cover, plastic |
| `08` | Barrels |
| `09` | Jerricans, cylindrical |
| `10` | Bags |
| `11` | Carboys, non-protected |
| `12` | Bottles, non-protected, cylindrical |
| `13` | Canisters |
| `14` | Tetra Briks |
| `15` | Centiliters |
| `16` | Centimeters |
| `17` | Bins |
| `18` | Dozens |
| `19` | Cases |
| `20` | Demijohns, non-protected |
| `21` | Grams |
| `22` | Kilometers |
| `23` | Cans, rectangular |
| `24` | Bunches |
| `25` | Meters |
| `26` | Millimeters |
| `27` | 6-Packs |
| `28` | Packages |
| `29` | Portions |
| `30` | Rolls |
| `31` | Envelopes |
| `32` | Tubs |
| `33` | Cubic meter |
| `34` | Second |
| `35` | Watt |
| `36` | Kilowatt-hour |

**`l10n_hu_edi_code`** — NAV UoM code

| Value | Label |
|---|---|
| `PIECE` | Piece |
| `KILOGRAM` | Kilogram |
| `TON` | Ton |
| `KWH` | Kilowatt hour |
| `DAY` | Day |
| `HOUR` | Hour |
| `MINUTE` | Minute |
| `MONTH` | Month |
| `LITER` | Liter |
| `KILOMETER` | Kilometer |
| `CUBIC_METER` | Cubic meter |
| `METER` | Meter |
| `LINEAR_METER` | Linear meter |
| `CARTON` | Carton |
| `PACK` | Package |

## `update.product.attribute.value`

**`mode`** — Mode

| Value | Label |
|---|---|
| `add` | Add to existing products |
| `update_extra_price` | Update the extra price on existing products |

## `utm.campaign`

**`ab_testing_sms_winner_selection`** — SMS Winner Selection

| Value | Label |
|---|---|
| `manual` | Manual |
| `clicks_ratio` | Highest Click Rate |
| `crm_lead_count` | Leads |
| `sale_quotation_count` | Quotations |
| `sale_invoiced_amount` | Revenues |

**`ab_testing_winner_selection`** — Winner Selection

| Value | Label |
|---|---|
| `manual` | Manual |
| `opened_ratio` | Highest Open Rate |
| `clicks_ratio` | Highest Click Rate |
| `replied_ratio` | Highest Reply Rate |
| `crm_lead_count` | Leads |
| `sale_quotation_count` | Quotations |
| `sale_invoiced_amount` | Revenues |

## `web_tour.tour.step`

**`tooltip_position`** — Tooltip Position

| Value | Label |
|---|---|
| `bottom` | Bottom |
| `top` | Top |
| `right` | Right |
| `left` | left |

## `website`

**`account_on_checkout`** — Customer Accounts

| Value | Label |
|---|---|
| `optional` | Optional |
| `disabled` | Disabled (buy as guest) |
| `mandatory` | Mandatory (no guest checkout) |

**`add_to_cart_action`** — Add To Cart Action

| Value | Label |
|---|---|
| `stay` | Stay on Product Page |
| `go_to_cart` | Go to cart |

**`auth_signup_uninvited`** — Customer Account

| Value | Label |
|---|---|
| `b2b` | On invitation |
| `b2c` | Free sign up |

**`ecommerce_access`** — Ecommerce Access

| Value | Label |
|---|---|
| `everyone` | All users |
| `logged_in` | Logged in users |

**`product_page_cols_order`** — Product Page main columns order

| Value | Label |
|---|---|
| `regular` | Regular order |
| `inverse` | Inverse order |

**`product_page_container`** — Product Page Container

| Value | Label |
|---|---|
| `unset` | Unset |
| `regular` | Regular |
| `fluid` | Full-width |

**`product_page_image_layout`** — Product Page Image Layout

| Value | Label |
|---|---|
| `carousel` | Carousel |
| `grid` | Grid |

**`product_page_image_ratio`** — Product Page Image Ratio

| Value | Label |
|---|---|
| `auto` | Auto |
| `21_9` | Wider (21/9) |
| `16_9` | Wide (16/9) |
| `4_3` | Landscape (4/3) |
| `6_5` | Horizontal (6/5) |
| `1_1` | Default (1/1) |
| `4_5` | Portrait (4/5) |
| `2_3` | Vertical (2/3) |

**`product_page_image_ratio_mobile`** — Product Page Image Ratio Mobile

| Value | Label |
|---|---|
| `auto` | Auto |
| `21_9` | Wider (21/9) |
| `16_9` | Wide (16/9) |
| `4_3` | Landscape (4/3) |
| `6_5` | Horizontal (6/5) |
| `1_1` | Default (1/1) |
| `4_5` | Portrait (4/5) |
| `2_3` | Vertical (2/3) |

**`product_page_image_roundness`** — Product Page Image Roundness

| Value | Label |
|---|---|
| `none` | None |
| `small` | Small |
| `medium` | Medium |
| `big` | Big |

**`product_page_image_spacing`** — Product Page Image Spacing

| Value | Label |
|---|---|
| `none` | None |
| `small` | Small |
| `medium` | Medium |
| `big` | Big |

**`product_page_image_width`** — Product Page Image Width

| Value | Label |
|---|---|
| `none` | Hidden |
| `33_pc` | 33 % |
| `50_pc` | 50 % |
| `66_pc` | 66 % |
| `100_pc` | 100 % |

**`shop_page_container`** — Shop Page Container

| Value | Label |
|---|---|
| `regular` | Regular |
| `fluid` | Full-width |

**`show_line_subtotals_tax_selection`** — Line Subtotals Tax Display

| Value | Label |
|---|---|
| `tax_excluded` | Tax Excluded |
| `tax_included` | Tax Included |

## `website.controller.page`

**`default_layout`** — Default Layout

| Value | Label |
|---|---|
| `grid` | Grid |
| `list` | List |

## `website.event.menu`

**`menu_type`** — Menu Type

| Value | Label |
|---|---|
| `community` | Community Menu |
| `introduction` | Home |
| `register` | Practical |
| `other` | Other |
| `booth` | Event Booth Menus |
| `exhibitor` | Exhibitors Menus |
| `track` | Event Tracks Menus |
| `track_proposal` | Event Proposals Menus |

## `website.page.properties`

**`redirect_type`** — Redirect Type (not stored)

| Value | Label |
|---|---|
| `301` | 301 Moved permanently |
| `302` | 302 Moved temporarily |

## `website.rewrite`

**`redirect_type`** — Action

| Value | Label |
|---|---|
| `404` | 404 Not Found |
| `301` | 301 Moved permanently |
| `302` | 302 Moved temporarily |
| `308` | 308 Redirect / Rewrite |

