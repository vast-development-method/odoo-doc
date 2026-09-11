# Selection values

Every field whose value comes from a fixed set: 1107 fields carrying 6334 values in total, as they exist once every capability package has contributed.

The value is stored and is part of any external contract, so it is reproduced exactly. The label is presentation.

## `account.account`

**`account_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `asset_receivable` | {'en_US': 'Receivable', 'tr_TR': 'Alacak', 'ar_001': 'المدين'} |
| `asset_cash` | {'en_US': 'Bank and Cash', 'tr_TR': 'Banka ve Kasa', 'ar_001': 'البنك والنقد'} |
| `asset_current` | {'en_US': 'Current Assets', 'tr_TR': 'Dönen Varlıklar', 'ar_001': 'الأصول المتداولة'} |
| `asset_non_current` | {'en_US': 'Non-current Assets', 'tr_TR': 'Cari Olmayan Varlıklar', 'ar_001': 'الأصول غير المتداولة'} |
| `asset_prepayments` | {'en_US': 'Prepayments', 'tr_TR': 'Avans Ödemeleri', 'ar_001': 'المدفوعات المسددة مقدمًا'} |
| `asset_fixed` | {'en_US': 'Fixed Assets', 'tr_TR': 'Duran Varlıklar', 'ar_001': 'أصول ثابتة'} |
| `liability_payable` | {'en_US': 'Payable', 'tr_TR': 'Borç', 'ar_001': 'الدائن'} |
| `liability_credit_card` | {'en_US': 'Credit Card', 'tr_TR': 'Kredi Kartı', 'ar_001': 'البطاقة الائتمانية'} |
| `liability_current` | {'en_US': 'Current Liabilities', 'tr_TR': 'Kısa Vadeli Borçlar', 'ar_001': 'الالتزامات الجارية'} |
| `liability_non_current` | {'en_US': 'Non-current Liabilities', 'tr_TR': 'Uzun Vadeli Yükümlülükler', 'ar_001': 'الالتزامات غير الجارية'} |
| `equity` | {'en_US': 'Equity', 'tr_TR': 'Özsermaye', 'ar_001': 'رأس المال'} |
| `equity_unaffected` | {'en_US': 'Current Year Earnings', 'tr_TR': 'Cari Yıldaki Kazanç', 'ar_001': 'أرباح السنة الجارية'} |
| `income` | {'en_US': 'Income', 'tr_TR': 'Gelir', 'ar_001': 'الدخل'} |
| `income_other` | {'en_US': 'Other Income', 'tr_TR': 'Diğer Gelirler', 'ar_001': 'دخل آخر'} |
| `expense` | {'en_US': 'Expenses', 'tr_TR': 'Masraflar', 'ar_001': 'النفقات'} |
| `expense_other` | {'en_US': 'Other Expenses', 'tr_TR': 'Diğer giderler', 'ar_001': 'النفقات الأخرى'} |
| `expense_depreciation` | {'en_US': 'Depreciation', 'tr_TR': 'Amortisman', 'ar_001': 'إهلاك'} |
| `expense_direct_cost` | {'en_US': 'Cost of Revenue', 'tr_TR': 'Gelir Maliyeti', 'ar_001': 'تكاليف الإيرادات'} |
| `off_balance` | {'en_US': 'Off-Balance Sheet', 'tr_TR': 'Bilanço Dışı', 'ar_001': 'خارج الميزانية العمومية'} |

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`internal_group`** — {'en_US': 'Internal Group', 'tr_TR': 'İç Grup', 'ar_001': 'مجموعة داخلية'} (not stored)

| Value | Label |
|---|---|
| `equity` | {'en_US': 'Equity', 'tr_TR': 'Özsermaye', 'ar_001': 'رأس المال'} |
| `asset` | {'en_US': 'Asset', 'tr_TR': 'Varlık', 'ar_001': 'أصل'} |
| `liability` | {'en_US': 'Liability', 'tr_TR': 'Kaynak', 'ar_001': 'التزام'} |
| `income` | {'en_US': 'Income', 'tr_TR': 'Gelir', 'ar_001': 'الدخل'} |
| `expense` | {'en_US': 'Expense', 'tr_TR': 'Masraf', 'ar_001': 'النفقة'} |
| `off` | {'en_US': 'Off Balance', 'tr_TR': 'Bakiye Dışı', 'ar_001': 'خارج الميزانية العمومية'} |

## `account.account.tag`

**`applicability`** — {'en_US': 'Applicability', 'tr_TR': 'Uygulanabilirlik', 'ar_001': 'قابلية التطبيق'}

| Value | Label |
|---|---|
| `accounts` | {'en_US': 'Accounts', 'tr_TR': 'Hesaplar', 'ar_001': 'الحسابات'} |
| `taxes` | {'en_US': 'Taxes', 'tr_TR': 'Vergiler', 'ar_001': 'الضرائب'} |
| `products` | {'en_US': 'Products', 'tr_TR': 'Ürünler', 'ar_001': 'المنتجات'} |

## `account.analytic.applicability`

**`applicability`** — {'en_US': 'Applicability', 'tr_TR': 'Uygulanabilirlik', 'ar_001': 'قابلية التطبيق'}

| Value | Label |
|---|---|
| `optional` | {'en_US': 'Optional', 'tr_TR': 'İsteğe bağlı', 'ar_001': 'اختياري'} |
| `mandatory` | {'en_US': 'Mandatory', 'tr_TR': 'Zorunlu', 'ar_001': 'إلزامي'} |
| `unavailable` | {'en_US': 'Unavailable', 'tr_TR': 'Kullanım dışı', 'ar_001': 'غير متاح'} |

**`business_domain`** — {'en_US': 'Domain', 'tr_TR': 'Alan Adı', 'ar_001': 'النطاق'}

| Value | Label |
|---|---|
| `general` | {'en_US': 'Miscellaneous', 'tr_TR': 'Diğer', 'ar_001': 'متفرقات'} |
| `invoice` | {'en_US': 'Invoice', 'tr_TR': 'Fatura', 'ar_001': 'الفاتورة'} |
| `bill` | {'en_US': 'Vendor Bill', 'tr_TR': 'Tedarikçi Faturası', 'ar_001': 'فاتورة المورد'} |
| `expense` | {'en_US': 'Expense', 'tr_TR': 'Masraf', 'ar_001': 'النفقة'} |
| `purchase_order` | {'en_US': 'Purchase Order', 'tr_TR': 'Satınalma Siparişi', 'ar_001': 'أمر شراء'} |
| `timesheet` | {'en_US': 'Timesheet', 'tr_TR': 'Çalışma Çizelgesi', 'ar_001': 'الجدول الزمني'} |
| `manufacturing_order` | {'en_US': 'Manufacturing Order', 'tr_TR': 'Üretim Emri', 'ar_001': 'أمر التصنيع'} |
| `stock_picking` | {'en_US': 'Stock Picking', 'tr_TR': 'Stok Transfer', 'ar_001': 'انتقاء المخزون'} |
| `sale_order` | {'en_US': 'Sale Order', 'tr_TR': 'Satış Siparişi', 'ar_001': 'أمر البيع'} |

## `account.analytic.line`

**`analytic_profitability`** — {'en_US': 'Profitability', 'ar_001': 'الربحية'} (not stored)

| Value | Label |
|---|---|
| `uncategorized` | {'en_US': 'Uncategorized', 'ar_001': 'غير مصنف'} |
| `revenue` | {'en_US': 'Revenue', 'tr_TR': 'Gelir', 'ar_001': 'الإيرادات'} |
| `loss` | {'en_US': 'Loss', 'tr_TR': 'Zarar', 'ar_001': 'خسارة'} |

**`category`** — {'en_US': 'Category', 'tr_TR': 'Kategori', 'ar_001': 'الفئة'}

| Value | Label |
|---|---|
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |
| `invoice` | {'en_US': 'Customer Invoice', 'tr_TR': 'Müşteri Faturası', 'ar_001': 'فاتورة العميل'} |
| `vendor_bill` | {'en_US': 'Vendor Bill', 'tr_TR': 'Tedarikçi Faturası', 'ar_001': 'فاتورة المورد'} |
| `manufacturing_order` | {'en_US': 'Manufacturing Order', 'tr_TR': 'Üretim Emri', 'ar_001': 'أمر التصنيع'} |
| `picking_entry` | {'en_US': 'Inventory Transfer', 'tr_TR': 'Envanter Transferi', 'ar_001': 'عملية نقل المخزون'} |

**`timesheet_invoice_type`** — {'en_US': 'Billable Type', 'tr_TR': 'Faturalanabilir Tür', 'ar_001': 'النوع القابل للفوترة'}

| Value | Label |
|---|---|
| `billable_time` | {'en_US': 'Billed on Timesheets', 'tr_TR': 'Çalışma Çizelgesinden Faturalanan', 'ar_001': 'مفوتر في الجداول الزمنية'} |
| `billable_fixed` | {'en_US': 'Billed at a Fixed price', 'tr_TR': 'Sabit Fiyatla Faturalanan', 'ar_001': 'مفوترة بسعر ثابت'} |
| `billable_milestones` | {'en_US': 'Billed on Milestones', 'tr_TR': 'Kilometre Taşlarına Göre Faturalandırılır', 'ar_001': 'الفوترة حسب مؤشرات التقدم'} |
| `billable_manual` | {'en_US': 'Billed Manually', 'tr_TR': 'Manuel Faturalandırma', 'ar_001': 'تتم الفوترة يدوياً'} |
| `non_billable` | {'en_US': 'Non-Billable', 'tr_TR': 'Faturalandırılamaz', 'ar_001': 'غير قابلة للفوترة'} |
| `timesheet_revenues` | {'en_US': 'Timesheet Revenues', 'tr_TR': 'Çalışma Çizelge Gelirleri', 'ar_001': 'إيرادات الجداول الزمنية'} |
| `service_revenues` | {'en_US': 'Service Revenues', 'tr_TR': 'Hizmet Gelirleri', 'ar_001': 'إيرادات الخدمة'} |
| `other_revenues` | {'en_US': 'Other revenues', 'tr_TR': 'Diğer gelirler', 'ar_001': 'الإيرادات الأخرى'} |
| `other_costs` | {'en_US': 'Other costs', 'tr_TR': 'Diğer maliyetler', 'ar_001': 'التكاليف الأخرى'} |

## `account.analytic.plan`

**`default_applicability`** — {'en_US': 'Default Applicability', 'tr_TR': 'Varsayılan Uygulanabilirlik', 'ar_001': 'قابلية التطبيق الافتراضية'}

| Value | Label |
|---|---|
| `optional` | {'en_US': 'Optional', 'tr_TR': 'İsteğe bağlı', 'ar_001': 'اختياري'} |
| `mandatory` | {'en_US': 'Mandatory', 'tr_TR': 'Zorunlu', 'ar_001': 'إلزامي'} |
| `unavailable` | {'en_US': 'Unavailable', 'tr_TR': 'Kullanım dışı', 'ar_001': 'غير متاح'} |

## `account.automatic.entry.wizard`

**`account_type`** — {'en_US': 'Account Type', 'tr_TR': 'Hesap Tipi', 'ar_001': 'نوع الحساب'}

| Value | Label |
|---|---|
| `income` | {'en_US': 'Revenue', 'tr_TR': 'Gelir', 'ar_001': 'الإيرادات'} |
| `expense` | {'en_US': 'Expense', 'tr_TR': 'Masraf', 'ar_001': 'النفقة'} |

**`action`** — {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'}

| Value | Label |
|---|---|
| `change_period` | {'en_US': 'Change Period', 'tr_TR': 'Değişim Dönemi', 'ar_001': 'تغيير الفترة الزمنية'} |
| `change_account` | {'en_US': 'Change Account', 'tr_TR': 'Hesabı Değiştir', 'ar_001': 'تغيير الحساب'} |

## `account.cash.rounding`

**`rounding_method`** — {'en_US': 'Rounding Method', 'tr_TR': 'Yuvarlama Yöntemi', 'ar_001': 'طريقة التقريب'}

| Value | Label |
|---|---|
| `UP` | {'en_US': 'Up', 'tr_TR': 'Yukarı', 'ar_001': 'أعلى'} |
| `DOWN` | {'en_US': 'Down', 'tr_TR': 'Aşağı', 'ar_001': 'أسفل'} |
| `HALF-UP` | {'en_US': 'Nearest', 'tr_TR': 'En yakın', 'ar_001': 'الأقرب'} |

**`strategy`** — {'en_US': 'Rounding Strategy', 'tr_TR': 'Yuvarlama Stratejisi', 'ar_001': 'استراتيجية التقريب'}

| Value | Label |
|---|---|
| `biggest_tax` | {'en_US': 'Modify tax amount', 'tr_TR': 'Vergi tutarı değiştirme', 'ar_001': 'تعديل مبلغ الضريبة'} |
| `add_invoice_line` | {'en_US': 'Add a rounding line', 'tr_TR': 'Yuvarlama kalemi ekleyin', 'ar_001': 'إضافة بند تقريب'} |

## `account.debit.note`

**`l10n_sa_reason`** — {'en_US': 'ZATCA Reason', 'ar_001': 'سبب هيئة الزكاة والضريبة والجمارك'}

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | {'en_US': 'Cancellation or suspension of the supplies after its occurrence either wholly or partially', 'ar_001': 'إلغاء أو تعليق التوريدات بعد حدوثها سواء كليًا أو جزئيًا'} |
| `BR-KSA-17-reason-2` | {'en_US': 'In case of essential change or amendment in the supply, which leads to the change of the VAT due', 'ar_001': 'في حالة حدوث تغيير أو تعديل جوهري في التوريد، مما يؤدي إلى تغيير ضريبة القيمة المضافة المستحقة'} |
| `BR-KSA-17-reason-3` | {'en_US': 'Amendment of the supply value which is pre-agreed upon between the supplier and consumer', 'ar_001': 'تعديل قيمة التوريد المتفق عليها مسبقًا بين المورد والمستهلك'} |
| `BR-KSA-17-reason-4` | {'en_US': 'In case of goods or services refund', 'ar_001': 'في حالة رد سلع أو خدمات'} |
| `BR-KSA-17-reason-5` | {'en_US': "In case of change in Seller's or Buyer's information", 'ar_001': 'في حالة تغيير بيانات البائع أو المشتري'} |

## `account.edi.document`

**`blocking_level`** — {'en_US': 'Blocking Level', 'tr_TR': 'Engelleme Seviyesi', 'ar_001': 'مستوى الحجب'}

| Value | Label |
|---|---|
| `info` | {'en_US': 'Info', 'tr_TR': 'Info', 'ar_001': 'معلومات'} |
| `warning` | {'en_US': 'Warning', 'tr_TR': 'Uyarı', 'ar_001': 'تحذير'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'tr_TR': 'Giden', 'ar_001': 'بانتظار الإرسال'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `to_cancel` | {'en_US': 'To Cancel', 'tr_TR': 'İptal etmek için', 'ar_001': 'للإلغاء'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `account.fiscal.position`

**`foreign_vat_header_mode`** — {'en_US': 'Foreign Vat Header Mode', 'tr_TR': 'Yabancı Kdv Başlığı Modu', 'ar_001': 'وضع ترويسة الضريبة الأجنبية'} (not stored)

| Value | Label |
|---|---|
| `templates_found` | {'en_US': 'Templates Found', 'tr_TR': 'Bulunan Şablonlar', 'ar_001': 'القوالب التي تم إيجادها'} |
| `no_template` | {'en_US': 'No Template', 'tr_TR': 'Şablon', 'ar_001': 'لا يوجد قالب'} |

**`l10n_br_fp_type`** — {'en_US': 'Interstate Fiscal Position Type'}

| Value | Label |
|---|---|
| `internal` | {'en_US': 'Internal'} |
| `ss_nnm` | {'en_US': 'South/Southeast selling to North/Northeast/Midwest'} |
| `interstate` | {'en_US': 'Other interstate'} |

## `account.invoice.report`

**`move_type`** — {'en_US': 'Move Type', 'tr_TR': 'Hareket türü', 'ar_001': 'نوع الحركة'}

| Value | Label |
|---|---|
| `out_invoice` | {'en_US': 'Customer Invoice', 'tr_TR': 'Müşteri Faturası', 'ar_001': 'فاتورة العميل'} |
| `in_invoice` | {'en_US': 'Vendor Bill', 'tr_TR': 'Tedarikçi Faturası', 'ar_001': 'فاتورة المورد'} |
| `out_refund` | {'en_US': 'Customer Credit Note', 'tr_TR': 'Müşteri İade/Fiyat Farkı Faturası', 'ar_001': 'إشعار دائن للعميل'} |
| `in_refund` | {'en_US': 'Vendor Credit Note', 'tr_TR': 'Tedarikçi İade/Fiyat Farkı', 'ar_001': 'إشعار المورد الدائن'} |

**`payment_state`** — {'en_US': 'Payment Status', 'tr_TR': 'Ödeme Durumu', 'ar_001': 'حالة الدفع'}

| Value | Label |
|---|---|
| `not_paid` | {'en_US': 'Not Paid', 'tr_TR': 'Ödenmemiş', 'ar_001': 'غير مدفوع'} |
| `in_payment` | {'en_US': 'In Payment', 'tr_TR': 'Ödeme', 'ar_001': 'بانتظار التسوية'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `partial` | {'en_US': 'Partially Paid', 'tr_TR': 'Kısmen ödenmiş', 'ar_001': 'مسدد جزئياً'} |
| `reversed` | {'en_US': 'Reversed', 'tr_TR': 'Tersine çevrildi', 'ar_001': 'معكوس'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `invoicing_legacy` | {'en_US': 'Invoicing App Legacy', 'tr_TR': 'Eski Faturalandırma Uygulaması', 'ar_001': 'تراث تطبيق الفوترة'} |

**`state`** — {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `posted` | {'en_US': 'Open', 'tr_TR': 'Açık', 'ar_001': 'فتح'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `account.journal`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`invoice_reference_model`** — {'en_US': 'Communication Standard', 'tr_TR': 'İletişim Standardı', 'ar_001': 'معيار الاتصال'}

| Value | Label |
|---|---|
| `odoo` | {'en_US': 'Full Reference (INV/2024/00001)', 'tr_TR': 'Tam Referans (INV/2024/00001)', 'ar_001': 'المرجع الكامل (INV/2024/00001)'} |
| `euro` | {'en_US': 'European (RF83INV202400001)', 'tr_TR': 'Avrupa (RF83INV202400001)', 'ar_001': 'European (RF83INV202400001)'} |
| `number` | {'en_US': 'Numbers only (202400001)', 'tr_TR': 'Yalnızca sayılar (202400001)', 'ar_001': 'أرقام فقط (202400001)'} |
| `be` | {'en_US': 'Belgium (+++000/2024/00182+++)'} |
| `ch` | {'en_US': 'Switzerland (12 34560 00103 88500 1000 19188)'} |
| `fi` | {'en_US': 'Finnish Standard Reference (2024000068)'} |
| `fi_rf` | {'en_US': 'Finnish Creditor Reference (RF) (RF952024000071)'} |
| `no` | {'en_US': 'Norway (000001024000083)'} |
| `se_ocr2` | {'en_US': 'Sweden OCR Level 1 & 2 (1255)'} |
| `se_ocr3` | {'en_US': 'Sweden OCR Level 3 (12658)'} |
| `se_ocr4` | {'en_US': 'Sweden OCR Level 4 (001271)'} |
| `si` | {'en_US': 'Slovenian 01 (SI01 25-1235-8403)'} |
| `dk_fik_71` | {'en_US': 'Denmark FIK Number (+71)'} |
| `dk_fik_75` | {'en_US': 'Denmark FIK Number (+75)'} |

**`invoice_reference_type`** — {'en_US': 'Communication Type', 'tr_TR': 'İletişim Türü', 'ar_001': 'نوع التواصل'}

| Value | Label |
|---|---|
| `partner` | {'en_US': 'Based on Customer', 'tr_TR': 'Müşteriye Göre', 'ar_001': 'بناءً على العميل'} |
| `invoice` | {'en_US': 'Based on Invoice', 'tr_TR': 'Faturaya dayalı', 'ar_001': 'بناءً على الفواتير'} |

**`type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `sale` | {'en_US': 'Sales', 'tr_TR': 'Satışlar', 'ar_001': 'المبيعات'} |
| `purchase` | {'en_US': 'Purchase', 'tr_TR': 'Satınalma', 'ar_001': 'الشراء'} |
| `cash` | {'en_US': 'Cash', 'tr_TR': 'Kasa', 'ar_001': 'نقدي'} |
| `bank` | {'en_US': 'Bank', 'tr_TR': 'Banka', 'ar_001': 'البنك'} |
| `credit` | {'en_US': 'Credit Card', 'tr_TR': 'Kredi Kartı', 'ar_001': 'البطاقة الائتمانية'} |
| `general` | {'en_US': 'Miscellaneous', 'tr_TR': 'Diğer', 'ar_001': 'متفرقات'} |

## `account.lock_exception`

**`lock_date_field`** — {'en_US': 'Lock Date Field', 'tr_TR': 'Tarih Kilitleme Alanı', 'ar_001': 'حقل تاريخ الإقفال'}

| Value | Label |
|---|---|
| `fiscalyear_lock_date` | {'en_US': 'Global Lock Date', 'tr_TR': 'Genel Kilit Tarihi', 'ar_001': 'تاريخ الإقفال العام'} |
| `tax_lock_date` | {'en_US': 'Tax Return Lock Date', 'tr_TR': 'Vergi İadesi Kilit Tarihi', 'ar_001': 'تاريخ إقفال الإقرار الضريبي'} |
| `sale_lock_date` | {'en_US': 'Sales Lock Date', 'tr_TR': 'Satış Kilit Tarihi', 'ar_001': 'تاريخ الإقفال للمبيعات'} |
| `purchase_lock_date` | {'en_US': 'Purchase Lock Date', 'tr_TR': 'Satınalma Kilit Tarihi', 'ar_001': 'تاريخ الإقفال للمشتريات'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'المحافظة'} (not stored)

| Value | Label |
|---|---|
| `active` | {'en_US': 'Active', 'tr_TR': 'Etkin', 'ar_001': 'نشط'} |
| `revoked` | {'en_US': 'Revoked', 'tr_TR': 'Geriye alındı', 'ar_001': 'ملغي'} |
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |

## `account.merge.wizard.line`

**`display_type`** — {'en_US': 'Display Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع العرض'}

| Value | Label |
|---|---|
| `line_section` | {'en_US': 'Section', 'tr_TR': 'Bölüm', 'ar_001': 'القسم'} |
| `line_subsection` | {'en_US': 'Subsection', 'tr_TR': 'Alt bölüm', 'ar_001': 'القسم الفرعي'} |
| `account` | {'en_US': 'Account', 'tr_TR': 'Hesap', 'ar_001': 'الحساب'} |

## `account.move`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`auto_post`** — {'en_US': 'Auto-post', 'tr_TR': 'Otomatik-onay', 'ar_001': 'الترحيل التلقائي'}

| Value | Label |
|---|---|
| `no` | {'en_US': 'No', 'tr_TR': 'Hayır', 'ar_001': 'لا'} |
| `at_date` | {'en_US': 'At Date', 'tr_TR': 'Bu Tarihte', 'ar_001': 'بتاريخ'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |
| `quarterly` | {'en_US': 'Quarterly', 'tr_TR': 'Üç Aylık', 'ar_001': 'ربع سنوي'} |
| `yearly` | {'en_US': 'Yearly', 'tr_TR': 'Yıllık', 'ar_001': 'سنويًا'} |

**`edi_blocking_level`** — {'en_US': 'Edi Blocking Level', 'tr_TR': 'Edi Engelleme Seviyesi', 'ar_001': 'مستوى حجب EDI'} (not stored)

| Value | Label |
|---|---|
| `info` | {'en_US': 'Info', 'tr_TR': 'Info', 'ar_001': 'معلومات'} |
| `warning` | {'en_US': 'Warning', 'tr_TR': 'Uyarı', 'ar_001': 'تحذير'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`edi_state`** — {'en_US': 'Electronic invoicing', 'tr_TR': 'Elektronik faturalama', 'ar_001': 'الفوترة الإلكترونية'}

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'tr_TR': 'Giden', 'ar_001': 'بانتظار الإرسال'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `to_cancel` | {'en_US': 'To Cancel', 'tr_TR': 'İptal etmek için', 'ar_001': 'للإلغاء'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`l10n_bg_exemption_reason`** — {'en_US': 'Exemption reason (BG)'}

| Value | Label |
|---|---|
| `01` | {'en_US': '01 - A delivery under Part 1 of Appendix 2 of LVAT'} |
| `02` | {'en_US': '02 - A delivery under Part 2 of Appendix 2 of LVAT'} |
| `03` | {'en_US': '03 - Import under Appendix 3 of VAT act'} |

**`l10n_es_edi_facturae_reason_code`** — {'en_US': 'Spanish Facturae EDI Reason Code'}

| Value | Label |
|---|---|
| `01` | {'en_US': 'Invoice number'} |
| `02` | {'en_US': 'Invoice serial number'} |
| `03` | {'en_US': 'Issue date'} |
| `04` | {'en_US': 'Name and surnames/Corporate name - Issuer (Sender)'} |
| `05` | {'en_US': 'Name and surnames/Corporate name - Receiver'} |
| `06` | {'en_US': "Issuer's Tax Identification Number"} |
| `07` | {'en_US': "Receiver's Tax Identification Number"} |
| `08` | {'en_US': "Issuer's address"} |
| `09` | {'en_US': "Receiver's address"} |
| `10` | {'en_US': 'Item line'} |
| `11` | {'en_US': 'Applicable Tax Rate'} |
| `12` | {'en_US': 'Applicable Tax Amount'} |
| `13` | {'en_US': 'Applicable Date/Period'} |
| `14` | {'en_US': 'Invoice Class'} |
| `15` | {'en_US': 'Legal literals'} |
| `16` | {'en_US': 'Taxable Base'} |
| `80` | {'en_US': 'Calculation of tax outputs'} |
| `81` | {'en_US': 'Calculation of tax inputs'} |
| `82` | {'en_US': 'Taxable Base modified due to return of packages and packaging materials'} |
| `83` | {'en_US': 'Taxable Base modified due to discounts and rebates'} |
| `84` | {'en_US': 'Taxable Base modified due to firm court ruling or administrative decision'} |
| `85` | {'en_US': 'Taxable Base modified due to unpaid outputs where there is a judgement opening insolvency proceedings'} |

**`l10n_es_edi_verifactu_refund_reason`** — {'en_US': 'Veri*Factu Refund Reason'}

| Value | Label |
|---|---|
| `R1` | {'en_US': 'R1: Art 80.1 and 80.2 and error of law'} |
| `R2` | {'en_US': 'R2: Art. 80.3'} |
| `R3` | {'en_US': 'R3: Art. 80.4'} |
| `R4` | {'en_US': 'R4: Rest'} |
| `R5` | {'en_US': 'R5: Corrective invoices concerning simplified invoices'} |

**`l10n_es_edi_verifactu_state`** — {'en_US': 'Veri*Factu Status'}

| Value | Label |
|---|---|
| `rejected` | {'en_US': 'Rejected'} |
| `registered_with_errors` | {'en_US': 'Registered with Errors'} |
| `accepted` | {'en_US': 'Accepted'} |
| `cancelled` | {'en_US': 'Cancelled'} |

**`l10n_es_payment_means`** — {'en_US': 'Payment Means'}

| Value | Label |
|---|---|
| `01` | {'en_US': 'In cash'} |
| `02` | {'en_US': 'Direct debit'} |
| `03` | {'en_US': 'Receipt'} |
| `04` | {'en_US': 'Credit transfer'} |
| `05` | {'en_US': 'Accepted bill of exchange'} |
| `06` | {'en_US': 'Documentary credit'} |
| `07` | {'en_US': 'Contract award'} |
| `08` | {'en_US': 'Bill of exchange'} |
| `09` | {'en_US': 'Transferable promissory note'} |
| `10` | {'en_US': 'Non transferable promissory note'} |
| `11` | {'en_US': 'Cheque'} |
| `12` | {'en_US': 'Open account reimbursement'} |
| `13` | {'en_US': 'Special payment'} |
| `14` | {'en_US': 'Set-off by reciprocal credits'} |
| `15` | {'en_US': 'Payment by postgiro'} |
| `16` | {'en_US': 'Certified cheque'} |
| `17` | {'en_US': 'Banker’s draft'} |
| `18` | {'en_US': 'Cash on delivery'} |
| `19` | {'en_US': 'Payment by card'} |

**`l10n_es_tbai_refund_reason`** — {'en_US': 'Invoice Refund Reason Code (TicketBai)'}

| Value | Label |
|---|---|
| `R1` | {'en_US': 'R1: Art. 80.1, 80.2, 80.6 and rights founded error'} |
| `R2` | {'en_US': 'R2: Art. 80.3'} |
| `R3` | {'en_US': 'R3: Art. 80.4'} |
| `R4` | {'en_US': 'R4: Art. 80 - other'} |
| `R5` | {'en_US': 'R5: Factura rectificativa en facturas simplificadas'} |

**`l10n_es_tbai_state`** — {'en_US': 'TicketBAI status'} (not stored)

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |
| `cancelled` | {'en_US': 'Cancelled'} |

**`l10n_fr_pdp_flow_10_operation_type`** — {'en_US': 'L10N Fr Pdp Flow 10 Operation Type'}

| Value | Label |
|---|---|
| `sale` | {'en_US': 'Sale'} |
| `purchase` | {'en_US': 'Purchase'} |

**`l10n_fr_pdp_flow_10_report_type`** — {'en_US': 'L10N Fr Pdp Flow 10 Report Type'}

| Value | Label |
|---|---|
| `transaction` | {'en_US': 'Transaction'} |
| `payment` | {'en_US': 'Payment'} |

**`l10n_fr_pdp_status`** — {'en_US': 'E-Reporting Status'}

| Value | Label |
|---|---|
| `out_of_scope` | {'en_US': 'Out of scope'} |
| `pending` | {'en_US': 'Pending'} |
| `ready` | {'en_US': 'Ready'} |
| `error` | {'en_US': 'Error'} |
| `sent` | {'en_US': 'Sent'} |
| `completed` | {'en_US': 'Completed'} |

**`l10n_gr_edi_inv_type`** — {'en_US': 'myDATA Invoice Type'}

| Value | Label |
|---|---|
| `1.1` | {'en_US': '1.1 - Sales Invoice'} |
| `1.2` | {'en_US': '1.2 - Sales Invoice/Intra-community Supplies'} |
| `1.3` | {'en_US': '1.3 - Sales Invoice/Third Country Supplies'} |
| `1.4` | {'en_US': '1.4 - Sales Invoice/Sale on Behalf of Third Parties'} |
| `1.5` | {'en_US': '1.5 - Sales Invoice/Clearance of Sales on Behalf of Third Parties – Fees from Sales on Behalf of Third Parties'} |
| `1.6` | {'en_US': '1.6 - Sales Invoice/Supplemental Accounting Source Document'} |
| `2.1` | {'en_US': '2.1 - Service Rendered Invoice'} |
| `2.2` | {'en_US': '2.2 - Intra-community Service Rendered Invoice'} |
| `2.3` | {'en_US': '2.3 - Third Country Service Rendered Invoice'} |
| `2.4` | {'en_US': '2.4 - Service Rendered Invoice/Supplemental Accounting Source Document'} |
| `3.1` | {'en_US': '3.1 - Proof of Expenditure (non-liable Issuer)'} |
| `3.2` | {'en_US': '3.2 - Proof of Expenditure (denial of issuance by liable Issuer)'} |
| `5.1` | {'en_US': '5.1 - Credit Invoice/Associated'} |
| `5.2` | {'en_US': '5.2 - Credit Invoice/Non-Associated'} |
| `6.1` | {'en_US': '6.1 - Self-Delivery Record'} |
| `6.2` | {'en_US': '6.2 - Self-Supply Record'} |
| `7.1` | {'en_US': '7.1 - Contract – Income'} |
| `8.1` | {'en_US': '8.1 - Rents – Income'} |
| `8.2` | {'en_US': '8.2 - Special Record – Accommodation Tax Collection/Payment Receipt'} |
| `11.1` | {'en_US': '11.1 - Retail Sales Receipt'} |
| `11.2` | {'en_US': '11.2 - Service Rendered Receipt'} |
| `11.3` | {'en_US': '11.3 - Simplified Invoice'} |
| `11.4` | {'en_US': '11.4 - Retail Sales Credit Note'} |
| `11.5` | {'en_US': '11.5 - Retail Sales Receipt on Behalf of Third Parties'} |
| `13.1` | {'en_US': '13.1 - Expenses – Domestic/Foreign Retail Transaction Purchases'} |
| `13.2` | {'en_US': '13.2 - Domestic/Foreign Retail Transaction Provision'} |
| `13.3` | {'en_US': '13.3 - Shared Utility Bills'} |
| `13.4` | {'en_US': '13.4 - Subscriptions'} |
| `13.30` | {'en_US': '13.30 - Self-Declared Entity Accounting Source Documents (Dynamic)'} |
| `13.31` | {'en_US': '13.31 - Domestic/Foreign Retail Sales Credit Note'} |
| `14.1` | {'en_US': '14.1 - Invoice/Intra-community Acquisitions'} |
| `14.2` | {'en_US': '14.2 - Invoice/Third Country Acquisitions'} |
| `14.3` | {'en_US': '14.3 - Invoice/Intra-community Services Receipt'} |
| `14.4` | {'en_US': '14.4 - Invoice/Third Country Services Receipt'} |
| `14.5` | {'en_US': '14.5 - EFKA'} |
| `14.30` | {'en_US': '14.30 - Self-Declared Entity Accounting Source Documents (Dynamic)'} |
| `14.31` | {'en_US': '14.31 - Domestic/Foreign Credit Note'} |
| `15.1` | {'en_US': '15.1 - Contract-Expense'} |
| `16.1` | {'en_US': '16.1 - Rent-Expense'} |
| `17.1` | {'en_US': '17.1 - Payroll'} |
| `17.2` | {'en_US': '17.2 - Amortisations'} |
| `17.3` | {'en_US': '17.3 - Other Income Adjustment/Regularisation Entries – Accounting Base'} |
| `17.4` | {'en_US': '17.4 - Other Income Adjustment/Regularisation Entries – Tax Base'} |
| `17.5` | {'en_US': '17.5 - Other Expense Adjustment/Regularisation Entries – Accounting Base'} |
| `17.6` | {'en_US': '17.6 - Other Expense Adjustment/Regularisation Entries – Tax Base'} |

**`l10n_gr_edi_payment_method`** — {'en_US': 'Payment Method'}

| Value | Label |
|---|---|
| `1` | {'en_US': '1 - Domestic Payments Account Number'} |
| `2` | {'en_US': '2 - Foreign Payments Account Number'} |
| `3` | {'en_US': '3 - Cash'} |
| `4` | {'en_US': '4 - Check'} |
| `5` | {'en_US': '5 - On credit'} |
| `6` | {'en_US': '6 - Web Banking'} |
| `7` | {'en_US': '7 - POS / e-POS'} |

**`l10n_gr_edi_state`** — {'en_US': 'myDATA Status'}

| Value | Label |
|---|---|
| `invoice_sent` | {'en_US': 'Invoice sent'} |
| `bill_fetched` | {'en_US': 'Expense classification ready to send'} |
| `bill_sent` | {'en_US': 'Expense classification sent'} |
| `invoice_pending` | {'en_US': 'Invoice submission pending'} |

**`l10n_hr_process_type`** — {'en_US': 'Business Process Type'}

| Value | Label |
|---|---|
| `P1` | {'en_US': 'P1: Issuing invoices for deliveries of goods and services according to purchase orders, based on contracts'} |
| `P2` | {'en_US': 'P2: Periodic invoicing for deliveries of goods and services based on contracts'} |
| `P3` | {'en_US': 'P3: Issuing invoices for delivery according to an independent purchase order'} |
| `P4` | {'en_US': 'P4: Prepayment (advance payment)'} |
| `P5` | {'en_US': 'P5: Payment on the spot (Sport payment)'} |
| `P6` | {'en_US': 'P6: Payment before delivery, based on purchase order'} |
| `P7` | {'en_US': 'P7: Issuing invoices with references to the delivery note'} |
| `P8` | {'en_US': 'P8: Issuing invoices with references to the shipping and receipt notes'} |
| `P9` | {'en_US': 'P9: Credits or invoices with negative amounts, issued for various reasons, including empty returns packaging'} |
| `P10` | {'en_US': 'P10: Issuing a corrective invoice (reversal/correction of invoice)'} |
| `P11` | {'en_US': 'P11: Issuing partial and final invoices'} |
| `P12` | {'en_US': 'P12: Self-issuance of invoice'} |
| `P99` | {'en_US': 'P99: Customer-defined process'} |

**`l10n_hu_edi_state`** — {'en_US': 'NAV 3.0 status'}

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent, waiting for response'} |
| `send_timeout` | {'en_US': 'Timeout when sending'} |
| `confirmed` | {'en_US': 'Confirmed'} |
| `confirmed_warning` | {'en_US': 'Confirmed with warnings'} |
| `rejected` | {'en_US': 'Rejected'} |
| `cancel_sent` | {'en_US': 'Cancellation request sent'} |
| `cancel_timeout` | {'en_US': 'Timeout when requesting cancellation'} |
| `cancel_pending` | {'en_US': 'Cancellation request pending'} |
| `cancelled` | {'en_US': 'Cancelled'} |

**`l10n_hu_payment_mode`** — {'en_US': 'Payment mode'}

| Value | Label |
|---|---|
| `TRANSFER` | {'en_US': 'Transfer'} |
| `CASH` | {'en_US': 'Cash'} |
| `CARD` | {'en_US': 'Credit/debit card'} |
| `VOUCHER` | {'en_US': 'Voucher'} |
| `OTHER` | {'en_US': 'Other'} |

**`l10n_id_coretax_add_info_07`** — {'en_US': 'L10N Id Coretax Add Info 07'}

| Value | Label |
|---|---|
| `TD.00501` | {'en_US': '1 - untuk Kawasan Bebas'} |
| `TD.00502` | {'en_US': '2 - untuk Tempat Penimbunan Berikat'} |
| `TD.00503` | {'en_US': '3 - untuk Hibah dan Bantuan Luar Negeri'} |
| `TD.00504` | {'en_US': '4 - untuk Avtur'} |
| `TD.00505` | {'en_US': '5 - untuk Lainnya'} |
| `TD.00506` | {'en_US': '6 - untuk Kontraktor Perjanjian Karya Pengusahaan Pertambangan Batubara Generasi I'} |
| `TD.00507` | {'en_US': '7 - untuk Penyerahan bahan bakar minyak untuk Kapal Angkutan Laut Luar Negeri'} |
| `TD.00508` | {'en_US': '8 - untuk Penyerahan jasa kena pajak terkait alat angkutan tertentu'} |
| `TD.00509` | {'en_US': '9 - untuk Penyerahan BKP Tertentu di KEK'} |
| `TD.00510` | {'en_US': '10 - untuk BKP tertentu yang bersifat strategis berupa anode slime'} |
| `TD.00511` | {'en_US': '11 - untuk Penyerahan alat angkutan tertentu dan/atau Jasa Kena Pajak terkait alat angkutan tertentu'} |
| `TD.00512` | {'en_US': '12 - untuk Penyerahan kepada Kontraktor Kerja Sama Migas yang mengikuti ketentuan Peraturan Pemerintah Nomor 27 Tahun 2017'} |
| `TD.00513` | {'en_US': '13 - Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2025'} |
| `TD.00514` | {'en_US': '14 - Penyerahan Jasa Sewa Ruangan atau Bangunan Kepada Pedagang Eceran yang Ditanggung Pemerintah Tahun Anggaran 2021'} |
| `TD.00515` | {'en_US': '15 - Penyerahan Barang dan Jasa Dalam Rangka Penanganan Pandemi COVID-19 (PMK 239/PMK. 03/2020)'} |
| `TD.00516` | {'en_US': '16 - Insentif PMK-103/PMK.010/2021 berupa PPN atas Penyerahan Rumah Tapak dan Unit Hunian Rumah Susun yang Ditanggung Pemerintah Tahun Anggaran 2021'} |
| `TD.00517` | {'en_US': '17 - Kawasan Ekonomi Khusus PP nomor 40 Tahun 2021'} |
| `TD.00518` | {'en_US': '18 - Kawasan Bebas PP nomor 41 Tahun 2021'} |
| `TD.00519` | {'en_US': '19 - Penyerahan Rumah Tapak dan Unit Hunian Rumah Susun yang Ditanggung Pemerintah Tahun Anggaran 2022'} |
| `TD.00520` | {'en_US': '20 - PPN Ditanggung Pemerintah dalam rangka Penanganan Pandemi Corona Virus'} |
| `TD.00521` | {'en_US': '21 - Penyerahan kepada Kontraktor Kerja Sama Migas yang mengikuti ketentuan Peraturan Pemerintah Nomor 53 Tahun 2017'} |
| `TD.00522` | {'en_US': '22 - BKP strategis tertentu dalam bentuk anode slime dan emas butiran'} |
| `TD.00523` | {'en_US': '23 - untuk penyerahan kertas koran dan/atau majalah'} |
| `TD.00524` | {'en_US': '24 - PPN Ditanggung Pemerintah'} |
| `TD.00525` | {'en_US': '25 - BKP dan JKP tertentu'} |
| `TD.00526` | {'en_US': '26 - Penyerahan BKP dan JKP di Ibu Kota Negara baru'} |
| `TD.00527` | {'en_US': '27 - Penyerahan kendaraan listrik berbasis baterai'} |
| `TD.00528` | {'en_US': '28 - Insentif Tambahan Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2025'} |
| `TD.00529` | {'en_US': '29 - PPN atas Penyerahan Hewan Khusus Tertentu Berupa Kuda serta Perlengkapan Pendukungnya Pemerintah Tahun Anggaran 2025'} |
| `TD.00530` | {'en_US': '30 - PPN atas Penyerahan Bekal Khusus Operasi Tertentu Yang Ditanggung Pemerintah Tahun Anggaran 2025'} |
| `TD.00531` | {'en_US': '31 - Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2026'} |

**`l10n_id_coretax_add_info_08`** — {'en_US': 'L10N Id Coretax Add Info 08'}

| Value | Label |
|---|---|
| `TD.00501` | {'en_US': '1 - untuk BKP dan JKP Tertentu'} |
| `TD.00502` | {'en_US': '2 - untuk BKP Tertentu yang Bersifat Strategis'} |
| `TD.00503` | {'en_US': '3 - untuk Jasa Kebandarudaraan'} |
| `TD.00504` | {'en_US': '4 - untuk Lainnya'} |
| `TD.00505` | {'en_US': '5 - untuk BKP Tertentu yang Bersifat Strategis sesuai PP Nomor 81 Tahun 2015'} |
| `TD.00506` | {'en_US': '6 - untuk Penyerahan Jasa Kepelabuhan Tertentu untuk kegiatan angkutan laut Luar Negeri'} |
| `TD.00507` | {'en_US': '7 - untuk Penyerahan Air Bersih'} |
| `TD.00508` | {'en_US': '8 - Penyerahan BKP tertentu yang bersifat strategis berdasarkan PP 48 Tahun 2020'} |
| `TD.00509` | {'en_US': '9 - Penyerahan kepada Perwakilan Negara Asing dan Badan Internasional serta Pejabatnya'} |
| `TD.00510` | {'en_US': '10 - BKP dan JKP tertentu'} |

**`l10n_id_coretax_facility_info_07`** — {'en_US': 'L10N Id Coretax Facility Info 07'}

| Value | Label |
|---|---|
| `TD.01101` | {'en_US': '1 - Pajak Pertambahan Nilai Tidak Dipungut berdasarkan PP Nomor 10 Tahun 2012'} |
| `TD.01102` | {'en_US': '2 - Pajak Pertambahan Nilai atau Pajak Pertambahan Nilai dan Pajak Penjualan atas Barang Mewah tidak dipungut'} |
| `TD.01103` | {'en_US': '3 - Pajak Pertambahan Nilai dan Pajak Penjualan atas Barang Mewah Tidak Dipungut'} |
| `TD.01104` | {'en_US': '4 - Pajak Pertambahan Nilai Tidak Dipungut Sesuai PP Nomor 71 Tahun 2012'} |
| `TD.01105` | {'en_US': '5 - (Tidak ada Cap)'} |
| `TD.01106` | {'en_US': '6 - PPN dan/atau PPnBM tidak dipungut berdasarkan PMK No. 194/PMK.03/2012'} |
| `TD.01107` | {'en_US': '7 - PPN Tidak Dipungut Berdasarkan PP Nomor 15 Tahun 2015'} |
| `TD.01108` | {'en_US': '8 - PPN Tidak Dipungut Berdasarkan PP Nomor 69 Tahun 2015'} |
| `TD.01109` | {'en_US': '9 - PPN Tidak Dipungut Berdasarkan PP Nomor 96 Tahun 2015'} |
| `TD.01110` | {'en_US': '10 - PPN Tidak Dipungut Berdasarkan PP Nomor 106 Tahun 2015'} |
| `TD.01111` | {'en_US': '11 - PPN Tidak Dipungut Sesuai PP Nomor 50 Tahun 2019'} |
| `TD.01112` | {'en_US': '12 - PPN atau PPN dan PPnBM Tidak Dipungut Sesuai Dengan PP Nomor 27 Tahun 2017'} |
| `TD.01113` | {'en_US': '13 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 13 TAHUN 2025'} |
| `TD.01114` | {'en_US': '14 - PPN DITANGGUNG PEMERINTAH EKS PMK 102/PMK.010/2021'} |
| `TD.01115` | {'en_US': '15 - PPN DITANGGUNG PEMERINTAH EKS PMK 239/PMK.03/2020'} |
| `TD.01116` | {'en_US': '16 - Insentif PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 103/PMK.010/2021'} |
| `TD.01117` | {'en_US': '17 - PAJAK PERTAMBAHAN NILAI TIDAK DIPUNGUT BERDASARKAN PP NOMOR 40 TAHUN 2021'} |
| `TD.01118` | {'en_US': '18 - PAJAK PERTAMBAHAN NILAI TIDAK DIPUNGUT BERDASARKAN PP NOMOR 41 TAHUN 2021'} |
| `TD.01119` | {'en_US': '19 - PPN DITANGGUNG PEMERINTAH EKS PMK 6/PMK.010/2022'} |
| `TD.01120` | {'en_US': '20 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 226/PMK.03/2021'} |
| `TD.01121` | {'en_US': '21 - PPN ATAU PPN DAN PPnBM TIDAK DIPUNGUT SESUAI DENGAN PP NOMOR 53 TAHUN 2017'} |
| `TD.01122` | {'en_US': '22 - PPN tidak dipungut berdasarkan PP Nomor 70 Tahun 2021'} |
| `TD.01123` | {'en_US': '23 - PPN ditanggung Pemerintah Ex PMK-125/PMK.01/2020'} |
| `TD.01124` | {'en_US': '24 - (Tidak ada Cap)'} |
| `TD.01125` | {'en_US': '25 - PPN tidak dipungut berdasarkan PP Nomor 49 Tahun 2022'} |
| `TD.01126` | {'en_US': '26 - PPN tidak dipungut berdasarkan PP Nomor 12 Tahun 2023'} |
| `TD.01127` | {'en_US': '27 - PPN Ditanggung Pemerintah berdasarkan PMK Nomor 12 Tahun 2025'} |
| `TD.01128` | {'en_US': '28 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 60 TAHUN 2025'} |
| `TD.01129` | {'en_US': '29 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 61 TAHUN 2025'} |
| `TD.01130` | {'en_US': '30 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 44 TAHUN 2025'} |
| `TD.01131` | {'en_US': '31 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 90 TAHUN 2025'} |

**`l10n_id_coretax_facility_info_08`** — {'en_US': 'L10N Id Coretax Facility Info 08'}

| Value | Label |
|---|---|
| `TD.01101` | {'en_US': '1 - PPN Dibebaskan Sesuai PP Nomor 146 Tahun 2000 Sebagaimana Telah Diubah Dengan PP Nomor 38 Tahun 2003'} |
| `TD.01102` | {'en_US': '2 - PPN Dibebaskan Sesuai PP Nomor 12 Tahun 2001 Sebagaimana Telah Beberapa Kali Diubah Terakhir Dengan PP Nomor 31 Tahun 2007'} |
| `TD.01103` | {'en_US': '3 - PPN dibebaskan berdasarkan Peraturan Pemerintah Nomor 28 Tahun 2009'} |
| `TD.01104` | {'en_US': '4 - (Tidak ada cap)'} |
| `TD.01105` | {'en_US': '5 - PPN Dibebaskan Sesuai Dengan PP Nomor 81 Tahun 2015'} |
| `TD.01106` | {'en_US': '6 - PPN Dibebaskan Berdasarkan PP Nomor 74 Tahun 2015'} |
| `TD.01107` | {'en_US': '7 - (tanpa cap)'} |
| `TD.01108` | {'en_US': '8 - PPN DIBEBASKAN SESUAI PP NOMOR 81 TAHUN 2015 SEBAGAIMANA TELAH DIUBAH DENGAN PP 48 TAHUN 2020'} |
| `TD.01109` | {'en_US': '9 - PPN DIBEBASKAN BERDASARKAN PP NOMOR 47 TAHUN 2020'} |
| `TD.01110` | {'en_US': '10 - PPN Dibebaskan berdasarkan PP Nomor 49 Tahun 2022'} |

**`l10n_id_kode_transaksi`** — {'en_US': 'Kode Transaksi'}

| Value | Label |
|---|---|
| `01` | {'en_US': '01 To the Parties that is not VAT Collector (Regular Customers)'} |
| `02` | {'en_US': '02 To the Treasurer'} |
| `03` | {'en_US': '03 To other VAT Collectors other than the Treasurer'} |
| `04` | {'en_US': '04 Other Value of VAT Imposition Base'} |
| `05` | {'en_US': '05 Specified Amount (Article 9A Paragraph (1) VAT Law)'} |
| `06` | {'en_US': '06 to individuals holding foreign passports'} |
| `07` | {'en_US': '07 Deliveries that the VAT is not Collected'} |
| `08` | {'en_US': '08 Deliveries that the VAT is Exempted'} |
| `09` | {'en_US': '09 Deliveries of Assets (Article 16D of VAT Law)'} |
| `10` | {'en_US': '10 Other deliveries'} |

**`l10n_in_edi_cancel_reason`** — {'en_US': 'E-Invoice(IN) Cancel Reason'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Duplicate'} |
| `2` | {'en_US': 'Data Entry Mistake'} |
| `3` | {'en_US': 'Order Cancelled'} |
| `4` | {'en_US': 'Others'} |

**`l10n_in_edi_status`** — {'en_US': 'India E-Invoice Status'}

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |
| `cancelled` | {'en_US': 'Cancelled'} |

**`l10n_in_gst_treatment`** — {'en_US': 'GST Treatment'}

| Value | Label |
|---|---|
| `regular` | {'en_US': 'Registered Business - Regular'} |
| `composition` | {'en_US': 'Registered Business - Composition'} |
| `unregistered` | {'en_US': 'Unregistered Business'} |
| `consumer` | {'en_US': 'Consumer'} |
| `overseas` | {'en_US': 'Overseas'} |
| `special_economic_zone` | {'en_US': 'Special Economic Zone'} |
| `deemed_export` | {'en_US': 'Deemed Export'} |
| `uin_holders` | {'en_US': 'UIN Holders'} |

**`l10n_it_edi_state`** — {'en_US': 'SDI State'}

| Value | Label |
|---|---|
| `being_sent` | {'en_US': 'Being Sent To SdI'} |
| `requires_user_signature` | {'en_US': 'Requires user signature'} |
| `processing` | {'en_US': 'SdI Processing'} |
| `rejected` | {'en_US': 'SdI Rejected'} |
| `forwarded` | {'en_US': 'SdI Accepted, Forwarded to Partner'} |
| `forward_failed` | {'en_US': 'SdI Accepted, Forward to Partner Failed'} |
| `forward_attempt` | {'en_US': 'SdI Accepted, Forwarding to Partner'} |
| `accepted_by_pa_partner` | {'en_US': 'SdI Accepted, Accepted by the PA Partner'} |
| `rejected_by_pa_partner` | {'en_US': 'SdI Accepted, Rejected by the PA Partner'} |
| `accepted_by_pa_partner_after_expiry` | {'en_US': 'SdI Accepted, PA Partner Expired Terms'} |

**`l10n_it_origin_document_type`** — {'en_US': 'Origin Document Type'}

| Value | Label |
|---|---|
| `purchase_order` | {'en_US': 'Purchase Order'} |
| `contract` | {'en_US': 'Contract'} |
| `agreement` | {'en_US': 'Agreement'} |

**`l10n_it_payment_method`** — {'en_US': 'L10N It Payment Method'}

| Value | Label |
|---|---|
| `MP01` | {'en_US': 'MP01 - Cash'} |
| `MP02` | {'en_US': 'MP02 - Check'} |
| `MP03` | {'en_US': "MP03 - Cashier's check"} |
| `MP04` | {'en_US': 'MP04 - Cash at the Treasury'} |
| `MP05` | {'en_US': 'MP05 - Wire transfer'} |
| `MP06` | {'en_US': 'MP06 - Promissory note'} |
| `MP07` | {'en_US': 'MP07 - Bank slip'} |
| `MP08` | {'en_US': 'MP08 - Payment card'} |
| `MP09` | {'en_US': 'MP09 - RID'} |
| `MP10` | {'en_US': 'MP10 - RID users'} |
| `MP11` | {'en_US': 'MP11 - Fast RID'} |
| `MP12` | {'en_US': 'MP12 - RIBA'} |
| `MP13` | {'en_US': 'MP13 - MAV'} |
| `MP14` | {'en_US': 'MP14 - Treasury receipt'} |
| `MP15` | {'en_US': 'MP15 - Transfer of special accounting accounts'} |
| `MP16` | {'en_US': 'MP16 - Bank direct debit'} |
| `MP17` | {'en_US': 'MP17 - Postal domiciliation'} |
| `MP18` | {'en_US': 'MP18 - Postal account slip'} |
| `MP19` | {'en_US': 'MP19 - SEPA Direct Debit'} |
| `MP20` | {'en_US': 'MP20 - SEPA Direct Debit CORE'} |
| `MP21` | {'en_US': 'MP21 - SEPA Direct Debit B2B'} |
| `MP22` | {'en_US': 'MP22 - Withholding from sums already collected'} |
| `MP23` | {'en_US': 'MP23 - PagoPA'} |

**`l10n_jo_edi_invoice_type`** — {'en_US': 'Invoice Type'}

| Value | Label |
|---|---|
| `local` | {'en_US': 'Local'} |
| `export` | {'en_US': 'Export', 'ar_001': 'تصدير'} |
| `development` | {'en_US': 'Development Area'} |
| `transit` | {'en_US': 'Transit'} |
| `foreign` | {'en_US': 'Foreign Trade'} |
| `freezone` | {'en_US': 'Free Zone Transfer'} |

**`l10n_jo_edi_state`** — {'en_US': 'JoFotara State'}

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'ar_001': 'بانتظار الإرسال'} |
| `sent` | {'en_US': 'Sent', 'ar_001': 'تم الإرسال'} |
| `demo` | {'en_US': 'Sent (Demo)'} |

**`l10n_my_edi_state`** — {'en_US': 'MyInvois State'}

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'Validation In Progress'} |
| `valid` | {'en_US': 'Valid'} |
| `rejected` | {'en_US': 'Rejected'} |
| `invalid` | {'en_US': 'Invalid'} |
| `cancelled` | {'en_US': 'Cancelled'} |

**`l10n_pl_edi_status`** — {'en_US': 'KSeF Status'}

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent (In Progress)'} |
| `accepted` | {'en_US': 'Accepted'} |
| `rejected` | {'en_US': 'Rejected'} |
| `fetch_ready` | {'en_US': 'Fetch Ready'} |
| `fetched` | {'en_US': 'Fetched'} |
| `fetch_failed` | {'en_US': 'Fetch Failed'} |

**`l10n_ro_edi_state`** — {'en_US': 'E-Factura Status'}

| Value | Label |
|---|---|
| `invoice_not_indexed` | {'en_US': 'Not indexed'} |
| `invoice_sent` | {'en_US': 'Sent'} |
| `invoice_refused` | {'en_US': 'Refused'} |
| `invoice_validated` | {'en_US': 'Validated'} |

**`l10n_rs_edi_state`** — {'en_US': 'Serbia E-Invoice state'}

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent'} |
| `sending_failed` | {'en_US': 'Error'} |

**`l10n_rs_tax_date_obligations_code`** — {'en_US': 'Tax Date Obligations'}

| Value | Label |
|---|---|
| `35` | {'en_US': 'By Delivery Date'} |
| `3` | {'en_US': 'By Issuance Date'} |
| `432` | {'en_US': 'By Billing System'} |

**`l10n_sa_reason`** — {'en_US': 'ZATCA Reason', 'ar_001': 'سبب هيئة الزكاة والضريبة والجمارك'}

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | {'en_US': 'Cancellation or suspension of the supplies after its occurrence either wholly or partially', 'ar_001': 'إلغاء أو تعليق التوريدات بعد حدوثها سواء كليًا أو جزئيًا'} |
| `BR-KSA-17-reason-2` | {'en_US': 'In case of essential change or amendment in the supply, which leads to the change of the VAT due', 'ar_001': 'في حالة حدوث تغيير أو تعديل جوهري في التوريد، مما يؤدي إلى تغيير ضريبة القيمة المضافة المستحقة'} |
| `BR-KSA-17-reason-3` | {'en_US': 'Amendment of the supply value which is pre-agreed upon between the supplier and consumer', 'ar_001': 'تعديل قيمة التوريد المتفق عليها مسبقًا بين المورد والمستهلك'} |
| `BR-KSA-17-reason-4` | {'en_US': 'In case of goods or services refund', 'ar_001': 'في حالة رد سلع أو خدمات'} |
| `BR-KSA-17-reason-5` | {'en_US': "In case of change in Seller's or Buyer's information", 'ar_001': 'في حالة تغيير بيانات البائع أو المشتري'} |

**`l10n_tr_gib_invoice_scenario`** — {'en_US': 'Invoice Scenario', 'tr_TR': 'GİB Fatura Senaryosu'}

| Value | Label |
|---|---|
| `TEMELFATURA` | {'en_US': 'Basic', 'tr_TR': 'Temel Fatura'} |
| `KAMU` | {'en_US': 'Public Sector', 'tr_TR': 'Kamu Sektörü'} |

**`l10n_tr_gib_invoice_type`** — {'en_US': 'GIB Invoice Type', 'tr_TR': 'GİB Fatura Türü'}

| Value | Label |
|---|---|
| `SATIS` | {'en_US': 'Sales', 'tr_TR': 'Satış'} |
| `TEVKIFAT` | {'en_US': 'Withholding', 'tr_TR': 'Tevkifat'} |
| `IHRACKAYITLI` | {'en_US': 'Registered for Export', 'tr_TR': 'İhracata Kayıtlı'} |
| `ISTISNA` | {'en_US': 'Tax Exempt', 'tr_TR': 'KDV Istisna'} |

**`l10n_tr_nilvera_send_status`** — {'en_US': 'Nilvera Status', 'tr_TR': 'Nilvera Durumu'}

| Value | Label |
|---|---|
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata'} |
| `not_sent` | {'en_US': 'Not sent', 'tr_TR': 'Gönderilmedi'} |
| `sent` | {'en_US': 'Sent and waiting response', 'tr_TR': 'Gönderildi ve yanıt bekleniyor'} |
| `succeed` | {'en_US': 'Successful', 'tr_TR': 'Başarılı'} |
| `waiting` | {'en_US': 'Waiting', 'tr_TR': 'Bekliyorum'} |
| `unknown` | {'en_US': 'Unknown', 'tr_TR': 'Bilinmiyor'} |

**`l10n_tr_shipping_type`** — {'en_US': 'Shipping Method', 'tr_TR': 'Nakliye Yöntemi'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Sea Transportation', 'tr_TR': 'Deniz Taşımacılığı'} |
| `2` | {'en_US': 'Railway Transportation', 'tr_TR': 'Demiryolu Taşımacılığı'} |
| `3` | {'en_US': 'Road Transportation', 'tr_TR': 'Karayolu Taşımacılığı'} |
| `4` | {'en_US': 'Air Transportation', 'tr_TR': 'Hava Taşımacılığı'} |
| `5` | {'en_US': 'Post', 'tr_TR': 'Postalamak'} |
| `6` | {'en_US': 'Combined Transportation', 'tr_TR': 'Kombine Taşımacılık'} |
| `7` | {'en_US': 'Fixed Transportation', 'tr_TR': 'Sabit Taşımacılık'} |
| `8` | {'en_US': 'Domestic Water Transportation', 'tr_TR': 'Yurtiçi Su Taşımacılığı'} |
| `9` | {'en_US': 'Invalid Transportation Method', 'tr_TR': 'Geçersiz Taşıma Yöntemi'} |

**`l10n_tw_edi_allowance_notify_way`** — {'en_US': 'Allowance Notify Way'}

| Value | Label |
|---|---|
| `email` | {'en_US': 'Email'} |
| `phone` | {'en_US': 'Phone'} |

**`l10n_tw_edi_carrier_type`** — {'en_US': 'Carrier Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'ECpay e-invoice carrier'} |
| `2` | {'en_US': 'Citizen Digital Certificate'} |
| `3` | {'en_US': 'Mobile Barcode'} |
| `4` | {'en_US': 'EasyCard'} |
| `5` | {'en_US': 'iPass'} |

**`l10n_tw_edi_clearance_mark`** — {'en_US': 'Clearance Mark'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'NOT via the customs'} |
| `2` | {'en_US': 'Via the customs'} |

**`l10n_tw_edi_invoice_type`** — {'en_US': 'Ecpay Invoice Type'}

| Value | Label |
|---|---|
| `07` | {'en_US': 'General Invoice'} |
| `08` | {'en_US': 'Special Invoice'} |

**`l10n_tw_edi_refund_agreement_type`** — {'en_US': 'Refund invoice Agreement Type'}

| Value | Label |
|---|---|
| `offline` | {'en_US': 'Offline Agreement'} |
| `online` | {'en_US': 'Online Agreement'} |

**`l10n_tw_edi_refund_state`** — {'en_US': 'Refund State'}

| Value | Label |
|---|---|
| `to_be_agreed` | {'en_US': 'To be agreed'} |
| `agreed` | {'en_US': 'Agreed'} |
| `disagreed` | {'en_US': 'Disagreed'} |

**`l10n_tw_edi_state`** — {'en_US': 'Invoice Status'}

| Value | Label |
|---|---|
| `invoiced` | {'en_US': 'Invoiced'} |
| `valid` | {'en_US': 'Valid'} |
| `invalid` | {'en_US': 'Invalid'} |

**`l10n_tw_edi_zero_tax_rate_reason`** — {'en_US': 'Zero Tax Rate Reason'}

| Value | Label |
|---|---|
| `71` | {'en_US': '71: No.1 export goods'} |
| `72` | {'en_US': '72: No.2 Services related to export sales, or services provided domestically but used abroad'} |
| `73` | {'en_US': '73: No.3 Duty-free shops established by law for the sale and transit or departure of passengers'} |
| `74` | {'en_US': '74: No.4 Sale of goods or services for operation by the operator of the FREE Trade Zone'} |
| `75` | {'en_US': "75: No.5 International transportation. However, foreign transport undertakings operating international transport business in Taiwan shall be limited to those whose countries shall give equal treatment to Taiwan's international transport undertakings or be exempt from similar taxes"} |
| `76` | {'en_US': '76: No.6 Ships, aircraft and distant-water fishing vessels for international transportation'} |
| `77` | {'en_US': '77: No.7 Goods or repair services used by ships, aircraft and distant-water fishing vessels for sale and international transport'} |
| `78` | {'en_US': '78: No.8 The bonded area operator sells goods that are not directly exported by the taxable area operator and the taxable area operator is not exported to the taxation area'} |
| `79` | {'en_US': '79: No.9 The bonded area operator sells the goods that the taxable area operator deposits into the bonded warehouse or logistics center managed by the free port area or customs administration for export'} |

**`l10n_vn_edi_adjustment_type`** — {'en_US': 'Adjustment type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Money adjustment'} |
| `2` | {'en_US': 'Information adjustment'} |

**`l10n_vn_edi_invoice_state`** — {'en_US': 'Sinvoice Status'}

| Value | Label |
|---|---|
| `ready_to_send` | {'en_US': 'Ready to send'} |
| `sent` | {'en_US': 'Sent'} |
| `payment_state_to_update` | {'en_US': 'Payment status to update'} |
| `canceled` | {'en_US': 'Canceled'} |
| `adjusted` | {'en_US': 'Adjusted'} |
| `replaced` | {'en_US': 'Replaced'} |

**`move_sent_values`** — {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} (not stored)

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `not_sent` | {'en_US': 'Not Sent', 'tr_TR': 'Gönderilmedi', 'ar_001': 'لم يتم الإرسال'} |

**`move_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `entry` | {'en_US': 'Journal Entry', 'tr_TR': 'Yevmiye Kaydı', 'ar_001': 'قيد اليومية'} |
| `out_invoice` | {'en_US': 'Customer Invoice', 'tr_TR': 'Müşteri Faturası', 'ar_001': 'فاتورة العميل'} |
| `out_refund` | {'en_US': 'Customer Credit Note', 'tr_TR': 'Müşteri İade/Fiyat Farkı Faturası', 'ar_001': 'إشعار دائن للعميل'} |
| `in_invoice` | {'en_US': 'Vendor Bill', 'tr_TR': 'Tedarikçi Faturası', 'ar_001': 'فاتورة المورد'} |
| `in_refund` | {'en_US': 'Vendor Credit Note', 'tr_TR': 'Tedarikçi İade/Fiyat Farkı', 'ar_001': 'إشعار المورد الدائن'} |
| `out_receipt` | {'en_US': 'Sales Receipt', 'tr_TR': 'Satış Makbuzu', 'ar_001': 'إيصال المبيعات'} |
| `in_receipt` | {'en_US': 'Purchase Receipt', 'tr_TR': 'Satınalma Makbuzu', 'ar_001': 'إيصال الشراء'} |

**`nemhandel_move_state`** — {'en_US': 'Nemhandel status'}

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready to send'} |
| `to_send` | {'en_US': 'Queued'} |
| `processing` | {'en_US': 'Pending Reception'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |
| `BusinessAccept` | {'en_US': 'Approved'} |
| `BusinessReject` | {'en_US': 'Rejected'} |

**`payment_state`** — {'en_US': 'Payment Status', 'tr_TR': 'Ödeme Durumu', 'ar_001': 'حالة الدفع'}

| Value | Label |
|---|---|
| `not_paid` | {'en_US': 'Not Paid', 'tr_TR': 'Ödenmemiş', 'ar_001': 'غير مدفوع'} |
| `in_payment` | {'en_US': 'In Payment', 'tr_TR': 'Ödeme', 'ar_001': 'بانتظار التسوية'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `partial` | {'en_US': 'Partially Paid', 'tr_TR': 'Kısmen ödenmiş', 'ar_001': 'مسدد جزئياً'} |
| `reversed` | {'en_US': 'Reversed', 'tr_TR': 'Tersine çevrildi', 'ar_001': 'معكوس'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `invoicing_legacy` | {'en_US': 'Invoicing App Legacy', 'tr_TR': 'Eski Faturalandırma Uygulaması', 'ar_001': 'تراث تطبيق الفوترة'} |

**`pdp_ppf_lifecycle_state`** — {'en_US': 'PPF Lifeycle Status'}

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'In Progress'} |
| `sent` | {'en_US': 'Sent'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |

**`pdp_ppf_move_state`** — {'en_US': 'PPF Invoice Status'}

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'In Progress'} |
| `sent` | {'en_US': 'Sent'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |

**`peppol_move_state`** — {'en_US': 'E-Invoicing Status'}

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready to send', 'tr_TR': 'Gönderilmeye hazır', 'ar_001': 'جاهز للإرسال'} |
| `to_send` | {'en_US': 'Queued', 'tr_TR': 'Kuyruğa Alındı', 'ar_001': 'في صف الانتظار'} |
| `skipped` | {'en_US': 'Skipped', 'tr_TR': 'Bir soru cevapsız ve atlandı', 'ar_001': 'تم التخطي'} |
| `processing` | {'en_US': 'Pending Reception', 'tr_TR': 'Alım Bekleniyor', 'ar_001': 'بانتظار الاستقبال'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `AB` | {'en_US': 'Received', 'ar_001': 'تم استلامه'} |
| `AP` | {'en_US': 'Approved', 'ar_001': 'تمت الموافقة'} |
| `RE` | {'en_US': 'Rejected', 'ar_001': 'تم الرفض'} |
| `PD` | {'en_US': 'With Payments'} |
| `submitted` | {'en_US': 'Submitted'} |
| `made_available` | {'en_US': 'Made Available'} |
| `refused` | {'en_US': 'Refused'} |
| `cancelled` | {'en_US': 'Cancelled'} |
| `sent` | {'en_US': 'Sent'} |
| `suspended` | {'en_US': 'Suspended'} |
| `completed` | {'en_US': 'Completed'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `posted` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`status_in_payment`** — {'en_US': 'Status In Payment', 'tr_TR': 'Ödeme Durumu', 'ar_001': 'الحالة قيد الدفع'} (not stored)

| Value | Label |
|---|---|
| `not_paid` | {'en_US': 'Not Paid', 'tr_TR': 'Ödenmemiş', 'ar_001': 'غير مدفوع'} |
| `in_payment` | {'en_US': 'In Payment', 'tr_TR': 'Ödeme', 'ar_001': 'بانتظار التسوية'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `partial` | {'en_US': 'Partially Paid', 'tr_TR': 'Kısmen ödenmiş', 'ar_001': 'مسدد جزئياً'} |
| `reversed` | {'en_US': 'Reversed', 'tr_TR': 'Tersine çevrildi', 'ar_001': 'معكوس'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `invoicing_legacy` | {'en_US': 'Invoicing App Legacy', 'tr_TR': 'Eski Faturalandırma Uygulaması', 'ar_001': 'تراث تطبيق الفوترة'} |
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `posted` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `account.move.line`

**`display_type`** — {'en_US': 'Display Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع العرض'}

| Value | Label |
|---|---|
| `product` | {'en_US': 'Product', 'tr_TR': 'Ürün', 'ar_001': 'المنتج'} |
| `cogs` | {'en_US': 'Cost of Goods Sold', 'tr_TR': 'Satılan malların maliyeti', 'ar_001': 'تكلفة البضائع المباعة'} |
| `tax` | {'en_US': 'Tax', 'tr_TR': 'Vergi', 'ar_001': 'الضريبة'} |
| `discount` | {'en_US': 'Discount', 'tr_TR': 'İndirim', 'ar_001': 'خصم'} |
| `rounding` | {'en_US': 'Rounding', 'tr_TR': 'Yuvarlama', 'ar_001': 'التقريب'} |
| `payment_term` | {'en_US': 'Payment Term', 'tr_TR': 'Ödeme Şartı', 'ar_001': 'شروط الدفع'} |
| `line_section` | {'en_US': 'Section', 'tr_TR': 'Bölüm', 'ar_001': 'القسم'} |
| `line_subsection` | {'en_US': 'Subsection', 'tr_TR': 'Alt bölüm', 'ar_001': 'القسم الفرعي'} |
| `line_note` | {'en_US': 'Note', 'tr_TR': 'Not', 'ar_001': 'الملاحظات'} |
| `epd` | {'en_US': 'Early Payment Discount', 'tr_TR': 'Erken Ödeme İndirimi', 'ar_001': 'خصم الدفع المبكر'} |
| `non_deductible_product_total` | {'en_US': 'Non Deductible Products Total', 'tr_TR': 'Muafiyetsiz Ürünler Toplam', 'ar_001': 'إجمالي المنتجات غير القابلة للخصم'} |
| `non_deductible_product` | {'en_US': 'Non Deductible Products', 'tr_TR': 'Muafiyetsiz Ürünler', 'ar_001': 'المنتجات غير القابلة للخصم'} |
| `non_deductible_tax` | {'en_US': 'Non Deductible Tax', 'tr_TR': 'İndirilemeyen Vergi', 'ar_001': 'الضريبة غير القابلة للخصم'} |

**`l10n_gr_edi_cls_category`** — {'en_US': 'myDATA Category'}

| Value | Label |
|---|---|
| `category1_1` | {'en_US': '1.1 - Commodity Sale Income'} |
| `category1_2` | {'en_US': '1.2 - Product Sale Income'} |
| `category1_3` | {'en_US': '1.3 - Provision of Services Income'} |
| `category1_4` | {'en_US': '1.4 - Sale of Fixed Assets Income'} |
| `category1_5` | {'en_US': '1.5 - Other Income/Profits'} |
| `category1_6` | {'en_US': '1.6 - Self-Deliveries/Self-Supplies'} |
| `category1_7` | {'en_US': '1.7 - Income on behalf of Third Parties'} |
| `category1_8` | {'en_US': '1.8 - Past fiscal years income'} |
| `category1_9` | {'en_US': '1.9 - Future fiscal years income'} |
| `category1_10` | {'en_US': '1.10 - Other Income Adjustment/Regularisation Entries'} |
| `category1_95` | {'en_US': '1.95 - Other Income-related Information'} |
| `category2_1` | {'en_US': '2.1 - Commodity Purchases'} |
| `category2_2` | {'en_US': '2.2 - Raw and Adjuvant Material Purchases'} |
| `category2_3` | {'en_US': '2.3 - Services Receipt'} |
| `category2_4` | {'en_US': '2.4 - General Expenses Subject to VAT Deduction'} |
| `category2_5` | {'en_US': '2.5 - General Expenses Not Subject to VAT Deduction'} |
| `category2_6` | {'en_US': '2.6 - Personnel Fees and Benefits'} |
| `category2_7` | {'en_US': '2.7 - Fixed Asset Purchases'} |
| `category2_8` | {'en_US': '2.8 - Fixed Asset Amortisations'} |
| `category2_9` | {'en_US': '2.9 - Expenses on behalf of Third Parties'} |
| `category2_10` | {'en_US': '2.10 - Past fiscal years expenses'} |
| `category2_11` | {'en_US': '2.11 - Future fiscal years expenses'} |
| `category2_12` | {'en_US': '2.12 - Other Expense Adjustment/Regularisation Entries'} |
| `category2_13` | {'en_US': '2.13 - Stock at Period Start'} |
| `category2_14` | {'en_US': '2.14 - Stock at Period End'} |
| `category2_95` | {'en_US': '2.95 - Other Expense-related Information'} |

**`l10n_gr_edi_cls_type`** — {'en_US': 'myDATA Type'}

| Value | Label |
|---|---|
| `E3_106` | {'en_US': 'E3_106 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Commodities'} |
| `E3_205` | {'en_US': 'E3_205 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials'} |
| `E3_210` | {'en_US': 'E3_210 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress'} |
| `E3_305` | {'en_US': 'E3_305 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials'} |
| `E3_310` | {'en_US': 'E3_310 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress'} |
| `E3_318` | {'en_US': 'E3_318 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Production expenses'} |
| `E3_561_001` | {'en_US': 'E3_561_001 - Wholesale Sales of Goods and Services – for Traders'} |
| `E3_561_002` | {'en_US': 'E3_561_002 - Wholesale Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000)'} |
| `E3_561_003` | {'en_US': 'E3_561_003 - Retail Sales of Goods and Services – Private Clientele'} |
| `E3_561_004` | {'en_US': 'E3_561_004 - Retail Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000)'} |
| `E3_561_005` | {'en_US': 'E3_561_005 - Intra-Community Foreign Sales of Goods and Services'} |
| `E3_561_006` | {'en_US': 'E3_561_006 - Third Country Foreign Sales of Goods and Services'} |
| `E3_561_007` | {'en_US': 'E3_561_007 - Other Sales of Goods and Services'} |
| `E3_562` | {'en_US': 'E3_562 - Other Ordinary Income'} |
| `E3_563` | {'en_US': 'E3_563 - Credit Interest and Related Income'} |
| `E3_564` | {'en_US': 'E3_564 - Credit Exchange Differences'} |
| `E3_565` | {'en_US': 'E3_565 - Income from Participations'} |
| `E3_566` | {'en_US': 'E3_566 - Profits from Disposing Non-Current Assets'} |
| `E3_567` | {'en_US': 'E3_567 - Profits from the Reversal of Provisions and Impairments'} |
| `E3_568` | {'en_US': 'E3_568 - Profits from Measurement at Fair Value'} |
| `E3_570` | {'en_US': 'E3_570 - Extraordinary income and profits'} |
| `E3_595` | {'en_US': 'E3_595 - Self-Production Expenses'} |
| `E3_596` | {'en_US': 'E3_596 - Subsidies - Grants'} |
| `E3_597` | {'en_US': 'E3_597 - Subsidies – Grants for Investment Purposes – Expense Coverage'} |
| `E3_880_001` | {'en_US': 'E3_880_001 - Wholesale Sales of Fixed Assets'} |
| `E3_880_002` | {'en_US': 'E3_880_002 - Retail Sales of Fixed Assets'} |
| `E3_880_003` | {'en_US': 'E3_880_003 - Intra-Community Foreign Sales of Fixed Assets'} |
| `E3_880_004` | {'en_US': 'E3_880_004 - Third Country Foreign Sales of Fixed Assets'} |
| `E3_881_001` | {'en_US': 'E3_881_001 - Wholesale Sales on behalf of Third Parties'} |
| `E3_881_002` | {'en_US': 'E3_881_002 - Retail Sales on behalf of Third Parties'} |
| `E3_881_003` | {'en_US': 'E3_881_003 - Intra-Community Foreign Sales on behalf of Third Parties'} |
| `E3_881_004` | {'en_US': 'E3_881_004 - Third Country Foreign Sales on behalf of Third Parties'} |
| `E3_598_001` | {'en_US': 'E3_598_001 - Sales of goods belonging to excise duty'} |
| `E3_598_003` | {'en_US': 'E3_598_003 - Sales on behalf of farmers through an agricultural cooperative e.t.c.'} |
| `E3_101` | {'en_US': 'E3_101 - Commodities at Period Start'} |
| `E3_102_001` | {'en_US': 'E3_102_001 - Fiscal Year Commodity Purchases (net amount)/Wholesale'} |
| `E3_102_002` | {'en_US': 'E3_102_002 - Fiscal Year Commodity Purchases (net amount)/Retail'} |
| `E3_102_003` | {'en_US': 'E3_102_003 - Fiscal Year Commodity Purchases (net amount)/Goods under article 39a paragraph 5 of the VAT Code (Law 2859/2000)'} |
| `E3_102_004` | {'en_US': 'E3_102_004 - Fiscal Year Commodity Purchases (net amount)/Foreign, Intra-Community'} |
| `E3_102_005` | {'en_US': 'E3_102_005 - Fiscal Year Commodity Purchases (net amount)/Foreign, Third Countries'} |
| `E3_102_006` | {'en_US': 'E3_102_006 - Fiscal Year Commodity Purchases (net amount)/Others'} |
| `E3_104` | {'en_US': 'E3_104 - Commodities at Period End'} |
| `E3_201` | {'en_US': 'E3_201 - Raw and Other Materials at Period Start/Production'} |
| `E3_202_001` | {'en_US': 'E3_202_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale'} |
| `E3_202_002` | {'en_US': 'E3_202_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail'} |
| `E3_202_003` | {'en_US': 'E3_202_003 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Intra-Community'} |
| `E3_202_004` | {'en_US': 'E3_202_004 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Third Countries'} |
| `E3_202_005` | {'en_US': 'E3_202_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others'} |
| `E3_204` | {'en_US': 'E3_204 - Raw and Other Material Stock at Period End/Production'} |
| `E3_207` | {'en_US': 'E3_207 - Products and Production in Progress at Period Start/Production'} |
| `E3_209` | {'en_US': 'E3_209 - Products and Production in Progress at Period End/Production'} |
| `E3_301` | {'en_US': 'E3_301 - Raw and Other Material at Period Start/Agricultural'} |
| `E3_302_001` | {'en_US': 'E3_302_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale'} |
| `E3_302_002` | {'en_US': 'E3_302_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail'} |
| `E3_302_003` | {'en_US': 'E3_302_003 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Intra-Community'} |
| `E3_302_004` | {'en_US': 'E3_302_004 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Third Countries'} |
| `E3_302_005` | {'en_US': 'E3_302_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others'} |
| `E3_304` | {'en_US': 'E3_304 - Raw and Other Material Stock at Period End/Agricultural'} |
| `E3_307` | {'en_US': 'E3_307 - Products and Production in Progress at Period Start/ Agricultural'} |
| `E3_309` | {'en_US': 'E3_309 - Products and Production in Progress at Period End/ Agricultural'} |
| `E3_312` | {'en_US': 'E3_312 - Stock at Period Start (Animals-Plants)'} |
| `E3_313_001` | {'en_US': 'E3_313_001 - Animal-Plant Purchases (net amount)/Wholesale'} |
| `E3_313_002` | {'en_US': 'E3_313_002 - Animal-Plant Purchases (net amount)/Retail'} |
| `E3_313_003` | {'en_US': 'E3_313_003 - Animal-Plant Purchases (net amount)/ Foreign, Intra-Community'} |
| `E3_313_004` | {'en_US': 'E3_313_004 - Animal-Plant Purchases (net amount)/ Foreign, Third Countries'} |
| `E3_313_005` | {'en_US': 'E3_313_005 - Animal-Plant Purchases/Others'} |
| `E3_315` | {'en_US': 'E3_315 - Stock at Period End (Animals-Plants)/Agricultural'} |
| `E3_581_001` | {'en_US': 'E3_581_001 - Employee Benefits/Gross Earnings'} |
| `E3_581_002` | {'en_US': 'E3_581_002 - Employee Benefits/Employer Contributions'} |
| `E3_581_003` | {'en_US': 'E3_581_003 - Employee Benefits/Other Benefits'} |
| `E3_582` | {'en_US': 'E3_582 - Asset Measurement Damages'} |
| `E3_583` | {'en_US': 'E3_583 - Debit Exchange Differences'} |
| `E3_584` | {'en_US': 'E3_584 - Damages from Disposing-Withdrawing Non-Current Assets'} |
| `E3_585_001` | {'en_US': 'E3_585_001 - Foreign/Domestic Management Fees'} |
| `E3_585_002` | {'en_US': 'E3_585_002 - Expenditures from Linked Enterprises'} |
| `E3_585_003` | {'en_US': 'E3_585_003 - Expenditures from Non-Cooperative States or Privileged Tax Regimes'} |
| `E3_585_004` | {'en_US': 'E3_585_004 - Expenditures for Information Day-Events'} |
| `E3_585_005` | {'en_US': 'E3_585_005 - Reception and Hospitality Expenses'} |
| `E3_585_006` | {'en_US': 'E3_585_006 - Travel expenses'} |
| `E3_585_007` | {'en_US': 'E3_585_007 - Self-Employed Social Security Contributions'} |
| `E3_585_008` | {'en_US': 'E3_585_008 - Commission Agent Expenses and Fees on behalf of Farmers'} |
| `E3_585_009` | {'en_US': 'E3_585_009 - Other Fees for Domestic Services'} |
| `E3_585_010` | {'en_US': 'E3_585_010 - Other Fees for Foreign Services'} |
| `E3_585_011` | {'en_US': 'E3_585_011 - Energy'} |
| `E3_585_012` | {'en_US': 'E3_585_012 - Water'} |
| `E3_585_013` | {'en_US': 'E3_585_013 - Telecommunications'} |
| `E3_585_014` | {'en_US': 'E3_585_014 - Rents'} |
| `E3_585_015` | {'en_US': 'E3_585_015 - Advertisement and promotion'} |
| `E3_585_016` | {'en_US': 'E3_585_016 - Other expenses'} |
| `E3_586` | {'en_US': 'E3_586 - Debit interests and related expenses'} |
| `E3_587` | {'en_US': 'E3_587 - Amortisations'} |
| `E3_588` | {'en_US': 'E3_588 - Extraordinary expenses, damages and fines'} |
| `E3_589` | {'en_US': 'E3_589 - Provisions (except for Personnel Provisions)'} |
| `E3_882_001` | {'en_US': 'E3_882_001 - Fiscal Year Tangible Asset Purchases/Wholesale'} |
| `E3_882_002` | {'en_US': 'E3_882_002 - Fiscal Year Tangible Asset Purchases/Retail'} |
| `E3_882_003` | {'en_US': 'E3_882_003 - Fiscal Year Tangible Asset Purchases/ Intra-Community Foreign'} |
| `E3_882_004` | {'en_US': 'E3_882_004 - Fiscal Year Tangible Asset Purchases/ Third Country Foreign'} |
| `E3_883_001` | {'en_US': 'E3_883_001 - Fiscal Year Intangible Asset Purchases/Wholesale'} |
| `E3_883_002` | {'en_US': 'E3_883_002 - Fiscal Year Intangible Asset Purchases/Retail'} |
| `E3_883_003` | {'en_US': 'E3_883_003 - Fiscal Year Intangible Asset Purchases/ Intra-Community Foreign'} |
| `E3_883_004` | {'en_US': 'E3_883_004 - Fiscal Year Intangible Asset Purchases/ Third Country Foreign'} |
| `E3_103` | {'en_US': 'E3_103 - Impairment of goods'} |
| `E3_203` | {'en_US': 'E3_203 - Impairment of raw materials and supplies'} |
| `E3_303` | {'en_US': 'E3_303 - Impairment of raw materials and supplies'} |
| `E3_208` | {'en_US': 'E3_208 - Impairment of products and production in progress'} |
| `E3_308` | {'en_US': 'E3_308 - Impairment of products and production in progress'} |
| `E3_314` | {'en_US': 'E3_314 - Impairment of animals-plants - goods'} |
| `XE3_106` | {'en_US': 'E3_106 - Own production of fixed assets – Self Deliveries – Inventory Disasters'} |
| `XE3_205` | {'en_US': 'E3_205 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_305` | {'en_US': 'E3_305 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_210` | {'en_US': 'E3_210 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_310` | {'en_US': 'E3_310 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_318` | {'en_US': 'E3_318 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `E3_598_002` | {'en_US': 'E3_598_002 - Purchases of goods falling into excise duty'} |

**`l10n_gr_edi_cls_vat`** — {'en_US': 'VAT Classification'}

| Value | Label |
|---|---|
| `VAT_361` | {'en_US': 'VAT_361 - Domestic Purchases & Expenditures'} |
| `VAT_362` | {'en_US': 'VAT_362 - Purchases & Imports of Investment Goods (Fixed Assets)'} |
| `VAT_363` | {'en_US': 'VAT_363 - Other Imports except for Investment Goods (Fixed Assets)'} |
| `VAT_364` | {'en_US': 'VAT_364 - Intra-Community Goods Acquisitions'} |
| `VAT_365` | {'en_US': 'VAT_365 - Intra-Community Services Receipts per article 14.2.a'} |
| `VAT_366` | {'en_US': 'VAT_366 - Other Recipient Actions'} |

**`l10n_gr_edi_detail_type`** — {'en_US': 'Detail Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': '1'} |
| `2` | {'en_US': '2'} |

**`l10n_gr_edi_tax_exemption_category`** — {'en_US': 'Tax Exemption Category'}

| Value | Label |
|---|---|
| `1` | {'en_US': '1 - Without VAT - article 3 of the VAT code'} |
| `2` | {'en_US': '2 - Without VAT - article 5 of the VAT code'} |
| `3` | {'en_US': '3 - Without VAT - article 13 of the VAT code'} |
| `4` | {'en_US': '4 - Without VAT - article 14 of the VAT code'} |
| `5` | {'en_US': '5 - Without VAT - article 16 of the VAT code'} |
| `6` | {'en_US': '6 - Without VAT - article 19 of the VAT code'} |
| `7` | {'en_US': '7 - Without VAT - article 22 of the VAT code'} |
| `8` | {'en_US': '8 - Without VAT - article 24 of the VAT code'} |
| `9` | {'en_US': '9 - Without VAT - article 25 of the VAT code'} |
| `10` | {'en_US': '10 - Without VAT - article 26 of the VAT code'} |
| `11` | {'en_US': '11 - Without VAT - article 27 of the VAT code'} |
| `12` | {'en_US': '12 - Without VAT - article 27 - Seagoing Vessels of the VAT code'} |
| `13` | {'en_US': '13 - Without VAT - article 27.1.γ - Seagoing Vessels of the VAT code'} |
| `14` | {'en_US': '14 - Without VAT - article 28 of the VAT code'} |
| `15` | {'en_US': '15 - Without VAT - article 39 of the VAT code'} |
| `16` | {'en_US': '16 - Without VAT - article 39a of the VAT code'} |
| `17` | {'en_US': '17 - Without VAT - article 40 of the VAT code'} |
| `18` | {'en_US': '18 - Without VAT - article 41 of the VAT code'} |
| `19` | {'en_US': '19 - Without VAT - article 47 of the VAT code'} |
| `20` | {'en_US': '20 - VAT included - article 43 of the VAT code'} |
| `21` | {'en_US': '21 - VAT included - article 44 of the VAT code'} |
| `22` | {'en_US': '22 - VAT included - article 45 of the VAT code'} |
| `23` | {'en_US': '23 - VAT included - article 46 of the VAT code'} |
| `24` | {'en_US': '24 - Without VAT - article 6 of the VAT code'} |
| `25` | {'en_US': '25 - Without VAT - ΠΟΛ.1029 / 1995'} |
| `26` | {'en_US': '26 - Without VAT - ΠΟΛ.1167 / 2015'} |
| `27` | {'en_US': '27 - Without VAT – Other VAT exceptions'} |
| `28` | {'en_US': '28 - Without VAT - Article 24 (b)(1) of the VAT Code(Tax Free)'} |
| `29` | {'en_US': '29 - Without VAT - Article 47 b of the VAT Code(OSS non - EU scheme)'} |
| `30` | {'en_US': '30 - Without VAT - Article 47 c of the VAT Code(OSS EU scheme)'} |
| `31` | {'en_US': '31 - Excluding VAT - Article 47 d of the VAT Code(IOSS)'} |

**`l10n_in_gstr_section`** — {'en_US': 'GSTR Section'}

| Value | Label |
|---|---|
| `sale_b2b_rcm` | {'en_US': 'B2B RCM'} |
| `sale_b2b_regular` | {'en_US': 'B2B Regular'} |
| `sale_b2cl` | {'en_US': 'B2CL'} |
| `sale_b2cs` | {'en_US': 'B2CS'} |
| `sale_exp_wp` | {'en_US': 'EXP(WP)'} |
| `sale_exp_wop` | {'en_US': 'EXP(WOP)'} |
| `sale_sez_wp` | {'en_US': 'SEZ(WP)'} |
| `sale_sez_wop` | {'en_US': 'SEZ(WOP)'} |
| `sale_deemed_export` | {'en_US': 'Deemed Export'} |
| `sale_cdnr_rcm` | {'en_US': 'CDNR RCM'} |
| `sale_cdnr_regular` | {'en_US': 'CDNR Regular'} |
| `sale_cdnr_deemed_export` | {'en_US': 'CDNR(Deemed Export)'} |
| `sale_cdnr_sez_wp` | {'en_US': 'CDNR(SEZ-WP)'} |
| `sale_cdnr_sez_wop` | {'en_US': 'CDNR(SEZ-WOP)'} |
| `sale_cdnur_b2cl` | {'en_US': 'CDNUR(B2CL)'} |
| `sale_cdnur_exp_wp` | {'en_US': 'CDNUR(EXP-WP)'} |
| `sale_cdnur_exp_wop` | {'en_US': 'CDNUR(EXP-WOP)'} |
| `sale_nil_rated` | {'en_US': 'Nil Rated'} |
| `sale_exempt` | {'en_US': 'Exempt'} |
| `sale_non_gst_supplies` | {'en_US': 'Non-GST Supplies'} |
| `sale_eco_9_5` | {'en_US': 'ECO 9(5)'} |
| `sale_out_of_scope` | {'en_US': 'Out of Scope'} |
| `purchase_b2b_regular` | {'en_US': 'B2B Regular'} |
| `purchase_b2c_regular` | {'en_US': 'B2C Regular'} |
| `purchase_b2b_rcm` | {'en_US': 'B2B RCM'} |
| `purchase_b2c_rcm` | {'en_US': 'B2C RCM'} |
| `purchase_imp_services` | {'en_US': 'IMP(services-RCM)'} |
| `purchase_imp_goods` | {'en_US': 'IMP(goods)'} |
| `purchase_cdnr_regular` | {'en_US': 'CDNR Regular'} |
| `purchase_cdnur_regular` | {'en_US': 'CDNUR Regular'} |
| `purchase_cdnr_rcm` | {'en_US': 'CDNR RCM'} |
| `purchase_cdnur_rcm` | {'en_US': 'CDNUR RCM'} |
| `purchase_nil_rated` | {'en_US': 'Nil Rated'} |
| `purchase_exempt` | {'en_US': 'Exempt'} |
| `purchase_non_gst_supplies` | {'en_US': 'Non-GST Supplies'} |
| `purchase_composition_supplies` | {'en_US': 'Composition Supplies'} |
| `purchase_out_of_scope` | {'en_US': 'Out of Scope'} |

**`l10n_my_edi_classification_code`** — {'en_US': 'Malaysian classification code'}

| Value | Label |
|---|---|
| `001` | {'en_US': '(001) Breastfeeding equipment '} |
| `002` | {'en_US': '(002) Child care centres and kindergartens fees'} |
| `003` | {'en_US': '(003) Computer, smartphone or tablet'} |
| `004` | {'en_US': '(004) Consolidated e-Invoice '} |
| `005` | {'en_US': '(005) Construction materials (as specified under Fourth Schedule of the Lembaga Pembangunan Industri Pembinaan Malaysia Act 1994)'} |
| `006` | {'en_US': '(006) Disbursement'} |
| `007` | {'en_US': '(007) Donation'} |
| `008` | {'en_US': '(008) -Commerce - e-Invoice to buyer / purchaser'} |
| `009` | {'en_US': '(009) e-Commerce - Self-billed e-Invoice to seller, logistics, etc. '} |
| `010` | {'en_US': '(010) Education fees'} |
| `011` | {'en_US': '(011) Goods on consignment (Consignor)'} |
| `012` | {'en_US': '(012) Goods on consignment (Consignee)'} |
| `013` | {'en_US': '(013) Gym membership'} |
| `014` | {'en_US': '(014) Insurance - Education and medical benefits'} |
| `015` | {'en_US': '(015) Insurance - Takaful or life insurance'} |
| `016` | {'en_US': '(016) Interest and financing expenses'} |
| `017` | {'en_US': '(017) Internet subscription'} |
| `018` | {'en_US': '(018) Land and building'} |
| `019` | {'en_US': '(019) Medical examination for learning disabilities and early intervention or rehabilitation treatments of learning disabilities'} |
| `020` | {'en_US': '(020) Medical examination or vaccination expenses'} |
| `021` | {'en_US': '(021) Medical expenses for serious diseases'} |
| `022` | {'en_US': '(022) Others'} |
| `023` | {'en_US': '(023) Petroleum operations (as defined in Petroleum (Income Tax) Act 1967)'} |
| `024` | {'en_US': '(024) Private retirement scheme or deferred annuity scheme'} |
| `025` | {'en_US': '(025) Motor vehicle'} |
| `026` | {'en_US': '(026) Subscription of books / journals / magazines / newspapers / other similar publications'} |
| `027` | {'en_US': '(027) Reimbursement'} |
| `028` | {'en_US': '(028) Rental of motor vehicle'} |
| `029` | {'en_US': '(029) EV charging facilities (Installation, rental, sale / purchase or subscription fees) '} |
| `030` | {'en_US': '(030) Repair and maintenance'} |
| `031` | {'en_US': '(031) Research and development'} |
| `032` | {'en_US': '(032) Foreign income'} |
| `033` | {'en_US': '(033) Self-billed - Betting and gaming'} |
| `034` | {'en_US': '(034) Self-billed - Importation of goods'} |
| `035` | {'en_US': '(035) Self-billed - Importation of services'} |
| `036` | {'en_US': '(036) Self-billed - Others'} |
| `037` | {'en_US': '(037) Self-billed - Monetary payment to agents, dealers or distributors'} |
| `038` | {'en_US': '(038) Fees related to sports equipment, facility rentals, competition registration, and training imposed by registered sports organizations under the Sports Development Act 1997'} |
| `039` | {'en_US': '(039) Supporting equipment for disabled person'} |
| `040` | {'en_US': '(040) Voluntary contribution to approved provident fund '} |
| `041` | {'en_US': '(041) Dental examination or treatment'} |
| `042` | {'en_US': '(042) Fertility treatment'} |
| `043` | {'en_US': '(043) Treatment and home care nursing, daycare centres and residential care centers'} |
| `044` | {'en_US': '(044) Vouchers, gift cards, loyalty points, etc'} |
| `045` | {'en_US': '(045) Self-billed - Non-monetary payment to agents, dealers or distributors'} |

## `account.move.reversal`

**`l10n_es_edi_verifactu_refund_reason`** — {'en_US': 'Veri*Factu Refund Reason'}

| Value | Label |
|---|---|
| `R1` | {'en_US': 'R1: Art 80.1 and 80.2 and error of law'} |
| `R2` | {'en_US': 'R2: Art. 80.3'} |
| `R3` | {'en_US': 'R3: Art. 80.4'} |
| `R4` | {'en_US': 'R4: Rest'} |
| `R5` | {'en_US': 'R5: Corrective invoices concerning simplified invoices'} |

**`l10n_es_tbai_refund_reason`** — {'en_US': 'Invoice Refund Reason Code (TicketBai)'}

| Value | Label |
|---|---|
| `R1` | {'en_US': 'R1: Art. 80.1, 80.2, 80.6 and rights founded error'} |
| `R2` | {'en_US': 'R2: Art. 80.3'} |
| `R3` | {'en_US': 'R3: Art. 80.4'} |
| `R4` | {'en_US': 'R4: Art. 80 - other'} |
| `R5` | {'en_US': 'R5: Factura rectificativa en facturas simplificadas'} |

**`l10n_sa_reason`** — {'en_US': 'ZATCA Reason', 'ar_001': 'سبب هيئة الزكاة والضريبة والجمارك'}

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | {'en_US': 'Cancellation or suspension of the supplies after its occurrence either wholly or partially', 'ar_001': 'إلغاء أو تعليق التوريدات بعد حدوثها سواء كليًا أو جزئيًا'} |
| `BR-KSA-17-reason-2` | {'en_US': 'In case of essential change or amendment in the supply, which leads to the change of the VAT due', 'ar_001': 'في حالة حدوث تغيير أو تعديل جوهري في التوريد، مما يؤدي إلى تغيير ضريبة القيمة المضافة المستحقة'} |
| `BR-KSA-17-reason-3` | {'en_US': 'Amendment of the supply value which is pre-agreed upon between the supplier and consumer', 'ar_001': 'تعديل قيمة التوريد المتفق عليها مسبقًا بين المورد والمستهلك'} |
| `BR-KSA-17-reason-4` | {'en_US': 'In case of goods or services refund', 'ar_001': 'في حالة رد سلع أو خدمات'} |
| `BR-KSA-17-reason-5` | {'en_US': "In case of change in Seller's or Buyer's information", 'ar_001': 'في حالة تغيير بيانات البائع أو المشتري'} |

**`l10n_tw_edi_allowance_notify_way`** — {'en_US': 'Allowance Notify Way'}

| Value | Label |
|---|---|
| `email` | {'en_US': 'Email'} |
| `phone` | {'en_US': 'Phone'} |

**`l10n_tw_edi_refund_agreement_type`** — {'en_US': 'Agreement Type'}

| Value | Label |
|---|---|
| `offline` | {'en_US': 'Offline'} |
| `online` | {'en_US': 'Online'} |

**`l10n_vn_edi_adjustment_type`** — {'en_US': 'Adjustment type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Money adjustment'} |
| `2` | {'en_US': 'Information adjustment'} |

## `account.payment`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`partner_type`** — {'en_US': 'Partner Type', 'tr_TR': 'İş Ortağı Türü', 'ar_001': 'نوع الشريك'}

| Value | Label |
|---|---|
| `customer` | {'en_US': 'Customer', 'tr_TR': 'Müşteri', 'ar_001': 'العميل'} |
| `supplier` | {'en_US': 'Vendor', 'tr_TR': 'Tedarikçi', 'ar_001': 'المورد'} |

**`payment_type`** — {'en_US': 'Payment Type', 'tr_TR': 'Ödeme Türü', 'ar_001': 'نوع السداد'}

| Value | Label |
|---|---|
| `outbound` | {'en_US': 'Send', 'tr_TR': 'Gönder', 'ar_001': 'إرسال'} |
| `inbound` | {'en_US': 'Receive', 'tr_TR': 'Ödeme Alma', 'ar_001': 'استلام'} |

**`reconciled_invoices_type`** — {'en_US': 'Reconciled Invoices Type', 'tr_TR': 'Denkleştirilmiş Fatura Türü', 'ar_001': 'نوع الفواتير المسواة'} (not stored)

| Value | Label |
|---|---|
| `credit_note` | {'en_US': 'Credit Note', 'tr_TR': 'İade/Fiyat Farkı', 'ar_001': 'إشعار دائن'} |
| `invoice` | {'en_US': 'Invoice', 'tr_TR': 'Fatura', 'ar_001': 'الفاتورة'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'المحافظة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `in_process` | {'en_US': 'In Process', 'tr_TR': 'İşleniyor', 'ar_001': 'قيد التنفيذ'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `canceled` | {'en_US': 'Canceled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

## `account.payment.method`

**`payment_type`** — {'en_US': 'Payment Type', 'tr_TR': 'Ödeme Türü', 'ar_001': 'نوع السداد'}

| Value | Label |
|---|---|
| `inbound` | {'en_US': 'Inbound', 'tr_TR': 'Gelen', 'ar_001': 'واردة'} |
| `outbound` | {'en_US': 'Outbound', 'tr_TR': 'Giden', 'ar_001': 'صادرة'} |

## `account.payment.method.line`

**`l10n_it_payment_method`** — {'en_US': 'Italian Payment Method'}

| Value | Label |
|---|---|
| `MP01` | {'en_US': 'MP01 - Cash'} |
| `MP02` | {'en_US': 'MP02 - Check'} |
| `MP03` | {'en_US': "MP03 - Cashier's check"} |
| `MP04` | {'en_US': 'MP04 - Cash at the Treasury'} |
| `MP05` | {'en_US': 'MP05 - Wire transfer'} |
| `MP06` | {'en_US': 'MP06 - Promissory note'} |
| `MP07` | {'en_US': 'MP07 - Bank slip'} |
| `MP08` | {'en_US': 'MP08 - Payment card'} |
| `MP09` | {'en_US': 'MP09 - RID'} |
| `MP10` | {'en_US': 'MP10 - RID users'} |
| `MP11` | {'en_US': 'MP11 - Fast RID'} |
| `MP12` | {'en_US': 'MP12 - RIBA'} |
| `MP13` | {'en_US': 'MP13 - MAV'} |
| `MP14` | {'en_US': 'MP14 - Treasury receipt'} |
| `MP15` | {'en_US': 'MP15 - Transfer of special accounting accounts'} |
| `MP16` | {'en_US': 'MP16 - Bank direct debit'} |
| `MP17` | {'en_US': 'MP17 - Postal domiciliation'} |
| `MP18` | {'en_US': 'MP18 - Postal account slip'} |
| `MP19` | {'en_US': 'MP19 - SEPA Direct Debit'} |
| `MP20` | {'en_US': 'MP20 - SEPA Direct Debit CORE'} |
| `MP21` | {'en_US': 'MP21 - SEPA Direct Debit B2B'} |
| `MP22` | {'en_US': 'MP22 - Withholding from sums already collected'} |
| `MP23` | {'en_US': 'MP23 - PagoPA'} |

## `account.payment.register`

**`installments_mode`** — {'en_US': 'Installments Mode'}

| Value | Label |
|---|---|
| `next` | {'en_US': 'Next Installment', 'tr_TR': 'Bir Sonra Taksit', 'ar_001': 'القسط التالي'} |
| `overdue` | {'en_US': 'Overdue Amount', 'tr_TR': 'Gecikmiş Tutar', 'ar_001': 'المبلغ المتأخر'} |
| `before_date` | {'en_US': 'Before Next Payment Date', 'tr_TR': 'Bir Sonraki Ödeme Tarihinden Önce', 'ar_001': 'قبل تاريخ الدفع التالي'} |
| `full` | {'en_US': 'Full Amount', 'tr_TR': 'Toplam Tutar', 'ar_001': 'المبلغ الكامل'} |

**`partner_type`** — {'en_US': 'Partner Type', 'tr_TR': 'İş Ortağı Türü', 'ar_001': 'نوع الشريك'}

| Value | Label |
|---|---|
| `customer` | {'en_US': 'Customer', 'tr_TR': 'Müşteri', 'ar_001': 'العميل'} |
| `supplier` | {'en_US': 'Vendor', 'tr_TR': 'Tedarikçi', 'ar_001': 'المورد'} |

**`payment_difference_handling`** — {'en_US': 'Payment Difference Handling', 'tr_TR': 'Ödeme Farkı İşleme', 'ar_001': 'التعامل مع فرق الدفع'}

| Value | Label |
|---|---|
| `open` | {'en_US': 'Keep open', 'tr_TR': 'Açık bırakın', 'ar_001': 'اتركها مفتوحة'} |
| `reconcile` | {'en_US': 'Mark as fully paid', 'tr_TR': 'Tamamen ödendi olarak işaretle', 'ar_001': 'التحديد كمدفوع كلياً'} |

**`payment_type`** — {'en_US': 'Payment Type', 'tr_TR': 'Ödeme Türü', 'ar_001': 'نوع السداد'}

| Value | Label |
|---|---|
| `outbound` | {'en_US': 'Send Money', 'tr_TR': 'Para Gönder', 'ar_001': 'إرسال المال'} |
| `inbound` | {'en_US': 'Receive Money', 'tr_TR': 'Para Al', 'ar_001': 'استلام المال'} |

## `account.payment.register.withholding.line`

**`comodel_payment_type`** — {'en_US': 'Comodel Payment Type'} (not stored)

| Value | Label |
|---|---|
| `outbound` | {'en_US': 'Send Money'} |
| `inbound` | {'en_US': 'Receive Money'} |

**`placeholder_type`** — {'en_US': 'Placeholder Type'}

| Value | Label |
|---|---|
| `given_by_sequence` | {'en_US': 'Given By the Sequence'} |
| `given_by_name` | {'en_US': 'Given By the Name'} |
| `not_defined` | {'en_US': 'Not defined'} |

**`previous_placeholder_type`** — {'en_US': 'Previous Placeholder Type'}

| Value | Label |
|---|---|
| `given_by_sequence` | {'en_US': 'Given By the Sequence'} |
| `given_by_name` | {'en_US': 'Given By the Name'} |
| `not_defined` | {'en_US': 'Not defined'} |

## `account.payment.term`

**`early_pay_discount_computation`** — {'en_US': 'Cash Discount Tax Reduction', 'tr_TR': 'Nakit İndirimi Vergi İndirimi', 'ar_001': 'تخفيض ضريبة الخصم النقدي'}

| Value | Label |
|---|---|
| `included` | {'en_US': 'On early payment', 'tr_TR': 'Erken ödemede', 'ar_001': 'عند الدفع المبكر'} |
| `excluded` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقًا'} |
| `mixed` | {'en_US': 'Always (upon invoice)', 'tr_TR': 'Her zaman (fatura üzerine)', 'ar_001': 'دائماً (على الفاتورة)'} |

## `account.payment.term.line`

**`delay_type`** — {'en_US': 'Delay Type', 'tr_TR': 'Gecikme Türü', 'ar_001': 'نوع التأخير'}

| Value | Label |
|---|---|
| `days_after` | {'en_US': 'Days after invoice date', 'tr_TR': 'Fatura tarihinden sonraki günler', 'ar_001': 'أيام بعد تاريخ الفاتورة'} |
| `days_after_end_of_month` | {'en_US': 'Days after end of month', 'tr_TR': 'Ay sonundan sonraki günler', 'ar_001': 'أيام بعد نهاية الشهر'} |
| `days_after_end_of_next_month` | {'en_US': 'Days after end of next month', 'tr_TR': 'Gelecek ay sonundan sonraki günler', 'ar_001': 'أيام بعد نهاية الشهر التالي'} |
| `days_end_of_month_on_the` | {'en_US': 'Days end of month on the', 'tr_TR': 'Ayın son günü', 'ar_001': 'أيام حتى نهاية الشهر في'} |

**`value`** — {'en_US': 'Value', 'tr_TR': 'Değer', 'ar_001': 'القيمة'}

| Value | Label |
|---|---|
| `percent` | {'en_US': 'Percent', 'tr_TR': 'Yüzde', 'ar_001': 'بالمئة'} |
| `fixed` | {'en_US': 'Fixed', 'tr_TR': 'Sabit', 'ar_001': 'ثابت'} |

## `account.payment.withholding.line`

**`comodel_payment_type`** — {'en_US': 'Comodel Payment Type'} (not stored)

| Value | Label |
|---|---|
| `outbound` | {'en_US': 'Send Money'} |
| `inbound` | {'en_US': 'Receive Money'} |

**`placeholder_type`** — {'en_US': 'Placeholder Type'}

| Value | Label |
|---|---|
| `given_by_sequence` | {'en_US': 'Given By the Sequence'} |
| `given_by_name` | {'en_US': 'Given By the Name'} |
| `not_defined` | {'en_US': 'Not defined'} |

**`previous_placeholder_type`** — {'en_US': 'Previous Placeholder Type'}

| Value | Label |
|---|---|
| `given_by_sequence` | {'en_US': 'Given By the Sequence'} |
| `given_by_name` | {'en_US': 'Given By the Name'} |
| `not_defined` | {'en_US': 'Not defined'} |

## `account.peppol.clarification`

**`list_identifier`** — {'en_US': 'List identifier', 'ar_001': 'معرف القائمة'}

| Value | Label |
|---|---|
| `OPStatusReason` | {'en_US': 'OPStatusReason', 'ar_001': 'OPStatusReason'} |
| `OPStatusAction` | {'en_US': 'OPStatusAction', 'ar_001': 'OPStatusAction'} |

## `account.peppol.response`

**`pdp_flow_number`** — {'en_US': 'Flow Number'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Tax Extract'} |
| `2` | {'en_US': 'Status'} |
| `6` | {'en_US': 'Mandatory Status'} |
| `10` | {'en_US': 'Report'} |

**`pdp_ppf_state`** — {'en_US': 'PPF Status'}

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent'} |
| `received` | {'en_US': 'received'} |
| `error` | {'en_US': 'Error'} |

**`pdp_ref_response_code`** — {'en_US': 'Original Response Code'}

| Value | Label |
|---|---|
| `AB` | {'en_US': 'Received'} |
| `AP` | {'en_US': 'Approved'} |
| `PD` | {'en_US': 'Payment'} |
| `RE` | {'en_US': 'Rejected'} |
| `submitted` | {'en_US': 'Submitted'} |
| `made_available` | {'en_US': 'Made Available'} |
| `refused` | {'en_US': 'Refused'} |
| `cancelled` | {'en_US': 'Cancelled'} |
| `sent` | {'en_US': 'Sent'} |
| `suspended` | {'en_US': 'Suspended'} |
| `completed` | {'en_US': 'Completed'} |

**`peppol_state`** — {'en_US': 'Peppol status', 'ar_001': 'Peppol status'}

| Value | Label |
|---|---|
| `processing` | {'en_US': 'Pending Reception', 'ar_001': 'بانتظار الاستقبال'} |
| `done` | {'en_US': 'Done', 'ar_001': 'تم الانتهاء '} |
| `error` | {'en_US': 'Error', 'ar_001': 'خطأ'} |
| `not_serviced` | {'en_US': 'Not Serviced', 'ar_001': 'غير متوفر'} |

**`response_code`** — {'en_US': 'Response Code', 'ar_001': 'رمز الاستجابة'}

| Value | Label |
|---|---|
| `AB` | {'en_US': 'Acknowledgement', 'ar_001': 'شكر وتقدير'} |
| `IP` | {'en_US': 'In Process', 'ar_001': 'قيد التنفيذ'} |
| `UQ` | {'en_US': 'Under query', 'ar_001': 'تحت الاستعلام'} |
| `CA` | {'en_US': 'Conditionally accepted', 'ar_001': 'مقبول بشروط'} |
| `RE` | {'en_US': 'Rejection', 'ar_001': 'الرفض'} |
| `AP` | {'en_US': 'Approval', 'ar_001': 'الموافقة'} |
| `PD` | {'en_US': 'Paid', 'ar_001': 'مدفوع'} |
| `submitted` | {'en_US': 'Submitted'} |
| `made_available` | {'en_US': 'Made Available'} |
| `refused` | {'en_US': 'Refused'} |
| `cancelled` | {'en_US': 'Cancelled'} |
| `sent` | {'en_US': 'Sent'} |
| `suspended` | {'en_US': 'Suspended'} |
| `completed` | {'en_US': 'Completed'} |

## `account.reconcile.model`

**`match_amount`** — {'en_US': 'Amount', 'tr_TR': 'Tutar', 'ar_001': 'مبلغ'}

| Value | Label |
|---|---|
| `lower` | {'en_US': 'Is lower than or equal to', 'tr_TR': "'den küçük veya eşittir", 'ar_001': 'أقل من أو يساوي'} |
| `greater` | {'en_US': 'Is greater than or equal to', 'tr_TR': 'Şundan büyük ya da eşit olan', 'ar_001': 'أكبر من أو يساوي'} |
| `between` | {'en_US': 'Is between', 'tr_TR': 'Şunun arasında olan', 'ar_001': 'بين'} |

**`match_label`** — {'en_US': 'Label', 'tr_TR': 'Etiket', 'ar_001': 'بطاقة عنوان'}

| Value | Label |
|---|---|
| `contains` | {'en_US': 'Contains', 'tr_TR': 'İçerir', 'ar_001': 'يحتوي على'} |
| `not_contains` | {'en_US': 'Not Contains', 'tr_TR': 'İçermez', 'ar_001': 'لا يحتوي'} |
| `match_regex` | {'en_US': 'Match Regex', 'tr_TR': 'Regex ile Eşleştir', 'ar_001': 'مطابقة التعابير النمطية (Regex)'} |

**`trigger`** — {'en_US': 'Trigger', 'tr_TR': 'Tetikleyici', 'ar_001': 'المشغّل'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `auto_reconcile` | {'en_US': 'Automated', 'tr_TR': 'Otomatik', 'ar_001': 'مؤتمت'} |

## `account.reconcile.model.line`

**`amount_type`** — {'en_US': 'Amount Type', 'tr_TR': 'Tutar Türü', 'ar_001': 'نوع المبلغ'}

| Value | Label |
|---|---|
| `fixed` | {'en_US': 'Fixed', 'tr_TR': 'Sabit', 'ar_001': 'ثابت'} |
| `percentage` | {'en_US': 'Percentage of balance', 'tr_TR': 'Bakiye Yüzdesi', 'ar_001': 'نسبة الرصيد'} |
| `percentage_st_line` | {'en_US': 'Percentage of statement line', 'tr_TR': 'Ekstre satırının yüzdesi', 'ar_001': 'نسبة بند الكشف'} |
| `regex` | {'en_US': 'From label', 'tr_TR': 'Açıklamadan', 'ar_001': 'من بطاقة العنوان'} |

## `account.report`

**`availability_condition`** — {'en_US': 'Availability', 'tr_TR': 'Müsaitlik', 'ar_001': 'التوافر'}

| Value | Label |
|---|---|
| `country` | {'en_US': 'Country Matches', 'tr_TR': 'Ülke Eşleşmeleri', 'ar_001': 'تطابقات الدول'} |
| `coa` | {'en_US': 'Chart of Accounts Matches', 'tr_TR': 'Hesap Planı Eşleşmeleri', 'ar_001': 'تطابقات شجرة الحسابات'} |
| `always` | {'en_US': 'Always', 'tr_TR': 'Daima', 'ar_001': 'دائمًا'} |

**`currency_translation`** — {'en_US': 'Currency Translation', 'tr_TR': 'Kur Çevrimleri', 'ar_001': 'ترجمة العملة'}

| Value | Label |
|---|---|
| `current` | {'en_US': 'Use the most recent rate at the date of the report', 'tr_TR': 'Rapor tarihindeki en güncel kuru kullanın.', 'ar_001': 'استخدم أحدث سعر لصرف العملة في التاريخ الذي صدر فيه التقرير'} |
| `cta` | {'en_US': 'Use CTA', 'tr_TR': 'CTA Kullan', 'ar_001': 'استخدم CTA'} |

**`default_opening_date_filter`** — {'en_US': 'Default Opening', 'tr_TR': 'Varsayılan Açılış', 'ar_001': 'الرصيد الافتتاحي الافتراضي'}

| Value | Label |
|---|---|
| `this_year` | {'en_US': 'This Year', 'tr_TR': 'Bu Yıl', 'ar_001': 'هذه السنة'} |
| `this_quarter` | {'en_US': 'This Quarter', 'tr_TR': 'Bu Çeyrek', 'ar_001': 'ربع السنة الجاري'} |
| `this_month` | {'en_US': 'This Month', 'tr_TR': 'Bu Ay', 'ar_001': 'هذا الشهر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `previous_month` | {'en_US': 'Last Month', 'tr_TR': 'Geçen Ay', 'ar_001': 'الشهر الماضي'} |
| `previous_quarter` | {'en_US': 'Last Quarter', 'tr_TR': 'Geçen Çeyrek', 'ar_001': 'ربع السنة الماضي'} |
| `previous_year` | {'en_US': 'Last Year', 'tr_TR': 'Geçen Yıl', 'ar_001': 'العام الماضي'} |
| `this_return_period` | {'en_US': 'This Return Period', 'tr_TR': 'Bu İade Dönemi', 'ar_001': 'فترة الإقرار هذه'} |
| `previous_return_period` | {'en_US': 'Last Return Period', 'tr_TR': 'Son İade Dönemi', 'ar_001': 'آخر فترة للإقرار'} |

**`filter_account_type`** — {'en_US': 'Account Types', 'tr_TR': 'Hesap Tipleri', 'ar_001': 'أنواع الحسابات'}

| Value | Label |
|---|---|
| `both` | {'en_US': 'Payable and receivable', 'tr_TR': 'Ödeme ve Alacak', 'ar_001': 'الدائنة والمدينة'} |
| `payable` | {'en_US': 'Payable', 'tr_TR': 'Borç', 'ar_001': 'الدائن'} |
| `receivable` | {'en_US': 'Receivable', 'tr_TR': 'Alacak', 'ar_001': 'المدين'} |
| `disabled` | {'en_US': 'Disabled', 'tr_TR': 'Devre Dışı', 'ar_001': 'معطل'} |

**`filter_hide_0_lines`** — {'en_US': 'Hide lines at 0', 'tr_TR': "0'da satırları gizle", 'ar_001': 'إخفاء البنود في 0'}

| Value | Label |
|---|---|
| `by_default` | {'en_US': 'Enabled by Default', 'tr_TR': 'Varsayılan Olarak Etkindir', 'ar_001': 'مفعّل تلقائياً'} |
| `optional` | {'en_US': 'Optional', 'tr_TR': 'İsteğe bağlı', 'ar_001': 'اختياري'} |
| `never` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقًا'} |

**`filter_hierarchy`** — {'en_US': 'Account Groups', 'tr_TR': 'Hesap Grupları', 'ar_001': 'مجموعات الحساب'}

| Value | Label |
|---|---|
| `by_default` | {'en_US': 'Enabled by Default', 'tr_TR': 'Varsayılan Olarak Etkindir', 'ar_001': 'مفعّل تلقائياً'} |
| `optional` | {'en_US': 'Optional', 'tr_TR': 'İsteğe bağlı', 'ar_001': 'اختياري'} |
| `never` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقًا'} |

**`filter_multi_company`** — {'en_US': 'Multi-Company', 'tr_TR': 'Çoklu-Şirket', 'ar_001': 'الشركات المتعددة'}

| Value | Label |
|---|---|
| `selector` | {'en_US': 'Use Company Selector', 'tr_TR': 'Şirket Seçiciyi Kullanın', 'ar_001': 'استخدم أداة تحديد الشركة'} |
| `tax_units` | {'en_US': 'Use Tax Units', 'tr_TR': 'Vergi Birimlerini Kullanın', 'ar_001': 'استخدم الوحدات الضريبية'} |

**`integer_rounding`** — {'en_US': 'Integer Rounding', 'tr_TR': 'Sayı (Integer) Yuvarlama', 'ar_001': 'تقريب العدد الصحيح'}

| Value | Label |
|---|---|
| `HALF-UP` | {'en_US': 'Nearest', 'tr_TR': 'En yakın', 'ar_001': 'الأقرب'} |
| `UP` | {'en_US': 'Up', 'tr_TR': 'Yukarı', 'ar_001': 'أعلى'} |
| `DOWN` | {'en_US': 'Down', 'tr_TR': 'Aşağı', 'ar_001': 'أسفل'} |

## `account.report.column`

**`figure_type`** — {'en_US': 'Figure Type', 'tr_TR': 'Şekil Türü', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `monetary` | {'en_US': 'Monetary', 'tr_TR': 'Parasal Değeri', 'ar_001': 'نقدي'} |
| `percentage` | {'en_US': 'Percentage', 'tr_TR': 'Yüzde', 'ar_001': 'النسبة'} |
| `integer` | {'en_US': 'Integer', 'tr_TR': 'Tamsayı', 'ar_001': 'عدد صحيح'} |
| `float` | {'en_US': 'Float', 'tr_TR': 'Float', 'ar_001': 'فاصلة عشرية'} |
| `date` | {'en_US': 'Date', 'tr_TR': 'Tarih', 'ar_001': 'التاريخ'} |
| `datetime` | {'en_US': 'Datetime', 'tr_TR': 'Tarih Saat', 'ar_001': 'التاريخ والوقت'} |
| `boolean` | {'en_US': 'Boolean', 'tr_TR': 'Boolean', 'ar_001': 'Boolean'} |
| `string` | {'en_US': 'String', 'tr_TR': 'Dizi metni', 'ar_001': 'نص'} |

## `account.report.expression`

**`date_scope`** — {'en_US': 'Date Scope', 'tr_TR': 'Tarih Kapsamı', 'ar_001': 'مجال التاريخ'}

| Value | Label |
|---|---|
| `from_beginning` | {'en_US': 'From the very start', 'tr_TR': 'En başından beri', 'ar_001': 'منذ البداية'} |
| `from_fiscalyear` | {'en_US': 'From the start of the fiscal year', 'tr_TR': 'Mali yılın başından itibaren', 'ar_001': 'من بداية العام المالي'} |
| `to_beginning_of_fiscalyear` | {'en_US': 'At the beginning of the fiscal year', 'tr_TR': 'Mali yıl başında', 'ar_001': 'في بداية العام المالي'} |
| `to_beginning_of_period` | {'en_US': 'At the beginning of the period', 'tr_TR': 'Dönemin başında', 'ar_001': 'في بداية الفترة'} |
| `strict_range` | {'en_US': 'Strictly on the given dates', 'tr_TR': 'Kesinlikle verilen tarihlerde', 'ar_001': 'فقط في التواريخ المعطاة'} |
| `previous_return_period` | {'en_US': 'From previous return period', 'tr_TR': 'Önceki dönüş döneminden', 'ar_001': 'من فترة الإقرار السابقة'} |

**`engine`** — {'en_US': 'Computation Engine', 'tr_TR': 'Hesaplama Motoru', 'ar_001': 'محرك الاحتساب'}

| Value | Label |
|---|---|
| `domain` | {'en_US': 'Odoo Domain', 'tr_TR': 'Odoo Domain', 'ar_001': 'نطاق أودو'} |
| `tax_tags` | {'en_US': 'Tax Tags', 'tr_TR': 'Vergi Etiketleri', 'ar_001': 'علامات تصنيف الضرائب'} |
| `aggregation` | {'en_US': 'Aggregate Other Formulas', 'tr_TR': 'Diğer Formülleri Topla', 'ar_001': 'تجميع الصيغ الأخرى'} |
| `account_codes` | {'en_US': 'Prefix of Account Codes', 'tr_TR': 'Hesap Kodlarının Öneki', 'ar_001': 'بادئات أكواد الحساب'} |
| `external` | {'en_US': 'External Value', 'tr_TR': 'Harici Değer', 'ar_001': 'القيمة الخارجية'} |
| `custom` | {'en_US': 'Custom Python Function', 'tr_TR': 'Özel Python İşlevi', 'ar_001': 'وظيفة بايثون مخصصة'} |

**`figure_type`** — {'en_US': 'Figure Type', 'tr_TR': 'Şekil Türü', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `monetary` | {'en_US': 'Monetary', 'tr_TR': 'Parasal Değeri', 'ar_001': 'نقدي'} |
| `percentage` | {'en_US': 'Percentage', 'tr_TR': 'Yüzde', 'ar_001': 'النسبة'} |
| `integer` | {'en_US': 'Integer', 'tr_TR': 'Tamsayı', 'ar_001': 'عدد صحيح'} |
| `float` | {'en_US': 'Float', 'tr_TR': 'Float', 'ar_001': 'فاصلة عشرية'} |
| `date` | {'en_US': 'Date', 'tr_TR': 'Tarih', 'ar_001': 'التاريخ'} |
| `datetime` | {'en_US': 'Datetime', 'tr_TR': 'Tarih Saat', 'ar_001': 'التاريخ والوقت'} |
| `boolean` | {'en_US': 'Boolean', 'tr_TR': 'Boolean', 'ar_001': 'Boolean'} |
| `string` | {'en_US': 'String', 'tr_TR': 'Dizi metni', 'ar_001': 'نص'} |

## `account.report.line`

**`horizontal_split_side`** — {'en_US': 'Horizontal Split Side', 'tr_TR': 'Yatay Ayırma Tarafı', 'ar_001': 'تقسيم الشاشة أفقياً'}

| Value | Label |
|---|---|
| `left` | {'en_US': 'Left', 'tr_TR': 'Sol', 'ar_001': 'يسار'} |
| `right` | {'en_US': 'Right', 'tr_TR': 'Sağ', 'ar_001': 'يمين'} |

## `account.resequence.wizard`

**`ordering`** — {'en_US': 'Ordering', 'tr_TR': 'Sipariş verme', 'ar_001': 'الطلب'}

| Value | Label |
|---|---|
| `keep` | {'en_US': 'Keep current order', 'tr_TR': 'Mevcut sıralamayı koru', 'ar_001': 'إبقاء الترتيب الحالي'} |
| `date` | {'en_US': 'Reorder by accounting date', 'tr_TR': 'Muhasebe tarihine göre yeniden sırala', 'ar_001': 'إعادة الطلب حسب تاريخ المحاسبة'} |

## `account.sale.closing`

**`frequency`** — {'en_US': 'Closing Type'}

| Value | Label |
|---|---|
| `daily` | {'en_US': 'Daily'} |
| `monthly` | {'en_US': 'Monthly'} |
| `annually` | {'en_US': 'Annual'} |

## `account.tax`

**`amount_type`** — {'en_US': 'Tax Computation', 'tr_TR': 'Vergi Hesaplaması', 'ar_001': 'حساب الضريبة'}

| Value | Label |
|---|---|
| `group` | {'en_US': 'Group of Taxes', 'tr_TR': 'Vergi Grupları', 'ar_001': 'مجموعة من الضرائب'} |
| `fixed` | {'en_US': 'Fixed', 'tr_TR': 'Sabit', 'ar_001': 'ثابت'} |
| `percent` | {'en_US': 'Percentage', 'tr_TR': 'Yüzde', 'ar_001': 'النسبة'} |
| `division` | {'en_US': 'Percentage Tax Included', 'tr_TR': 'Yüzde Vergi Dahil', 'ar_001': 'النسبة شاملة الضريبة'} |
| `code` | {'en_US': 'Custom Formula', 'tr_TR': 'Özel Formül', 'ar_001': 'معادلة مخصصة'} |

**`l10n_ar_tax_type`** — {'en_US': 'WTH Tax'}

| Value | Label |
|---|---|
| `earnings` | {'en_US': 'Earnings'} |
| `earnings_scale` | {'en_US': 'Earnings Scale'} |
| `iibb_untaxed` | {'en_US': 'IIBB Untaxed'} |
| `iibb_total` | {'en_US': 'IIBB Total Amount'} |

**`l10n_ar_type_tax_use`** — {'en_US': 'Argentina Tax Type'} (not stored)

| Value | Label |
|---|---|
| `sale` | {'en_US': 'Sales'} |
| `purchase` | {'en_US': 'Purchases'} |
| `none` | {'en_US': 'Other'} |
| `supplier` | {'en_US': 'Vendor Payment Withholding'} |
| `customer` | {'en_US': 'Customer Payment Withholding'} |

**`l10n_ar_withholding_payment_type`** — {'en_US': 'Argentina Withholding Payment Type'}

| Value | Label |
|---|---|
| `supplier` | {'en_US': 'Vendor Payment'} |
| `customer` | {'en_US': 'Customer Payment'} |

**`l10n_ee_kmd_inf_code`** — {'en_US': 'KMD INF Code'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Sale KMS §41/42'} |
| `2` | {'en_US': 'Sale KMS §41^1'} |
| `11` | {'en_US': 'Purchase KMS §29(4)/30/32'} |
| `12` | {'en_US': 'Purchase KMS §41^1'} |

**`l10n_eg_eta_code`** — {'en_US': 'ETA Code (Egypt)'}

| Value | Label |
|---|---|
| `t1_v001` | {'en_US': 'T1 - V001 - Export'} |
| `t1_v002` | {'en_US': 'T1 - V002 - Export to free areas and other areas'} |
| `t1_v003` | {'en_US': 'T1 - V003 - Exempted good or service'} |
| `t1_v004` | {'en_US': 'T1 - V004 - A non-taxable good or service'} |
| `t1_v005` | {'en_US': 'T1 - V005 - Exemptions for diplomats, consulates and embassies'} |
| `t1_v006` | {'en_US': 'T1 - V006 - Defence and National security Exemptions'} |
| `t1_v007` | {'en_US': 'T1 - V007 - Agreements exemptions'} |
| `t1_v008` | {'en_US': 'T1 - V008 - Special Exemption and other reasons'} |
| `t1_v009` | {'en_US': 'T1 - V009 - General Item sales'} |
| `t1_v010` | {'en_US': 'T1 - V010 - Other Rates'} |
| `t2_tbl01` | {'en_US': 'T2 - Tbl01 - Table tax (percentage)'} |
| `t3_tbl02` | {'en_US': 'T3 - Tbl02 - Table tax (Fixed Amount)'} |
| `t4_w001` | {'en_US': 'T4 - W001 - Contracting'} |
| `t4_w002` | {'en_US': 'T4 - W002 - Supplies'} |
| `t4_w003` | {'en_US': 'T4 - W003 - Purchases'} |
| `t4_w004` | {'en_US': 'T4 - W004 - Services'} |
| `t4_w005` | {'en_US': 'T4 - W005 - Sums paid by the cooperative societies for car transportation to their members'} |
| `t4_w006` | {'en_US': 'T4 - W006 - Commission agency & brokerage'} |
| `t4_w007` | {'en_US': 'T4 - W007 - Discounts & grants & additional exceptional incentives (smoke, cement companies)'} |
| `t4_w008` | {'en_US': 'T4 - W008 - All discounts & grants & commissions (petroleum, telecommunications, and other)'} |
| `t4_w009` | {'en_US': 'T4 - W009 - Supporting export subsidies'} |
| `t4_w010` | {'en_US': 'T4 - W010 - Professional fees'} |
| `t4_w011` | {'en_US': 'T4 - W011 - Commission & brokerage _A_57'} |
| `t4_w012` | {'en_US': 'T4 - W012 - Hospitals collecting from doctors'} |
| `t4_w013` | {'en_US': 'T4 - W013 - Royalties'} |
| `t4_w014` | {'en_US': 'T4 - W014 - Customs clearance'} |
| `t4_w015` | {'en_US': 'T4 - W015 - Exemption'} |
| `t4_w016` | {'en_US': 'T4 - W016 - advance payments'} |
| `t5_st01` | {'en_US': 'T5 - ST01 - Stamping tax (percentage)'} |
| `t6_st02` | {'en_US': 'T6 - ST02 - Stamping Tax (amount)'} |
| `t7_ent01` | {'en_US': 'T7 - Ent01 - Entertainment tax (rate)'} |
| `t7_ent02` | {'en_US': 'T7 - Ent02 - Entertainment tax (amount)'} |
| `t8_rd01` | {'en_US': 'T8 - RD01 - Resource development fee (rate)'} |
| `t8_rd02` | {'en_US': 'T8 - RD02 - Resource development fee (amount)'} |
| `t9_sc01` | {'en_US': 'T9 - SC01 - Service charges (rate)'} |
| `t9_sc02` | {'en_US': 'T9 - SC02 - Service charges (amount)'} |
| `t10_mn01` | {'en_US': 'T10 - Mn01 - Municipality Fees (rate)'} |
| `t10_mn02` | {'en_US': 'T10 - Mn02 - Municipality Fees (amount)'} |
| `t11_mi01` | {'en_US': 'T11 - MI01 - Medical insurance fee (rate)'} |
| `t11_mi02` | {'en_US': 'T11 - MI02 - Medical insurance fee (amount)'} |
| `t12_of01` | {'en_US': 'T12 - OF01 - Other fees (rate)'} |
| `t12_of02` | {'en_US': 'T12 - OF02 - Other fees (amount)'} |
| `t13_st03` | {'en_US': 'T13 - ST03 - Stamping tax (percentage)'} |
| `t14_st04` | {'en_US': 'T14 - ST04 - Stamping Tax (amount)'} |
| `t15_ent03` | {'en_US': 'T15 - Ent03 - Entertainment tax (rate)'} |
| `t15_ent04` | {'en_US': 'T15 - Ent04 - Entertainment tax (amount)'} |
| `t16_rd03` | {'en_US': 'T16 - RD03 - Resource development fee (rate)'} |
| `t16_rd04` | {'en_US': 'T16 - RD04 - Resource development fee (amount)'} |
| `t17_sc03` | {'en_US': 'T17 - SC03 - Service charges (rate)'} |
| `t17_sc04` | {'en_US': 'T17 - SC04 - Service charges (amount)'} |
| `t18_mn03` | {'en_US': 'T18 - Mn03 - Municipality Fees (rate)'} |
| `t18_mn04` | {'en_US': 'T18 - Mn04 - Municipality Fees (amount)'} |
| `t19_mi03` | {'en_US': 'T19 - MI03 - Medical insurance fee (rate)'} |
| `t19_mi04` | {'en_US': 'T19 - MI04 - Medical insurance fee (amount)'} |
| `t20_of03` | {'en_US': 'T20 - OF03 - Other fees (rate)'} |
| `t20_of04` | {'en_US': 'T20 - OF04 - Other fees (amount)'} |

**`l10n_es_applicability`** — {'en_US': 'Applicability (Spain)'}

| Value | Label |
|---|---|
| `01` | {'en_US': 'VAT'} |
| `02` | {'en_US': 'IPSI'} |
| `03` | {'en_US': 'IGIC'} |

**`l10n_es_edi_facturae_tax_type`** — {'en_US': 'Spanish Facturae EDI Tax Type'}

| Value | Label |
|---|---|
| `01` | {'en_US': 'Value-Added Tax'} |
| `02` | {'en_US': 'Taxes on production, services and imports in Ceuta and Melilla'} |
| `03` | {'en_US': 'IGIC: Canaries General Indirect Tax'} |
| `04` | {'en_US': 'IRPF: Personal Income Tax'} |
| `05` | {'en_US': 'Other'} |
| `06` | {'en_US': 'ITPAJD: Tax on wealth transfers and stamp duty'} |
| `07` | {'en_US': 'IE: Excise duties and consumption taxes'} |
| `08` | {'en_US': 'RA: Customs duties'} |
| `09` | {'en_US': 'IGTECM: Sales tax in Ceuta and Melilla'} |
| `10` | {'en_US': 'IECDPCAC: Excise duties on oil derivates in Canaries'} |
| `11` | {'en_US': 'IIIMAB: Tax on premises that affect the environment in the Balearic Islands'} |
| `12` | {'en_US': 'ICIO: Tax on construction, installation and works'} |
| `13` | {'en_US': 'IMVDN: Local tax on unoccupied homes in Navarre'} |
| `14` | {'en_US': 'IMSN: Local tax on building plots in Navarre'} |
| `15` | {'en_US': 'IMGSN: Local sumptuary tax in Navarre'} |
| `16` | {'en_US': 'IMPN: Local tax on advertising in Navarre'} |
| `17` | {'en_US': 'REIVA: Special VAT for travel agencies'} |
| `18` | {'en_US': 'REIGIC: Special IGIC: for travel agencies'} |
| `19` | {'en_US': 'REIPSI: Special IPSI for travel agencies'} |
| `20` | {'en_US': 'IPS: Insurance premiums Tax'} |
| `21` | {'en_US': 'SWUA: Surcharge for Winding Up Activity'} |
| `22` | {'en_US': 'IVPEE: Tax on the value of electricity generation'} |
| `23` | {'en_US': 'Tax on the production of spent nuclear fuel and radioactive waste from the generation of nuclear electric power'} |
| `24` | {'en_US': 'Tax on the storage of spent nuclear energy and radioactive waste in centralised facilities'} |
| `25` | {'en_US': 'IDEC: Tax on bank deposits'} |
| `26` | {'en_US': 'Excise duty applied to manufactured tobacco in Canaries'} |
| `27` | {'en_US': 'IGFEI: Tax on Fluorinated Greenhouse Gases'} |
| `28` | {'en_US': 'IRNR: Non-resident Income Tax'} |
| `29` | {'en_US': 'Corporation Tax'} |

**`l10n_es_exempt_reason`** — {'en_US': 'Exempt Reason (Spain)'}

| Value | Label |
|---|---|
| `E1` | {'en_US': 'Art. 20'} |
| `E2` | {'en_US': 'Art. 21'} |
| `E3` | {'en_US': 'Art. 22'} |
| `E4` | {'en_US': 'Art. 23 y 24'} |
| `E5` | {'en_US': 'Art. 25'} |
| `E6` | {'en_US': 'Otros'} |

**`l10n_es_type`** — {'en_US': 'Tax Type (Spain)'}

| Value | Label |
|---|---|
| `exento` | {'en_US': 'Exento'} |
| `sujeto` | {'en_US': 'Sujeto'} |
| `sujeto_agricultura` | {'en_US': 'Sujeto Agricultura'} |
| `sujeto_isp` | {'en_US': 'Sujeto ISP'} |
| `no_sujeto` | {'en_US': 'No Sujeto'} |
| `no_sujeto_loc` | {'en_US': 'No Sujeto por reglas de Localization'} |
| `no_deducible` | {'en_US': 'No Deducible'} |
| `retencion` | {'en_US': 'Retencion'} |
| `recargo` | {'en_US': 'Recargo de Equivalencia'} |
| `dua` | {'en_US': 'DUA'} |
| `ignore` | {'en_US': 'Ignore even the base amount'} |

**`l10n_gr_edi_default_tax_exemption_category`** — {'en_US': 'Default Tax Exemption Category'}

| Value | Label |
|---|---|
| `1` | {'en_US': '1 - Without VAT - article 3 of the VAT code'} |
| `2` | {'en_US': '2 - Without VAT - article 5 of the VAT code'} |
| `3` | {'en_US': '3 - Without VAT - article 13 of the VAT code'} |
| `4` | {'en_US': '4 - Without VAT - article 14 of the VAT code'} |
| `5` | {'en_US': '5 - Without VAT - article 16 of the VAT code'} |
| `6` | {'en_US': '6 - Without VAT - article 19 of the VAT code'} |
| `7` | {'en_US': '7 - Without VAT - article 22 of the VAT code'} |
| `8` | {'en_US': '8 - Without VAT - article 24 of the VAT code'} |
| `9` | {'en_US': '9 - Without VAT - article 25 of the VAT code'} |
| `10` | {'en_US': '10 - Without VAT - article 26 of the VAT code'} |
| `11` | {'en_US': '11 - Without VAT - article 27 of the VAT code'} |
| `12` | {'en_US': '12 - Without VAT - article 27 - Seagoing Vessels of the VAT code'} |
| `13` | {'en_US': '13 - Without VAT - article 27.1.γ - Seagoing Vessels of the VAT code'} |
| `14` | {'en_US': '14 - Without VAT - article 28 of the VAT code'} |
| `15` | {'en_US': '15 - Without VAT - article 39 of the VAT code'} |
| `16` | {'en_US': '16 - Without VAT - article 39a of the VAT code'} |
| `17` | {'en_US': '17 - Without VAT - article 40 of the VAT code'} |
| `18` | {'en_US': '18 - Without VAT - article 41 of the VAT code'} |
| `19` | {'en_US': '19 - Without VAT - article 47 of the VAT code'} |
| `20` | {'en_US': '20 - VAT included - article 43 of the VAT code'} |
| `21` | {'en_US': '21 - VAT included - article 44 of the VAT code'} |
| `22` | {'en_US': '22 - VAT included - article 45 of the VAT code'} |
| `23` | {'en_US': '23 - VAT included - article 46 of the VAT code'} |
| `24` | {'en_US': '24 - Without VAT - article 6 of the VAT code'} |
| `25` | {'en_US': '25 - Without VAT - ΠΟΛ.1029 / 1995'} |
| `26` | {'en_US': '26 - Without VAT - ΠΟΛ.1167 / 2015'} |
| `27` | {'en_US': '27 - Without VAT – Other VAT exceptions'} |
| `28` | {'en_US': '28 - Without VAT - Article 24 (b)(1) of the VAT Code(Tax Free)'} |
| `29` | {'en_US': '29 - Without VAT - Article 47 b of the VAT Code(OSS non - EU scheme)'} |
| `30` | {'en_US': '30 - Without VAT - Article 47 c of the VAT Code(OSS EU scheme)'} |
| `31` | {'en_US': '31 - Excluding VAT - Article 47 d of the VAT Code(IOSS)'} |

**`l10n_hu_tax_type`** — {'en_US': 'NAV VAT Tax Type'}

| Value | Label |
|---|---|
| `VAT` | {'en_US': 'Normal VAT (percent based)'} |
| `AAM` | {'en_US': 'AAM - Personal tax exemption'} |
| `TAM` | {'en_US': 'TAM - tax-exempt activity or tax-exempt due to being in public interest or special in nature'} |
| `KBAET` | {'en_US': 'KBAET - intra-Community exempt supply, without new means of transport'} |
| `KBAUK` | {'en_US': 'KBAUK - tax-exempt, intra-Community sales of new means of transport'} |
| `EAM` | {'en_US': 'EAM - tax-exempt, extra-Community sales of goods (export of goods to a non-EU country)'} |
| `NAM` | {'en_US': 'NAM - tax-exempt on other grounds related to international transactions'} |
| `ATK` | {'en_US': 'ATK - Outside the scope of VAT'} |
| `EUFAD37` | {'en_US': 'EUFAD37 - Based on section 37 of the VAT Act, a reverse charge transaction carried out in another Member State'} |
| `EUFADE` | {'en_US': 'EUFADE - Reverse charge transaction carried out in another Member State, not subject to Section 37 of the VAT Act'} |
| `EUE` | {'en_US': 'EUE - Non-reverse charge transaction performed in another Member State'} |
| `HO` | {'en_US': 'HO - Transaction in a third country'} |
| `DOMESTIC_REVERSE` | {'en_US': 'DOMESTIC_REVERSE - Domestic reverse-charge regime'} |
| `TRAVEL_AGENCY` | {'en_US': 'TRAVEL_AGENCY - Profit-margin based regime for travel agencies'} |
| `SECOND_HAND` | {'en_US': 'SECOND_HAND - Profit-margin based regime for second-hand sales'} |
| `ARTWORK` | {'en_US': 'ARTWORK - Profit-margin based regime for artwork sales'} |
| `ANTIQUES` | {'en_US': 'ANTIQUES - Profit-margin based regime for antique sales'} |
| `REFUNDABLE_VAT` | {'en_US': 'REFUNDABLE_VAT - VAT incurred under sections 11 or 14, without an agreement from the beneficiary to reimburse VAT'} |
| `NONREFUNDABLE_VAT` | {'en_US': 'NONREFUNDABLE_VAT - VAT incurred under sections 11 or 14, with an agreement from the beneficiary to reimburse VAT'} |
| `NO_VAT` | {'en_US': 'VAT not applicable pursuant to section 17 of the VAT Act'} |

**`l10n_in_gst_tax_type`** — {'en_US': 'L10N In Gst Tax Type'} (not stored)

| Value | Label |
|---|---|
| `igst` | {'en_US': 'igst'} |
| `cgst` | {'en_US': 'cgst'} |
| `sgst` | {'en_US': 'sgst'} |
| `cess` | {'en_US': 'cess'} |

**`l10n_in_tax_type`** — {'en_US': 'Indian Tax Type'}

| Value | Label |
|---|---|
| `gst` | {'en_US': 'GST'} |
| `tcs` | {'en_US': 'TCS'} |
| `tds_sale` | {'en_US': 'TDS Sale'} |
| `tds_purchase` | {'en_US': 'TDS Purchase'} |
| `nil_rated` | {'en_US': 'Nil Rated'} |
| `exempt` | {'en_US': 'Exempt'} |
| `non_gst` | {'en_US': 'Non-GST'} |

**`l10n_it_exempt_reason`** — {'en_US': 'Exoneration'}

| Value | Label |
|---|---|
| `N1` | {'en_US': '[N1] Escluse ex art. 15'} |
| `N2.1` | {'en_US': '[N2.1] Non soggette ad IVA ai sensi degli artt. Da 7 a 7-septies del DPR 633/72'} |
| `N2.2` | {'en_US': '[N2.2] Non soggette - altri casi'} |
| `N3.1` | {'en_US': '[N3.1] Non imponibili - esportazioni'} |
| `N3.2` | {'en_US': '[N3.2] Non imponibili - cessioni intracomunitarie'} |
| `N3.3` | {'en_US': '[N3.3] Non imponibili - cessioni verso San Marino'} |
| `N3.4` | {'en_US': "[N3.4] Non imponibili - operazioni assimilate alle cessioni all'esportazione"} |
| `N3.5` | {'en_US': "[N3.5] Non imponibili - a seguito di dichiarazioni d'intento"} |
| `N3.6` | {'en_US': '[N3.6] Non imponibili - altre operazioni che non concorrono alla formazione del plafond'} |
| `N4` | {'en_US': '[N4] Esenti'} |
| `N5` | {'en_US': '[N5] Regime del margine / IVA non esposta in fattura'} |
| `N6.1` | {'en_US': '[N6.1] Inversione contabile - cessione di rottami e altri materiali di recupero'} |
| `N6.2` | {'en_US': '[N6.2] Inversione contabile - cessione di oro e argento puro'} |
| `N6.3` | {'en_US': '[N6.3] Inversione contabile - subappalto nel settore edile'} |
| `N6.4` | {'en_US': '[N6.4] Inversione contabile - cessione di fabbricati'} |
| `N6.5` | {'en_US': '[N6.5] Inversione contabile - cessione di telefoni cellulari'} |
| `N6.6` | {'en_US': '[N6.6] Inversione contabile - cessione di prodotti elettronici'} |
| `N6.7` | {'en_US': '[N6.7] Inversione contabile - prestazioni comparto edile esettori connessi'} |
| `N6.8` | {'en_US': '[N6.8] Inversione contabile - operazioni settore energetico'} |
| `N6.9` | {'en_US': '[N6.9] Inversione contabile - altri casi'} |
| `N7` | {'en_US': '[N7] IVA assolta in altro stato UE (prestazione di servizi di telecomunicazioni, tele-radiodiffusione ed elettronici ex art. 7-octies, comma 1 lett. a, b, art. 74-sexies DPR 633/72)'} |

**`l10n_it_pension_fund_type`** — {'en_US': 'Pension fund type (Italy)'}

| Value | Label |
|---|---|
| `TC01` | {'en_US': '[TC01] National pension fund for lawyers and solicitors'} |
| `TC02` | {'en_US': '[TC02] Pension fund for accountants with a degree'} |
| `TC03` | {'en_US': '[TC03] Pension fund for surveyors'} |
| `TC04` | {'en_US': '[TC04] National pension fund for associated engineers and architects'} |
| `TC05` | {'en_US': '[TC05] National pension fund for notaries'} |
| `TC06` | {'en_US': '[TC06] Pension fund for accountants without a degree and commercial experts'} |
| `TC07` | {'en_US': '[TC07] ENASARCO pension fund for sales agents'} |
| `TC08` | {'en_US': '[TC08] ENPACL pension fund for labor consultants'} |
| `TC09` | {'en_US': '[TC09] ENPAM pension fund for doctors'} |
| `TC10` | {'en_US': '[TC10] ENPAF pension fund for chemists'} |
| `TC11` | {'en_US': '[TC11] ENPAV pension fund for veterinaries'} |
| `TC12` | {'en_US': '[TC12] ENPAIA pension fund for people working in agriculture'} |
| `TC13` | {'en_US': '[TC13] Pension fund for employees in delivery and marine agencies'} |
| `TC14` | {'en_US': '[TC14] INPGI pension fund for journalists'} |
| `TC15` | {'en_US': '[TC15] ONAOSI fund for sanitary orphans'} |
| `TC16` | {'en_US': '[TC16] CASAGIT Additional pension fund for journalists'} |
| `TC17` | {'en_US': '[TC17] EPPI pension fund for industrial experts'} |
| `TC18` | {'en_US': '[TC18] EPAP pension fund'} |
| `TC19` | {'en_US': '[TC19] ENPAB national pension fund for biologists'} |
| `TC20` | {'en_US': '[TC20] ENPAPI national pension fund for nurses'} |
| `TC21` | {'en_US': '[TC21] ENPAP national pension fund for psychologists'} |
| `TC22` | {'en_US': '[TC22] INPS national pension fund'} |

**`l10n_it_withholding_reason`** — {'en_US': 'Withholding tax reason (Italy)'}

| Value | Label |
|---|---|
| `A` | {'en_US': '[A] Autonomous work in the fields of art or profession'} |
| `B` | {'en_US': '[B] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science'} |
| `C` | {'en_US': '[C] Income from work as part of association groups or other cooperation determined by contracts'} |
| `D` | {'en_US': '[D] Income as partner or founder of a corporation'} |
| `E` | {'en_US': '[E] Income from client-related bill protests made by town secretaries'} |
| `G` | {'en_US': '[G] Compensation for the end of a professional sport career'} |
| `H` | {'en_US': '[H] Compensation for the end of a societary career (excluded those earned before 31.12.2003) and already taxed'} |
| `I` | {'en_US': '[I] Compensation for the end of a notary career'} |
| `K` | {'en_US': '[K] Civil service checks, ref art. 16 D.lgs. n.40 6/03/2017'} |
| `L` | {'en_US': '[L] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science, but not made by the author/inventor'} |
| `L1` | {'en_US': '[L1] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science, from someone who actively bought the use rights'} |
| `M` | {'en_US': "[M] Autonomous work which isn't part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow"} |
| `M1` | {'en_US': '[M1] Incomes due for an obligation to act, not to act, or to allow'} |
| `M2` | {'en_US': '[M2] Autonomous work which isn\'t part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow - that require being registered to the "Gestione separata"'} |
| `N` | {'en_US': '[N] Compensation for travel, expenses, prizes, or other compensations for amateur sport activities'} |
| `O` | {'en_US': '[O] Autonomous work which isn\'t part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow - that do not require being registered to the "Gestione separata"'} |
| `O1` | {'en_US': '[O1] Incomes due for an obligation to act, not to act, or to allow - that do not require being registered to the "Gestione Separata"'} |
| `P` | {'en_US': '[P] Compensation for people residing abroad for continuous use or concession of industrial machinery, commercial or scientific tools that are on the Italian soil'} |
| `Q` | {'en_US': "[Q] Provisions for exclusive agents or sales representatives' work"} |
| `R` | {'en_US': "[R] Provisions for non-exclusive agents or sales representatives' work"} |
| `S` | {'en_US': '[S] Provisions for commissioner work'} |
| `T` | {'en_US': '[T] Provisions for mediator work'} |
| `U` | {'en_US': '[U] Provisions for procurer work'} |
| `V` | {'en_US': '[V] Provisions for door-to-door sales persons and newspaper selling in kiosks'} |
| `V1` | {'en_US': '[V1] Income from unusual commercial activities (such as provisions for occasional work or sales representative, mediator, procurer)'} |
| `V2` | {'en_US': '[V2] Income from unusual work activities from door-to-door sales representatives'} |
| `W` | {'en_US': '[W] Income from tenders subject to Art. 25-ter of Presidential Decree 600/1973'} |
| `X` | {'en_US': '[X] Income from 2014 for foreign companies or institutions subject to law art. 26-quater, c. 1, lett. a) and b) D.P.R. 600/1973'} |
| `Y` | {'en_US': '[Y] Income from 1.01.2005 to 26.07.2005 from companies or institutions not included in the description above'} |
| `Z` | {'en_US': '[Z] Deprecated'} |
| `ZO` | {'en_US': '[ZO] Other reason'} |

**`l10n_it_withholding_type`** — {'en_US': 'Withholding tax type (Italy)'}

| Value | Label |
|---|---|
| `RT01` | {'en_US': '[RT01] Withholding for persons'} |
| `RT02` | {'en_US': '[RT02] Withholding for personal businesses'} |
| `RT03` | {'en_US': '[RT03] INPS Pension fund contribution'} |
| `RT04` | {'en_US': '[RT04] ENASARCO pension fund contribution'} |
| `RT05` | {'en_US': '[RT05] ENPAM pension fund contribution'} |
| `RT06` | {'en_US': '[RT06] Other pension fund contribution'} |

**`l10n_mx_factor_type`** — {'en_US': 'Factor Type'}

| Value | Label |
|---|---|
| `Tasa` | {'en_US': 'Tasa'} |
| `Cuota` | {'en_US': 'Cuota'} |
| `Exento` | {'en_US': 'Exento'} |

**`l10n_mx_tax_type`** — {'en_US': 'SAT Tax Type'}

| Value | Label |
|---|---|
| `isr` | {'en_US': 'ISR'} |
| `iva` | {'en_US': 'IVA'} |
| `ieps` | {'en_US': 'IEPS'} |
| `local` | {'en_US': 'Local'} |

**`l10n_my_tax_type`** — {'en_US': 'Malaysian Tax Type'}

| Value | Label |
|---|---|
| `01` | {'en_US': 'Sales Tax'} |
| `02` | {'en_US': 'Service Tax'} |
| `03` | {'en_US': 'Tourism Tax'} |
| `04` | {'en_US': 'High-Value Goods Tax'} |
| `05` | {'en_US': 'Sales Tax on Low Value Goods'} |
| `06` | {'en_US': 'Not Applicable'} |
| `E` | {'en_US': 'Tax exemption (where applicable)'} |

**`l10n_pe_edi_isc_type`** — {'en_US': 'ISC Type'}

| Value | Label |
|---|---|
| `01` | {'en_US': 'System to value'} |
| `02` | {'en_US': 'Application of the Fixed Amount'} |
| `03` | {'en_US': 'Retail Price System'} |

**`l10n_pe_edi_tax_code`** — {'en_US': 'Code'}

| Value | Label |
|---|---|
| `1000` | {'en_US': 'IGV - General Sales Tax'} |
| `1016` | {'en_US': 'IVAP - Tax on Sale Paddy Rice'} |
| `2000` | {'en_US': 'ISC - Selective Excise Tax'} |
| `7152` | {'en_US': 'ICBPER - Plastic bag tax'} |
| `9995` | {'en_US': 'EXP - Exportation'} |
| `9996` | {'en_US': 'GRA - Free'} |
| `9997` | {'en_US': 'EXO - Exonerated'} |
| `9998` | {'en_US': 'INA - Unaffected'} |
| `9999` | {'en_US': 'OTHERS - Other taxes'} |

**`l10n_pe_edi_unece_category`** — {'en_US': 'UNECE Code'}

| Value | Label |
|---|---|
| `E` | {'en_US': 'Exempt from tax'} |
| `G` | {'en_US': 'Free export item, tax not charged'} |
| `O` | {'en_US': 'Services outside scope of tax'} |
| `S` | {'en_US': 'Standard rate'} |
| `Z` | {'en_US': 'Zero rated goods'} |

**`l10n_sa_exemption_reason_code`** — {'en_US': 'Exemption Reason Code', 'ar_001': 'كود سبب الإعفاء'}

| Value | Label |
|---|---|
| `VATEX-SA-29` | {'en_US': 'VATEX-SA-29 Financial services mentioned in Article 29 of the VAT Regulations.', 'ar_001': 'VATEX-SA-29 الخدمات المالية المذكورة في القانون 29 في لوائح ضريبة القيمة المضافة.'} |
| `VATEX-SA-29-7` | {'en_US': 'VATEX-SA-29-7 Life insurance services mentioned in Article 29 of the VAT Regulations.'} |
| `VATEX-SA-30` | {'en_US': 'VATEX-SA-30 Real estate transactions mentioned in Article 30 of the VAT Regulations.', 'ar_001': 'VATEX-SA-30 المعاملات العقارية المذكورة في القانون 30 في لوائح ضريبة القيمة المضافة.'} |
| `VATEX-SA-32` | {'en_US': 'VATEX-SA-32 Export of goods.', 'ar_001': 'VATEX-SA-32 تصدير البضائع.'} |
| `VATEX-SA-33` | {'en_US': 'VATEX-SA-33 Export of Services.', 'ar_001': 'VATEX-SA-33 تصدير الخدمات.'} |
| `VATEX-SA-34-1` | {'en_US': 'VATEX-SA-34-1 The international transport of Goods.', 'ar_001': 'VATEX-SA-34-1 الشحن الدولي للبضائع.'} |
| `VATEX-SA-34-2` | {'en_US': 'VATEX-SA-34-2 The international transport of Passengers.', 'ar_001': 'VATEX-SA-34-2 المواصلات الدولية للركاب.'} |
| `VATEX-SA-34-3` | {'en_US': 'VATEX-SA-34-3 Services directly connected and incidental to a Supply of international passenger transport.', 'ar_001': 'VATEX-SA-34-3 الخدمات المتصلة مباشرة وعرضاً بوسيلة مواصلات الركاب الدولية.'} |
| `VATEX-SA-34-4` | {'en_US': 'VATEX-SA-34-4 Supply of a qualifying means of transport.', 'ar_001': 'VATEX-SA-34-4 التزويد بوسائل نقل مؤهلة.'} |
| `VATEX-SA-34-5` | {'en_US': 'VATEX-SA-34-5 Any services relating to Goods or passenger transportation, as defined in article twenty five of these Regulations.', 'ar_001': 'VATEX-SA-34-5 أي خدمة متعلقة بنقل الركاب أو البضائع، كما هو محدد في القانون خمسة وعشرين في تلك اللوائح.'} |
| `VATEX-SA-35` | {'en_US': 'VATEX-SA-35 Medicines and medical equipment.', 'ar_001': 'VATEX-SA-35 الأدوية والمعدات الطبية.'} |
| `VATEX-SA-36` | {'en_US': 'VATEX-SA-36 Qualifying metals.', 'ar_001': 'VATEX-SA-36 المعادن المؤهلة.'} |
| `VATEX-SA-EDU` | {'en_US': 'VATEX-SA-EDU Private education to citizen.', 'ar_001': 'VATEX-SA-EDU تعليم خاص للمواطن.'} |
| `VATEX-SA-HEA` | {'en_US': 'VATEX-SA-HEA Private healthcare to citizen.', 'ar_001': 'VATEX-SA-HEA رعاية صحية خاصة للمواطن.'} |
| `VATEX-SA-OOS` | {'en_US': 'VATEX-SA-OOS Not subject to VAT.', 'ar_001': 'VATEX-SA-OOS غير خاضعة لضريبة القيمة المضافة. '} |

**`l10n_tw_edi_special_tax_type`** — {'en_US': 'Ecpay Special Tax Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Saloons and tea rooms, coffee shops and bars offering companionship services: Tax rate is 25%'} |
| `2` | {'en_US': 'Night clubs or restaurants providing entertaining show programs: Tax rate is 15%'} |
| `3` | {'en_US': 'Banking businesses, insurance businesses, trust investment businesses, securities businesses, futures businesses, commercial paper businesses and pawn-broking businesses: Tax rate is 2%'} |
| `4` | {'en_US': 'The sales amounts from reinsurance premiums shall be taxed at 1%'} |
| `5` | {'en_US': 'Banking businesses, insurance businesses, trust investment businesses, securities businesses, futures businesses, commercial paper businesses and pawn-broking businesses: Tax rate is 5%'} |
| `6` | {'en_US': 'Core business revenues from the banking and insurance business of the banking and insurance industries (Applicable to sales after July 2014): Tax rate is 5%'} |
| `7` | {'en_US': 'Core business revenues from the banking and insurance business of the banking and insurance industries (Applicable to sales after June 2014): Tax rate is 5%'} |
| `8` | {'en_US': 'Duty free or non-output data'} |

**`l10n_tw_edi_tax_type`** — {'en_US': 'Ecpay Tax Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Taxable'} |
| `2` | {'en_US': 'Zero tax rate'} |
| `3` | {'en_US': 'Duty free'} |
| `4` | {'en_US': 'Taxable (special tax rate)'} |

**`l10n_uy_tax_category`** — {'en_US': 'Tax Category'}

| Value | Label |
|---|---|
| `vat` | {'en_US': 'VAT'} |

**`price_include_override`** — {'en_US': 'Included in Price', 'tr_TR': 'Fiyata Dahil Edilir', 'ar_001': 'مشمول في السعر'}

| Value | Label |
|---|---|
| `tax_included` | {'en_US': 'Tax Included', 'tr_TR': 'Vergi Dahil', 'ar_001': 'شامل الضريبة'} |
| `tax_excluded` | {'en_US': 'Tax Excluded', 'tr_TR': 'Vergi Hariç', 'ar_001': 'غير شامل الضريبة'} |

**`tax_exigibility`** — {'en_US': 'Tax Exigibility', 'tr_TR': 'Vergi Münhasırlığı', 'ar_001': 'الأهلية الضريبية'}

| Value | Label |
|---|---|
| `on_invoice` | {'en_US': 'Based on Invoice', 'tr_TR': 'Faturaya dayalı', 'ar_001': 'بناءً على الفواتير'} |
| `on_payment` | {'en_US': 'Based on Payment', 'tr_TR': 'Ödemeye Dayalı', 'ar_001': 'بناءً على السداد'} |

**`tax_scope`** — {'en_US': 'Tax Scope', 'tr_TR': 'Vergi Kapsamı', 'ar_001': 'نطاق الضريبة'}

| Value | Label |
|---|---|
| `service` | {'en_US': 'Services', 'tr_TR': 'Hizmetler', 'ar_001': 'الخدمات'} |
| `consu` | {'en_US': 'Goods', 'tr_TR': 'Mallar', 'ar_001': 'البضائع'} |
| `merch` | {'en_US': 'Merchandise'} |
| `invest` | {'en_US': 'Investment'} |

**`type_tax_use`** — {'en_US': 'Tax Type', 'tr_TR': 'Vergi Tipi', 'ar_001': 'نوع الضريبة'}

| Value | Label |
|---|---|
| `sale` | {'en_US': 'Sales', 'tr_TR': 'Satışlar', 'ar_001': 'المبيعات'} |
| `purchase` | {'en_US': 'Purchases', 'tr_TR': 'Satınalma', 'ar_001': 'المشتريات'} |
| `none` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |

**`ubl_cii_tax_category_code`** — {'en_US': 'Tax Category Code', 'tr_TR': 'Vergi Kategori Kodu', 'ar_001': 'رمز فئة الضريبة'}

| Value | Label |
|---|---|
| `AE` | {'en_US': 'AE - Vat Reverse Charge', 'tr_TR': 'AE - KDV Tersine Tahsilat', 'ar_001': 'نظام عكس ضريبة القيمة المضافة في الإمارات العربية المتحدة'} |
| `E` | {'en_US': 'E - Exempt from Tax', 'tr_TR': 'M - Vergiden Muaf', 'ar_001': 'E - معفى من الضريبة'} |
| `S` | {'en_US': 'S - Standard rate', 'tr_TR': 'S - Standart oran', 'ar_001': 'س - السعر القياسي'} |
| `Z` | {'en_US': 'Z - Zero rated goods', 'tr_TR': 'S - Sıfır oranlı mallar', 'ar_001': 'Z - Zero rated goods'} |
| `G` | {'en_US': 'G - Free export item, VAT not charged', 'tr_TR': 'G - Serbest ihracat kalemi, KDV uygulanmaz', 'ar_001': 'G - سلعة تصدير مجانية، ضريبة القيمة المضافة غير مفروضة'} |
| `O` | {'en_US': 'O - Services outside scope of tax', 'tr_TR': 'O - Vergi kapsamı dışında kalan hizmetler', 'ar_001': 'O - خدمات خارج نطاق الضريبة'} |
| `K` | {'en_US': 'K - VAT exempt for EEA intra-community supply of goods and services', 'tr_TR': "K - EEA içi mal ve hizmet teslimlerinde KDV'den muaf", 'ar_001': 'K - معفى من ضريبة القيمة المضافة للإمدادات الداخلية للسلع والخدمات داخل المنطقة الاقتصادية الأوروبية'} |
| `L` | {'en_US': 'L - Canary Islands general indirect tax', 'tr_TR': 'L - Kanarya Adaları genel dolaylı vergisi', 'ar_001': 'L - ضريبة القيمة المضافة العامة في جزر الكناري'} |
| `M` | {'en_US': 'M - Tax for production, services and importation in Ceuta and Melilla', 'tr_TR': 'M - Ceuta ve Melilla’da üretim, hizmet ve ithalata ilişkin vergi', 'ar_001': 'م - الضرائب على الإنتاج والخدمات والاستيراد في سبتة ومليلية'} |
| `B` | {'en_US': 'B - Transferred (VAT), In Italy', 'tr_TR': "B - Aktarıldı (KDV), İtalya'da", 'ar_001': 'ب - تم التحويل (ضريبة القيمة المضافة)، في إيطاليا'} |
| `SR` | {'en_US': 'SG - Local supply of goods and services'} |
| `SRCA-S` | {'en_US': 'SG - Customer accounting supply made by the supplier'} |
| `SRCA-C` | {'en_US': 'SG - Customer accounting supply made by the customer on supplier’s behalf'} |
| `SROVR-RS` | {'en_US': 'SG - Supply of remote services accountable by the electronic marketplace under the Overseas Vendor Registration Regime'} |
| `SROVR-LVG` | {'en_US': 'SG - Supply of low-value goods accountable by the redeliverer or electronic marketplace on behalf of third-party suppliers'} |
| `SRRC` | {'en_US': 'SG - Reverse charge regime for Business-to-Business (“B2B”) supplies of imported services'} |
| `SRLVG` | {'en_US': 'SG - Own supply of low-value goods'} |
| `ZR` | {'en_US': 'SG - Supplies involving goods for export/ provision of international services'} |
| `ES33` | {'en_US': 'SG - Specific categories of exempt supplies listed under regulation 33 of the GST (General) Regulations'} |
| `ESN33` | {'en_US': 'SG - Exempt supplies other than those listed under regulation 33 of the GST (General) Regulations'} |
| `DS` | {'en_US': 'SG - Supplies required to be reported pursuant to the GST legislation'} |
| `OS` | {'en_US': 'SG - Supplies outside the scope of the GST Act'} |
| `NG` | {'en_US': 'SG - Supplies from a company which is not registered for GST'} |
| `NA` | {'en_US': 'SG - Taxable supplies where GST need not be charged'} |

**`ubl_cii_tax_exemption_reason_code`** — {'en_US': 'Tax Exemption Reason Code', 'tr_TR': 'Vergi Muafiyet Gerekçe Kodu', 'ar_001': 'رمز سبب الإعفاء الضريبي'}

| Value | Label |
|---|---|
| `VATEX-EU-79-C` | {'en_US': 'VATEX-EU-79-C - Exempt based on article 79, point c of Council Directive 2006/112/EC', 'tr_TR': 'VATEX-EU-79-C - Avrupa Konseyi 2006/112/EC Direktifi Madde 79, bent c uyarınca KDV istisnası', 'ar_001': 'VATEX-EU-79-C - Exempt based on article 79, point c of Council Directive 2006/112/EC'} |
| `VATEX-EU-132` | {'en_US': 'VATEX-EU-132 - Exempt based on article 132 of Council Directive 2006/112/EC', 'tr_TR': 'Lütfen EAS kodunu ve Katılımcı Kimlik kodunu doldurun.', 'ar_001': 'VATEX-EU-132 - معفى بموجب المادة 132 من توجيه المجلس 2006/112/EC'} |
| `VATEX-EU-132-1A` | {'en_US': 'VATEX-EU-132-1A - Exempt based on article 132, section 1 (a) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1A 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (a) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1A - معفى بناءً على المادة 132، القسم 1 (أ) من توجيه المجلس 2006/112/EC'} |
| `VATEX-EU-132-1B` | {'en_US': 'VATEX-EU-132-1B - Exempt based on article 132, section 1 (b) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1B 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (b) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1A - معفى وفقًا للمادة 132، القسم 1 (أ) من توجيه المجلس 2006/112/EC'} |
| `VATEX-EU-132-1C` | {'en_US': 'VATEX-EU-132-1C - Exempt based on article 132, section 1 (c) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1C 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (c) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1C - معفى بناءً على المادة 132، القسم 1 (ج) من توجيه المجلس 2006/112/EC'} |
| `VATEX-EU-132-1D` | {'en_US': 'VATEX-EU-132-1D - Exempt based on article 132, section 1 (d) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1D 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (d) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1D - Exempt based on article 132, section 1 (d) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1E` | {'en_US': 'VATEX-EU-132-1E - Exempt based on article 132, section 1 (e) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1E 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (e) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1E - Exempt based on article 132, section 1 (e) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1F` | {'en_US': 'VATEX-EU-132-1F - Exempt based on article 132, section 1 (f) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1F 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (f) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1F - Exempt based on article 132, section 1 (f) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1G` | {'en_US': 'VATEX-EU-132-1G - Exempt based on article 132, section 1 (g) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1G 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (g) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1G - Exempt based on article 132, section 1 (g) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1H` | {'en_US': 'VATEX-EU-132-1H - Exempt based on article 132, section 1 (h) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1H 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (h) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1H - Exempt based on article 132, section 1 (h) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1I` | {'en_US': 'VATEX-EU-132-1I - Exempt based on article 132, section 1 (i) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1I 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (i) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1I - Exempt based on article 132, section 1 (i) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1J` | {'en_US': 'VATEX-EU-132-1J - Exempt based on article 132, section 1 (j) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1J 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (j) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1J - Exempt based on article 132, section 1 (j) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1K` | {'en_US': 'VATEX-EU-132-1K - Exempt based on article 132, section 1 (k) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1K 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (k) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1K - Exempt based on article 132, section 1 (k) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1L` | {'en_US': 'VATEX-EU-132-1L - Exempt based on article 132, section 1 (l) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1L 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (l) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1L - Exempt based on article 132, section 1 (l) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1M` | {'en_US': 'VATEX-EU-132-1M - Exempt based on article 132, section 1 (m) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1M 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (m) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1M - Exempt based on article 132, section 1 (m) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1N` | {'en_US': 'VATEX-EU-132-1N - Exempt based on article 132, section 1 (n) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1N 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (n) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1N - Exempt based on article 132, section 1 (n) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1O` | {'en_US': 'VATEX-EU-132-1O - Exempt based on article 132, section 1 (o) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1O 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (o) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1O - Exempt based on article 132, section 1 (o) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1P` | {'en_US': 'VATEX-EU-132-1P - Exempt based on article 132, section 1 (p) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1P 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (p) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1P - Exempt based on article 132, section 1 (p) of Council Directive 2006/112/EC'} |
| `VATEX-EU-132-1Q` | {'en_US': 'VATEX-EU-132-1Q - Exempt based on article 132, section 1 (q) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-132-1Q 2006/112/EC sayılı AB Konsey Direktifi'nin 132. maddesi 1 (q) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-132-1Q - Exempt based on article 132, section 1 (q) of Council Directive 2006/112/EC'} |
| `VATEX-EU-135-1` | {'en_US': 'VATEX-EU-135-1 - Exempt based on article 135, section 1 of Council Directive 2006/112/EC', 'ar_001': 'VATEX-EU-135-1 - معفى بموجب المادة 135، الفقرة 1 من توجيه المجلس 2006/112/EC'} |
| `VATEX-EU-143` | {'en_US': 'VATEX-EU-143 - Exempt based on article 143 of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143 - 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesine göre KDV muafiyeti", 'ar_001': 'VATEX-EU-143 - Exempt based on article 143 of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1A` | {'en_US': 'VATEX-EU-143-1A - Exempt based on article 143, section 1 (a) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1A 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (a) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1A - Exempt based on article 143, section 1 (a) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1B` | {'en_US': 'VATEX-EU-143-1B - Exempt based on article 143, section 1 (b) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1B 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (b) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1B - Exempt based on article 143, section 1 (b) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1C` | {'en_US': 'VATEX-EU-143-1C - Exempt based on article 143, section 1 (c) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1C 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (c) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1C - Exempt based on article 143, section 1 (c) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1D` | {'en_US': 'VATEX-EU-143-1D - Exempt based on article 143, section 1 (d) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1D 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (d) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1D - Exempt based on article 143, section 1 (d) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1E` | {'en_US': 'VATEX-EU-143-1E - Exempt based on article 143, section 1 (e) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1E 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (e) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1E - Exempt based on article 143, section 1 (e) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1F` | {'en_US': 'VATEX-EU-143-1F - Exempt based on article 143, section 1 (f) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1F 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (f) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1F - Exempt based on article 143, section 1 (f) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1FA` | {'en_US': 'VATEX-EU-143-1FA - Exempt based on article 143, section 1 (fa) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1FA 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (fa) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1FA - Exempt based on article 143, section 1 (fa) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1G` | {'en_US': 'VATEX-EU-143-1G - Exempt based on article 143, section 1 (g) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1G 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (g) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1G - Exempt based on article 143, section 1 (g) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1H` | {'en_US': 'VATEX-EU-143-1H - Exempt based on article 143, section 1 (h) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1H 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (h) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1H - Exempt based on article 143, section 1 (h) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1I` | {'en_US': 'VATEX-EU-143-1I - Exempt based on article 143, section 1 (i) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1I 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (i) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1I - Exempt based on article 143, section 1 (i) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1J` | {'en_US': 'VATEX-EU-143-1J - Exempt based on article 143, section 1 (j) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1J 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (j) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1J - Exempt based on article 143, section 1 (j) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1K` | {'en_US': 'VATEX-EU-143-1K - Exempt based on article 143, section 1 (k) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1K 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (k) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1K - Exempt based on article 143, section 1 (k) of Council Directive 2006/112/EC'} |
| `VATEX-EU-143-1L` | {'en_US': 'VATEX-EU-143-1L - Exempt based on article 143, section 1 (l) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-143-1L 2006/112/EC sayılı AB Konsey Direktifi'nin 143. maddesi 1 (l) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-143-1L - Exempt based on article 143, section 1 (l) of Council Directive 2006/112/EC'} |
| `VATEX-EU-144` | {'en_US': 'VATEX-EU-144 - Exempt based on article 144 of Council Directive 2006/112/EC', 'ar_001': 'VATEX-EU-144 - Exempt based on article 144 of Council Directive 2006/112/EC'} |
| `VATEX-EU-146-1E` | {'en_US': 'VATEX-EU-146-1E - Exempt based on article 146 section 1 (e) of Council Directive 2006/112/EC', 'ar_001': 'VATEX-EU-146-1E - Exempt based on article 146 section 1 (e) of Council Directive 2006/112/EC'} |
| `VATEX-EU-148` | {'en_US': 'VATEX-EU-148 - Exempt based on article 148 of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148 - 2006/112/EC sayılı AB Konsey Direktifi'nin 148. maddesine göre KDV muafiyeti", 'ar_001': 'VATEX-EU-148 - Exempt based on article 148 of Council Directive 2006/112/EC'} |
| `VATEX-EU-148-A` | {'en_US': 'VATEX-EU-148-A - Exempt based on article 148, section (a) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148-A 2006/112/EC sayılı AB Konsey Direktifi'nin 148. maddesi (a) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-148-A - Exempt based on article 148, section (a) of Council Directive 2006/112/EC'} |
| `VATEX-EU-148-B` | {'en_US': 'VATEX-EU-148-B - Exempt based on article 148, section (b) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148-B 2006/112/EC sayılı AB Konsey Direktifi'nin 148. maddesi (b) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-148-B - Exempt based on article 148, section (b) of Council Directive 2006/112/EC'} |
| `VATEX-EU-148-C` | {'en_US': 'VATEX-EU-148-C - Exempt based on article 148, section (c) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148-C 2006/112/EC sayılı AB Konsey Direktifi'nin 148. maddesi (c) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-148-C - Exempt based on article 148, section (c) of Council Directive 2006/112/EC'} |
| `VATEX-EU-148-D` | {'en_US': 'VATEX-EU-148-D - Exempt based on article 148, section (d) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148-D 2006/112/EC sayılı AB Konsey Direktifi'nin 148. maddesi (d) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-148-D - Exempt based on article 148, section (d) of Council Directive 2006/112/EC'} |
| `VATEX-EU-148-E` | {'en_US': 'VATEX-EU-148-E - Exempt based on article 148, section (e) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148-E 2006/112/EC sayılı AB Konsey Direktifi'nin 148. maddesi (e) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-148-E - Exempt based on article 148, section (e) of Council Directive 2006/112/EC'} |
| `VATEX-EU-148-F` | {'en_US': 'VATEX-EU-148-F - Exempt based on article 148, section (f) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148-F 2006/112/EC sayılı AB Konsey Direktifi'nin 148. maddesi (f) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-148-F - Exempt based on article 148, section (f) of Council Directive 2006/112/EC'} |
| `VATEX-EU-148-G` | {'en_US': 'VATEX-EU-148-G - Exempt based on article 148, section (g) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-148-G sayılı AB Konsey Direktifi'nin 148. mddesine göre KDV muafiyeti", 'ar_001': 'VATEX-EU-148-G - Exempt based on article 148, section (g) of Council Directive 2006/112/EC'} |
| `VATEX-EU-151` | {'en_US': 'VATEX-EU-151 - Exempt based on article 151 of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-151 - 2006/112/EC sayılı AB Konsey Direktifi'nin 151. maddesine göre KDV muafiyeti", 'ar_001': 'VATEX-EU-151 - Exempt based on article 151 of Council Directive 2006/112/EC'} |
| `VATEX-EU-151-1A` | {'en_US': 'VATEX-EU-151-1A - Exempt based on article 151, section 1 (a) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-151-1A 2006/112/EC sayılı AB Konsey Direktifi'nin 151. maddesi 1 (a) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-151-1A - Exempt based on article 151, section 1 (a) of Council Directive 2006/112/EC'} |
| `VATEX-EU-151-1AA` | {'en_US': 'VATEX-EU-151-1AA - Exempt based on article 151, section 1 (aa) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-151-1AA 2006/112/EC sayılı AB Konsey Direktifi'nin 151. maddesi 1 (aa) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-151-1AA - Exempt based on article 151, section 1 (aa) of Council Directive 2006/112/EC'} |
| `VATEX-EU-151-1B` | {'en_US': 'VATEX-EU-151-1B - Exempt based on article 151, section 1 (b) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-151-1B 2006/112/EC sayılı AB Konsey Direktifi'nin 151. maddesi 1 (b) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-151-1B - Exempt based on article 151, section 1 (b) of Council Directive 2006/112/EC'} |
| `VATEX-EU-151-1C` | {'en_US': 'VATEX-EU-151-1C - Exempt based on article 151, section 1 (c) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-151-1C 2006/112/EC sayılı AB Konsey Direktifi'nin 151. maddesi 1 (c) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-151-1C - Exempt based on article 151, section 1 (c) of Council Directive 2006/112/EC'} |
| `VATEX-EU-151-1D` | {'en_US': 'VATEX-EU-151-1D - Exempt based on article 151, section 1 (d) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-151-1D 2006/112/EC sayılı AB Konsey Direktifi'nin 151. maddesi 1 (d) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-151-1D - Exempt based on article 151, section 1 (d) of Council Directive 2006/112/EC'} |
| `VATEX-EU-151-1E` | {'en_US': 'VATEX-EU-151-1E - Exempt based on article 151, section 1 (e) of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-151-1E 2006/112/EC sayılı AB Konsey Direktifi'nin 151. maddesi 1 (e) bendi uyarınca muaf", 'ar_001': 'VATEX-EU-151-1E - Exempt based on article 151, section 1 (e) of Council Directive 2006/112/EC'} |
| `VATEX-EU-153` | {'en_US': 'VATEX-EU-153 - Exempt based on article 153 of Council Directive 2006/112/EC', 'ar_001': 'VATEX-EU-153 - معفى بموجب المادة 153 من توجيه المجلس 2006/112/EC'} |
| `VATEX-EU-159` | {'en_US': 'VATEX-EU-159 - Exempt based on article 159 of Council Directive 2006/112/EC', 'ar_001': 'VATEX-EU-159 - Exempt based on article 159 of Council Directive 2006/112/EC'} |
| `VATEX-EU-309` | {'en_US': 'VATEX-EU-309 - Exempt based on article 309 of Council Directive 2006/112/EC', 'tr_TR': "VATEX-EU-309 - 2006/112/EC sayılı AB Konsey Direktifi'nin 309. maddesine göre KDV muafiyeti", 'ar_001': 'VATEX-EU-309 - Exempt based on article 309 of Council Directive 2006/112/EC'} |
| `VATEX-EU-AE` | {'en_US': 'VATEX-EU-AE - Reverse charge', 'tr_TR': 'VATEX-EU-AE - Tersine vergi yükümlülüğü', 'ar_001': 'VATEX-EU-AE - نظام عكس ضريبة القيمة المضافة'} |
| `VATEX-EU-D` | {'en_US': 'VATEX-EU-D - Intra-Community acquisition from second hand means of transport', 'tr_TR': 'VATEX-EU-D - İkinci el taşıma araçlarının Topluluk içi edinimi (AB içi)', 'ar_001': 'VATEX-EU-D - Intra-Community acquisition from second hand means of transport'} |
| `VATEX-EU-F` | {'en_US': 'VATEX-EU-F - Intra-Community acquisition of second hand goods', 'tr_TR': 'VATEX-EU-F - AB içi ikinci el eşya edinimi', 'ar_001': 'VATEX-EU-F - Intra-Community acquisition of second hand goods'} |
| `VATEX-EU-G` | {'en_US': 'VATEX-EU-G - Export outside the EU', 'tr_TR': 'VATEX-EU-G – AB dışına ihracat', 'ar_001': 'VATEX-EU-G - Export outside the EU'} |
| `VATEX-EU-I` | {'en_US': 'VATEX-EU-I - Intra-Community acquisition of works of art', 'tr_TR': 'VATEX-EU-I - AB içi sanat eseri edinimi', 'ar_001': 'VATEX-EU-I - Intra-Community acquisition of works of art'} |
| `VATEX-EU-IC` | {'en_US': 'VATEX-EU-IC - Intra-Community supply', 'tr_TR': 'VATEX-EU-IC - Topluluk içi teslimat', 'ar_001': 'VATEX-EU-IC - Intra-Community supply'} |
| `VATEX-EU-O` | {'en_US': 'VATEX-EU-O - Not subject to VAT', 'tr_TR': 'VATEX-EU-O – KDV’ye tabi değil', 'ar_001': 'VATEX-EU-O - Not subject to VAT'} |
| `VATEX-EU-J` | {'en_US': 'VATEX-EU-J - Intra-Community acquisition of collectors items and antiques', 'tr_TR': 'VATEX-EU-J - Koleksiyon ürünleri ve antikaların Topluluk içi edinimi', 'ar_001': 'VATEX-EU-J - Intra-Community acquisition of collectors items and antiques'} |
| `VATEX-FR-FRANCHISE` | {'en_US': 'VATEX-FR-FRANCHISE - France domestic VAT franchise in base', 'tr_TR': 'VATEX-FR-FRANCHISE - Fransa içi KDV muafiyeti uygulaması', 'ar_001': 'VATEX-FR-FRANCHISE - France domestic VAT franchise in base'} |
| `VATEX-FR-CNWVAT` | {'en_US': 'VATEX-FR-CNWVAT - France domestic Credit Notes without VAT, due to supplier forfeit of VAT for discount', 'tr_TR': "VATEX-FR-CNWVAT - Fransa içi KDV'siz İade Faturaları (satıcının indirime bağlı KDV feragatinden dolayı).", 'ar_001': 'VATEX-FR-CNWVAT - France domestic Credit Notes without VAT, due to supplier forfeit of VAT for discount'} |
| `VATEX-FR-CGI261-1` | {'en_US': 'VATEX-FR-CGI261-1 - Exempt based on 1 of article 261 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261-1 - Exempt based on 1 of article 261 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261-2` | {'en_US': 'VATEX-FR-CGI261-2 - Exempt based on 2 of article 261 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261-2 - Exempt based on 2 of article 261 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261-3` | {'en_US': 'VATEX-FR-CGI261-3 - Exempt based on 3 of article 261 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261-3 - Exempt based on 3 of article 261 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261-4` | {'en_US': 'VATEX-FR-CGI261-4 - Exempt based on 4 of article 261 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261-4 - Exempt based on 4 of article 261 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261-5` | {'en_US': 'VATEX-FR-CGI261-5 - Exempt based on 5 of article 261 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261-5 - Exempt based on 5 of article 261 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261-7` | {'en_US': 'VATEX-FR-CGI261-7 - Exempt based on 7 of article 261 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261-7 - Exempt based on 7 of article 261 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261-8` | {'en_US': 'VATEX-FR-CGI261-8 - Exempt based on 8 of article 261 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261-8 - Exempt based on 8 of article 261 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261A` | {'en_US': 'VATEX-FR-CGI261A - Exempt based on article 261 A of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261A - Exempt based on article 261 A of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261B` | {'en_US': 'VATEX-FR-CGI261B - Exempt based on article 261 B of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261B - Exempt based on article 261 B of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261C-1` | {'en_US': 'VATEX-FR-CGI261C-1 - Exempt based on 1° of article 261 C of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261C-1 - Exempt based on 1° of article 261 C of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261C-2` | {'en_US': 'VATEX-FR-CGI261C-2 - Exempt based on 2° of article 261 C of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261C-2 - Exempt based on 2° of article 261 C of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261C-3` | {'en_US': 'VATEX-FR-CGI261C-3 - Exempt based on 3° of article 261 C of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261C-3 - Exempt based on 3° of article 261 C of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261D-1` | {'en_US': 'VATEX-FR-CGI261D-1 - Exempt based on 1° of article 261 D of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261D-1 - Exempt based on 1° of article 261 D of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261D-1BIS` | {'en_US': 'VATEX-FR-CGI261D-1BIS - Exempt based on 1°bis of article 261 D of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261D-1BIS - Exempt based on 1°bis of article 261 D of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261D-2` | {'en_US': 'VATEX-FR-CGI261D-2 - Exempt based on 2° of article 261 D of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261D-2 - Exempt based on 2° of article 261 D of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261D-3` | {'en_US': 'VATEX-FR-CGI261D-3 - Exempt based on 3° of article 261 D of the Code Général des Impôts (CGI ; General tax code) Exonération de TVA - Article 261 D-3° du Code Général des Impôts', 'ar_001': 'VATEX-FR-CGI261D-3 - معفى بموجب الفقرة 3 من المادة 261 د من القانون العام للضرائب (CGI) إعفاء من ضريبة القيمة المضافة - المادة 261 د-3 من القانون العام للضرائب'} |
| `VATEX-FR-CGI261D-4` | {'en_US': 'VATEX-FR-CGI261D-4 - Exempt based on 4° of article 261 D of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261D-4 - معفى بموجب المادة 261 دال من القانون العام للضرائب (CGI)'} |
| `VATEX-FR-CGI261E-1` | {'en_US': 'VATEX-FR-CGI261E-1 - Exempt based on 1° of article 261 E of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261E-1 - Exempt based on 1° of article 261 E of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI261E-2` | {'en_US': 'VATEX-FR-CGI261E-2 - Exempt based on 2° of article 261 E of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI261E-2 - Exempt based on 2° of article 261 E of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI277A` | {'en_US': 'VATEX-FR-CGI277A - Exempt based on article 277 A of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI277A - Exempt based on article 277 A of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI275` | {'en_US': 'VATEX-FR-CGI275 - Exempt based on article 275 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI275 - Exempt based on article 275 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-298SEXDECIESA` | {'en_US': 'VATEX-FR-298SEXDECIESA - Exempt based on article 298 sexdecies A of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-298SEXDECIESA - Exempt based on article 298 sexdecies A of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-CGI295` | {'en_US': 'VATEX-FR-CGI295 - Exempt based on article 295 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-CGI295 - Exempt based on article 295 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-AE` | {'en_US': 'VATEX-FR-AE - Exempt based on 2 of article 283 of the Code Général des Impôts (CGI ; General tax code)', 'ar_001': 'VATEX-FR-AE - Exempt based on 2 of article 283 of the Code Général des Impôts (CGI ; General tax code)'} |
| `VATEX-FR-F` | {'en_US': 'VATEX-FR-F - Second-hand sales'} |
| `VATEX-FR-I` | {'en_US': 'VATEX-FR-I - Sales of works of art'} |
| `VATEX-FR-J` | {'en_US': 'VATEX-FR-J - Sales of antiques'} |

## `account.tax.group`

**`l10n_ar_tribute_afip_code`** — {'en_US': 'Tribute ARCA Code'}

| Value | Label |
|---|---|
| `01` | {'en_US': '01 - National Taxes'} |
| `02` | {'en_US': '02 - Provincial Taxes'} |
| `03` | {'en_US': '03 - Municipal Taxes'} |
| `04` | {'en_US': '04 - Internal Taxes'} |
| `06` | {'en_US': '06 - VAT perception'} |
| `07` | {'en_US': '07 - IIBB perception'} |
| `08` | {'en_US': '08 - Municipal Taxes Perceptions'} |
| `09` | {'en_US': '09 - Other Perceptions'} |
| `99` | {'en_US': '99 - Others'} |

**`l10n_ar_vat_afip_code`** — {'en_US': 'VAT ARCA Code'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Not Applicable'} |
| `1` | {'en_US': 'Untaxed'} |
| `2` | {'en_US': 'Exempt'} |
| `3` | {'en_US': '0%'} |
| `4` | {'en_US': '10.5%'} |
| `5` | {'en_US': '21%'} |
| `6` | {'en_US': '27%'} |
| `8` | {'en_US': '5%'} |
| `9` | {'en_US': '2,5%'} |

**`l10n_ec_type`** — {'en_US': 'Type Ecuadorian Tax'}

| Value | Label |
|---|---|
| `vat05` | {'en_US': 'VAT 5%'} |
| `vat08` | {'en_US': 'VAT 8%'} |
| `vat12` | {'en_US': 'VAT 12%'} |
| `vat13` | {'en_US': 'VAT 13%'} |
| `vat14` | {'en_US': 'VAT 14%'} |
| `vat15` | {'en_US': 'VAT 15%'} |
| `zero_vat` | {'en_US': 'VAT 0%'} |
| `not_charged_vat` | {'en_US': 'VAT Not Charged'} |
| `exempt_vat` | {'en_US': 'VAT Exempt'} |
| `ice` | {'en_US': 'Special Consumptions Tax (ICE)'} |
| `irbpnr` | {'en_US': 'Plastic Bottles (IRBPNR)'} |
| `withhold_vat_sale` | {'en_US': 'VAT Withhold on Sales'} |
| `withhold_vat_purchase` | {'en_US': 'VAT Withhold on Purchases'} |
| `withhold_income_sale` | {'en_US': 'Profit Withhold on Sales'} |
| `withhold_income_purchase` | {'en_US': 'Profit Withhold on Purchases'} |
| `outflows_tax` | {'en_US': 'Exchange Outflows'} |
| `other` | {'en_US': 'Others'} |

## `account.tax.repartition.line`

**`document_type`** — {'en_US': 'Related to', 'tr_TR': 'Şununla ilgili', 'ar_001': 'متعلق بـ'}

| Value | Label |
|---|---|
| `invoice` | {'en_US': 'Invoice', 'tr_TR': 'Fatura', 'ar_001': 'الفاتورة'} |
| `refund` | {'en_US': 'Refund', 'tr_TR': 'İade/Fiyat Farkı', 'ar_001': 'استرداد الأموال'} |

**`repartition_type`** — {'en_US': 'Based On', 'tr_TR': 'Matrahı Üzerinden', 'ar_001': 'بناءً على'}

| Value | Label |
|---|---|
| `base` | {'en_US': 'Base', 'tr_TR': 'Matrah', 'ar_001': 'قاعدة'} |
| `tax` | {'en_US': 'of tax', 'tr_TR': 'vergisi', 'ar_001': 'من الضريبة'} |

## `account.withholding.line`

**`comodel_payment_type`** — {'en_US': 'Comodel Payment Type'} (not stored)

| Value | Label |
|---|---|
| `outbound` | {'en_US': 'Send Money'} |
| `inbound` | {'en_US': 'Receive Money'} |

**`placeholder_type`** — {'en_US': 'Placeholder Type'}

| Value | Label |
|---|---|
| `given_by_sequence` | {'en_US': 'Given By the Sequence'} |
| `given_by_name` | {'en_US': 'Given By the Name'} |
| `not_defined` | {'en_US': 'Not defined'} |

**`previous_placeholder_type`** — {'en_US': 'Previous Placeholder Type'}

| Value | Label |
|---|---|
| `given_by_sequence` | {'en_US': 'Given By the Sequence'} |
| `given_by_name` | {'en_US': 'Given By the Name'} |
| `not_defined` | {'en_US': 'Not defined'} |

## `account_edi_proxy_client.user`

**`edi_mode`** — {'en_US': 'EDI operating mode', 'tr_TR': 'EDI Çalışma Modu', 'ar_001': 'وضع تشغيل تبادل المعلومات إلكترونياً'}

| Value | Label |
|---|---|
| `prod` | {'en_US': 'Production mode', 'tr_TR': 'Canlı Mod', 'ar_001': 'وضع الإنتاج'} |
| `test` | {'en_US': 'Test mode', 'tr_TR': 'Test Modu', 'ar_001': 'وضع الاختبار'} |
| `demo` | {'en_US': 'Demo mode', 'tr_TR': 'Demo modu', 'ar_001': 'وضع العرض التوضيحي'} |

**`proxy_type`** — {'en_US': 'Proxy Type', 'tr_TR': 'Vekil Türü', 'ar_001': 'نوع الوكيل'}

| Value | Label |
|---|---|
| `peppol` | {'en_US': 'PEPPOL', 'tr_TR': 'PEPPOL', 'ar_001': 'PEPPOL'} |
| `nemhandel` | {'en_US': 'Nemhandel'} |
| `l10n_it_edi` | {'en_US': 'Italian EDI'} |
| `l10n_my_edi` | {'en_US': 'Malaysian EDI'} |
| `pdp` | {'en_US': 'Approved Platform'} |
| `l10n_gr_edi` | {'en_US': 'Greek EDI'} |

## `auth.totp.rate.limit.log`

**`limit_type`** — {'en_US': 'Limit Type', 'tr_TR': 'Limit türü', 'ar_001': 'نوع الحد'}

| Value | Label |
|---|---|
| `send_email` | {'en_US': 'Send Email', 'tr_TR': 'E-posta Gönder', 'ar_001': 'إرسال بريد إلكتروني'} |
| `code_check` | {'en_US': 'Code Checking', 'tr_TR': 'Kod Kontrolü', 'ar_001': 'التحقق من الكود'} |

## `barcode.nomenclature`

**`upc_ean_conv`** — {'en_US': 'UPC/EAN Conversion', 'tr_TR': 'UPC/EAN dönüştürme', 'ar_001': 'التحويل بين UPC وEAN'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقًا'} |
| `ean2upc` | {'en_US': 'EAN-13 to UPC-A', 'tr_TR': 'EAN-13 to UPC-A', 'ar_001': 'EAN-13 إلى UPC-A'} |
| `upc2ean` | {'en_US': 'UPC-A to EAN-13', 'tr_TR': 'UPC-A dan EAN-13', 'ar_001': 'UPC-A إلى EAN-13'} |
| `always` | {'en_US': 'Always', 'tr_TR': 'Daima', 'ar_001': 'دائمًا'} |

## `barcode.rule`

**`encoding`** — {'en_US': 'Encoding', 'tr_TR': 'Kodlama', 'ar_001': 'التشفير'}

| Value | Label |
|---|---|
| `any` | {'en_US': 'Any', 'tr_TR': 'Hiç', 'ar_001': 'أي'} |
| `ean13` | {'en_US': 'EAN-13', 'tr_TR': 'EAN-13', 'ar_001': 'EAN-13'} |
| `ean8` | {'en_US': 'EAN-8', 'tr_TR': 'EAN-8', 'ar_001': 'EAN-8'} |
| `upca` | {'en_US': 'UPC-A', 'tr_TR': 'UPC-A', 'ar_001': 'UPC-A'} |
| `gs1-128` | {'en_US': 'GS1-128', 'tr_TR': 'GS1-128', 'ar_001': 'GS1-128'} |

**`gs1_content_type`** — {'en_US': 'GS1 Content Type', 'tr_TR': 'GS1 İçerik Türü', 'ar_001': 'نوع محتوى GS1'}

| Value | Label |
|---|---|
| `date` | {'en_US': 'Date', 'tr_TR': 'Tarih', 'ar_001': 'التاريخ'} |
| `measure` | {'en_US': 'Measure', 'tr_TR': 'Ölçüm', 'ar_001': 'المقياس'} |
| `identifier` | {'en_US': 'Numeric Identifier', 'tr_TR': 'Sayısal Tanımlayıcı', 'ar_001': 'المعرف الرقمي'} |
| `alpha` | {'en_US': 'Alpha-Numeric Name', 'tr_TR': 'Alfanümerik İsim', 'ar_001': 'اسم يحتوي على أحرف وأرقام'} |

**`type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `alias` | {'en_US': 'Alias', 'tr_TR': 'Rumuz', 'ar_001': 'لقب'} |
| `product` | {'en_US': 'Unit Product', 'tr_TR': 'Birim Ürün', 'ar_001': 'وحدة المنتج'} |
| `quantity` | {'en_US': 'Quantity', 'tr_TR': 'Miktar', 'ar_001': 'الكمية'} |
| `weight` | {'en_US': 'Weighted Product', 'tr_TR': 'Tartılmış Ürün', 'ar_001': 'منتج موزون'} |
| `location` | {'en_US': 'Location', 'tr_TR': 'Konum', 'ar_001': 'الموقع'} |
| `location_dest` | {'en_US': 'Destination location', 'tr_TR': 'Hedef konum', 'ar_001': 'موقع الوجهة'} |
| `lot` | {'en_US': 'Lot', 'tr_TR': 'Lot', 'ar_001': 'المجموعة'} |
| `package` | {'en_US': 'Package', 'tr_TR': 'Paket', 'ar_001': 'الطرد'} |
| `use_date` | {'en_US': 'Best before Date', 'tr_TR': 'Tavsiye Edilen Tüketim Tarihi', 'ar_001': 'مثالي قبل تاريخ'} |
| `expiration_date` | {'en_US': 'Expiration Date', 'tr_TR': 'Son Kullanma Tarihi', 'ar_001': 'تاريخ الانتهاء'} |
| `package_type` | {'en_US': 'Package Type', 'tr_TR': 'Paket Türü', 'ar_001': 'نوع الطرد'} |
| `pack_date` | {'en_US': 'Pack Date', 'tr_TR': 'Paket Tarihi', 'ar_001': 'تاريخ التعبئة'} |
| `price` | {'en_US': 'Priced Product', 'tr_TR': 'Fiyatlandırılmış Ürünler', 'ar_001': 'منتج مسعر'} |
| `discount` | {'en_US': 'Discounted Product', 'tr_TR': 'İndirimli Ürün', 'ar_001': 'منتج مخفض'} |
| `client` | {'en_US': 'Client', 'tr_TR': 'Müşteri', 'ar_001': 'العميل'} |
| `cashier` | {'en_US': 'Cashier', 'tr_TR': 'Kasiyer', 'ar_001': 'أمين الصندوق'} |
| `coupon` | {'en_US': 'Coupon', 'tr_TR': 'Kupon', 'ar_001': 'كوبون'} |

## `base.automation`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`trg_date_range_mode`** — {'en_US': 'Delay mode', 'tr_TR': 'Gecikme modu', 'ar_001': 'وضع التأخير'}

| Value | Label |
|---|---|
| `after` | {'en_US': 'After', 'tr_TR': 'Sonra', 'ar_001': 'بعد'} |
| `before` | {'en_US': 'Before', 'tr_TR': 'Önce', 'ar_001': 'قبل'} |

**`trg_date_range_type`** — {'en_US': 'Delay unit', 'tr_TR': 'Gecikme birimi', 'ar_001': 'وحدة التأخير'}

| Value | Label |
|---|---|
| `minutes` | {'en_US': 'Minutes', 'tr_TR': 'Dakika', 'ar_001': 'الدقائق'} |
| `hour` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |
| `day` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `month` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |

**`trigger`** — {'en_US': 'Trigger', 'tr_TR': 'Tetikleyici', 'ar_001': 'المشغّل'}

| Value | Label |
|---|---|
| `on_stage_set` | {'en_US': 'Stage is set to', 'tr_TR': 'Aşama şuna ayarlandı:', 'ar_001': 'تم تعيين المرحلة إلى'} |
| `on_user_set` | {'en_US': 'User is set', 'tr_TR': 'Kullanıcı atandı', 'ar_001': 'تم إعداد المستخدم'} |
| `on_tag_set` | {'en_US': 'Tag is added', 'tr_TR': 'Etiket eklendi', 'ar_001': 'تمت إضافة علامة التصنيف'} |
| `on_state_set` | {'en_US': 'State is set to', 'tr_TR': 'Durum şuna ayarlandı:', 'ar_001': 'تم تعيين الحالة كـ'} |
| `on_priority_set` | {'en_US': 'Priority is set to', 'tr_TR': 'Öncelik şuna ayarlandı:', 'ar_001': 'تم تعيين الأولوية إلى'} |
| `on_archive` | {'en_US': 'On archived', 'tr_TR': 'Arşivlendiğinde', 'ar_001': 'عند الأرشفة'} |
| `on_unarchive` | {'en_US': 'On unarchived', 'tr_TR': 'Arşivden Çıkarıldığında', 'ar_001': 'عند الإزالة من الأرشيف'} |
| `on_create` | {'en_US': 'On create', 'tr_TR': 'Oluşturma üzerine', 'ar_001': 'عند الإنشاء'} |
| `on_create_or_write` | {'en_US': 'On create and edit', 'tr_TR': 'Oluşturma ve düzenlemede', 'ar_001': 'عند الإنشاء والتحرير'} |
| `on_write` | {'en_US': 'On update', 'tr_TR': 'Güncellendiğinde', 'ar_001': 'عند التحديث'} |
| `on_unlink` | {'en_US': 'On deletion', 'tr_TR': 'Silindiğinde', 'ar_001': 'عند الحذف'} |
| `on_change` | {'en_US': 'On UI change', 'tr_TR': 'Arayüzde değişiklik olunca', 'ar_001': 'عند تغير واجهة المستخدم'} |
| `on_time` | {'en_US': 'Based on date field', 'tr_TR': 'Tarih alanına göre', 'ar_001': 'بناءً على حقل التاريخ'} |
| `on_time_created` | {'en_US': 'After creation', 'tr_TR': 'Oluşturulduktan sonra', 'ar_001': 'بعد الإنشاء'} |
| `on_time_updated` | {'en_US': 'After last update', 'tr_TR': 'Son güncellemeden sonra', 'ar_001': 'بعد التحديث الأخير'} |
| `on_message_received` | {'en_US': 'On incoming message', 'tr_TR': 'Gelen mesajda', 'ar_001': 'عند الرسائل الواردة'} |
| `on_message_sent` | {'en_US': 'On outgoing message', 'tr_TR': 'Giden mesajda', 'ar_001': 'عند الرسائل الصادرة'} |
| `on_webhook` | {'en_US': 'On webhook', 'tr_TR': 'Webhook tetiklendiğinde', 'ar_001': 'في webhook'} |

## `base.enable.profiling.wizard`

**`duration`** — {'en_US': 'Enable profiling for', 'tr_TR': 'Profil oluşturmayı etkinleştir', 'ar_001': 'تمكين التحليل لـ'}

| Value | Label |
|---|---|
| `minutes_5` | {'en_US': '5 Minutes', 'tr_TR': '5 Dakika', 'ar_001': '5 دقائق'} |
| `hours_1` | {'en_US': '1 Hour', 'tr_TR': '1 Saat', 'ar_001': 'ساعة 1'} |
| `days_1` | {'en_US': '1 Day', 'tr_TR': '1 Gün', 'ar_001': 'يوم 1'} |
| `months_1` | {'en_US': '1 Month', 'tr_TR': '1 Ay', 'ar_001': 'شهر 1'} |

## `base.import.module`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `init` | {'en_US': 'init', 'tr_TR': 'başlatma', 'ar_001': 'إطلاق'} |
| `done` | {'en_US': 'done', 'tr_TR': 'biten', 'ar_001': 'تم'} |

## `base.language.export`

**`export_type`** — {'en_US': 'Export Type', 'tr_TR': 'Dışa Aktarma Türü', 'ar_001': 'نوع التصدير'}

| Value | Label |
|---|---|
| `module` | {'en_US': 'Module', 'tr_TR': 'Modül', 'ar_001': 'التطبيق'} |
| `model` | {'en_US': 'Model', 'tr_TR': 'Model', 'ar_001': 'النموذج'} |

**`format`** — {'en_US': 'File Format', 'tr_TR': 'Dosya Formatı', 'ar_001': 'صيغة الملف'}

| Value | Label |
|---|---|
| `csv` | {'en_US': 'CSV File', 'tr_TR': 'CSV Dosyası', 'ar_001': 'ملف CSV'} |
| `po` | {'en_US': 'PO File', 'tr_TR': 'PO Dosyası', 'ar_001': 'ملف PO'} |
| `tgz` | {'en_US': 'TGZ Archive', 'tr_TR': 'TGZ Arşivi', 'ar_001': 'أرشيف TGZ'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}

| Value | Label |
|---|---|
| `choose` | {'en_US': 'choose', 'tr_TR': 'seç', 'ar_001': 'اختر'} |
| `get` | {'en_US': 'get', 'tr_TR': 'alma', 'ar_001': 'احصل على'} |

## `base.module.update`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `init` | {'en_US': 'init', 'tr_TR': 'başlatma', 'ar_001': 'إطلاق'} |
| `done` | {'en_US': 'done', 'tr_TR': 'biten', 'ar_001': 'تم'} |

## `base.partner.merge.automatic.wizard`

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}

| Value | Label |
|---|---|
| `option` | {'en_US': 'Option', 'tr_TR': 'Seçenek', 'ar_001': 'الخيار'} |
| `selection` | {'en_US': 'Selection', 'tr_TR': 'Seçim', 'ar_001': 'قائمة خيارات'} |
| `finished` | {'en_US': 'Finished', 'tr_TR': 'Bitmiş', 'ar_001': 'مُنتهي'} |

## `calendar.alarm`

**`alarm_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `notification` | {'en_US': 'Notification', 'tr_TR': 'Bildirimler', 'ar_001': 'إشعار'} |
| `email` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `sms` | {'en_US': 'SMS Text Message', 'tr_TR': 'SMS Metin Mesajı', 'ar_001': 'الرسائة النصية القصيرة'} |

**`interval`** — {'en_US': 'Unit', 'tr_TR': 'Birim', 'ar_001': 'الوحدة'}

| Value | Label |
|---|---|
| `minutes` | {'en_US': 'Minutes', 'tr_TR': 'Dakika', 'ar_001': 'الدقائق'} |
| `hours` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |

## `calendar.attendee`

**`availability`** — {'en_US': 'Available/Busy', 'tr_TR': 'Müsait/Meşgul', 'ar_001': 'متاح/مشغول'}

| Value | Label |
|---|---|
| `free` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `busy` | {'en_US': 'Busy', 'tr_TR': 'Meşgul', 'ar_001': 'مشغول'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `accepted` | {'en_US': 'Yes', 'tr_TR': 'Evet', 'ar_001': 'نعم'} |
| `declined` | {'en_US': 'No', 'tr_TR': 'Hayır', 'ar_001': 'لا'} |
| `tentative` | {'en_US': 'Maybe', 'tr_TR': 'Belki', 'ar_001': 'ربما'} |
| `needsAction` | {'en_US': 'Needs Action', 'tr_TR': 'Eylem Gerektiriyor', 'ar_001': 'يتطلب اتخاذ إجراء'} |

## `calendar.event`

**`byday`** — {'en_US': 'By day', 'tr_TR': 'Gündüz', 'ar_001': 'باليوم'} (not stored)

| Value | Label |
|---|---|
| `1` | {'en_US': 'First', 'tr_TR': 'İlk', 'ar_001': 'الأول'} |
| `2` | {'en_US': 'Second', 'tr_TR': 'Saniye', 'ar_001': 'الثاني'} |
| `3` | {'en_US': 'Third', 'tr_TR': 'Üçüncü', 'ar_001': 'ثالثًا'} |
| `4` | {'en_US': 'Fourth', 'tr_TR': 'Dördüncü', 'ar_001': 'رابعًا'} |
| `-1` | {'en_US': 'Last', 'tr_TR': 'Sonuncu', 'ar_001': 'آخِر'} |

**`effective_privacy`** — {'en_US': 'Effective Privacy', 'tr_TR': 'Etkili Gizlilik', 'ar_001': 'الخصوصية المطبقة'} (not stored)

| Value | Label |
|---|---|
| `public` | {'en_US': 'Public', 'tr_TR': 'Genel', 'ar_001': 'عام'} |
| `private` | {'en_US': 'Private', 'tr_TR': 'Özel', 'ar_001': 'خاص'} |
| `confidential` | {'en_US': 'Only internal users', 'tr_TR': 'Sadece dahili kullanıcılar', 'ar_001': 'المستخدمين الداخليين فقط'} |

**`end_type`** — {'en_US': 'Recurrence Termination', 'tr_TR': 'Yineleme Sonlandırma', 'ar_001': 'إنهاء التكرار'} (not stored)

| Value | Label |
|---|---|
| `count` | {'en_US': 'Number of repetitions', 'tr_TR': 'Tekrarlama Sayısı', 'ar_001': 'مرات التكرار'} |
| `end_date` | {'en_US': 'End date', 'tr_TR': 'Bitiş tarihi', 'ar_001': 'تاريخ الانتهاء'} |
| `forever` | {'en_US': 'Forever', 'tr_TR': 'Sonsuza dek', 'ar_001': 'للأبد'} |

**`month_by`** — {'en_US': 'Option', 'tr_TR': 'Seçenek', 'ar_001': 'الخيار'} (not stored)

| Value | Label |
|---|---|
| `date` | {'en_US': 'Date of month', 'tr_TR': 'Ayın Günü', 'ar_001': 'اليوم من الشهر'} |
| `day` | {'en_US': 'Day of month', 'tr_TR': 'Ayın günü', 'ar_001': 'اليوم من الشهر'} |

**`privacy`** — {'en_US': 'Privacy', 'tr_TR': 'Özel', 'ar_001': 'الخصوصية'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Public', 'tr_TR': 'Genel', 'ar_001': 'عام'} |
| `private` | {'en_US': 'Private', 'tr_TR': 'Özel', 'ar_001': 'خاص'} |
| `confidential` | {'en_US': 'Only internal users', 'tr_TR': 'Sadece dahili kullanıcılar', 'ar_001': 'المستخدمين الداخليين فقط'} |

**`recurrence_update`** — {'en_US': 'Recurrence Update', 'tr_TR': 'Yineleme Güncellemesi', 'ar_001': 'تحديث التكرار'} (not stored)

| Value | Label |
|---|---|
| `self_only` | {'en_US': 'This event', 'tr_TR': 'Bu etkinlik', 'ar_001': 'هذه الفعالية'} |
| `future_events` | {'en_US': 'This and following events', 'tr_TR': 'Bu ve aşağıdaki olaylar', 'ar_001': 'هذه الفعالية والفعاليات التالية'} |
| `all_events` | {'en_US': 'All events', 'tr_TR': 'Tüm Etkinlikler', 'ar_001': 'كافة الفعاليات'} |

**`rrule_type`** — {'en_US': 'Recurrence', 'tr_TR': 'Yinelenme', 'ar_001': 'التكرار'} (not stored)

| Value | Label |
|---|---|
| `daily` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weekly` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `monthly` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |
| `yearly` | {'en_US': 'Years', 'tr_TR': 'Yıllar', 'ar_001': 'سنوات'} |

**`rrule_type_ui`** — {'en_US': 'Repeat', 'tr_TR': 'Tekrarla', 'ar_001': 'تكرار'} (not stored)

| Value | Label |
|---|---|
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `weekly` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |
| `yearly` | {'en_US': 'Yearly', 'tr_TR': 'Yıllık', 'ar_001': 'سنويًا'} |
| `custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |

**`show_as`** — {'en_US': 'Show as', 'tr_TR': 'Olarak göster', 'ar_001': 'الإظهار كـ'}

| Value | Label |
|---|---|
| `free` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `busy` | {'en_US': 'Busy', 'tr_TR': 'Meşgul', 'ar_001': 'مشغول'} |

**`videocall_source`** — {'en_US': 'Videocall Source', 'tr_TR': 'Görüntülü Görüşme Kaynağı', 'ar_001': 'مصدر مكالمة الفيديو'} (not stored)

| Value | Label |
|---|---|
| `discuss` | {'en_US': 'Discuss', 'tr_TR': 'Mesajlaşma', 'ar_001': 'المناقشة'} |
| `custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |
| `google_meet` | {'en_US': 'Google Meet', 'tr_TR': 'Google Meet', 'ar_001': 'اجتماعات Google'} |

**`weekday`** — {'en_US': 'Weekday', 'tr_TR': 'Hafta Günü', 'ar_001': 'يوم العمل'} (not stored)

| Value | Label |
|---|---|
| `MON` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `TUE` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `WED` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `THU` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `FRI` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `SAT` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |
| `SUN` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |

## `calendar.popover.delete.wizard`

**`delete`** — {'en_US': 'Delete', 'tr_TR': 'Sil', 'ar_001': 'حذف'}

| Value | Label |
|---|---|
| `one` | {'en_US': 'Delete this event', 'tr_TR': 'Bu etkinliği sil', 'ar_001': 'حذف هذه الفعالية'} |
| `next` | {'en_US': 'Delete this and following events', 'tr_TR': 'Bunu ve bundan sonraki etkinlikleri sil', 'ar_001': 'حذف هذه الفعالية والفعاليات القادمة'} |
| `all` | {'en_US': 'Delete all the events', 'tr_TR': 'Tüm etkinlikleri sil', 'ar_001': 'حذف كافة الفعاليات'} |

## `calendar.provider.config`

**`external_calendar_provider`** — {'en_US': 'Choose an external calendar to configure', 'tr_TR': 'Yapılandırılacak harici takvim seçin', 'ar_001': 'اختر تقويماً خارجياً لتهيئته'}

| Value | Label |
|---|---|
| `google` | {'en_US': 'Google', 'tr_TR': 'Google', 'ar_001': 'Google'} |
| `microsoft` | {'en_US': 'Outlook', 'tr_TR': 'Outlook', 'ar_001': 'Outlook'} |

## `calendar.recurrence`

**`byday`** — {'en_US': 'By day', 'tr_TR': 'Gündüz', 'ar_001': 'باليوم'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'First', 'tr_TR': 'İlk', 'ar_001': 'الأول'} |
| `2` | {'en_US': 'Second', 'tr_TR': 'Saniye', 'ar_001': 'الثاني'} |
| `3` | {'en_US': 'Third', 'tr_TR': 'Üçüncü', 'ar_001': 'ثالثًا'} |
| `4` | {'en_US': 'Fourth', 'tr_TR': 'Dördüncü', 'ar_001': 'رابعًا'} |
| `-1` | {'en_US': 'Last', 'tr_TR': 'Sonuncu', 'ar_001': 'آخِر'} |

**`end_type`** — {'en_US': 'End Type', 'tr_TR': 'End Type', 'ar_001': 'نوع النهاية'}

| Value | Label |
|---|---|
| `count` | {'en_US': 'Number of repetitions', 'tr_TR': 'Tekrarlama Sayısı', 'ar_001': 'مرات التكرار'} |
| `end_date` | {'en_US': 'End date', 'tr_TR': 'Bitiş tarihi', 'ar_001': 'تاريخ الانتهاء'} |
| `forever` | {'en_US': 'Forever', 'tr_TR': 'Sonsuza dek', 'ar_001': 'للأبد'} |

**`month_by`** — {'en_US': 'Month By', 'tr_TR': 'Month By', 'ar_001': 'الشهر'}

| Value | Label |
|---|---|
| `date` | {'en_US': 'Date of month', 'tr_TR': 'Ayın Günü', 'ar_001': 'اليوم من الشهر'} |
| `day` | {'en_US': 'Day of month', 'tr_TR': 'Ayın günü', 'ar_001': 'اليوم من الشهر'} |

**`rrule_type`** — {'en_US': 'Rrule Type', 'tr_TR': 'Rrule Type', 'ar_001': 'نوع قاعدة التكرار'}

| Value | Label |
|---|---|
| `daily` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weekly` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `monthly` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |
| `yearly` | {'en_US': 'Years', 'tr_TR': 'Yıllar', 'ar_001': 'سنوات'} |

**`weekday`** — {'en_US': 'Weekday', 'tr_TR': 'Hafta Günü', 'ar_001': 'يوم العمل'}

| Value | Label |
|---|---|
| `MON` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `TUE` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `WED` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `THU` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `FRI` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `SAT` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |
| `SUN` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |

## `card.campaign`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

## `card.card`

**`share_status`** — {'en_US': 'Share Status', 'tr_TR': 'Durumu Paylaş', 'ar_001': 'مشاركة الحالة'}

| Value | Label |
|---|---|
| `shared` | {'en_US': 'Shared', 'tr_TR': 'Paylaşılan', 'ar_001': 'مشارك'} |
| `visited` | {'en_US': 'Visited', 'tr_TR': 'Ziyaret edildi', 'ar_001': 'قام بالزيارة'} |

## `certificate.certificate`

**`content_format`** — {'en_US': 'Original certificate format', 'tr_TR': 'Orijinal sertifika formatı', 'ar_001': 'التنسيق الأصلي للشهادة'}

| Value | Label |
|---|---|
| `der` | {'en_US': 'DER', 'tr_TR': 'DER', 'ar_001': 'DER'} |
| `pem` | {'en_US': 'PEM', 'tr_TR': 'PEM', 'ar_001': 'PEM'} |
| `pkcs12` | {'en_US': 'PKCS12', 'tr_TR': 'PKCS12', 'ar_001': 'PKCS12'} |

**`scope`** — {'en_US': 'Certificate scope', 'tr_TR': 'Sertifika kapsamı', 'ar_001': 'مجال الشهادة'}

| Value | Label |
|---|---|
| `general` | {'en_US': 'General', 'tr_TR': 'Genel', 'ar_001': 'عام'} |
| `facturae` | {'en_US': 'Facturae'} |
| `sii` | {'en_US': 'SII'} |
| `tbai` | {'en_US': 'TBAI'} |
| `verifactu` | {'en_US': 'Veri*Factu'} |

## `chatbot.script`

**`first_step_warning`** — {'en_US': 'First Step Warning', 'tr_TR': 'İlk Adım Uyarısı', 'ar_001': 'تحذير الخطوة الأولى'} (not stored)

| Value | Label |
|---|---|
| `first_step_operator` | {'en_US': 'First Step Operator', 'tr_TR': 'İlk Adım Operatörü', 'ar_001': 'موظف الخطوة الأولى'} |
| `first_step_invalid` | {'en_US': 'First Step Invalid', 'tr_TR': 'İlk Adım Geçersiz', 'ar_001': 'الخطوة الأولى غير صالحة'} |

## `chatbot.script.step`

**`step_type`** — {'en_US': 'Step Type', 'tr_TR': 'Adım Türü', 'ar_001': 'نوع الخطوة'}

| Value | Label |
|---|---|
| `text` | {'en_US': 'Text', 'tr_TR': 'Metin', 'ar_001': 'النص'} |
| `question_selection` | {'en_US': 'Question', 'tr_TR': 'Soru', 'ar_001': 'السؤال'} |
| `question_email` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `question_phone` | {'en_US': 'Phone', 'tr_TR': 'Telefon', 'ar_001': 'رقم الهاتف'} |
| `forward_operator` | {'en_US': 'Forward to Operator', 'tr_TR': 'Operatöre İletme', 'ar_001': 'التحويل إلى الموظف'} |
| `free_input_single` | {'en_US': 'Free Input', 'tr_TR': 'Ücretsiz Giriş', 'ar_001': 'مدخلات حرة'} |
| `free_input_multi` | {'en_US': 'Free Input (Multi-Line)', 'tr_TR': 'Ücretsiz Giriş (Çoklu)', 'ar_001': 'مدخلات حرة (متعددة الأسطر)'} |
| `create_lead` | {'en_US': 'Create Lead', 'tr_TR': 'Müşteri Adayı Oluştur', 'ar_001': 'إنشاء عميل مهتم'} |
| `create_lead_and_forward` | {'en_US': 'Create Lead & Forward', 'tr_TR': 'Potansiyel müşteri adayı oluşturun ve yönlendirin', 'ar_001': 'إنشاء العميل المهتم وإعادة توجيهه'} |

## `crm.activity.report`

**`lead_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `lead` | {'en_US': 'Lead', 'tr_TR': 'Aday', 'ar_001': 'عميل مهتم'} |
| `opportunity` | {'en_US': 'Opportunity', 'tr_TR': 'Fırsat', 'ar_001': 'الفرصة'} |

**`won_status`** — {'en_US': 'Is Won', 'tr_TR': 'Kazanıldı', 'ar_001': 'رابحة'}

| Value | Label |
|---|---|
| `won` | {'en_US': 'Won', 'tr_TR': 'Kazanıldı', 'ar_001': 'تم الفوز بها'} |
| `lost` | {'en_US': 'Lost', 'tr_TR': 'Kayıp', 'ar_001': 'ضائع'} |
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |

## `crm.iap.lead.mining.request`

**`contact_filter_type`** — {'en_US': 'Filter on', 'tr_TR': 'Buna göre Süz', 'ar_001': 'التصفية حسب'}

| Value | Label |
|---|---|
| `role` | {'en_US': 'Role', 'tr_TR': 'Rol', 'ar_001': 'الدور'} |
| `seniority` | {'en_US': 'Seniority', 'tr_TR': 'Kıdem', 'ar_001': 'الأقدمية'} |

**`error_type`** — {'en_US': 'Error Type', 'tr_TR': 'Hata Türü', 'ar_001': 'نوع الخطأ'}

| Value | Label |
|---|---|
| `credits` | {'en_US': 'Insufficient Credits', 'tr_TR': 'Yetersiz Kredi', 'ar_001': 'ليس لديك الرصيد الكافي'} |
| `no_result` | {'en_US': 'No Result', 'tr_TR': 'Sonuç Yok', 'ar_001': 'لا توجد نتائج'} |

**`lead_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `lead` | {'en_US': 'Leads', 'tr_TR': 'Adaylar', 'ar_001': 'العملاء المهتمين'} |
| `opportunity` | {'en_US': 'Opportunities', 'tr_TR': 'Fırsatlar', 'ar_001': 'الفرص'} |

**`search_type`** — {'en_US': 'Target', 'tr_TR': 'Hedef', 'ar_001': 'الهدف'}

| Value | Label |
|---|---|
| `companies` | {'en_US': 'Companies', 'tr_TR': 'Şirketler', 'ar_001': 'الشركات'} |
| `people` | {'en_US': 'Companies and their Contacts', 'tr_TR': 'Şirketler ve Bağlantıları', 'ar_001': 'الشركات وجهات الاتصال الخاصة بها'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `crm.lead`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`email_state`** — {'en_US': 'Email Quality', 'tr_TR': 'E-Posta Kalitesi', 'ar_001': 'جودة البريد الإلكتروني'}

| Value | Label |
|---|---|
| `correct` | {'en_US': 'Correct', 'tr_TR': 'Doğru', 'ar_001': 'صحيحة'} |
| `incorrect` | {'en_US': 'Incorrect', 'tr_TR': 'Yanlış', 'ar_001': 'غير صحيح'} |

**`phone_state`** — {'en_US': 'Phone Quality', 'tr_TR': 'Telefon Kalitesi', 'ar_001': 'جودة الهاتف'}

| Value | Label |
|---|---|
| `correct` | {'en_US': 'Correct', 'tr_TR': 'Doğru', 'ar_001': 'صحيحة'} |
| `incorrect` | {'en_US': 'Incorrect', 'tr_TR': 'Yanlış', 'ar_001': 'غير صحيح'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Low', 'tr_TR': 'Düşük', 'ar_001': 'منخفض'} |
| `1` | {'en_US': 'Medium', 'tr_TR': 'Aracı:', 'ar_001': 'متوسط'} |
| `2` | {'en_US': 'High', 'tr_TR': 'Yüksek', 'ar_001': 'مرتفع'} |
| `3` | {'en_US': 'Very High', 'tr_TR': 'Çok Yüksek', 'ar_001': 'عالٍ جداً'} |

**`type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `lead` | {'en_US': 'Lead', 'tr_TR': 'Aday', 'ar_001': 'عميل مهتم'} |
| `opportunity` | {'en_US': 'Opportunity', 'tr_TR': 'Fırsat', 'ar_001': 'الفرصة'} |

**`won_status`** — {'en_US': 'Won/Lost', 'tr_TR': 'Kazanılan / Kaybedilen', 'ar_001': 'رابحة/ضائعة'}

| Value | Label |
|---|---|
| `won` | {'en_US': 'Won', 'tr_TR': 'Kazanıldı', 'ar_001': 'تم الفوز بها'} |
| `lost` | {'en_US': 'Lost', 'tr_TR': 'Kayıp', 'ar_001': 'ضائع'} |
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |

## `crm.lead.forward.to.partner`

**`forward_type`** — {'en_US': 'Forward selected leads to', 'tr_TR': 'Seçili adayları ilet', 'ar_001': 'إرسال العملاء المهتمين المحددين إلى'}

| Value | Label |
|---|---|
| `single` | {'en_US': 'a single partner: manual selection of partner', 'tr_TR': 'tek bir iş ortağı: iş ortağının mauel seçimi', 'ar_001': 'شريك واحد: دليل اختيار شريك'} |
| `assigned` | {'en_US': "several partners: automatic assignment, using GPS coordinates and partner's grades", 'tr_TR': 'birkaç ortak: GPS koordinatlarını ve ortakların notlarını kullanarak otomatik atama', 'ar_001': 'عدة شركاء: الإسناد التلقائي، باستخدام إحداثيات GPS ودرجات الشريك'} |

## `crm.lead2opportunity.partner`

**`action`** — {'en_US': 'Related Customer', 'tr_TR': 'İlgili Müşteri', 'ar_001': 'العميل ذو الصلة'}

| Value | Label |
|---|---|
| `create` | {'en_US': 'Create a new customer', 'tr_TR': 'Yeni bir müşteri oluştur', 'ar_001': 'إنشاء عميل جديد'} |
| `exist` | {'en_US': 'Link to an existing customer', 'tr_TR': 'Varolan bir müşteriye bağlantıla', 'ar_001': 'الربط بعميل موجود بالفعل'} |

**`name`** — {'en_US': 'Conversion Action', 'tr_TR': 'Dönüştürme Eylemi', 'ar_001': 'إجراء التحويل'}

| Value | Label |
|---|---|
| `convert` | {'en_US': 'Convert to opportunity', 'tr_TR': 'Fırsata dönüştür', 'ar_001': 'التحويل إلى فرصة'} |
| `merge` | {'en_US': 'Merge with existing opportunities', 'tr_TR': 'Mevcut fırsatlarla birleştir', 'ar_001': 'دمج مع فرص موجودة بالفعل'} |

## `crm.lead2opportunity.partner.mass`

**`action`** — {'en_US': 'Related Customer', 'tr_TR': 'İlgili Müşteri', 'ar_001': 'العميل ذو الصلة'}

| Value | Label |
|---|---|
| `create` | {'en_US': 'Create a new customer', 'tr_TR': 'Yeni bir müşteri oluştur', 'ar_001': 'إنشاء عميل جديد'} |
| `exist` | {'en_US': 'Link to an existing customer', 'tr_TR': 'Varolan bir müşteriye bağlantıla', 'ar_001': 'الربط بعميل موجود بالفعل'} |
| `each_exist_or_create` | {'en_US': 'Use existing partner or create', 'tr_TR': 'Varolan iş ortağını kullan yada oluştur', 'ar_001': 'استخدام شريك موجود بالفعل أو إنشاء واحد'} |

**`name`** — {'en_US': 'Conversion Action', 'tr_TR': 'Dönüştürme Eylemi', 'ar_001': 'إجراء التحويل'}

| Value | Label |
|---|---|
| `convert` | {'en_US': 'Convert to opportunity', 'tr_TR': 'Fırsata dönüştür', 'ar_001': 'التحويل إلى فرصة'} |
| `merge` | {'en_US': 'Merge with existing opportunities', 'tr_TR': 'Mevcut fırsatlarla birleştir', 'ar_001': 'دمج مع فرص موجودة بالفعل'} |

## `crm.quotation.partner`

**`action`** — {'en_US': 'Quotation Customer', 'tr_TR': 'Teklif Müşteri', 'ar_001': 'العميل المعني بعرض السعر'}

| Value | Label |
|---|---|
| `create` | {'en_US': 'Create a new customer', 'tr_TR': 'Yeni bir müşteri oluştur', 'ar_001': 'إنشاء عميل جديد'} |
| `exist` | {'en_US': 'Link to an existing customer', 'tr_TR': 'Varolan bir müşteriye bağlantıla', 'ar_001': 'الربط بعميل موجود بالفعل'} |
| `nothing` | {'en_US': 'Do not link to a customer', 'tr_TR': 'Bir müşteriye bağlanmayın', 'ar_001': 'عدم الربط بعميل'} |

## `crm.reveal.rule`

**`contact_filter_type`** — {'en_US': 'Filter On', 'tr_TR': 'Filtre Açık', 'ar_001': 'التصفية حسب'}

| Value | Label |
|---|---|
| `role` | {'en_US': 'Role', 'tr_TR': 'Rol', 'ar_001': 'الدور'} |
| `seniority` | {'en_US': 'Seniority', 'tr_TR': 'Kıdem', 'ar_001': 'الأقدمية'} |

**`lead_for`** — {'en_US': 'Data Tracking', 'tr_TR': 'Veri İzleme', 'ar_001': 'تعقب البيانات'}

| Value | Label |
|---|---|
| `companies` | {'en_US': 'Companies', 'tr_TR': 'Şirketler', 'ar_001': 'الشركات'} |
| `people` | {'en_US': 'Companies and their Contacts', 'tr_TR': 'Şirketler ve Bağlantıları', 'ar_001': 'الشركات وجهات الاتصال الخاصة بها'} |

**`lead_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `lead` | {'en_US': 'Lead', 'tr_TR': 'Aday', 'ar_001': 'عميل مهتم'} |
| `opportunity` | {'en_US': 'Opportunity', 'tr_TR': 'Fırsat', 'ar_001': 'الفرصة'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Low', 'tr_TR': 'Düşük', 'ar_001': 'منخفض'} |
| `1` | {'en_US': 'Medium', 'tr_TR': 'Aracı:', 'ar_001': 'متوسط'} |
| `2` | {'en_US': 'High', 'tr_TR': 'Yüksek', 'ar_001': 'مرتفع'} |
| `3` | {'en_US': 'Very High', 'tr_TR': 'Çok Yüksek', 'ar_001': 'عالٍ جداً'} |

## `crm.reveal.view`

**`reveal_state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `to_process` | {'en_US': 'To Process', 'tr_TR': 'İşlenecek', 'ar_001': 'للمعالجة'} |
| `not_found` | {'en_US': 'Not Found', 'tr_TR': 'Bulunamadı', 'ar_001': 'لم يتم العثور عليه'} |

## `data_recycle.model`

**`notify_frequency_period`** — {'en_US': 'Notify Frequency Period', 'tr_TR': 'Sıklık Dönemini Bildir', 'ar_001': 'فترة تواتر الإخطار'}

| Value | Label |
|---|---|
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weeks` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |

**`recycle_action`** — {'en_US': 'Recycle Action', 'tr_TR': 'Geri Dönüşüm İşlemi', 'ar_001': 'إعادة تدوير الإجراء'}

| Value | Label |
|---|---|
| `archive` | {'en_US': 'Archive', 'tr_TR': 'Arşivle', 'ar_001': 'الأرشيف'} |
| `unlink` | {'en_US': 'Delete', 'tr_TR': 'Sil', 'ar_001': 'حذف'} |

**`recycle_mode`** — {'en_US': 'Recycle Mode', 'tr_TR': 'Geri Dönüşüm Modu', 'ar_001': 'إعادة تدوير الوضع'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `automatic` | {'en_US': 'Automatic', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |

**`time_field_delta_unit`** — {'en_US': 'Delta Unit', 'tr_TR': 'Delta Birimi', 'ar_001': 'وحدة دلتا'}

| Value | Label |
|---|---|
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weeks` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |
| `years` | {'en_US': 'Years', 'tr_TR': 'Yıllar', 'ar_001': 'سنوات'} |

## `delivery.carrier`

**`delivery_type`** — {'en_US': 'Provider', 'tr_TR': 'Sağlayıcı', 'ar_001': 'المزود'}

| Value | Label |
|---|---|
| `base_on_rule` | {'en_US': 'Based on Rules', 'tr_TR': 'Bu kurallara göre', 'ar_001': 'حسب القواعد'} |
| `fixed` | {'en_US': 'Fixed Price', 'tr_TR': 'Sabit Fiyat', 'ar_001': 'سعر ثابت'} |
| `gelato` | {'en_US': 'Gelato', 'tr_TR': 'Gelato', 'ar_001': 'Gelato'} |
| `in_store` | {'en_US': 'Pick up in store', 'tr_TR': 'Mağazadan teslim al', 'ar_001': 'الاستلام من المتجر'} |

**`gelato_shipping_service_type`** — {'en_US': 'Gelato Shipping Service Type', 'tr_TR': 'Gelato Nakliye Hizmet Türü', 'ar_001': 'نوع خدمة شحن Gelato'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Standard Delivery', 'tr_TR': 'Standart Teslimat', 'ar_001': 'التوصيل القياسي'} |
| `express` | {'en_US': 'Express Delivery', 'tr_TR': 'Hızlı Teslimat', 'ar_001': 'التوصيل السريع'} |

**`integration_level`** — {'en_US': 'Integration Level', 'tr_TR': 'Entegrasyon Seviyesi', 'ar_001': 'مستوى الربط'}

| Value | Label |
|---|---|
| `rate` | {'en_US': 'Get Rate', 'tr_TR': 'Oran Al', 'ar_001': 'حساب السعر'} |
| `rate_and_ship` | {'en_US': 'Get Rate and Create Shipment', 'tr_TR': 'Oranı Alın ve Sevkiyatı Oluşturun', 'ar_001': 'حساب السعر وإنشاء الشحنة'} |

**`invoice_policy`** — {'en_US': 'Invoicing Policy', 'tr_TR': 'Faturalama Kuralı', 'ar_001': 'سياسة الفوترة'}

| Value | Label |
|---|---|
| `estimated` | {'en_US': 'Estimated cost', 'tr_TR': 'Tahmini maliyet', 'ar_001': 'التكلفة التقديرية'} |
| `real` | {'en_US': 'Real cost', 'tr_TR': 'Gerçek Maliyet', 'ar_001': 'التكلفة الفعلية'} |

## `delivery.price.rule`

**`operator`** — {'en_US': 'Operator', 'tr_TR': 'Operatör', 'ar_001': 'موظف الدعم'}

| Value | Label |
|---|---|
| `==` | {'en_US': '='} |
| `<=` | {'en_US': '<='} |
| `<` | {'en_US': '<'} |
| `>=` | {'en_US': '>='} |
| `>` | {'en_US': '>'} |

**`variable`** — {'en_US': 'Variable', 'tr_TR': 'Değişken', 'ar_001': 'متغير'}

| Value | Label |
|---|---|
| `weight` | {'en_US': 'Weight', 'tr_TR': 'Ağırlık', 'ar_001': 'الوزن'} |
| `volume` | {'en_US': 'Volume', 'tr_TR': 'Hacim', 'ar_001': 'الحجم'} |
| `wv` | {'en_US': 'Weight * Volume', 'tr_TR': 'Ağırlık * Hacim', 'ar_001': 'الوزن × الحجم'} |
| `price` | {'en_US': 'Price', 'tr_TR': 'Fiyat', 'ar_001': 'السعر'} |
| `quantity` | {'en_US': 'Quantity', 'tr_TR': 'Miktar', 'ar_001': 'الكمية'} |

**`variable_factor`** — {'en_US': 'Variable Factor', 'tr_TR': 'Değişken Faktörü', 'ar_001': 'معامل متغير'}

| Value | Label |
|---|---|
| `weight` | {'en_US': 'Weight', 'tr_TR': 'Ağırlık', 'ar_001': 'الوزن'} |
| `volume` | {'en_US': 'Volume', 'tr_TR': 'Hacim', 'ar_001': 'الحجم'} |
| `wv` | {'en_US': 'Weight * Volume', 'tr_TR': 'Ağırlık * Hacim', 'ar_001': 'الوزن × الحجم'} |
| `price` | {'en_US': 'Price', 'tr_TR': 'Fiyat', 'ar_001': 'السعر'} |
| `quantity` | {'en_US': 'Quantity', 'tr_TR': 'Miktar', 'ar_001': 'الكمية'} |

## `digest.digest`

**`periodicity`** — {'en_US': 'Periodicity', 'tr_TR': 'Dönemsellik', 'ar_001': 'الوتيرة'}

| Value | Label |
|---|---|
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `weekly` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |
| `quarterly` | {'en_US': 'Quarterly', 'tr_TR': 'Üç Aylık', 'ar_001': 'ربع سنوي'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `activated` | {'en_US': 'Activated', 'tr_TR': 'Etkinleştirildi', 'ar_001': 'مفعل'} |
| `deactivated` | {'en_US': 'Deactivated', 'tr_TR': 'Devre dışı', 'ar_001': 'معطل'} |

## `discuss.channel`

**`channel_type`** — {'en_US': 'Channel Type', 'tr_TR': 'Kanal Türü', 'ar_001': 'نوع القناة'}

| Value | Label |
|---|---|
| `chat` | {'en_US': 'Chat', 'tr_TR': 'Sohbet', 'ar_001': 'الدردشة'} |
| `channel` | {'en_US': 'Channel', 'tr_TR': 'Kanal', 'ar_001': 'القناة'} |
| `group` | {'en_US': 'Group', 'tr_TR': 'Grup', 'ar_001': 'المجموعة'} |
| `livechat` | {'en_US': 'Livechat Conversation', 'tr_TR': 'Canlı Sohbetler', 'ar_001': 'محادثة الدردشة المباشرة'} |

**`default_display_mode`** — {'en_US': 'Default Display Mode', 'tr_TR': 'Varsayılan Görünüm Modu', 'ar_001': 'طريقة العرض الافتراضية'}

| Value | Label |
|---|---|
| `video_full_screen` | {'en_US': 'Full screen video', 'tr_TR': 'Tam ekran video', 'ar_001': 'مقطع فيديو بوضع ملء الشاشة'} |

**`livechat_failure`** — {'en_US': 'Live Chat Session Failure', 'tr_TR': 'Canlı Sohbet Oturumu Hatası', 'ar_001': 'فشلت جلسة الدردشة المباشرة'}

| Value | Label |
|---|---|
| `no_answer` | {'en_US': 'Never Answered', 'tr_TR': 'Hiç Cevaplanmadı', 'ar_001': 'لم يتم الإجابة عنها قط'} |
| `no_agent` | {'en_US': 'No one Available', 'tr_TR': 'Müsait kimse yok', 'ar_001': 'لا يوجد شخص متاح'} |
| `no_failure` | {'en_US': 'No Failure', 'tr_TR': 'Başarısızlık Yok', 'ar_001': 'لم يفشل'} |

**`livechat_outcome`** — {'en_US': 'Livechat Outcome', 'tr_TR': 'Canlı Sohbet Sonuçları', 'ar_001': 'نتائج الدردشة المباشرة'}

| Value | Label |
|---|---|
| `no_answer` | {'en_US': 'Never Answered', 'tr_TR': 'Hiç Cevaplanmadı', 'ar_001': 'لم يتم الإجابة عنها قط'} |
| `no_agent` | {'en_US': 'No one Available', 'tr_TR': 'Müsait kimse yok', 'ar_001': 'لا يوجد شخص متاح'} |
| `no_failure` | {'en_US': 'Success', 'tr_TR': 'Başarılı', 'ar_001': 'نجاح'} |
| `escalated` | {'en_US': 'Escalated', 'tr_TR': 'Üst destek birimi', 'ar_001': 'تصعيد'} |

**`livechat_status`** — {'en_US': 'Livechat Status', 'tr_TR': 'Canlı Sohbet  Durumu', 'ar_001': 'حالة الدردشة المباشرة'}

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'In progress', 'tr_TR': 'Devam Eden', 'ar_001': 'قيد التنفيذ'} |
| `waiting` | {'en_US': 'Waiting for customer', 'tr_TR': 'Müşteri bekleniyor', 'ar_001': 'بانتظار العميل'} |
| `need_help` | {'en_US': 'Looking for help', 'tr_TR': 'Yardım arıyorum', 'ar_001': 'أبحث عن المساعدة'} |

**`livechat_week_day`** — {'en_US': 'Day of the Week', 'tr_TR': 'Haftanın Günü', 'ar_001': 'اليوم من الأسبوع'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `1` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `2` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `3` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `4` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `5` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |
| `6` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |

**`rating_avg_text`** — {'en_US': 'Rating Avg Text', 'tr_TR': 'Değerlendirme Ort Metni', 'ar_001': 'متوسط نص التقييم'} (not stored)

| Value | Label |
|---|---|
| `top` | {'en_US': 'Happy', 'tr_TR': 'Mutlu', 'ar_001': 'سعيد'} |
| `ok` | {'en_US': 'Neutral', 'tr_TR': 'Nötr', 'ar_001': 'محايد'} |
| `ko` | {'en_US': 'Unhappy', 'tr_TR': 'Mutsuz', 'ar_001': 'غير سعيد'} |
| `none` | {'en_US': 'Not Rated yet', 'tr_TR': 'Henüz Değerlendirilmedi', 'ar_001': 'لم يتم التقييم بعد'} |

## `discuss.channel.member`

**`custom_notifications`** — {'en_US': 'Customized Notifications', 'tr_TR': 'Özelleştirilmiş Bildirimler', 'ar_001': 'الإشعارات المخصصة'}

| Value | Label |
|---|---|
| `all` | {'en_US': 'All Messages', 'tr_TR': 'Tüm Mesajlar', 'ar_001': 'كافة الرسائل'} |
| `mentions` | {'en_US': 'Mentions Only', 'tr_TR': 'Yalnızca Bahsedilenler', 'ar_001': 'الإشارات فقط'} |
| `no_notif` | {'en_US': 'Nothing', 'tr_TR': 'Hiçbir şey', 'ar_001': 'لا شيء'} |

**`livechat_member_type`** — {'en_US': 'Livechat Member Type', 'tr_TR': 'Canlı Sohbet  Üye Tipi', 'ar_001': 'نوع أعضاء الدردشة المباشرة'} (not stored)

| Value | Label |
|---|---|
| `agent` | {'en_US': 'Agent', 'tr_TR': 'Ajan', 'ar_001': 'الوكيل'} |
| `visitor` | {'en_US': 'Visitor', 'tr_TR': 'Ziyaretçi', 'ar_001': 'زائر'} |
| `bot` | {'en_US': 'Chatbot', 'tr_TR': 'Chatbot', 'ar_001': 'برنامج الدردشة الآلي'} |

## `event.booth`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `unavailable` | {'en_US': 'Unavailable', 'tr_TR': 'Kullanım dışı', 'ar_001': 'غير متاح'} |

## `event.event`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`badge_format`** — {'en_US': 'Badge Dimension', 'tr_TR': 'Yaka Kartı Boyutu', 'ar_001': 'أبعاد الشارة'}

| Value | Label |
|---|---|
| `A4_french_fold` | {'en_US': 'A4 foldable', 'tr_TR': 'A4 katlanabilir', 'ar_001': 'A4 قابلة للطي'} |
| `A6` | {'en_US': 'A6', 'tr_TR': 'A6', 'ar_001': 'A6'} |
| `four_per_sheet` | {'en_US': '4 per sheet', 'tr_TR': 'Sayfa başına 4', 'ar_001': '4 لكل ورقة'} |

**`kanban_state`** — {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Ready for Next Stage', 'tr_TR': 'Bir Sonraki Aşama için Hazır', 'ar_001': 'جاهز للمرحلة التالية'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`website_visibility`** — {'en_US': 'Website Visibility', 'tr_TR': 'Web Sitesi Görünürlüğü', 'ar_001': 'الظهور على الموقع الإلكتروني'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Public', 'tr_TR': 'Genel', 'ar_001': 'عام'} |
| `link` | {'en_US': 'Via a Link', 'tr_TR': 'Bir Bağlantı Üzerinden', 'ar_001': 'عن طريق الرابط'} |
| `logged_users` | {'en_US': 'Logged Users', 'tr_TR': 'Kayıtlı Kullanıcılar', 'ar_001': 'المستخدمون المسجلون'} |

## `event.lead.rule`

**`lead_creation_basis`** — {'en_US': 'Create', 'tr_TR': 'Oluştur', 'ar_001': 'إنشاء'}

| Value | Label |
|---|---|
| `attendee` | {'en_US': 'Per Attendee', 'tr_TR': 'Katılımcı Başına', 'ar_001': 'لكل حاضر'} |
| `order` | {'en_US': 'Per Order', 'tr_TR': 'Sipariş Başına', 'ar_001': 'لكل أمر'} |

**`lead_creation_trigger`** — {'en_US': 'When', 'tr_TR': 'Ne zaman', 'ar_001': 'الزمان'}

| Value | Label |
|---|---|
| `create` | {'en_US': 'Attendees are created', 'tr_TR': 'Katılımcılar oluşturuldu', 'ar_001': 'تم إنشاء الحاضرين'} |
| `confirm` | {'en_US': 'Attendees are registered', 'tr_TR': 'Katılımcılar kaydedildi', 'ar_001': 'الحاضرون مسجلون'} |
| `done` | {'en_US': 'Attendees attended', 'tr_TR': 'Katılan katılımcılar', 'ar_001': 'الحاضرين الذين حضروا'} |

**`lead_type`** — {'en_US': 'Lead Type', 'tr_TR': 'Teslim Türü', 'ar_001': 'نوع المهلة'}

| Value | Label |
|---|---|
| `lead` | {'en_US': 'Lead', 'tr_TR': 'Aday', 'ar_001': 'عميل مهتم'} |
| `opportunity` | {'en_US': 'Opportunity', 'tr_TR': 'Fırsat', 'ar_001': 'الفرصة'} |

## `event.mail`

**`interval_type`** — {'en_US': 'Trigger ', 'tr_TR': 'Tetikleyici ', 'ar_001': 'تشغيل '}

| Value | Label |
|---|---|
| `after_sub` | {'en_US': 'After each registration', 'tr_TR': 'Her kayıt sonrası', 'ar_001': 'بعد كل تسجيل'} |
| `before_event` | {'en_US': 'Before the event starts', 'tr_TR': 'Etkinlik başlamadan önce', 'ar_001': 'قبل بدء الفعالية'} |
| `after_event_start` | {'en_US': 'After the event started', 'tr_TR': 'Etkinlik başladıktan sonra', 'ar_001': 'بعد بدء الفعالية'} |
| `after_event` | {'en_US': 'After the event ended', 'tr_TR': 'Etkinlik sona erdikten sonra', 'ar_001': 'بعد انتهاء الفعالية'} |
| `before_event_end` | {'en_US': 'Before the event ends', 'tr_TR': 'Etkinlik sona ermeden önce', 'ar_001': 'قبل انتهاء الفعالية'} |

**`interval_unit`** — {'en_US': 'Unit', 'tr_TR': 'Birim', 'ar_001': 'الوحدة'}

| Value | Label |
|---|---|
| `now` | {'en_US': 'Immediately', 'tr_TR': 'Acil Olarak', 'ar_001': 'فورًا'} |
| `hours` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weeks` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |

**`mail_state`** — {'en_US': 'Global communication Status', 'tr_TR': 'Küresel iletişim Durumu', 'ar_001': 'حالة التواصل العالمي'} (not stored)

| Value | Label |
|---|---|
| `running` | {'en_US': 'Running', 'tr_TR': 'Devam Eden', 'ar_001': 'جاري'} |
| `scheduled` | {'en_US': 'Scheduled', 'tr_TR': 'Planlandı', 'ar_001': 'تمت الجدولة'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`notification_type`** — {'en_US': 'Send', 'tr_TR': 'Gönder', 'ar_001': 'إرسال'} (not stored)

| Value | Label |
|---|---|
| `mail` | {'en_US': 'Mail', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `sms` | {'en_US': 'SMS', 'tr_TR': 'SMS', 'ar_001': 'الرسائل النصية القصيرة'} |

## `event.question`

**`question_type`** — {'en_US': 'Question Type', 'tr_TR': 'Soru Tipi', 'ar_001': 'نوع السؤال'}

| Value | Label |
|---|---|
| `simple_choice` | {'en_US': 'Selection', 'tr_TR': 'Seçim', 'ar_001': 'قائمة خيارات'} |
| `text_box` | {'en_US': 'Text Input', 'tr_TR': 'Metin Girişi', 'ar_001': 'مدخلات النص'} |
| `name` | {'en_US': 'Name', 'tr_TR': 'Adı', 'ar_001': 'الاسم'} |
| `email` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `phone` | {'en_US': 'Phone', 'tr_TR': 'Telefon', 'ar_001': 'رقم الهاتف'} |
| `company_name` | {'en_US': 'Company', 'tr_TR': 'Firma', 'ar_001': 'الشركة'} |

## `event.registration`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`sale_status`** — {'en_US': 'Sale Status', 'tr_TR': 'Satış Durumu', 'ar_001': 'حالة البيع'}

| Value | Label |
|---|---|
| `to_pay` | {'en_US': 'Not Sold', 'tr_TR': 'Satılmadı', 'ar_001': 'لم تباع'} |
| `sold` | {'en_US': 'Sold', 'tr_TR': 'Satılan', 'ar_001': 'المبيعات'} |
| `free` | {'en_US': 'Free', 'tr_TR': 'Ücretsiz', 'ar_001': 'مجاني'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Unconfirmed', 'tr_TR': 'Onaysız', 'ar_001': 'غير مؤكد'} |
| `open` | {'en_US': 'Registered', 'tr_TR': 'Kayıtlı', 'ar_001': 'مُسجل'} |
| `done` | {'en_US': 'Attended', 'tr_TR': 'Katıldı', 'ar_001': 'حضر'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `event.sale.report`

**`event_registration_state`** — {'en_US': 'Registration Status', 'tr_TR': 'Kayıt durumu', 'ar_001': 'حالة التسجيل'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Unconfirmed', 'tr_TR': 'Onaysız', 'ar_001': 'غير مؤكد'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `open` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `done` | {'en_US': 'Attended', 'tr_TR': 'Katıldı', 'ar_001': 'حضر'} |

**`sale_order_state`** — {'en_US': 'Sale Order Status', 'tr_TR': 'Satış Siparişi Durumu', 'ar_001': 'حالة أمر البيع'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Quotation', 'tr_TR': 'Teklif', 'ar_001': 'عرض سعر'} |
| `sent` | {'en_US': 'Quotation Sent', 'tr_TR': 'Teklif Gönderildi', 'ar_001': 'تم إرسال عرض السعر'} |
| `sale` | {'en_US': 'Sales Order', 'tr_TR': 'Satış Siparişi', 'ar_001': 'أمر البيع'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`sale_status`** — {'en_US': 'Payment Status', 'tr_TR': 'Ödeme Durumu', 'ar_001': 'حالة الدفع'}

| Value | Label |
|---|---|
| `to_pay` | {'en_US': 'Not Sold', 'tr_TR': 'Satılmadı', 'ar_001': 'لم تباع'} |
| `sold` | {'en_US': 'Sold', 'tr_TR': 'Satılan', 'ar_001': 'المبيعات'} |
| `free` | {'en_US': 'Free', 'tr_TR': 'Ücretsiz', 'ar_001': 'مجاني'} |

## `event.sponsor`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`exhibitor_type`** — {'en_US': 'Sponsor Type', 'tr_TR': 'Sponsor Tipi', 'ar_001': 'نوع الراعي'}

| Value | Label |
|---|---|
| `sponsor` | {'en_US': 'Footer Logo Only', 'tr_TR': 'Yalnızca Altbilgi Logosu', 'ar_001': 'شعار التذييل فقط'} |
| `exhibitor` | {'en_US': 'Exhibitor', 'tr_TR': 'Sergileyici', 'ar_001': 'العارض'} |
| `online` | {'en_US': 'Online Exhibitor', 'tr_TR': 'Çevrimiçi Sergileyici', 'ar_001': 'عارض عبر الإنترنت'} |

## `event.sponsor.type`

**`display_ribbon_style`** — {'en_US': 'Ribbon Style', 'tr_TR': 'Şerit Stili', 'ar_001': 'شكل الشريطة'}

| Value | Label |
|---|---|
| `no_ribbon` | {'en_US': 'No Ribbon', 'tr_TR': 'Şerit Yok', 'ar_001': 'لا توجد شريطة'} |
| `Gold` | {'en_US': 'Gold', 'tr_TR': 'Altın', 'ar_001': 'ذهبي'} |
| `Silver` | {'en_US': 'Silver', 'tr_TR': 'Gümüş', 'ar_001': 'فضي'} |
| `Bronze` | {'en_US': 'Bronze', 'tr_TR': 'Bronz', 'ar_001': 'برونزي'} |

## `event.track`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`kanban_state`** — {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Grey', 'tr_TR': 'Gri', 'ar_001': 'رمادي'} |
| `done` | {'en_US': 'Green', 'tr_TR': 'Yeşil', 'ar_001': 'أخضر'} |
| `blocked` | {'en_US': 'Red', 'tr_TR': 'Kırmızı', 'ar_001': 'أحمر'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Low', 'tr_TR': 'Düşük', 'ar_001': 'منخفض'} |
| `1` | {'en_US': 'Medium', 'tr_TR': 'Aracı:', 'ar_001': 'متوسط'} |
| `2` | {'en_US': 'High', 'tr_TR': 'Yüksek', 'ar_001': 'مرتفع'} |
| `3` | {'en_US': 'Highest', 'tr_TR': 'En yüksek', 'ar_001': 'الأعلى'} |

## `event.type.mail`

**`interval_type`** — {'en_US': 'Trigger', 'tr_TR': 'Tetikleyici', 'ar_001': 'المشغّل'}

| Value | Label |
|---|---|
| `after_sub` | {'en_US': 'After each registration', 'tr_TR': 'Her kayıt sonrası', 'ar_001': 'بعد كل تسجيل'} |
| `before_event` | {'en_US': 'Before the event starts', 'tr_TR': 'Etkinlik başlamadan önce', 'ar_001': 'قبل بدء الفعالية'} |
| `after_event_start` | {'en_US': 'After the event started', 'tr_TR': 'Etkinlik başladıktan sonra', 'ar_001': 'بعد بدء الفعالية'} |
| `after_event` | {'en_US': 'After the event ended', 'tr_TR': 'Etkinlik sona erdikten sonra', 'ar_001': 'بعد انتهاء الفعالية'} |
| `before_event_end` | {'en_US': 'Before the event ends', 'tr_TR': 'Etkinlik sona ermeden önce', 'ar_001': 'قبل انتهاء الفعالية'} |

**`interval_unit`** — {'en_US': 'Unit', 'tr_TR': 'Birim', 'ar_001': 'الوحدة'}

| Value | Label |
|---|---|
| `now` | {'en_US': 'Immediately', 'tr_TR': 'Acil Olarak', 'ar_001': 'فورًا'} |
| `hours` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weeks` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |

**`notification_type`** — {'en_US': 'Send', 'tr_TR': 'Gönder', 'ar_001': 'إرسال'} (not stored)

| Value | Label |
|---|---|
| `mail` | {'en_US': 'Mail', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `sms` | {'en_US': 'SMS', 'tr_TR': 'SMS', 'ar_001': 'الرسائل النصية القصيرة'} |

## `fetchmail.server`

**`server_type`** — {'en_US': 'Server Type', 'tr_TR': 'Sunucu Türü', 'ar_001': 'نوع الخادم'}

| Value | Label |
|---|---|
| `imap` | {'en_US': 'IMAP Server', 'tr_TR': 'IMAP Sunucusu', 'ar_001': 'خادم IMAP'} |
| `pop` | {'en_US': 'POP Server', 'tr_TR': 'POP Sunucusu', 'ar_001': 'خادم POP'} |
| `local` | {'en_US': 'Local Server', 'tr_TR': 'Yerel Sunucu', 'ar_001': 'الخادم المحلي'} |
| `gmail` | {'en_US': 'Gmail OAuth Authentication', 'tr_TR': 'Gmail OAuth Kimlik Doğrulaması', 'ar_001': 'مصادقة Gmail OAuth'} |
| `outlook` | {'en_US': 'Outlook OAuth Authentication', 'tr_TR': 'Outlook OAuth Kimlik Doğrulaması', 'ar_001': 'مصادقة Outlook OAuth'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Not Confirmed', 'tr_TR': 'Onaylanmadı', 'ar_001': 'غير مؤكد'} |
| `done` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |

## `fleet.service.type`

**`category`** — {'en_US': 'Category', 'tr_TR': 'Kategori', 'ar_001': 'الفئة'}

| Value | Label |
|---|---|
| `contract` | {'en_US': 'Contract', 'tr_TR': 'Sözleşme', 'ar_001': 'العقد'} |
| `service` | {'en_US': 'Service', 'tr_TR': 'Hizmet', 'ar_001': 'الخدمة'} |

## `fleet.vehicle`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`co2_emission_unit`** — {'en_US': 'Co2 Emission Unit', 'tr_TR': 'Karbon Emisyonları Birimi', 'ar_001': 'وحدة انبعاثات ثاني أكسيد الكربون'}

| Value | Label |
|---|---|
| `g/km` | {'en_US': 'g/km', 'tr_TR': 'g/km', 'ar_001': 'ج/كم'} |
| `g/mi` | {'en_US': 'g/mi', 'tr_TR': 'g/mi', 'ar_001': 'ج/ميل'} |

**`contract_state`** — {'en_US': 'Last Contract State', 'tr_TR': 'Son Sözleşme Durumu', 'ar_001': 'حالة آخر عقد'} (not stored)

| Value | Label |
|---|---|
| `futur` | {'en_US': 'Incoming', 'tr_TR': 'Gelen', 'ar_001': 'واردة'} |
| `open` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `closed` | {'en_US': 'Closed', 'tr_TR': 'Kapanmış', 'ar_001': 'مغلق'} |

**`frame_type`** — {'en_US': 'Bike Frame Type', 'tr_TR': 'Bisiklet Çerçeve Tipi', 'ar_001': 'نوع إطار الدراجة'}

| Value | Label |
|---|---|
| `diamant` | {'en_US': 'Diamant', 'tr_TR': 'Elmas', 'ar_001': 'ديامانت'} |
| `trapez` | {'en_US': 'Trapez', 'tr_TR': 'Trapez', 'ar_001': 'Trapez'} |
| `wave` | {'en_US': 'Wave', 'tr_TR': 'Grup', 'ar_001': 'مموج'} |

**`fuel_type`** — {'en_US': 'Fuel Type', 'tr_TR': 'Yakıt Türü', 'ar_001': 'نوع الوقود'}

| Value | Label |
|---|---|
| `diesel` | {'en_US': 'Diesel', 'tr_TR': 'Dizel', 'ar_001': 'ديزل'} |
| `gasoline` | {'en_US': 'Gasoline', 'tr_TR': 'Benzin', 'ar_001': 'بنزين'} |
| `full_hybrid` | {'en_US': 'Full Hybrid', 'tr_TR': 'Tam Hibrit', 'ar_001': 'هجين كامل'} |
| `plug_in_hybrid_diesel` | {'en_US': 'Plug-in Hybrid Diesel', 'tr_TR': 'Eklenti Hibrit Dizel', 'ar_001': 'ديزل لمركبة هجينة قابلة للشحن الخارجي'} |
| `plug_in_hybrid_gasoline` | {'en_US': 'Plug-in Hybrid Gasoline', 'tr_TR': 'Eklenti Hibrit Benzinli', 'ar_001': 'غازولين لمركبة هجينة قابلة للشحن الخارجي'} |
| `cng` | {'en_US': 'CNG', 'tr_TR': 'CNG', 'ar_001': 'CNG'} |
| `lpg` | {'en_US': 'LPG', 'tr_TR': 'LPG', 'ar_001': 'غاز نفطي مسال'} |
| `hydrogen` | {'en_US': 'Hydrogen', 'tr_TR': 'Hidrojen', 'ar_001': 'هيدروجين'} |
| `electric` | {'en_US': 'Electric', 'tr_TR': 'Elektrikli', 'ar_001': 'كهربائية'} |

**`odometer_unit`** — {'en_US': 'Odometer Unit', 'tr_TR': 'KilometreSayaç Ünitesi', 'ar_001': 'وحدة عداد المسافات'}

| Value | Label |
|---|---|
| `kilometers` | {'en_US': 'km', 'tr_TR': 'km', 'ar_001': 'كم'} |
| `miles` | {'en_US': 'mi', 'tr_TR': 'mil', 'ar_001': 'ميل'} |

**`power_unit`** — {'en_US': 'Power Unit', 'tr_TR': 'Güç Ünitesi', 'ar_001': 'وحدة قياس القوة'}

| Value | Label |
|---|---|
| `power` | {'en_US': 'kW', 'tr_TR': 'kW', 'ar_001': 'kW'} |
| `horsepower` | {'en_US': 'Horsepower', 'tr_TR': 'BeygirGücü', 'ar_001': 'قوة حصان'} |

**`range_unit`** — {'en_US': 'Range Unit', 'tr_TR': 'Menzil Birimi', 'ar_001': 'وحدة النطاق'}

| Value | Label |
|---|---|
| `km` | {'en_US': 'km', 'tr_TR': 'km', 'ar_001': 'كم'} |
| `mi` | {'en_US': 'mi', 'tr_TR': 'mil', 'ar_001': 'ميل'} |

**`service_activity`** — {'en_US': 'Service Activity', 'tr_TR': 'Servis Aktivitesi', 'ar_001': 'نشاط الخدمة'} (not stored)

| Value | Label |
|---|---|
| `none` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |

**`transmission`** — {'en_US': 'Transmission', 'tr_TR': 'Şanzıman', 'ar_001': 'ناقل الحركة'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `automatic` | {'en_US': 'Automatic', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |

## `fleet.vehicle.cost.report`

**`cost_type`** — {'en_US': 'Cost Type', 'tr_TR': 'Maliyet Türü', 'ar_001': 'نوع التكلفة'}

| Value | Label |
|---|---|
| `contract` | {'en_US': 'Contract', 'tr_TR': 'Sözleşme', 'ar_001': 'العقد'} |
| `service` | {'en_US': 'Service', 'tr_TR': 'Hizmet', 'ar_001': 'الخدمة'} |

**`vehicle_type`** — {'en_US': 'Vehicle Type', 'tr_TR': 'Araç Türü', 'ar_001': 'نوع المركبة'}

| Value | Label |
|---|---|
| `car` | {'en_US': 'Car', 'tr_TR': 'Car', 'ar_001': 'السيارة'} |
| `bike` | {'en_US': 'Bike', 'tr_TR': 'Bisiklet', 'ar_001': 'الدراجة'} |

## `fleet.vehicle.log.contract`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`cost_frequency`** — {'en_US': 'Recurring Cost Frequency', 'tr_TR': 'Yinelenen Maliyet Sıklığı', 'ar_001': 'مدى تواتر التكلفة الدورية'}

| Value | Label |
|---|---|
| `no` | {'en_US': 'No', 'tr_TR': 'Hayır', 'ar_001': 'لا'} |
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `weekly` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |
| `yearly` | {'en_US': 'Yearly', 'tr_TR': 'Yıllık', 'ar_001': 'سنويًا'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `futur` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `open` | {'en_US': 'Running', 'tr_TR': 'Devam Eden', 'ar_001': 'جاري'} |
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `closed` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `fleet.vehicle.log.services`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`state`** — {'en_US': 'Stage', 'tr_TR': 'Aşama', 'ar_001': 'المرحلة'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `running` | {'en_US': 'Running', 'tr_TR': 'Devam Eden', 'ar_001': 'جاري'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `fleet.vehicle.model`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`co2_emission_unit`** — {'en_US': 'Co2 Emission Unit', 'tr_TR': 'Karbon Emisyonları Birimi', 'ar_001': 'وحدة انبعاثات ثاني أكسيد الكربون'} (not stored)

| Value | Label |
|---|---|
| `g/km` | {'en_US': 'g/km', 'tr_TR': 'g/km', 'ar_001': 'ج/كم'} |
| `g/mi` | {'en_US': 'g/mi', 'tr_TR': 'g/mi', 'ar_001': 'ج/ميل'} |

**`default_fuel_type`** — {'en_US': 'Fuel Type', 'tr_TR': 'Yakıt Türü', 'ar_001': 'نوع الوقود'}

| Value | Label |
|---|---|
| `diesel` | {'en_US': 'Diesel', 'tr_TR': 'Dizel', 'ar_001': 'ديزل'} |
| `gasoline` | {'en_US': 'Gasoline', 'tr_TR': 'Benzin', 'ar_001': 'بنزين'} |
| `full_hybrid` | {'en_US': 'Full Hybrid', 'tr_TR': 'Tam Hibrit', 'ar_001': 'هجين كامل'} |
| `plug_in_hybrid_diesel` | {'en_US': 'Plug-in Hybrid Diesel', 'tr_TR': 'Eklenti Hibrit Dizel', 'ar_001': 'ديزل لمركبة هجينة قابلة للشحن الخارجي'} |
| `plug_in_hybrid_gasoline` | {'en_US': 'Plug-in Hybrid Gasoline', 'tr_TR': 'Eklenti Hibrit Benzinli', 'ar_001': 'غازولين لمركبة هجينة قابلة للشحن الخارجي'} |
| `cng` | {'en_US': 'CNG', 'tr_TR': 'CNG', 'ar_001': 'CNG'} |
| `lpg` | {'en_US': 'LPG', 'tr_TR': 'LPG', 'ar_001': 'غاز نفطي مسال'} |
| `hydrogen` | {'en_US': 'Hydrogen', 'tr_TR': 'Hidrojen', 'ar_001': 'هيدروجين'} |
| `electric` | {'en_US': 'Electric', 'tr_TR': 'Elektrikli', 'ar_001': 'كهربائية'} |

**`drive_type`** — {'en_US': 'Drive Type', 'tr_TR': 'Sürüş Tipi', 'ar_001': 'نوع القيادة'}

| Value | Label |
|---|---|
| `fwd` | {'en_US': 'Front-Wheel Drive (FWD)', 'tr_TR': 'Önden Çekişli (FWD)', 'ar_001': 'نظام الدفع بالعجلات الأمامية (FWD)'} |
| `awd` | {'en_US': 'All-Wheel Drive (AWD)', 'tr_TR': 'Dört Tekerlekten Çekiş (AWD)', 'ar_001': 'نظام الدفع بكل العجلات (AWD)'} |
| `rwd` | {'en_US': 'Rear-Wheel Drive (RWD)', 'tr_TR': 'Arkadan İtişli (RWD)', 'ar_001': 'نظام الدفع بالعجلات الخلفية (RWD)'} |
| `4wd` | {'en_US': 'Four-Wheel Drive (4WD)', 'tr_TR': 'Dört Tekerlekten Çekiş (4WD)', 'ar_001': 'الدفع رباعي العجلات (4WD)'} |

**`power_unit`** — {'en_US': 'Power Unit', 'tr_TR': 'Güç Ünitesi', 'ar_001': 'وحدة قياس القوة'}

| Value | Label |
|---|---|
| `power` | {'en_US': 'kW', 'tr_TR': 'kW', 'ar_001': 'kW'} |
| `horsepower` | {'en_US': 'Horsepower (hp)', 'tr_TR': 'Beygir gücü (hp)', 'ar_001': 'قوة حصان (hp)'} |

**`range_unit`** — {'en_US': 'Range Unit', 'tr_TR': 'Menzil Birimi', 'ar_001': 'وحدة النطاق'}

| Value | Label |
|---|---|
| `km` | {'en_US': 'km', 'tr_TR': 'km', 'ar_001': 'كم'} |
| `mi` | {'en_US': 'mi', 'tr_TR': 'mil', 'ar_001': 'ميل'} |

**`transmission`** — {'en_US': 'Transmission', 'tr_TR': 'Şanzıman', 'ar_001': 'ناقل الحركة'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `automatic` | {'en_US': 'Automatic', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |

**`vehicle_type`** — {'en_US': 'Vehicle Type', 'tr_TR': 'Araç Türü', 'ar_001': 'نوع المركبة'}

| Value | Label |
|---|---|
| `car` | {'en_US': 'Car', 'tr_TR': 'Car', 'ar_001': 'السيارة'} |
| `bike` | {'en_US': 'Bike', 'tr_TR': 'Bisiklet', 'ar_001': 'الدراجة'} |

## `forum.forum`

**`default_order`** — {'en_US': 'Default', 'tr_TR': 'Öntanımlı', 'ar_001': 'افتراضي'}

| Value | Label |
|---|---|
| `create_date desc` | {'en_US': 'Newest', 'tr_TR': 'En Yeni', 'ar_001': 'الأحدث'} |
| `last_activity_date desc` | {'en_US': 'Last Updated', 'tr_TR': 'Son Güncelleme', 'ar_001': 'آخر تحديث'} |
| `vote_count desc` | {'en_US': 'Most Voted', 'tr_TR': 'En fazla oylanan', 'ar_001': 'الأكثر تصويتًا'} |
| `relevancy desc` | {'en_US': 'Relevance', 'tr_TR': 'İlgili', 'ar_001': 'الصلة'} |
| `child_count desc` | {'en_US': 'Answered', 'tr_TR': 'Yanıtlanan', 'ar_001': 'تمت الإجابة'} |

**`mode`** — {'en_US': 'Mode', 'tr_TR': 'Şekli', 'ar_001': 'الوضع'}

| Value | Label |
|---|---|
| `questions` | {'en_US': 'Questions (1 answer)', 'tr_TR': 'Sorular (1 cevap)', 'ar_001': 'الأسئلة (إجابة واحدة)'} |
| `discussions` | {'en_US': 'Discussions (multiple answers)', 'tr_TR': 'Tartışmalar (birden fazla cevap)', 'ar_001': 'المناقشات (عدة إجابات)'} |

**`privacy`** — {'en_US': 'Privacy', 'tr_TR': 'Özel', 'ar_001': 'الخصوصية'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Public', 'tr_TR': 'Genel', 'ar_001': 'عام'} |
| `connected` | {'en_US': 'Signed In', 'tr_TR': 'Giriş', 'ar_001': 'تم تسجيل الدخول'} |
| `private` | {'en_US': 'Some users', 'tr_TR': 'Bazı kullanıcılar', 'ar_001': 'بعض المستخدمين'} |

## `forum.post`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `active` | {'en_US': 'Active', 'tr_TR': 'Etkin', 'ar_001': 'نشط'} |
| `pending` | {'en_US': 'Waiting Validation', 'tr_TR': 'Bekleyen Doğrulama', 'ar_001': 'بانتظار التصديق'} |
| `close` | {'en_US': 'Closed', 'tr_TR': 'Kapanmış', 'ar_001': 'مغلق'} |
| `offensive` | {'en_US': 'Offensive', 'tr_TR': 'Saldırgan', 'ar_001': 'مسيء'} |
| `flagged` | {'en_US': 'Flagged', 'tr_TR': 'İşaretli', 'ar_001': 'مُعلّم بعلامة'} |

## `forum.post.reason`

**`reason_type`** — {'en_US': 'Reason Type', 'tr_TR': 'Sebep Türü', 'ar_001': 'نوع السبب'}

| Value | Label |
|---|---|
| `basic` | {'en_US': 'Basic', 'tr_TR': 'Temel', 'ar_001': 'الأساسي'} |
| `offensive` | {'en_US': 'Offensive', 'tr_TR': 'Saldırgan', 'ar_001': 'مسيء'} |

## `forum.post.vote`

**`vote`** — {'en_US': 'Vote', 'tr_TR': 'Oy ver', 'ar_001': 'التصويت'}

| Value | Label |
|---|---|
| `1` | {'en_US': '1'} |
| `-1` | {'en_US': '-1'} |
| `0` | {'en_US': '0'} |

## `gamification.badge`

**`level`** — {'en_US': 'Forum Badge Level', 'tr_TR': 'Forum Rozet Seviye', 'ar_001': 'مستوى شارة المنتدى'}

| Value | Label |
|---|---|
| `bronze` | {'en_US': 'Bronze', 'tr_TR': 'Bronz', 'ar_001': 'برونزي'} |
| `silver` | {'en_US': 'Silver', 'tr_TR': 'Gümüş', 'ar_001': 'فضي'} |
| `gold` | {'en_US': 'Gold', 'tr_TR': 'Altın', 'ar_001': 'ذهبي'} |

**`rule_auth`** — {'en_US': 'Allowance to Grant', 'tr_TR': 'Verme Yetkisi', 'ar_001': 'المصروف لمنحه'}

| Value | Label |
|---|---|
| `everyone` | {'en_US': 'Everyone', 'tr_TR': 'Herkes', 'ar_001': 'الجميع'} |
| `users` | {'en_US': 'A selected list of users', 'tr_TR': 'Seçilmiş kullanıcılar listesi', 'ar_001': 'قائمة مختارة من المستخدمين'} |
| `having` | {'en_US': 'People having some badges', 'tr_TR': 'Bazı rozetlere sahip kişiler', 'ar_001': 'الأشخاص الذين يملكون بعض الشارات'} |
| `nobody` | {'en_US': 'No one, assigned through challenges', 'tr_TR': 'Hiç kimse, yarışmalar üzerinden atanır', 'ar_001': 'لا أحد، يتم تعيينه من خلال التحديات'} |

## `gamification.challenge`

**`challenge_category`** — {'en_US': 'Appears in', 'tr_TR': 'Görüneceği Yer', 'ar_001': 'يظهر في'}

| Value | Label |
|---|---|
| `hr` | {'en_US': 'Human Resources / Engagement', 'tr_TR': 'İnsan Kaynakları / Bağlılık', 'ar_001': 'الموارد البشرية/المشاركة'} |
| `other` | {'en_US': 'Settings / Gamification Tools', 'tr_TR': 'Ayarlar / Ödüllendirme Araçları', 'ar_001': 'الإعدادات/أدوات التلعيب'} |
| `certification` | {'en_US': 'Certifications', 'tr_TR': 'Sertifikalar', 'ar_001': 'الشهادات'} |
| `forum` | {'en_US': 'Website / Forum', 'tr_TR': 'Web Sitesi / Forum', 'ar_001': 'الموقع الإلكتروني/المنتدى'} |
| `slides` | {'en_US': 'Website / Slides', 'tr_TR': 'Web sitesi / Slaytlar', 'ar_001': 'الموقع الإلكتروني / الشرائح'} |

**`period`** — {'en_US': 'Periodicity', 'tr_TR': 'Dönemsellik', 'ar_001': 'الوتيرة'}

| Value | Label |
|---|---|
| `once` | {'en_US': 'Non recurring', 'tr_TR': 'Yenilemesiz', 'ar_001': 'غير متكرر'} |
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `weekly` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |
| `yearly` | {'en_US': 'Yearly', 'tr_TR': 'Yıllık', 'ar_001': 'سنويًا'} |

**`report_message_frequency`** — {'en_US': 'Report Frequency', 'tr_TR': 'Rapor Sıklığı', 'ar_001': 'تواتر التقارير'}

| Value | Label |
|---|---|
| `never` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقًا'} |
| `onchange` | {'en_US': 'On change', 'tr_TR': 'Değiştiğinde', 'ar_001': 'عند التغيير'} |
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `weekly` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |
| `yearly` | {'en_US': 'Yearly', 'tr_TR': 'Yıllık', 'ar_001': 'سنويًا'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `inprogress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

**`visibility_mode`** — {'en_US': 'Display Mode', 'tr_TR': 'Görünüm Modu', 'ar_001': 'وضع العرض'}

| Value | Label |
|---|---|
| `personal` | {'en_US': 'Individual Goals', 'tr_TR': 'Bireysel Hedefler', 'ar_001': 'الأهداف الفردية'} |
| `ranking` | {'en_US': 'Leader Board (Group Ranking)', 'tr_TR': 'Lider Tablosu (Grup Sıralaması)', 'ar_001': 'لوحة الصدارة (التصنيف الجماعي)'} |

## `gamification.goal`

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `inprogress` | {'en_US': 'In progress', 'tr_TR': 'Devam Eden', 'ar_001': 'قيد التنفيذ'} |
| `reached` | {'en_US': 'Reached', 'tr_TR': 'Ulaşıldı', 'ar_001': 'تم الوصول'} |
| `failed` | {'en_US': 'Failed', 'tr_TR': 'Başarısız', 'ar_001': 'فشل'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `gamification.goal.definition`

**`computation_mode`** — {'en_US': 'Computation Mode', 'tr_TR': 'Hesaplama Modu', 'ar_001': 'طريقة الاحتساب'}

| Value | Label |
|---|---|
| `manually` | {'en_US': 'Recorded manually', 'tr_TR': 'Manuel kaydedilmiş', 'ar_001': 'تم التسجيل يدويًا'} |
| `count` | {'en_US': 'Automatic: number of records', 'tr_TR': 'Otomatik: kayıtların sayısı', 'ar_001': 'تلقائي: عدد من السجلات'} |
| `sum` | {'en_US': 'Automatic: sum on a field', 'tr_TR': 'Otomatik: Bir alanın toplamı', 'ar_001': 'تلقائي: المجموع في حقل'} |
| `python` | {'en_US': 'Automatic: execute a specific Python code', 'tr_TR': 'Otomatik: belirli bir Python kodu çalıştırma', 'ar_001': 'تلقائي: تنفيذ كود بايثون معين'} |

**`condition`** — {'en_US': 'Goal Performance', 'tr_TR': 'Hedef Performansı', 'ar_001': 'أداء الهدف'}

| Value | Label |
|---|---|
| `higher` | {'en_US': 'The higher the better', 'tr_TR': 'Yüksek daha iyi', 'ar_001': 'كلما كانت القيمة أعلى كان ذلك أفضل'} |
| `lower` | {'en_US': 'The lower the better', 'tr_TR': 'Düşük daha iyi', 'ar_001': 'كلما كانت القيمة أقل كان ذلك أفضل'} |

**`display_mode`** — {'en_US': 'Displayed as', 'tr_TR': 'Olarak Görüntüle', 'ar_001': 'معروض كـ'}

| Value | Label |
|---|---|
| `progress` | {'en_US': 'Progressive (using numerical values)', 'tr_TR': 'İlerleyen (sayısal değerleri kullanarak)', 'ar_001': 'التقدمية (باستخدام القيم العددية)'} |
| `boolean` | {'en_US': 'Exclusive (done or not-done)', 'tr_TR': 'Özel (bitmiş veya bitmemiş)', 'ar_001': 'حصري (المنتهي أو غير المنتهي)'} |

## `google.calendar.account.reset`

**`delete_policy`** — {'en_US': "User's Existing Events", 'tr_TR': 'Kullanıcının Mevcut Etkinlikleri', 'ar_001': 'فعاليات المستخدم الموجودة بالفعل'}

| Value | Label |
|---|---|
| `dont_delete` | {'en_US': 'Leave them untouched', 'tr_TR': 'Onları el değmeden bırakın', 'ar_001': 'اتركهم كما هم'} |
| `delete_google` | {'en_US': 'Delete from the current Google Calendar account', 'tr_TR': 'Delete from the current Google Calendar account', 'ar_001': 'الحذف من حساب تقويم Google الحالي'} |
| `delete_odoo` | {'en_US': 'Delete from Odoo', 'tr_TR': "Odoo'dan sil", 'ar_001': 'الحذف من أودو'} |
| `delete_both` | {'en_US': 'Delete from both', 'tr_TR': 'İkisinden de sil', 'ar_001': 'الحذف من كليهما'} |

**`sync_policy`** — {'en_US': 'Next Synchronization', 'tr_TR': 'Sonraki Senkronizasyon', 'ar_001': 'المزامنة التالية'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'Synchronize only new events', 'tr_TR': 'Yalnızca yeni etkinlikleri senkronize et', 'ar_001': 'مزامنة الفعاليات الجديدة فقط'} |
| `all` | {'en_US': 'Synchronize all existing events', 'tr_TR': 'Mevcut tüm etkinlikleri senkronize et', 'ar_001': 'مزامنة كافة الفعاليات الموجودة'} |

## `hr.applicant`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`application_status`** — {'en_US': 'Application Status', 'tr_TR': 'Başvuru Durumu', 'ar_001': 'حالة طلب التقديم'} (not stored)

| Value | Label |
|---|---|
| `ongoing` | {'en_US': 'Ongoing', 'tr_TR': 'Devam eden', 'ar_001': 'جاري'} |
| `hired` | {'en_US': 'Hired', 'tr_TR': 'İşe Alınan', 'ar_001': 'تم التعيين'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `archived` | {'en_US': 'Archived', 'tr_TR': 'Arşivlendi', 'ar_001': 'مؤرشف'} |

**`kanban_state`** — {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Ready for Next Stage', 'tr_TR': 'Bir Sonraki Aşama için Hazır', 'ar_001': 'جاهز للمرحلة التالية'} |
| `waiting` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |

**`priority`** — {'en_US': 'Evaluation', 'tr_TR': 'Değerlendirme', 'ar_001': 'التقييم'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `1` | {'en_US': 'Good', 'tr_TR': 'İyi', 'ar_001': 'جيد'} |
| `2` | {'en_US': 'Very Good', 'tr_TR': 'Çok İyi', 'ar_001': 'جيد جداً'} |
| `3` | {'en_US': 'Excellent', 'tr_TR': 'Mükemmel', 'ar_001': 'ممتاز'} |

## `hr.attendance`

**`in_mode`** — {'en_US': 'Mode', 'tr_TR': 'Şekli', 'ar_001': 'الوضع'}

| Value | Label |
|---|---|
| `kiosk` | {'en_US': 'Kiosk', 'tr_TR': 'Dijital sipariş ekranı', 'ar_001': 'كشك'} |
| `systray` | {'en_US': 'Systray', 'tr_TR': 'Sistem Çubuğu', 'ar_001': 'Systray'} |
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `technical` | {'en_US': 'Technical', 'tr_TR': 'Teknik', 'ar_001': 'تقني'} |

**`out_mode`** — {'en_US': 'Out Mode', 'tr_TR': 'Dış Mod', 'ar_001': 'الوضع في الخارج'}

| Value | Label |
|---|---|
| `kiosk` | {'en_US': 'Kiosk', 'tr_TR': 'Dijital sipariş ekranı', 'ar_001': 'كشك'} |
| `systray` | {'en_US': 'Systray', 'tr_TR': 'Sistem Çubuğu', 'ar_001': 'Systray'} |
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `technical` | {'en_US': 'Technical', 'tr_TR': 'Teknik', 'ar_001': 'تقني'} |
| `auto_check_out` | {'en_US': 'Automatic Check-Out', 'tr_TR': 'Otomatik Çıkış', 'ar_001': 'تسجيل خروج الموظفين تلقائياً'} |

**`overtime_status`** — {'en_US': 'Overtime Status', 'tr_TR': 'Fazla Mesai Durumu', 'ar_001': 'حالة ساعات العمل الإصافية'}

| Value | Label |
|---|---|
| `to_approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

## `hr.attendance.overtime.line`

**`status`** — {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `to_approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

## `hr.attendance.overtime.rule`

**`base_off`** — {'en_US': 'Based Off', 'tr_TR': 'Dayanarak'}

| Value | Label |
|---|---|
| `quantity` | {'en_US': 'Quantity', 'tr_TR': 'Miktar', 'ar_001': 'الكمية'} |
| `timing` | {'en_US': 'Timing', 'tr_TR': 'Vardiya', 'ar_001': 'التوقيت'} |

**`quantity_period`** — {'en_US': 'Quantity Period', 'tr_TR': 'Miktar Dönem'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Day', 'tr_TR': 'Gün', 'ar_001': 'اليوم'} |
| `week` | {'en_US': 'Week', 'tr_TR': 'Hafta', 'ar_001': 'الأسبوع'} |

**`timing_type`** — {'en_US': 'Timing Type', 'tr_TR': 'Zamanlama Tipi'}

| Value | Label |
|---|---|
| `work_days` | {'en_US': 'On any working day', 'tr_TR': 'Herhangi bir iş gününde'} |
| `non_work_days` | {'en_US': 'On any non-working day', 'tr_TR': 'Çalışılmayan herhangi bir günde'} |
| `leave` | {'en_US': 'When employee is off', 'tr_TR': 'Çalışan izinli olduğunda'} |
| `schedule` | {'en_US': 'Outside of a specific schedule', 'tr_TR': 'Belirli bir programın dışında'} |

## `hr.attendance.overtime.ruleset`

**`rate_combination_mode`** — {'en_US': 'Rate Combination Mode', 'tr_TR': 'Derecelendirme Kombinasyon Modu'}

| Value | Label |
|---|---|
| `max` | {'en_US': 'Maximum Rate', 'tr_TR': 'Maksimum Oran'} |
| `sum` | {'en_US': 'Sum of all rates', 'tr_TR': 'Tüm oranların toplamı'} |

## `hr.department`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

## `hr.employee`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`attendance_state`** — {'en_US': 'Attendance Status', 'tr_TR': 'Katılım Durumu', 'ar_001': 'حالة الحضور'} (not stored)

| Value | Label |
|---|---|
| `checked_out` | {'en_US': 'Checked out', 'tr_TR': 'Çıkış Yapıldı', 'ar_001': 'تم تسجيل الخروج'} |
| `checked_in` | {'en_US': 'Checked in', 'tr_TR': 'Giriş Yapıldı', 'ar_001': 'تم تسجيل الحضور'} |

**`current_leave_state`** — {'en_US': 'Current Time Off Status', 'tr_TR': 'Mevcut İzin Durumu', 'ar_001': 'حالة الإجازة الحالية'} (not stored)

| Value | Label |
|---|---|
| `confirm` | {'en_US': 'Waiting Approval', 'tr_TR': 'Onay Bekliyor', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Waiting Second Approval', 'tr_TR': 'İkinci Onayı Bekliyor', 'ar_001': 'بانتظار الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`hr_icon_display`** — {'en_US': 'Hr Icon Display', 'tr_TR': 'Hr Icon Display', 'ar_001': 'عرض أيقونة الموارد البشرية'} (not stored)

| Value | Label |
|---|---|
| `presence_present` | {'en_US': 'Present', 'tr_TR': 'Mevcut', 'ar_001': 'حاضر'} |
| `presence_out_of_working_hour` | {'en_US': 'Off-Hours', 'tr_TR': 'Mesai Dışı', 'ar_001': 'خارج ساعات العمل'} |
| `presence_absent` | {'en_US': 'Absent', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غائب'} |
| `presence_archive` | {'en_US': 'Archived', 'tr_TR': 'Arşivlendi', 'ar_001': 'مؤرشف'} |
| `presence_undetermined` | {'en_US': 'Undetermined', 'tr_TR': 'Belirsiz', 'ar_001': 'غير محدد'} |
| `presence_holiday_absent` | {'en_US': 'On leave', 'tr_TR': 'İzinli', 'ar_001': 'في إجازة'} |
| `presence_holiday_present` | {'en_US': 'Present but on leave', 'tr_TR': 'Mevcut ancak izinde', 'ar_001': 'حاضر ولكن في إجازة'} |
| `presence_home` | {'en_US': 'At Home', 'tr_TR': 'Evde', 'ar_001': 'في المنزل'} |
| `presence_office` | {'en_US': 'At Office', 'tr_TR': 'Ofiste', 'ar_001': 'في المكتب'} |
| `presence_other` | {'en_US': 'At Other', 'tr_TR': 'Diğer', 'ar_001': 'في غير ذلك'} |

**`hr_presence_state`** — {'en_US': 'Hr Presence State', 'tr_TR': 'İk Mevcudiyet Durumu', 'ar_001': 'حالة حضور الموارد البشرية'} (not stored)

| Value | Label |
|---|---|
| `present` | {'en_US': 'Present', 'tr_TR': 'Mevcut', 'ar_001': 'حاضر'} |
| `absent` | {'en_US': 'Absent', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غائب'} |
| `archive` | {'en_US': 'Archived', 'tr_TR': 'Arşivlendi', 'ar_001': 'مؤرشف'} |
| `out_of_working_hour` | {'en_US': 'Off-Hours', 'tr_TR': 'Mesai Dışı', 'ar_001': 'خارج ساعات العمل'} |

**`hr_presence_state_display`** — {'en_US': 'Hr Presence State Display', 'tr_TR': 'İk Hazır Bulunma Durumu Ekranı', 'ar_001': 'عرض حالة حضور الموارد البشرية'}

| Value | Label |
|---|---|
| `out_of_working_hour` | {'en_US': 'Off-Hours', 'tr_TR': 'Mesai Dışı', 'ar_001': 'خارج ساعات العمل'} |
| `present` | {'en_US': 'Present', 'tr_TR': 'Mevcut', 'ar_001': 'حاضر'} |
| `absent` | {'en_US': 'Absent', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غائب'} |

**`work_location_type`** — {'en_US': 'Work Location Type', 'tr_TR': 'İş Konumu Türü', 'ar_001': 'نوع موقع العمل'} (not stored)

| Value | Label |
|---|---|
| `home` | {'en_US': 'Home', 'tr_TR': 'Ana Sayfa', 'ar_001': 'الرئيسية'} |
| `office` | {'en_US': 'Office', 'tr_TR': 'Ofis', 'ar_001': 'المكتب'} |
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |

## `hr.employee.public`

**`hr_presence_state`** — {'en_US': 'Hr Presence State', 'tr_TR': 'İk Mevcudiyet Durumu', 'ar_001': 'حالة حضور الموارد البشرية'} (not stored)

| Value | Label |
|---|---|
| `present` | {'en_US': 'Present', 'tr_TR': 'Mevcut', 'ar_001': 'حاضر'} |
| `absent` | {'en_US': 'Absent', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غائب'} |
| `archive` | {'en_US': 'Archived', 'tr_TR': 'Arşivlendi', 'ar_001': 'مؤرشف'} |
| `out_of_working_hour` | {'en_US': 'Off-Hours', 'tr_TR': 'Mesai Dışı', 'ar_001': 'خارج ساعات العمل'} |

**`hr_presence_state_display`** — {'en_US': 'Hr Presence State Display', 'tr_TR': 'İk Hazır Bulunma Durumu Ekranı', 'ar_001': 'عرض حالة حضور الموارد البشرية'}

| Value | Label |
|---|---|
| `out_of_working_hour` | {'en_US': 'Off-Hours', 'tr_TR': 'Mesai Dışı', 'ar_001': 'خارج ساعات العمل'} |
| `present` | {'en_US': 'Present', 'tr_TR': 'Mevcut', 'ar_001': 'حاضر'} |
| `absent` | {'en_US': 'Absent', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غائب'} |

## `hr.expense`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`approval_state`** — {'en_US': 'Approval State', 'tr_TR': 'Onay Durumu', 'ar_001': 'حالة الموافقة'}

| Value | Label |
|---|---|
| `submitted` | {'en_US': 'Submitted', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

**`payment_mode`** — {'en_US': 'Paid By', 'tr_TR': 'Ödeyen', 'ar_001': 'مدفوعة بواسطة'}

| Value | Label |
|---|---|
| `own_account` | {'en_US': 'Employee (to reimburse)', 'tr_TR': 'Personel (geri ödenecek)', 'ar_001': 'الموظف (لرد الأموال)'} |
| `company_account` | {'en_US': 'Company', 'tr_TR': 'Firma', 'ar_001': 'الشركة'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `submitted` | {'en_US': 'Submitted', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `posted` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |
| `in_payment` | {'en_US': 'In Payment', 'tr_TR': 'Ödeme', 'ar_001': 'بانتظار التسوية'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

## `hr.expense.split`

**`approval_state`** — {'en_US': 'Approval State', 'tr_TR': 'Onay Durumu', 'ar_001': 'حالة الموافقة'}

| Value | Label |
|---|---|
| `submitted` | {'en_US': 'Submitted', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

## `hr.holidays.summary.employee`

**`holiday_type`** — {'en_US': 'Select Time Off Type', 'tr_TR': 'İzin Türü Seç', 'ar_001': 'قم بتحديد نوع الإجازة'}

| Value | Label |
|---|---|
| `Approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `Confirmed` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `both` | {'en_US': 'Both Approved and Confirmed', 'tr_TR': 'Onaylanmış ve Doğrulanmış', 'ar_001': 'تم قبولها وتأكيدها'} |

## `hr.job`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

## `hr.leave`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`request_date_from_period`** — {'en_US': 'Date Period Start', 'tr_TR': 'Dönem Başlangıç Tarihi', 'ar_001': 'تاريخ بداية الفترة'}

| Value | Label |
|---|---|
| `am` | {'en_US': 'Morning', 'tr_TR': 'Sabah', 'ar_001': 'صباحاً'} |
| `pm` | {'en_US': 'Afternoon', 'tr_TR': 'Öğleden Sonra', 'ar_001': 'بعد الظهر'} |

**`request_date_to_period`** — {'en_US': 'Date Period End', 'tr_TR': 'Tarih Dönem Sonu'}

| Value | Label |
|---|---|
| `am` | {'en_US': 'Morning', 'tr_TR': 'Sabah', 'ar_001': 'صباحاً'} |
| `pm` | {'en_US': 'Afternoon', 'tr_TR': 'Öğleden Sonra', 'ar_001': 'بعد الظهر'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `hr.leave.accrual.level`

**`accrual_validity_type`** — {'en_US': 'Accrual Validity Type'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `month` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |

**`action_with_unused_accruals`** — {'en_US': 'Action With Unused Accruals'}

| Value | Label |
|---|---|
| `lost` | {'en_US': 'Lost', 'tr_TR': 'Kayıp', 'ar_001': 'ضائع'} |
| `all` | {'en_US': 'Carried over', 'tr_TR': 'Devredilen', 'ar_001': 'مُرحّلة'} |

**`added_value_type`** — {'en_US': 'Added Value Type'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Day(s)', 'tr_TR': 'Gün', 'ar_001': 'يوم'} |
| `hour` | {'en_US': 'Hour(s)', 'tr_TR': 'Saat(ler)', 'ar_001': 'ساعات'} |

**`carryover_options`** — {'en_US': 'Carryover Options'}

| Value | Label |
|---|---|
| `unlimited` | {'en_US': 'Unlimited', 'tr_TR': 'Sınırsız', 'ar_001': 'غير محدودة'} |
| `limited` | {'en_US': 'Up to', 'tr_TR': 'Up to', 'ar_001': 'يصل إلى'} |

**`first_month`** — {'en_US': 'First Month'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'January', 'tr_TR': 'Ocak', 'ar_001': 'يناير'} |
| `2` | {'en_US': 'February', 'tr_TR': 'Şubat', 'ar_001': 'فبراير'} |
| `3` | {'en_US': 'March', 'tr_TR': 'Mart', 'ar_001': 'مارس'} |
| `4` | {'en_US': 'April', 'tr_TR': 'Nisan', 'ar_001': 'أبريل'} |
| `5` | {'en_US': 'May', 'tr_TR': 'Mayıs', 'ar_001': 'مايو'} |
| `6` | {'en_US': 'June', 'tr_TR': 'Haziran', 'ar_001': 'يونيو'} |

**`frequency`** — {'en_US': 'Frequency', 'tr_TR': 'Sıklık', 'ar_001': 'معدل الحدوث'}

| Value | Label |
|---|---|
| `hourly` | {'en_US': 'Hourly', 'tr_TR': 'Saatlik', 'ar_001': 'في الساعة'} |
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `weekly` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `bimonthly` | {'en_US': 'Twice a month', 'tr_TR': 'Ayda iki kez', 'ar_001': 'مرتان في الشهر'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |
| `biyearly` | {'en_US': 'Twice a year', 'tr_TR': 'Yılda iki kez', 'ar_001': 'مرتان في السنة'} |
| `yearly` | {'en_US': 'Yearly', 'tr_TR': 'Yıllık', 'ar_001': 'سنويًا'} |
| `worked_hours` | {'en_US': 'Per Hour Worked', 'tr_TR': 'Çalışılan Saat Başına', 'ar_001': 'لكل ساعة عمل'} |

**`milestone_date`** — {'en_US': 'Milestone Date'}

| Value | Label |
|---|---|
| `creation` | {'en_US': 'At allocation creation', 'tr_TR': 'Tahsis oluşturma sırasında', 'ar_001': 'عند إنشاء المخصصات'} |
| `after` | {'en_US': 'After', 'tr_TR': 'Sonra', 'ar_001': 'بعد'} |

**`second_month`** — {'en_US': 'Second Month'}

| Value | Label |
|---|---|
| `7` | {'en_US': 'July', 'tr_TR': 'Temmuz', 'ar_001': 'يوليو'} |
| `8` | {'en_US': 'August', 'tr_TR': 'Ağustos', 'ar_001': 'أغسطس'} |
| `9` | {'en_US': 'September', 'tr_TR': 'Eylül', 'ar_001': 'سبتمبر'} |
| `10` | {'en_US': 'October', 'tr_TR': 'Ekim', 'ar_001': 'أكتوبر'} |
| `11` | {'en_US': 'November', 'tr_TR': 'Kasım', 'ar_001': 'نوفمبر'} |
| `12` | {'en_US': 'December', 'tr_TR': 'Aralık', 'ar_001': 'ديسمبر'} |

**`start_type`** — {'en_US': 'Start Type'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `month` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |
| `year` | {'en_US': 'Years', 'tr_TR': 'Yıllar', 'ar_001': 'سنوات'} |

**`week_day`** — {'en_US': 'Allocation on', 'tr_TR': 'Tahsisat', 'ar_001': 'المخصصات في'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `1` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `2` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `3` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `4` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `5` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |
| `6` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |

**`yearly_month`** — {'en_US': 'Yearly Month'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'January', 'tr_TR': 'Ocak', 'ar_001': 'يناير'} |
| `2` | {'en_US': 'February', 'tr_TR': 'Şubat', 'ar_001': 'فبراير'} |
| `3` | {'en_US': 'March', 'tr_TR': 'Mart', 'ar_001': 'مارس'} |
| `4` | {'en_US': 'April', 'tr_TR': 'Nisan', 'ar_001': 'أبريل'} |
| `5` | {'en_US': 'May', 'tr_TR': 'Mayıs', 'ar_001': 'مايو'} |
| `6` | {'en_US': 'June', 'tr_TR': 'Haziran', 'ar_001': 'يونيو'} |
| `7` | {'en_US': 'July', 'tr_TR': 'Temmuz', 'ar_001': 'يوليو'} |
| `8` | {'en_US': 'August', 'tr_TR': 'Ağustos', 'ar_001': 'أغسطس'} |
| `9` | {'en_US': 'September', 'tr_TR': 'Eylül', 'ar_001': 'سبتمبر'} |
| `10` | {'en_US': 'October', 'tr_TR': 'Ekim', 'ar_001': 'أكتوبر'} |
| `11` | {'en_US': 'November', 'tr_TR': 'Kasım', 'ar_001': 'نوفمبر'} |
| `12` | {'en_US': 'December', 'tr_TR': 'Aralık', 'ar_001': 'ديسمبر'} |

## `hr.leave.accrual.plan`

**`accrued_gain_time`** — {'en_US': 'Accrued Gain Time'}

| Value | Label |
|---|---|
| `start` | {'en_US': 'At the start of the accrual period', 'tr_TR': 'Tahsis döneminin başında', 'ar_001': 'في بداية فترة الاستحقاق'} |
| `end` | {'en_US': 'At the end of the accrual period', 'tr_TR': 'Tahsis döneminin sonunda', 'ar_001': 'في نهاية فترة الاستحقاق'} |

**`added_value_type`** — {'en_US': 'Added Value Type'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `hour` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |

**`carryover_date`** — {'en_US': 'Carry-Over Time'}

| Value | Label |
|---|---|
| `year_start` | {'en_US': 'At the start of the year', 'tr_TR': 'Senenin başında', 'ar_001': 'في بداية العام'} |
| `allocation` | {'en_US': 'At the allocation date', 'tr_TR': 'Tahsis edilme tarihinde', 'ar_001': 'في تاريخ التخصيص'} |
| `other` | {'en_US': 'Custom date', 'tr_TR': 'Özel tarih', 'ar_001': 'تاريخ مخصص'} |

**`carryover_month`** — {'en_US': 'Carryover Month'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'January', 'tr_TR': 'Ocak', 'ar_001': 'يناير'} |
| `2` | {'en_US': 'February', 'tr_TR': 'Şubat', 'ar_001': 'فبراير'} |
| `3` | {'en_US': 'March', 'tr_TR': 'Mart', 'ar_001': 'مارس'} |
| `4` | {'en_US': 'April', 'tr_TR': 'Nisan', 'ar_001': 'أبريل'} |
| `5` | {'en_US': 'May', 'tr_TR': 'Mayıs', 'ar_001': 'مايو'} |
| `6` | {'en_US': 'June', 'tr_TR': 'Haziran', 'ar_001': 'يونيو'} |
| `7` | {'en_US': 'July', 'tr_TR': 'Temmuz', 'ar_001': 'يوليو'} |
| `8` | {'en_US': 'August', 'tr_TR': 'Ağustos', 'ar_001': 'أغسطس'} |
| `9` | {'en_US': 'September', 'tr_TR': 'Eylül', 'ar_001': 'سبتمبر'} |
| `10` | {'en_US': 'October', 'tr_TR': 'Ekim', 'ar_001': 'أكتوبر'} |
| `11` | {'en_US': 'November', 'tr_TR': 'Kasım', 'ar_001': 'نوفمبر'} |
| `12` | {'en_US': 'December', 'tr_TR': 'Aralık', 'ar_001': 'ديسمبر'} |

**`transition_mode`** — {'en_US': 'Transition Mode'}

| Value | Label |
|---|---|
| `immediately` | {'en_US': 'Immediately', 'tr_TR': 'Acil Olarak', 'ar_001': 'فورًا'} |
| `end_of_accrual` | {'en_US': "After this accrual's period", 'tr_TR': 'Bu tahakkuk döneminden sonra', 'ar_001': 'بعد فترة المستحق هذه'} |

## `hr.leave.allocation`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`allocation_type`** — {'en_US': 'Allocation Type', 'tr_TR': 'Tahsis Türü', 'ar_001': 'نوع المخصصات'}

| Value | Label |
|---|---|
| `regular` | {'en_US': 'Regular Allocation', 'tr_TR': 'Normal Tahsis', 'ar_001': 'التخصيص المنتظم'} |
| `accrual` | {'en_US': 'Accrual Allocation', 'tr_TR': 'İzin Tahakkuk Tahsisi', 'ar_001': 'تخصيص المستحقات'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

**`type_request_unit`** — {'en_US': 'Type Request Unit', 'tr_TR': 'Talep Birimi Türü', 'ar_001': 'اكتب وحدة الطلب'} (not stored)

| Value | Label |
|---|---|
| `hour` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |
| `half_day` | {'en_US': 'Half-Day', 'tr_TR': 'Yarım Gün', 'ar_001': 'نصف يوم'} |
| `day` | {'en_US': 'Day', 'tr_TR': 'Gün', 'ar_001': 'اليوم'} |

## `hr.leave.allocation.generate.multi.wizard`

**`allocation_mode`** — {'en_US': 'Allocation Mode', 'tr_TR': 'Tahsis Şekli', 'ar_001': 'وضع تخصيص رصيد الإجازات'}

| Value | Label |
|---|---|
| `employee` | {'en_US': 'By Employee', 'tr_TR': 'Personele Göre', 'ar_001': 'حسب الموظف'} |
| `company` | {'en_US': 'By Company', 'tr_TR': 'Firmaya Göre', 'ar_001': 'حسب المؤسسة'} |
| `department` | {'en_US': 'By Department', 'tr_TR': 'Departmana Göre', 'ar_001': 'حسب القسم'} |
| `category` | {'en_US': 'By Employee Tag', 'tr_TR': 'Personel Etiketine Göre', 'ar_001': 'حسب علامة تصنيف الموظف'} |

**`allocation_type`** — {'en_US': 'Allocation Type', 'tr_TR': 'Tahsis Türü', 'ar_001': 'نوع المخصصات'}

| Value | Label |
|---|---|
| `regular` | {'en_US': 'Regular Allocation', 'tr_TR': 'Normal Tahsis', 'ar_001': 'التخصيص المنتظم'} |
| `accrual` | {'en_US': 'Based on Accrual Plan', 'tr_TR': 'Tahakkuk Planına Göre', 'ar_001': 'بناءً على خطة الاستحقاق'} |

## `hr.leave.employee.type.report`

**`holiday_status`** — {'en_US': 'Holiday Status', 'tr_TR': 'Tatil Durumu', 'ar_001': 'حالة الإجازة'}

| Value | Label |
|---|---|
| `taken` | {'en_US': 'Taken', 'tr_TR': 'Alınmış', 'ar_001': 'تم أخذها'} |
| `left` | {'en_US': 'Left', 'tr_TR': 'Sol', 'ar_001': 'يسار'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

## `hr.leave.generate.multi.wizard`

**`allocation_mode`** — {'en_US': 'Allocation Mode', 'tr_TR': 'Tahsis Şekli', 'ar_001': 'وضع تخصيص رصيد الإجازات'}

| Value | Label |
|---|---|
| `employee` | {'en_US': 'By Employee', 'tr_TR': 'Personele Göre', 'ar_001': 'حسب الموظف'} |
| `company` | {'en_US': 'By Company', 'tr_TR': 'Firmaya Göre', 'ar_001': 'حسب المؤسسة'} |
| `department` | {'en_US': 'By Department', 'tr_TR': 'Departmana Göre', 'ar_001': 'حسب القسم'} |
| `category` | {'en_US': 'By Employee Tag', 'tr_TR': 'Personel Etiketine Göre', 'ar_001': 'حسب علامة تصنيف الموظف'} |

## `hr.leave.report`

**`leave_type`** — {'en_US': 'Request Type', 'tr_TR': 'İstek Türü', 'ar_001': 'نوع الطلب'}

| Value | Label |
|---|---|
| `allocation` | {'en_US': 'Allocation', 'tr_TR': 'Tahsis', 'ar_001': 'التخصيص'} |
| `request` | {'en_US': 'Time Off', 'tr_TR': 'İzin', 'ar_001': 'الإجازات'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

## `hr.leave.report.calendar`

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

## `hr.leave.type`

**`allocation_validation_type`** — {'en_US': 'Approval', 'tr_TR': 'Onay', 'ar_001': 'الموافقة'}

| Value | Label |
|---|---|
| `no_validation` | {'en_US': 'None needed', 'tr_TR': 'Gerekli değil', 'ar_001': 'لا توجد حاجة لأي منها'} |
| `hr` | {'en_US': 'By Time Off Officer', 'tr_TR': 'İzin Yetkilisi tarafından', 'ar_001': 'بواسطة موظف الإجازات'} |
| `manager` | {'en_US': "By Employee's Approver", 'tr_TR': 'Çalışanın Onaylayanı Tarafından', 'ar_001': 'بواسطة مانح الموافقة للموظف'} |
| `both` | {'en_US': "By Employee's Approver and Time Off Officer", 'tr_TR': 'Çalışanın Onaylayan ve İzin Görevlisi Tarafından', 'ar_001': 'بواسطة مانح الموافقة للموظف وموظف الإجازات'} |

**`leave_validation_type`** — {'en_US': 'Time Off Validation', 'tr_TR': 'İzin Doğrulama', 'ar_001': 'تصديق الإجازة'}

| Value | Label |
|---|---|
| `no_validation` | {'en_US': 'None needed', 'tr_TR': 'Gerekli değil', 'ar_001': 'لا توجد حاجة لأي منها'} |
| `hr` | {'en_US': 'By Time Off Officer', 'tr_TR': 'İzin Yetkilisi tarafından', 'ar_001': 'بواسطة موظف الإجازات'} |
| `manager` | {'en_US': "By Employee's Approver", 'tr_TR': 'Çalışanın Onaylayanı Tarafından', 'ar_001': 'بواسطة مانح الموافقة للموظف'} |
| `both` | {'en_US': "By Employee's Approver and Time Off Officer", 'tr_TR': 'Çalışanın Onaylayan ve İzin Görevlisi Tarafından', 'ar_001': 'بواسطة مانح الموافقة للموظف وموظف الإجازات'} |

**`request_unit`** — {'en_US': 'Duration Type', 'tr_TR': 'Süre Tipi'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Day', 'tr_TR': 'Gün', 'ar_001': 'اليوم'} |
| `half_day` | {'en_US': 'Half-Day', 'tr_TR': 'Yarım Gün', 'ar_001': 'نصف يوم'} |
| `hour` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |

**`time_type`** — {'en_US': 'Kind of Time Off', 'tr_TR': 'Bir Çeşit İzin', 'ar_001': 'نوع الإجازة'}

| Value | Label |
|---|---|
| `other` | {'en_US': 'Worked Time', 'tr_TR': 'Çalışma Süresi', 'ar_001': 'الوقت المقضي في العمل'} |
| `leave` | {'en_US': 'Absence', 'tr_TR': 'İzinli', 'ar_001': 'الغياب'} |

## `hr.resume.line`

**`course_type`** — {'en_US': 'Course Type', 'tr_TR': 'Ders Türü', 'ar_001': 'نوع الدورة'}

| Value | Label |
|---|---|
| `external` | {'en_US': 'External', 'tr_TR': 'Dış', 'ar_001': 'خارجي'} |
| `onsite` | {'en_US': 'Onsite', 'tr_TR': 'Yerinde İçinde', 'ar_001': 'في الموقع'} |
| `elearning` | {'en_US': 'eLearning', 'tr_TR': 'e-Eğitim', 'ar_001': 'التعلم الإلكتروني'} |

**`expiration_status`** — {'en_US': 'Expiration Status', 'tr_TR': 'Son geçerlilik tarihi durumu', 'ar_001': 'حالة انتهاء الصلاحية'}

| Value | Label |
|---|---|
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `expiring` | {'en_US': 'Expiring', 'tr_TR': 'Süresi dolmak üzere', 'ar_001': 'انتهاء الصلاحية'} |
| `valid` | {'en_US': 'Valid', 'tr_TR': 'Geçerli', 'ar_001': 'صالح'} |

## `hr.version`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`distance_home_work_unit`** — {'en_US': 'Home-Work Distance unit', 'tr_TR': 'Ev-İş Arası Mesafe Birimi', 'ar_001': 'وحدة قياس المسافة بين المنزل والعمل'}

| Value | Label |
|---|---|
| `kilometers` | {'en_US': 'km', 'tr_TR': 'km', 'ar_001': 'كم'} |
| `miles` | {'en_US': 'mi', 'tr_TR': 'mil', 'ar_001': 'ميل'} |

**`employee_type`** — {'en_US': 'Employee Type', 'tr_TR': 'Personel Türü', 'ar_001': 'نوع الموظف'}

| Value | Label |
|---|---|
| `employee` | {'en_US': 'Employee', 'tr_TR': 'Çalışan', 'ar_001': 'الموظف'} |
| `worker` | {'en_US': 'Worker', 'tr_TR': 'İşçi', 'ar_001': 'العامل'} |
| `student` | {'en_US': 'Student', 'tr_TR': 'Öğrenci', 'ar_001': 'طالب'} |
| `trainee` | {'en_US': 'Trainee', 'tr_TR': 'Stajyer', 'ar_001': 'متدرب'} |
| `contractor` | {'en_US': 'Contractor', 'tr_TR': 'Sözleşmeli', 'ar_001': 'المتعاقد'} |
| `freelance` | {'en_US': 'Freelancer', 'tr_TR': 'Serbest Çalışan', 'ar_001': 'مستقل'} |

**`sex`** — {'en_US': 'Gender', 'tr_TR': 'Cinsiyeti', 'ar_001': 'الجنس'}

| Value | Label |
|---|---|
| `male` | {'en_US': 'Male', 'tr_TR': 'Bay', 'ar_001': 'ذكر'} |
| `female` | {'en_US': 'Female', 'tr_TR': 'Bayan', 'ar_001': 'أنثى'} |
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |

**`work_entry_source`** — {'en_US': 'Work Entry Source', 'tr_TR': 'Puantaj Kaydı Kaynağı', 'ar_001': 'مصدر قيد العمل'}

| Value | Label |
|---|---|
| `calendar` | {'en_US': 'Working Schedule', 'tr_TR': 'Çalışma Saatleri', 'ar_001': 'جدول العمل'} |

## `hr.work.entry`

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `conflict` | {'en_US': 'In Conflict', 'tr_TR': 'Çatışma Halinde'} |
| `validated` | {'en_US': 'In Payslip', 'tr_TR': 'Maaş Bordrosunda', 'ar_001': 'في كشوف الرواتب'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `hr.work.location`

**`location_type`** — {'en_US': 'Cover Image', 'tr_TR': 'Kapak Resmi', 'ar_001': 'صورة الغلاف'}

| Value | Label |
|---|---|
| `home` | {'en_US': 'Home', 'tr_TR': 'Ana Sayfa', 'ar_001': 'الرئيسية'} |
| `office` | {'en_US': 'Office', 'tr_TR': 'Ofis', 'ar_001': 'المكتب'} |
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |

## `html_editor.converter.test`

**`selection_str`** — {'en_US': "Lorsqu'un pancake prend l'avion à destination de Toronto et qu'il fait une escale technique à St Claude, on dit:"}

| Value | Label |
|---|---|
| `A` | {'en_US': "Qu'il n'est pas arrivé à Toronto", 'tr_TR': "Toronto'ya gelmediğini", 'ar_001': "Qu'il n'est pas arrivé à Toronto"} |
| `B` | {'en_US': "Qu'il était supposé arriver à Toronto", 'tr_TR': "Toronto'ya gelmesi gerekiyordu", 'ar_001': "Qu'il était supposé arriver à Toronto"} |
| `C` | {'en_US': "Qu'est-ce qu'il fout ce maudit pancake, tabernacle ?", 'tr_TR': 'O kahrolası gözleme ne yapıyor, Çadır?', 'ar_001': "Qu'est-ce qu'il fout ce maudit pancake, tabernacle ?"} |
| `D` | {'en_US': 'La réponse D'} |

## `iap.account`

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `banned` | {'en_US': 'Banned', 'tr_TR': 'Yasaklı', 'ar_001': 'محظور'} |
| `registered` | {'en_US': 'Registered', 'tr_TR': 'Kayıtlı', 'ar_001': 'مُسجل'} |
| `unregistered` | {'en_US': 'Unregistered', 'tr_TR': 'Kaydedilmemiş', 'ar_001': 'غير مسجل'} |

## `im_livechat.channel`

**`max_sessions_mode`** — {'en_US': 'Sessions per Operator', 'tr_TR': 'Operatör Başına Oturumlar', 'ar_001': 'عدد الجلسات لكل موظف دعم'}

| Value | Label |
|---|---|
| `unlimited` | {'en_US': 'Unlimited', 'tr_TR': 'Sınırsız', 'ar_001': 'غير محدودة'} |
| `limited` | {'en_US': 'Limited', 'tr_TR': 'Sınırlı', 'ar_001': 'محدود'} |

## `im_livechat.channel.member.history`

**`help_status`** — {'en_US': 'Help Status', 'tr_TR': 'Yardım Durumu', 'ar_001': 'حالة المساعدة'}

| Value | Label |
|---|---|
| `requested` | {'en_US': 'Help Requested', 'tr_TR': 'Yardım İstendi', 'ar_001': 'تم طلب المساعدة'} |
| `provided` | {'en_US': 'Help Provided', 'tr_TR': 'Sağlanan Yardım', 'ar_001': 'تم تقديم المساعدة'} |

**`livechat_member_type`** — {'en_US': 'Livechat Member Type', 'tr_TR': 'Canlı Sohbet  Üye Tipi', 'ar_001': 'نوع أعضاء الدردشة المباشرة'}

| Value | Label |
|---|---|
| `agent` | {'en_US': 'Agent', 'tr_TR': 'Ajan', 'ar_001': 'الوكيل'} |
| `visitor` | {'en_US': 'Visitor', 'tr_TR': 'Ziyaretçi', 'ar_001': 'زائر'} |
| `bot` | {'en_US': 'Chatbot', 'tr_TR': 'Chatbot', 'ar_001': 'برنامج الدردشة الآلي'} |

## `im_livechat.channel.rule`

**`action`** — {'en_US': 'Live Chat Button', 'tr_TR': 'Canlı Sohbet Butonu', 'ar_001': 'زر الدردشة المباشرة'}

| Value | Label |
|---|---|
| `display_button` | {'en_US': 'Show', 'tr_TR': 'Göster', 'ar_001': 'إظهار'} |
| `display_button_and_text` | {'en_US': 'Show with notification', 'tr_TR': 'Bildirimle göster', 'ar_001': 'الإظهار مع الإشعار'} |
| `auto_popup` | {'en_US': 'Open automatically', 'tr_TR': 'Otomatik olarak aç', 'ar_001': 'الفتح تلقائياً'} |
| `hide_button` | {'en_US': 'Hide', 'tr_TR': 'Gizle', 'ar_001': 'إخفاء'} |

**`chatbot_enabled_condition`** — {'en_US': 'Enable ChatBot', 'tr_TR': "ChatBot'u Etkinleştir", 'ar_001': 'تمكين برامج الدردشة الآلية'}

| Value | Label |
|---|---|
| `always` | {'en_US': 'Always', 'tr_TR': 'Daima', 'ar_001': 'دائمًا'} |
| `only_if_no_operator` | {'en_US': 'Only when no operator is available', 'tr_TR': 'Yalnızca operatör bulunmadığında', 'ar_001': 'فقط عندما لا يكون هناك موظف دعم متوفر'} |
| `only_if_operator` | {'en_US': 'Only when an operator is available', 'tr_TR': 'Sadece bir operatör mevcut olduğunda', 'ar_001': 'فقط عندما يكون هناك موظف دعم متوفر'} |

## `im_livechat.report.channel`

**`day_number`** — {'en_US': 'Day of the Week', 'tr_TR': 'Haftanın Günü', 'ar_001': 'اليوم من الأسبوع'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |
| `1` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `2` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `3` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `4` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `5` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `6` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |

**`session_outcome`** — {'en_US': 'Session Outcome', 'tr_TR': 'Oturum Sonuçları', 'ar_001': 'نواتج الجلسة'}

| Value | Label |
|---|---|
| `no_answer` | {'en_US': 'Never Answered', 'tr_TR': 'Hiç Cevaplanmadı', 'ar_001': 'لم يتم الإجابة عنها قط'} |
| `no_agent` | {'en_US': 'No one Available', 'tr_TR': 'Müsait kimse yok', 'ar_001': 'لا يوجد شخص متاح'} |
| `no_failure` | {'en_US': 'Success', 'tr_TR': 'Başarılı', 'ar_001': 'نجاح'} |
| `escalated` | {'en_US': 'Escalated', 'tr_TR': 'Üst destek birimi', 'ar_001': 'تصعيد'} |

## `ir.actions.act_url`

**`binding_type`** — {'en_US': 'Binding Type', 'tr_TR': 'Bağlantı Türü', 'ar_001': 'نوع الربط'}

| Value | Label |
|---|---|
| `action` | {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'} |
| `report` | {'en_US': 'Report', 'tr_TR': 'Form', 'ar_001': 'التقرير'} |

**`target`** — {'en_US': 'Action Target', 'tr_TR': 'İşlem Hedefi', 'ar_001': 'هدف الإجراء'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'New Window', 'tr_TR': 'Yeni Pencere', 'ar_001': 'نافذة جديدة'} |
| `self` | {'en_US': 'This Window', 'tr_TR': 'Bu Pencere', 'ar_001': 'هذه النافذة'} |
| `download` | {'en_US': 'Download', 'tr_TR': 'İndir', 'ar_001': 'تنزيل'} |

## `ir.actions.act_window`

**`binding_type`** — {'en_US': 'Binding Type', 'tr_TR': 'Bağlantı Türü', 'ar_001': 'نوع الربط'}

| Value | Label |
|---|---|
| `action` | {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'} |
| `report` | {'en_US': 'Report', 'tr_TR': 'Form', 'ar_001': 'التقرير'} |

**`target`** — {'en_US': 'Target Window', 'tr_TR': 'Hedef Pencere', 'ar_001': 'النافذة المستهدفة'}

| Value | Label |
|---|---|
| `current` | {'en_US': 'Current Window', 'tr_TR': 'Geçerli Pencere', 'ar_001': 'النافذة الحالية'} |
| `new` | {'en_US': 'New Window', 'tr_TR': 'Yeni Pencere', 'ar_001': 'نافذة جديدة'} |
| `fullscreen` | {'en_US': 'Full Screen', 'tr_TR': 'Tam Ekran', 'ar_001': 'ملء الشاشة'} |
| `main` | {'en_US': 'Main action of Current Window', 'tr_TR': 'Geçerli Pencerenin ana aksiyonu', 'ar_001': 'الإجراء الرئيسي للنافذة الحالية'} |

## `ir.actions.act_window.view`

**`view_mode`** — {'en_US': 'View Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع واجهة العرض'}

| Value | Label |
|---|---|
| `list` | {'en_US': 'List', 'tr_TR': 'Liste', 'ar_001': 'القائمة'} |
| `form` | {'en_US': 'Form', 'tr_TR': 'Form', 'ar_001': 'الاستمارة'} |
| `graph` | {'en_US': 'Graph', 'tr_TR': 'Grafik', 'ar_001': 'الرسم البياني'} |
| `pivot` | {'en_US': 'Pivot', 'tr_TR': 'Pivot', 'ar_001': 'محور'} |
| `calendar` | {'en_US': 'Calendar', 'tr_TR': 'Takvim', 'ar_001': 'التقويم'} |
| `kanban` | {'en_US': 'Kanban', 'tr_TR': 'Kanban', 'ar_001': 'كانبان'} |
| `hierarchy` | {'en_US': 'Hierarchy', 'tr_TR': 'Hiyerarşi', 'ar_001': 'التسلسل الهرمي'} |
| `activity` | {'en_US': 'Activity', 'tr_TR': 'Aktivite', 'ar_001': 'النشاط'} |

## `ir.actions.act_window_close`

**`binding_type`** — {'en_US': 'Binding Type', 'tr_TR': 'Bağlantı Türü', 'ar_001': 'نوع الربط'}

| Value | Label |
|---|---|
| `action` | {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'} |
| `report` | {'en_US': 'Report', 'tr_TR': 'Form', 'ar_001': 'التقرير'} |

## `ir.actions.actions`

**`binding_type`** — {'en_US': 'Binding Type', 'tr_TR': 'Bağlantı Türü', 'ar_001': 'نوع الربط'}

| Value | Label |
|---|---|
| `action` | {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'} |
| `report` | {'en_US': 'Report', 'tr_TR': 'Form', 'ar_001': 'التقرير'} |

## `ir.actions.client`

**`binding_type`** — {'en_US': 'Binding Type', 'tr_TR': 'Bağlantı Türü', 'ar_001': 'نوع الربط'}

| Value | Label |
|---|---|
| `action` | {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'} |
| `report` | {'en_US': 'Report', 'tr_TR': 'Form', 'ar_001': 'التقرير'} |

**`target`** — {'en_US': 'Target Window', 'tr_TR': 'Hedef Pencere', 'ar_001': 'النافذة المستهدفة'}

| Value | Label |
|---|---|
| `current` | {'en_US': 'Current Window', 'tr_TR': 'Geçerli Pencere', 'ar_001': 'النافذة الحالية'} |
| `new` | {'en_US': 'New Window', 'tr_TR': 'Yeni Pencere', 'ar_001': 'نافذة جديدة'} |
| `fullscreen` | {'en_US': 'Full Screen', 'tr_TR': 'Tam Ekran', 'ar_001': 'ملء الشاشة'} |
| `main` | {'en_US': 'Main action of Current Window', 'tr_TR': 'Geçerli Pencerenin ana aksiyonu', 'ar_001': 'الإجراء الرئيسي للنافذة الحالية'} |

## `ir.actions.report`

**`binding_type`** — {'en_US': 'Binding Type', 'tr_TR': 'Bağlantı Türü', 'ar_001': 'نوع الربط'}

| Value | Label |
|---|---|
| `action` | {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'} |
| `report` | {'en_US': 'Report', 'tr_TR': 'Form', 'ar_001': 'التقرير'} |

**`report_type`** — {'en_US': 'Report Type', 'tr_TR': 'Rapor Tipi', 'ar_001': 'نوع التقرير'}

| Value | Label |
|---|---|
| `qweb-html` | {'en_US': 'HTML', 'tr_TR': 'HTML', 'ar_001': 'HTML'} |
| `qweb-pdf` | {'en_US': 'PDF', 'tr_TR': 'PDF', 'ar_001': 'PDF'} |
| `qweb-text` | {'en_US': 'Text', 'tr_TR': 'Metin', 'ar_001': 'النص'} |

## `ir.actions.server`

**`activity_date_deadline_range_type`** — {'en_US': 'Due type', 'tr_TR': 'Beklenen Türü', 'ar_001': 'نوع الاستحقاق'}

| Value | Label |
|---|---|
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weeks` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`activity_user_type`** — {'en_US': 'User Type', 'tr_TR': 'Kullanıcı Türü', 'ar_001': 'نوع المستخدم'}

| Value | Label |
|---|---|
| `specific` | {'en_US': 'Specific User', 'tr_TR': 'Belirli Kullanıcı', 'ar_001': 'مستخدم معين'} |
| `generic` | {'en_US': 'Dynamic User (based on record)', 'tr_TR': 'Dinamik Kullanıcı (kayda göre)', 'ar_001': 'مستخدم ديناميكي (بناءً على السجل)'} |

**`binding_type`** — {'en_US': 'Binding Type', 'tr_TR': 'Bağlantı Türü', 'ar_001': 'نوع الربط'}

| Value | Label |
|---|---|
| `action` | {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'} |
| `report` | {'en_US': 'Report', 'tr_TR': 'Form', 'ar_001': 'التقرير'} |

**`evaluation_type`** — {'en_US': 'Value Type', 'tr_TR': 'Değer Türü', 'ar_001': 'نوع القيمة'}

| Value | Label |
|---|---|
| `value` | {'en_US': 'Update', 'tr_TR': 'Güncelle', 'ar_001': 'تحديث'} |
| `sequence` | {'en_US': 'Sequence', 'tr_TR': 'Sıralama', 'ar_001': 'تسلسل'} |
| `equation` | {'en_US': 'Compute', 'tr_TR': 'Hesapla', 'ar_001': 'احتساب'} |

**`followers_type`** — {'en_US': 'Followers Type', 'tr_TR': 'Takipçi Türü', 'ar_001': 'نوع المتابعين'}

| Value | Label |
|---|---|
| `specific` | {'en_US': 'Specific Followers', 'tr_TR': 'Belirli Takipçiler', 'ar_001': 'متابعين محددين'} |
| `generic` | {'en_US': 'Dynamic Followers', 'tr_TR': 'Dinamik Takipçiler', 'ar_001': 'المتابعون الديناميكيون'} |

**`mail_post_method`** — {'en_US': 'Send Email As', 'tr_TR': 'E-posta Gönderici Adı', 'ar_001': 'إرسال البريد الإلكتروني كـ'}

| Value | Label |
|---|---|
| `email` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `comment` | {'en_US': 'Message', 'tr_TR': 'Mesaj', 'ar_001': 'الرسالة'} |
| `note` | {'en_US': 'Note', 'tr_TR': 'Not', 'ar_001': 'الملاحظات'} |

**`sms_method`** — {'en_US': 'Send SMS As', 'tr_TR': 'SMS Olarak Gönder', 'ar_001': 'إرسال الرسائل النصية القصيرة كـ'}

| Value | Label |
|---|---|
| `sms` | {'en_US': 'SMS (without note)', 'tr_TR': 'SMS (notsuz)', 'ar_001': 'الرسائل النصية القصيرة (دون ملاحظة)'} |
| `comment` | {'en_US': 'SMS (with note)', 'tr_TR': 'SMS (not ile)', 'ar_001': 'الرسائل النصية القصيرة (مع ملاحظة)'} |
| `note` | {'en_US': 'Note only', 'tr_TR': 'Sadece not edin', 'ar_001': 'الملاحظة فقط'} |

**`state`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `object_write` | {'en_US': 'Update Record', 'tr_TR': 'Kaydı Güncelle', 'ar_001': 'تحديث السجل'} |
| `object_create` | {'en_US': 'Create Record', 'tr_TR': 'Kayıt Oluştur', 'ar_001': 'إنشاء سجل'} |
| `object_copy` | {'en_US': 'Duplicate Record', 'tr_TR': 'Mükerrer Kayıt', 'ar_001': 'نسخ السجل'} |
| `next_activity` | {'en_US': 'Create Activity', 'tr_TR': 'Aktivite Oluştur', 'ar_001': 'إنشاء نشاط'} |
| `mail_post` | {'en_US': 'Send Email', 'tr_TR': 'E-posta Gönder', 'ar_001': 'إرسال بريد إلكتروني'} |
| `sms` | {'en_US': 'Send SMS', 'tr_TR': 'SMS Gönder', 'ar_001': 'إرسال رسالة نصية قصيرة'} |
| `followers` | {'en_US': 'Add Followers', 'tr_TR': 'Takipçi Ekle', 'ar_001': 'إضافة متابعين'} |
| `remove_followers` | {'en_US': 'Remove Followers', 'tr_TR': 'Takipçileri Kaldır', 'ar_001': 'إزالة المتابعين'} |
| `code` | {'en_US': 'Execute Code', 'tr_TR': 'Kodu Çalıştırın', 'ar_001': 'تنفيذ الكود'} |
| `webhook` | {'en_US': 'Send Webhook Notification', 'tr_TR': 'Webhook Bildirimi Gönder', 'ar_001': 'أرسل إشعارات Webhook'} |
| `multi` | {'en_US': 'Multi Actions', 'tr_TR': 'Çoklu Eylemler', 'ar_001': 'الإجراءات المتعددة'} |

**`update_boolean_value`** — {'en_US': 'Boolean Value', 'tr_TR': 'Boole Değeri', 'ar_001': 'قيمة Boolean'}

| Value | Label |
|---|---|
| `true` | {'en_US': 'Yes (True)', 'tr_TR': 'Evet (Doğru)', 'ar_001': 'نعم (صحيح)'} |
| `false` | {'en_US': 'No (False)', 'tr_TR': 'Hayır (False)', 'ar_001': 'لا (خطأ)'} |

**`update_m2m_operation`** — {'en_US': 'Many2many Operations', 'tr_TR': 'Çoktan-Çoka İşlemler', 'ar_001': 'عمليات علاقة متعدد إلى متعدد'}

| Value | Label |
|---|---|
| `add` | {'en_US': 'Adding', 'tr_TR': 'Ekleme', 'ar_001': 'جاري إضافة'} |
| `remove` | {'en_US': 'Removing', 'tr_TR': 'Kaldırma', 'ar_001': 'جاري الإزالة'} |
| `set` | {'en_US': 'Setting it to', 'tr_TR': 'Şuna ayarlanıyor:', 'ar_001': 'التعيين إلى'} |
| `clear` | {'en_US': 'Clearing it', 'tr_TR': 'Kontrol ediliyor', 'ar_001': 'جاري المسح'} |

**`usage`** — {'en_US': 'Usage', 'tr_TR': 'Kullanımı', 'ar_001': 'الاستخدام'}

| Value | Label |
|---|---|
| `ir_actions_server` | {'en_US': 'Server Action', 'tr_TR': 'Sunucu İşlemi', 'ar_001': 'إجراء الخادم'} |
| `ir_cron` | {'en_US': 'Scheduled Action', 'tr_TR': 'Planlanmış İşlem', 'ar_001': 'إجراء مجدول'} |
| `base_automation` | {'en_US': 'Automation Rule', 'tr_TR': 'Otomasyon Kuralları', 'ar_001': 'قاعدة الأتمتة'} |

**`value_field_to_show`** — {'en_US': 'Value Field To Show', 'tr_TR': 'Gösterilecek Değer Alanı', 'ar_001': 'حقل القيمة لإظهاره'} (not stored)

| Value | Label |
|---|---|
| `value` | {'en_US': 'value', 'tr_TR': 'değer', 'ar_001': 'قيمة'} |
| `html_value` | {'en_US': 'html_value', 'tr_TR': 'html_value', 'ar_001': 'html_value'} |
| `sequence_id` | {'en_US': 'sequence_id', 'tr_TR': 'sequence_id', 'ar_001': 'sequence_id'} |
| `resource_ref` | {'en_US': 'reference', 'tr_TR': 'referans', 'ar_001': 'المرجع'} |
| `update_boolean_value` | {'en_US': 'update_boolean_value', 'tr_TR': 'mantıksal_değeri_güncelleyin', 'ar_001': 'update_boolean_value'} |
| `selection_value` | {'en_US': 'selection_value', 'tr_TR': 'seçim_değeri', 'ar_001': 'selection_value'} |

## `ir.actions.todo`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `open` | {'en_US': 'To Do', 'tr_TR': 'Yapılacak', 'ar_001': 'المهام المراد تنفيذها'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `ir.asset`

**`directive`** — {'en_US': 'Directive', 'tr_TR': 'Yönerge', 'ar_001': 'التوجيه'}

| Value | Label |
|---|---|
| `append` | {'en_US': 'Append', 'tr_TR': 'Ekle', 'ar_001': 'إضافة'} |
| `prepend` | {'en_US': 'Prepend', 'tr_TR': 'Başına Ekle', 'ar_001': 'إضافة إلى البداية'} |
| `after` | {'en_US': 'After', 'tr_TR': 'Sonra', 'ar_001': 'بعد'} |
| `before` | {'en_US': 'Before', 'tr_TR': 'Önce', 'ar_001': 'قبل'} |
| `remove` | {'en_US': 'Remove', 'tr_TR': 'Kaldır', 'ar_001': 'إزالة'} |
| `replace` | {'en_US': 'Replace', 'tr_TR': 'Yerleştirme', 'ar_001': 'استبدال'} |
| `include` | {'en_US': 'Include', 'tr_TR': 'Dahil', 'ar_001': 'تضمين'} |

## `ir.attachment`

**`type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `url` | {'en_US': 'URL', 'tr_TR': 'URL', 'ar_001': 'رابط URL'} |
| `binary` | {'en_US': 'File', 'tr_TR': 'Dosya', 'ar_001': 'الملف'} |
| `cloud_storage` | {'en_US': 'Cloud Storage', 'tr_TR': 'Bulut Depolama', 'ar_001': 'مساحة التخزين السحابية'} |

## `ir.cron`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`interval_type`** — {'en_US': 'Interval Unit', 'tr_TR': 'Aralık Birimi', 'ar_001': 'وحدة الفترة'}

| Value | Label |
|---|---|
| `minutes` | {'en_US': 'Minutes', 'tr_TR': 'Dakika', 'ar_001': 'الدقائق'} |
| `hours` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weeks` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |

## `ir.logging`

**`type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `client` | {'en_US': 'Client', 'tr_TR': 'Müşteri', 'ar_001': 'العميل'} |
| `server` | {'en_US': 'Server', 'tr_TR': 'Sunucu', 'ar_001': 'الخادم'} |

## `ir.mail_server`

**`smtp_authentication`** — {'en_US': 'Authenticate with', 'tr_TR': 'İle kimlik doğrulaması yapın', 'ar_001': 'المصادقة مع'}

| Value | Label |
|---|---|
| `login` | {'en_US': 'Username', 'tr_TR': 'Kullanıcı Adı', 'ar_001': 'اسم المستخدم'} |
| `certificate` | {'en_US': 'SSL Certificate', 'tr_TR': 'SSL Sertifikası', 'ar_001': 'شهادة SSL'} |
| `cli` | {'en_US': 'Command Line Interface', 'tr_TR': 'Komut Satırı Arayüzü', 'ar_001': 'واجهة بند الأمر'} |
| `gmail` | {'en_US': 'Gmail OAuth Authentication', 'tr_TR': 'Gmail OAuth Kimlik Doğrulaması', 'ar_001': 'مصادقة Gmail OAuth'} |
| `outlook` | {'en_US': 'Outlook OAuth Authentication', 'tr_TR': 'Outlook OAuth Kimlik Doğrulaması', 'ar_001': 'مصادقة Outlook OAuth'} |

**`smtp_encryption`** — {'en_US': 'Connection Encryption', 'tr_TR': 'Bağlantı Şifreleme', 'ar_001': 'تشفير الاتصال'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |
| `starttls_strict` | {'en_US': 'TLS (STARTTLS), encryption and validation', 'tr_TR': 'TLS (STARTTLS), şifreleme ve doğrulama', 'ar_001': 'TLS (STARTTLS)، التشفير والتصديق'} |
| `starttls` | {'en_US': 'TLS (STARTTLS), encryption only', 'tr_TR': 'TLS (STARTTLS), yalnızca şifreleme', 'ar_001': 'TLS (STARTTLS)، التشفير فقط'} |
| `ssl_strict` | {'en_US': 'SSL/TLS, encryption and validation', 'tr_TR': 'SSL/TLS, şifreleme ve doğrulama', 'ar_001': 'SSL/TLS، التشفير والتصديق'} |
| `ssl` | {'en_US': 'SSL/TLS, encryption only', 'tr_TR': 'SSL/TLS, yalnızca şifreleme', 'ar_001': 'SSL/TLS، التشفير فقط'} |

## `ir.model`

**`state`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Custom Object', 'tr_TR': 'Özel Nesne', 'ar_001': 'كائن مخصص'} |
| `base` | {'en_US': 'Base Object', 'tr_TR': 'Temel Nesne', 'ar_001': 'الكائن الأساسي'} |

## `ir.model.fields`

**`on_delete`** — {'en_US': 'On Delete', 'tr_TR': 'Sil üzerinden', 'ar_001': 'عند الحذف'}

| Value | Label |
|---|---|
| `cascade` | {'en_US': 'Cascade', 'tr_TR': 'Basamakla', 'ar_001': 'تعاقُب'} |
| `set null` | {'en_US': 'Set NULL', 'tr_TR': 'Set NULL', 'ar_001': 'التعيين كقيمة باطلة'} |
| `restrict` | {'en_US': 'Restrict', 'tr_TR': 'Kısıtlama', 'ar_001': 'تقييد'} |

**`state`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Custom Field', 'tr_TR': 'Özel Alan', 'ar_001': 'حقل مخصص'} |
| `base` | {'en_US': 'Base Field', 'tr_TR': 'Temel Alan', 'ar_001': 'الحقل الأساسي'} |

**`translate`** — {'en_US': 'Translatable', 'tr_TR': 'Çevirilebilir', 'ar_001': 'قابل للترجمة'}

| Value | Label |
|---|---|
| `standard` | {'en_US': 'Translate as a whole', 'tr_TR': 'Bir bütün olarak çevir', 'ar_001': 'الترجمة ككل'} |
| `html_translate` | {'en_US': 'Translate HTML terms', 'tr_TR': 'HTML terimlerini çevirin', 'ar_001': 'ترجمة مصطلحات HTML'} |
| `xml_translate` | {'en_US': 'Translate XML terms', 'tr_TR': 'XML terimlerini çevirin', 'ar_001': 'ترجمة مصطلحات XML'} |

**`ttype`** — {'en_US': 'Field Type', 'tr_TR': 'Alan Türü', 'ar_001': 'نوع الحقل'}

| Value | Label |
|---|---|
| `binary` | {'en_US': 'binary', 'tr_TR': 'ikili', 'ar_001': 'ثنائي'} |
| `boolean` | {'en_US': 'boolean', 'tr_TR': 'mantıksal', 'ar_001': 'قيمة منطقية'} |
| `char` | {'en_US': 'char', 'tr_TR': 'char', 'ar_001': 'حرف'} |
| `date` | {'en_US': 'date', 'tr_TR': 'tarih', 'ar_001': 'التاريخ'} |
| `datetime` | {'en_US': 'datetime', 'tr_TR': 'tarih saat', 'ar_001': 'التاريخ والوقت'} |
| `float` | {'en_US': 'float', 'tr_TR': 'sayısal', 'ar_001': 'عدد عشري'} |
| `html` | {'en_US': 'html', 'tr_TR': 'html', 'ar_001': 'html'} |
| `integer` | {'en_US': 'integer', 'tr_TR': 'tamsayı', 'ar_001': 'عدد صحيح'} |
| `json` | {'en_US': 'json', 'tr_TR': 'json', 'ar_001': 'json'} |
| `many2many` | {'en_US': 'many2many', 'tr_TR': 'many2many', 'ar_001': 'علاقة متعدد لمتعدد'} |
| `many2one` | {'en_US': 'many2one', 'tr_TR': 'many2one', 'ar_001': 'علاقة متعدد لواحد'} |
| `many2one_reference` | {'en_US': 'many2one_reference', 'tr_TR': 'many2one_reference', 'ar_001': 'many2one_reference'} |
| `monetary` | {'en_US': 'monetary', 'tr_TR': 'parasal', 'ar_001': 'نقدي'} |
| `one2many` | {'en_US': 'one2many', 'tr_TR': 'one2many', 'ar_001': 'علاقة واحد لمتعدد'} |
| `properties` | {'en_US': 'properties', 'tr_TR': 'ilanlar', 'ar_001': 'الخصائص'} |
| `properties_definition` | {'en_US': 'properties_definition', 'tr_TR': 'properties_definition', 'ar_001': 'properties_definition'} |
| `reference` | {'en_US': 'reference', 'tr_TR': 'referans', 'ar_001': 'المرجع'} |
| `selection` | {'en_US': 'selection', 'tr_TR': 'seçim', 'ar_001': 'اختيار'} |
| `text` | {'en_US': 'text', 'tr_TR': 'metin', 'ar_001': 'نص'} |
| `serialized` | {'en_US': 'serialized', 'tr_TR': 'serileştirme', 'ar_001': 'متسلسل'} |

## `ir.module.module`

**`license`** — {'en_US': 'License', 'tr_TR': 'Lisans', 'ar_001': 'الرخصة'}

| Value | Label |
|---|---|
| `GPL-2` | {'en_US': 'GPL Version 2', 'tr_TR': 'GPL Version 2', 'ar_001': 'GPL الإصدار 2'} |
| `GPL-2 or any later version` | {'en_US': 'GPL-2 or later version', 'tr_TR': 'GPL-2 or later version', 'ar_001': 'GPL-2 أو أحدث'} |
| `GPL-3` | {'en_US': 'GPL Version 3', 'tr_TR': 'GPL Version 3', 'ar_001': 'GPL الإصدار 3'} |
| `GPL-3 or any later version` | {'en_US': 'GPL-3 or later version', 'tr_TR': 'GPL-3 or later version', 'ar_001': 'GPL-3 أو أحدث'} |
| `AGPL-3` | {'en_US': 'Affero GPL-3', 'tr_TR': 'Affero GLP-3', 'ar_001': 'رخصة Affero GPL-3'} |
| `LGPL-3` | {'en_US': 'LGPL Version 3', 'tr_TR': 'LGPL Sürüm 3', 'ar_001': 'LGPL الإصدار 3'} |
| `Other OSI approved licence` | {'en_US': 'Other OSI Approved License', 'tr_TR': 'Diğer OSI Onaylı Lisans', 'ar_001': 'رخصة OSI معتمدة أخرى'} |
| `OEEL-1` | {'en_US': 'Odoo Enterprise Edition License v1.0', 'tr_TR': 'Odoo Kurumsal Sürüm Lisansı v1.0', 'ar_001': 'رخصة إصدار أودو للمؤسسات V1.0'} |
| `OPL-1` | {'en_US': 'Odoo Proprietary License v1.0', 'tr_TR': 'Odoo Tescilli Lisansı v1.0', 'ar_001': 'رخصة ملكية أودو V1.0'} |
| `Other proprietary` | {'en_US': 'Other Proprietary', 'tr_TR': 'Diğer Sahiplik', 'ar_001': 'الملكية الأخرى'} |

**`module_type`** — {'en_US': 'Module Type', 'tr_TR': 'Modül Türü', 'ar_001': 'نوع التطبيق'}

| Value | Label |
|---|---|
| `official` | {'en_US': 'Official Apps', 'tr_TR': 'Resmi Uygulamalar', 'ar_001': 'التطبيقات الرسمية'} |
| `industries` | {'en_US': 'Industries', 'tr_TR': 'Sektörler', 'ar_001': 'مجالات العمل'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `uninstallable` | {'en_US': 'Uninstallable', 'tr_TR': 'Kaldırılamaz', 'ar_001': 'يمكن إلغاء تثبيته'} |
| `uninstalled` | {'en_US': 'Not Installed', 'tr_TR': 'Yüklü Değil', 'ar_001': 'غير مثبت'} |
| `installed` | {'en_US': 'Installed', 'tr_TR': 'Yüklü', 'ar_001': 'تم التثبيت'} |
| `to upgrade` | {'en_US': 'To be upgraded', 'tr_TR': 'Yükseltilecek', 'ar_001': 'بانتظار الترقية'} |
| `to remove` | {'en_US': 'To be removed', 'tr_TR': 'Kaldırılacak', 'ar_001': 'بانتظار الإزالة'} |
| `to install` | {'en_US': 'To be installed', 'tr_TR': 'Yüklenecek', 'ar_001': 'بانتظار التثبيت'} |

## `ir.module.module.dependency`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'} (not stored)

| Value | Label |
|---|---|
| `uninstallable` | {'en_US': 'Uninstallable', 'tr_TR': 'Kaldırılamaz', 'ar_001': 'يمكن إلغاء تثبيته'} |
| `uninstalled` | {'en_US': 'Not Installed', 'tr_TR': 'Yüklü Değil', 'ar_001': 'غير مثبت'} |
| `installed` | {'en_US': 'Installed', 'tr_TR': 'Yüklü', 'ar_001': 'تم التثبيت'} |
| `to upgrade` | {'en_US': 'To be upgraded', 'tr_TR': 'Yükseltilecek', 'ar_001': 'بانتظار الترقية'} |
| `to remove` | {'en_US': 'To be removed', 'tr_TR': 'Kaldırılacak', 'ar_001': 'بانتظار الإزالة'} |
| `to install` | {'en_US': 'To be installed', 'tr_TR': 'Yüklenecek', 'ar_001': 'بانتظار التثبيت'} |
| `unknown` | {'en_US': 'Unknown', 'tr_TR': 'Bilinmeyen', 'ar_001': 'غير معروف'} |

## `ir.module.module.exclusion`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'} (not stored)

| Value | Label |
|---|---|
| `uninstallable` | {'en_US': 'Uninstallable', 'tr_TR': 'Kaldırılamaz', 'ar_001': 'يمكن إلغاء تثبيته'} |
| `uninstalled` | {'en_US': 'Not Installed', 'tr_TR': 'Yüklü Değil', 'ar_001': 'غير مثبت'} |
| `installed` | {'en_US': 'Installed', 'tr_TR': 'Yüklü', 'ar_001': 'تم التثبيت'} |
| `to upgrade` | {'en_US': 'To be upgraded', 'tr_TR': 'Yükseltilecek', 'ar_001': 'بانتظار الترقية'} |
| `to remove` | {'en_US': 'To be removed', 'tr_TR': 'Kaldırılacak', 'ar_001': 'بانتظار الإزالة'} |
| `to install` | {'en_US': 'To be installed', 'tr_TR': 'Yüklenecek', 'ar_001': 'بانتظار التثبيت'} |
| `unknown` | {'en_US': 'Unknown', 'tr_TR': 'Bilinmeyen', 'ar_001': 'غير معروف'} |

## `ir.sequence`

**`implementation`** — {'en_US': 'Implementation', 'tr_TR': 'Uygulama', 'ar_001': 'التطبيق'}

| Value | Label |
|---|---|
| `standard` | {'en_US': 'Standard', 'tr_TR': 'Standart', 'ar_001': 'قياسي'} |
| `no_gap` | {'en_US': 'No gap', 'tr_TR': 'Boşluk Yok', 'ar_001': 'لا توجد فجوة'} |

## `ir.ui.view`

**`mode`** — {'en_US': 'View inheritance mode', 'tr_TR': 'Görünüm devralma modu', 'ar_001': 'عرض وضع الوراثة'}

| Value | Label |
|---|---|
| `primary` | {'en_US': 'Base view', 'tr_TR': 'Temel görünüm', 'ar_001': 'واجهة العرض الأساسية'} |
| `extension` | {'en_US': 'Extension View', 'tr_TR': 'Uzantı Görünümü', 'ar_001': 'معاينة الامتداد'} |

**`type`** — {'en_US': 'View Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع واجهة العرض'}

| Value | Label |
|---|---|
| `list` | {'en_US': 'List', 'tr_TR': 'Liste', 'ar_001': 'القائمة'} |
| `form` | {'en_US': 'Form', 'tr_TR': 'Form', 'ar_001': 'الاستمارة'} |
| `graph` | {'en_US': 'Graph', 'tr_TR': 'Grafik', 'ar_001': 'الرسم البياني'} |
| `pivot` | {'en_US': 'Pivot', 'tr_TR': 'Pivot', 'ar_001': 'محور'} |
| `calendar` | {'en_US': 'Calendar', 'tr_TR': 'Takvim', 'ar_001': 'التقويم'} |
| `kanban` | {'en_US': 'Kanban', 'tr_TR': 'Kanban', 'ar_001': 'كانبان'} |
| `search` | {'en_US': 'Search', 'tr_TR': 'Arama', 'ar_001': 'بحث'} |
| `qweb` | {'en_US': 'QWeb', 'tr_TR': 'QWeb', 'ar_001': 'QWeb'} |
| `hierarchy` | {'en_US': 'Hierarchy', 'tr_TR': 'Hiyerarşi', 'ar_001': 'التسلسل الهرمي'} |
| `activity` | {'en_US': 'Activity', 'tr_TR': 'Aktivite', 'ar_001': 'النشاط'} |

**`visibility`** — {'en_US': 'Visibility', 'tr_TR': 'Görünürlük', 'ar_001': 'الظهور'}

| Value | Label |
|---|---|
| `` | {'en_US': 'Public', 'tr_TR': 'Genel', 'ar_001': 'عام'} |
| `connected` | {'en_US': 'Signed In', 'tr_TR': 'Giriş', 'ar_001': 'تم تسجيل الدخول'} |
| `restricted_group` | {'en_US': 'Restricted Group', 'tr_TR': 'Kısıtlı Grup', 'ar_001': 'مجموعة مقيدة'} |
| `password` | {'en_US': 'With Password', 'tr_TR': 'Şifre ile', 'ar_001': 'مع كلمة سر'} |

## `l10n.fr.pdp.reports.flow`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`operation_type`** — {'en_US': 'Operation Type'}

| Value | Label |
|---|---|
| `sale` | {'en_US': 'Sales'} |
| `purchase` | {'en_US': 'Acquisitions'} |

**`period_status`** — {'en_US': 'Period Status'} (not stored)

| Value | Label |
|---|---|
| `open` | {'en_US': 'Open'} |
| `grace` | {'en_US': 'Grace'} |
| `closed` | {'en_US': 'Closed'} |

**`report_type`** — {'en_US': 'Report Type'}

| Value | Label |
|---|---|
| `transaction` | {'en_US': 'Transaction'} |
| `payment` | {'en_US': 'Payment'} |

**`state`** — {'en_US': 'Status'}

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready'} |
| `error` | {'en_US': 'Error'} |
| `sent` | {'en_US': 'Sent'} |
| `completed` | {'en_US': 'Completed'} |

**`transmission_type`** — {'en_US': 'Transmission Type'} (not stored)

| Value | Label |
|---|---|
| `initial` | {'en_US': 'Initial'} |
| `rectificative` | {'en_US': 'Rectificative'} |

## `l10n.in.ewaybill`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`blocking_level`** — {'en_US': 'Blocking Level'}

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Warning'} |
| `error` | {'en_US': 'Error'} |

**`cancel_reason`** — {'en_US': 'Cancel reason'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Duplicate'} |
| `2` | {'en_US': 'Data Entry Mistake'} |
| `3` | {'en_US': 'Order Cancelled'} |
| `4` | {'en_US': 'Others'} |

**`mode`** — {'en_US': 'Transportation Mode'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'By Road'} |
| `2` | {'en_US': 'Rail'} |
| `3` | {'en_US': 'Air'} |
| `4` | {'en_US': 'Ship or Ship Cum Road/Rail'} |

**`state`** — {'en_US': 'Status'}

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Pending'} |
| `generated` | {'en_US': 'Generated'} |
| `cancel` | {'en_US': 'Cancelled'} |
| `challan` | {'en_US': 'Challan'} |

**`supply_type`** — {'en_US': 'Supply Type'} (not stored)

| Value | Label |
|---|---|
| `O` | {'en_US': 'Outward'} |
| `I` | {'en_US': 'Inward'} |

**`vehicle_type`** — {'en_US': 'Vehicle Type'}

| Value | Label |
|---|---|
| `R` | {'en_US': 'Regular'} |
| `O` | {'en_US': 'Over Dimensional Cargo'} |

## `l10n.in.ewaybill.cancel`

**`cancel_reason`** — {'en_US': 'Cancel Reason'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Duplicate'} |
| `2` | {'en_US': 'Data Entry Mistake'} |
| `3` | {'en_US': 'Order Cancelled'} |
| `4` | {'en_US': 'Others'} |

## `l10n.in.ewaybill.type`

**`allowed_supply_type`** — {'en_US': 'Allowed for supply type'}

| Value | Label |
|---|---|
| `both` | {'en_US': 'Incoming and Outgoing'} |
| `out` | {'en_US': 'Outgoing'} |
| `in` | {'en_US': 'Incoming'} |

## `l10n_es_edi_tbai.document`

**`state`** — {'en_US': 'status'}

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `accepted` | {'en_US': 'Accepted'} |
| `rejected` | {'en_US': 'Rejected'} |

## `l10n_es_edi_verifactu.document`

**`document_type`** — {'en_US': 'Document Type'}

| Value | Label |
|---|---|
| `submission` | {'en_US': 'Submission'} |
| `cancellation` | {'en_US': 'Cancellation'} |

**`state`** — {'en_US': 'Status'}

| Value | Label |
|---|---|
| `rejected` | {'en_US': 'Rejected'} |
| `registered_with_errors` | {'en_US': 'Registered with Errors'} |
| `accepted` | {'en_US': 'Accepted'} |

## `l10n_fr.fec.export.wizard`

**`export_type`** — {'en_US': 'Export Type'}

| Value | Label |
|---|---|
| `official` | {'en_US': 'Official FEC report (posted entries only)'} |
| `nonofficial` | {'en_US': 'Non-official FEC report (posted and unposted entries)'} |

## `l10n_gr_edi.document`

**`provider_pdf_state`** — {'en_US': 'Final PDF Status'}

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Pending'} |
| `sent` | {'en_US': 'Sent'} |
| `error` | {'en_US': 'Failed'} |

**`state`** — {'en_US': 'myDATA Status'}

| Value | Label |
|---|---|
| `invoice_sent` | {'en_US': 'Invoice sent'} |
| `invoice_error` | {'en_US': 'Invoice send failed'} |
| `bill_fetched` | {'en_US': 'Expense classification ready to send'} |
| `bill_sent` | {'en_US': 'Expense classification sent'} |
| `bill_error` | {'en_US': 'Expense classification send failed'} |
| `invoice_pending` | {'en_US': 'Invoice submission pending'} |

## `l10n_gr_edi.preferred_classification`

**`l10n_gr_edi_cls_category`** — {'en_US': 'myDATA Category'}

| Value | Label |
|---|---|
| `category1_1` | {'en_US': '1.1 - Commodity Sale Income'} |
| `category1_2` | {'en_US': '1.2 - Product Sale Income'} |
| `category1_3` | {'en_US': '1.3 - Provision of Services Income'} |
| `category1_4` | {'en_US': '1.4 - Sale of Fixed Assets Income'} |
| `category1_5` | {'en_US': '1.5 - Other Income/Profits'} |
| `category1_6` | {'en_US': '1.6 - Self-Deliveries/Self-Supplies'} |
| `category1_7` | {'en_US': '1.7 - Income on behalf of Third Parties'} |
| `category1_8` | {'en_US': '1.8 - Past fiscal years income'} |
| `category1_9` | {'en_US': '1.9 - Future fiscal years income'} |
| `category1_10` | {'en_US': '1.10 - Other Income Adjustment/Regularisation Entries'} |
| `category1_95` | {'en_US': '1.95 - Other Income-related Information'} |
| `category2_1` | {'en_US': '2.1 - Commodity Purchases'} |
| `category2_2` | {'en_US': '2.2 - Raw and Adjuvant Material Purchases'} |
| `category2_3` | {'en_US': '2.3 - Services Receipt'} |
| `category2_4` | {'en_US': '2.4 - General Expenses Subject to VAT Deduction'} |
| `category2_5` | {'en_US': '2.5 - General Expenses Not Subject to VAT Deduction'} |
| `category2_6` | {'en_US': '2.6 - Personnel Fees and Benefits'} |
| `category2_7` | {'en_US': '2.7 - Fixed Asset Purchases'} |
| `category2_8` | {'en_US': '2.8 - Fixed Asset Amortisations'} |
| `category2_9` | {'en_US': '2.9 - Expenses on behalf of Third Parties'} |
| `category2_10` | {'en_US': '2.10 - Past fiscal years expenses'} |
| `category2_11` | {'en_US': '2.11 - Future fiscal years expenses'} |
| `category2_12` | {'en_US': '2.12 - Other Expense Adjustment/Regularisation Entries'} |
| `category2_13` | {'en_US': '2.13 - Stock at Period Start'} |
| `category2_14` | {'en_US': '2.14 - Stock at Period End'} |
| `category2_95` | {'en_US': '2.95 - Other Expense-related Information'} |

**`l10n_gr_edi_cls_type`** — {'en_US': 'myDATA Type'}

| Value | Label |
|---|---|
| `E3_106` | {'en_US': 'E3_106 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Commodities'} |
| `E3_205` | {'en_US': 'E3_205 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials'} |
| `E3_210` | {'en_US': 'E3_210 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress'} |
| `E3_305` | {'en_US': 'E3_305 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Raw and other materials'} |
| `E3_310` | {'en_US': 'E3_310 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Products and production in progress'} |
| `E3_318` | {'en_US': 'E3_318 - Self-Production of Fixed Assets – Self-Deliveries – Destroying inventory/Production expenses'} |
| `E3_561_001` | {'en_US': 'E3_561_001 - Wholesale Sales of Goods and Services – for Traders'} |
| `E3_561_002` | {'en_US': 'E3_561_002 - Wholesale Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000)'} |
| `E3_561_003` | {'en_US': 'E3_561_003 - Retail Sales of Goods and Services – Private Clientele'} |
| `E3_561_004` | {'en_US': 'E3_561_004 - Retail Sales of Goods and Services pursuant to article 39a paragraph 5 of the VAT Code (Law 2859/2000)'} |
| `E3_561_005` | {'en_US': 'E3_561_005 - Intra-Community Foreign Sales of Goods and Services'} |
| `E3_561_006` | {'en_US': 'E3_561_006 - Third Country Foreign Sales of Goods and Services'} |
| `E3_561_007` | {'en_US': 'E3_561_007 - Other Sales of Goods and Services'} |
| `E3_562` | {'en_US': 'E3_562 - Other Ordinary Income'} |
| `E3_563` | {'en_US': 'E3_563 - Credit Interest and Related Income'} |
| `E3_564` | {'en_US': 'E3_564 - Credit Exchange Differences'} |
| `E3_565` | {'en_US': 'E3_565 - Income from Participations'} |
| `E3_566` | {'en_US': 'E3_566 - Profits from Disposing Non-Current Assets'} |
| `E3_567` | {'en_US': 'E3_567 - Profits from the Reversal of Provisions and Impairments'} |
| `E3_568` | {'en_US': 'E3_568 - Profits from Measurement at Fair Value'} |
| `E3_570` | {'en_US': 'E3_570 - Extraordinary income and profits'} |
| `E3_595` | {'en_US': 'E3_595 - Self-Production Expenses'} |
| `E3_596` | {'en_US': 'E3_596 - Subsidies - Grants'} |
| `E3_597` | {'en_US': 'E3_597 - Subsidies – Grants for Investment Purposes – Expense Coverage'} |
| `E3_880_001` | {'en_US': 'E3_880_001 - Wholesale Sales of Fixed Assets'} |
| `E3_880_002` | {'en_US': 'E3_880_002 - Retail Sales of Fixed Assets'} |
| `E3_880_003` | {'en_US': 'E3_880_003 - Intra-Community Foreign Sales of Fixed Assets'} |
| `E3_880_004` | {'en_US': 'E3_880_004 - Third Country Foreign Sales of Fixed Assets'} |
| `E3_881_001` | {'en_US': 'E3_881_001 - Wholesale Sales on behalf of Third Parties'} |
| `E3_881_002` | {'en_US': 'E3_881_002 - Retail Sales on behalf of Third Parties'} |
| `E3_881_003` | {'en_US': 'E3_881_003 - Intra-Community Foreign Sales on behalf of Third Parties'} |
| `E3_881_004` | {'en_US': 'E3_881_004 - Third Country Foreign Sales on behalf of Third Parties'} |
| `E3_598_001` | {'en_US': 'E3_598_001 - Sales of goods belonging to excise duty'} |
| `E3_598_003` | {'en_US': 'E3_598_003 - Sales on behalf of farmers through an agricultural cooperative e.t.c.'} |
| `E3_101` | {'en_US': 'E3_101 - Commodities at Period Start'} |
| `E3_102_001` | {'en_US': 'E3_102_001 - Fiscal Year Commodity Purchases (net amount)/Wholesale'} |
| `E3_102_002` | {'en_US': 'E3_102_002 - Fiscal Year Commodity Purchases (net amount)/Retail'} |
| `E3_102_003` | {'en_US': 'E3_102_003 - Fiscal Year Commodity Purchases (net amount)/Goods under article 39a paragraph 5 of the VAT Code (Law 2859/2000)'} |
| `E3_102_004` | {'en_US': 'E3_102_004 - Fiscal Year Commodity Purchases (net amount)/Foreign, Intra-Community'} |
| `E3_102_005` | {'en_US': 'E3_102_005 - Fiscal Year Commodity Purchases (net amount)/Foreign, Third Countries'} |
| `E3_102_006` | {'en_US': 'E3_102_006 - Fiscal Year Commodity Purchases (net amount)/Others'} |
| `E3_104` | {'en_US': 'E3_104 - Commodities at Period End'} |
| `E3_201` | {'en_US': 'E3_201 - Raw and Other Materials at Period Start/Production'} |
| `E3_202_001` | {'en_US': 'E3_202_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale'} |
| `E3_202_002` | {'en_US': 'E3_202_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail'} |
| `E3_202_003` | {'en_US': 'E3_202_003 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Intra-Community'} |
| `E3_202_004` | {'en_US': 'E3_202_004 - Fiscal Year Raw and Other Material Purchases (net amount)/ Foreign, Third Countries'} |
| `E3_202_005` | {'en_US': 'E3_202_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others'} |
| `E3_204` | {'en_US': 'E3_204 - Raw and Other Material Stock at Period End/Production'} |
| `E3_207` | {'en_US': 'E3_207 - Products and Production in Progress at Period Start/Production'} |
| `E3_209` | {'en_US': 'E3_209 - Products and Production in Progress at Period End/Production'} |
| `E3_301` | {'en_US': 'E3_301 - Raw and Other Material at Period Start/Agricultural'} |
| `E3_302_001` | {'en_US': 'E3_302_001 - Fiscal Year Raw and Other Material Purchases (net amount)/Wholesale'} |
| `E3_302_002` | {'en_US': 'E3_302_002 - Fiscal Year Raw and Other Material Purchases (net amount)/Retail'} |
| `E3_302_003` | {'en_US': 'E3_302_003 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Intra-Community'} |
| `E3_302_004` | {'en_US': 'E3_302_004 - Fiscal Year Raw and Other Material Purchases (net amount)/Foreign, Third Countries'} |
| `E3_302_005` | {'en_US': 'E3_302_005 - Fiscal Year Raw and Other Material Purchases (net amount)/Others'} |
| `E3_304` | {'en_US': 'E3_304 - Raw and Other Material Stock at Period End/Agricultural'} |
| `E3_307` | {'en_US': 'E3_307 - Products and Production in Progress at Period Start/ Agricultural'} |
| `E3_309` | {'en_US': 'E3_309 - Products and Production in Progress at Period End/ Agricultural'} |
| `E3_312` | {'en_US': 'E3_312 - Stock at Period Start (Animals-Plants)'} |
| `E3_313_001` | {'en_US': 'E3_313_001 - Animal-Plant Purchases (net amount)/Wholesale'} |
| `E3_313_002` | {'en_US': 'E3_313_002 - Animal-Plant Purchases (net amount)/Retail'} |
| `E3_313_003` | {'en_US': 'E3_313_003 - Animal-Plant Purchases (net amount)/ Foreign, Intra-Community'} |
| `E3_313_004` | {'en_US': 'E3_313_004 - Animal-Plant Purchases (net amount)/ Foreign, Third Countries'} |
| `E3_313_005` | {'en_US': 'E3_313_005 - Animal-Plant Purchases/Others'} |
| `E3_315` | {'en_US': 'E3_315 - Stock at Period End (Animals-Plants)/Agricultural'} |
| `E3_581_001` | {'en_US': 'E3_581_001 - Employee Benefits/Gross Earnings'} |
| `E3_581_002` | {'en_US': 'E3_581_002 - Employee Benefits/Employer Contributions'} |
| `E3_581_003` | {'en_US': 'E3_581_003 - Employee Benefits/Other Benefits'} |
| `E3_582` | {'en_US': 'E3_582 - Asset Measurement Damages'} |
| `E3_583` | {'en_US': 'E3_583 - Debit Exchange Differences'} |
| `E3_584` | {'en_US': 'E3_584 - Damages from Disposing-Withdrawing Non-Current Assets'} |
| `E3_585_001` | {'en_US': 'E3_585_001 - Foreign/Domestic Management Fees'} |
| `E3_585_002` | {'en_US': 'E3_585_002 - Expenditures from Linked Enterprises'} |
| `E3_585_003` | {'en_US': 'E3_585_003 - Expenditures from Non-Cooperative States or Privileged Tax Regimes'} |
| `E3_585_004` | {'en_US': 'E3_585_004 - Expenditures for Information Day-Events'} |
| `E3_585_005` | {'en_US': 'E3_585_005 - Reception and Hospitality Expenses'} |
| `E3_585_006` | {'en_US': 'E3_585_006 - Travel expenses'} |
| `E3_585_007` | {'en_US': 'E3_585_007 - Self-Employed Social Security Contributions'} |
| `E3_585_008` | {'en_US': 'E3_585_008 - Commission Agent Expenses and Fees on behalf of Farmers'} |
| `E3_585_009` | {'en_US': 'E3_585_009 - Other Fees for Domestic Services'} |
| `E3_585_010` | {'en_US': 'E3_585_010 - Other Fees for Foreign Services'} |
| `E3_585_011` | {'en_US': 'E3_585_011 - Energy'} |
| `E3_585_012` | {'en_US': 'E3_585_012 - Water'} |
| `E3_585_013` | {'en_US': 'E3_585_013 - Telecommunications'} |
| `E3_585_014` | {'en_US': 'E3_585_014 - Rents'} |
| `E3_585_015` | {'en_US': 'E3_585_015 - Advertisement and promotion'} |
| `E3_585_016` | {'en_US': 'E3_585_016 - Other expenses'} |
| `E3_586` | {'en_US': 'E3_586 - Debit interests and related expenses'} |
| `E3_587` | {'en_US': 'E3_587 - Amortisations'} |
| `E3_588` | {'en_US': 'E3_588 - Extraordinary expenses, damages and fines'} |
| `E3_589` | {'en_US': 'E3_589 - Provisions (except for Personnel Provisions)'} |
| `E3_882_001` | {'en_US': 'E3_882_001 - Fiscal Year Tangible Asset Purchases/Wholesale'} |
| `E3_882_002` | {'en_US': 'E3_882_002 - Fiscal Year Tangible Asset Purchases/Retail'} |
| `E3_882_003` | {'en_US': 'E3_882_003 - Fiscal Year Tangible Asset Purchases/ Intra-Community Foreign'} |
| `E3_882_004` | {'en_US': 'E3_882_004 - Fiscal Year Tangible Asset Purchases/ Third Country Foreign'} |
| `E3_883_001` | {'en_US': 'E3_883_001 - Fiscal Year Intangible Asset Purchases/Wholesale'} |
| `E3_883_002` | {'en_US': 'E3_883_002 - Fiscal Year Intangible Asset Purchases/Retail'} |
| `E3_883_003` | {'en_US': 'E3_883_003 - Fiscal Year Intangible Asset Purchases/ Intra-Community Foreign'} |
| `E3_883_004` | {'en_US': 'E3_883_004 - Fiscal Year Intangible Asset Purchases/ Third Country Foreign'} |
| `E3_103` | {'en_US': 'E3_103 - Impairment of goods'} |
| `E3_203` | {'en_US': 'E3_203 - Impairment of raw materials and supplies'} |
| `E3_303` | {'en_US': 'E3_303 - Impairment of raw materials and supplies'} |
| `E3_208` | {'en_US': 'E3_208 - Impairment of products and production in progress'} |
| `E3_308` | {'en_US': 'E3_308 - Impairment of products and production in progress'} |
| `E3_314` | {'en_US': 'E3_314 - Impairment of animals-plants - goods'} |
| `XE3_106` | {'en_US': 'E3_106 - Own production of fixed assets – Self Deliveries – Inventory Disasters'} |
| `XE3_205` | {'en_US': 'E3_205 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_305` | {'en_US': 'E3_305 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_210` | {'en_US': 'E3_210 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_310` | {'en_US': 'E3_310 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `XE3_318` | {'en_US': 'E3_318 - Own production of fixed assets - Self Deliveries – Inventory Disasters'} |
| `E3_598_002` | {'en_US': 'E3_598_002 - Purchases of goods falling into excise duty'} |

**`l10n_gr_edi_inv_type`** — {'en_US': 'Invoice Type'}

| Value | Label |
|---|---|
| `1.1` | {'en_US': '1.1 - Sales Invoice'} |
| `1.2` | {'en_US': '1.2 - Sales Invoice/Intra-community Supplies'} |
| `1.3` | {'en_US': '1.3 - Sales Invoice/Third Country Supplies'} |
| `1.4` | {'en_US': '1.4 - Sales Invoice/Sale on Behalf of Third Parties'} |
| `1.5` | {'en_US': '1.5 - Sales Invoice/Clearance of Sales on Behalf of Third Parties – Fees from Sales on Behalf of Third Parties'} |
| `1.6` | {'en_US': '1.6 - Sales Invoice/Supplemental Accounting Source Document'} |
| `2.1` | {'en_US': '2.1 - Service Rendered Invoice'} |
| `2.2` | {'en_US': '2.2 - Intra-community Service Rendered Invoice'} |
| `2.3` | {'en_US': '2.3 - Third Country Service Rendered Invoice'} |
| `2.4` | {'en_US': '2.4 - Service Rendered Invoice/Supplemental Accounting Source Document'} |
| `3.1` | {'en_US': '3.1 - Proof of Expenditure (non-liable Issuer)'} |
| `3.2` | {'en_US': '3.2 - Proof of Expenditure (denial of issuance by liable Issuer)'} |
| `5.1` | {'en_US': '5.1 - Credit Invoice/Associated'} |
| `5.2` | {'en_US': '5.2 - Credit Invoice/Non-Associated'} |
| `6.1` | {'en_US': '6.1 - Self-Delivery Record'} |
| `6.2` | {'en_US': '6.2 - Self-Supply Record'} |
| `7.1` | {'en_US': '7.1 - Contract – Income'} |
| `8.1` | {'en_US': '8.1 - Rents – Income'} |
| `8.2` | {'en_US': '8.2 - Special Record – Accommodation Tax Collection/Payment Receipt'} |
| `11.1` | {'en_US': '11.1 - Retail Sales Receipt'} |
| `11.2` | {'en_US': '11.2 - Service Rendered Receipt'} |
| `11.3` | {'en_US': '11.3 - Simplified Invoice'} |
| `11.4` | {'en_US': '11.4 - Retail Sales Credit Note'} |
| `11.5` | {'en_US': '11.5 - Retail Sales Receipt on Behalf of Third Parties'} |
| `13.1` | {'en_US': '13.1 - Expenses – Domestic/Foreign Retail Transaction Purchases'} |
| `13.2` | {'en_US': '13.2 - Domestic/Foreign Retail Transaction Provision'} |
| `13.3` | {'en_US': '13.3 - Shared Utility Bills'} |
| `13.4` | {'en_US': '13.4 - Subscriptions'} |
| `13.30` | {'en_US': '13.30 - Self-Declared Entity Accounting Source Documents (Dynamic)'} |
| `13.31` | {'en_US': '13.31 - Domestic/Foreign Retail Sales Credit Note'} |
| `14.1` | {'en_US': '14.1 - Invoice/Intra-community Acquisitions'} |
| `14.2` | {'en_US': '14.2 - Invoice/Third Country Acquisitions'} |
| `14.3` | {'en_US': '14.3 - Invoice/Intra-community Services Receipt'} |
| `14.4` | {'en_US': '14.4 - Invoice/Third Country Services Receipt'} |
| `14.5` | {'en_US': '14.5 - EFKA'} |
| `14.30` | {'en_US': '14.30 - Self-Declared Entity Accounting Source Documents (Dynamic)'} |
| `14.31` | {'en_US': '14.31 - Domestic/Foreign Credit Note'} |
| `15.1` | {'en_US': '15.1 - Contract-Expense'} |
| `16.1` | {'en_US': '16.1 - Rent-Expense'} |
| `17.1` | {'en_US': '17.1 - Payroll'} |
| `17.2` | {'en_US': '17.2 - Amortisations'} |
| `17.3` | {'en_US': '17.3 - Other Income Adjustment/Regularisation Entries – Accounting Base'} |
| `17.4` | {'en_US': '17.4 - Other Income Adjustment/Regularisation Entries – Tax Base'} |
| `17.5` | {'en_US': '17.5 - Other Expense Adjustment/Regularisation Entries – Accounting Base'} |
| `17.6` | {'en_US': '17.6 - Other Expense Adjustment/Regularisation Entries – Tax Base'} |

## `l10n_hr_edi.addendum`

**`business_document_status`** — {'en_US': 'Business document status'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'APPROVED'} |
| `1` | {'en_US': 'REJECTED'} |
| `2` | {'en_US': 'PAYMENT_FULFILLED'} |
| `3` | {'en_US': 'PAYMENT_PARTIALLY_FULLFILLED'} |
| `4` | {'en_US': 'RECEIVING_CONFIRMED'} |
| `99` | {'en_US': 'RECEIVED'} |
| `None` | {'en_US': 'None'} |

**`fiscalization_channel_type`** — {'en_US': 'Delivery channel type'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Delivered via EDI'} |
| `1` | {'en_US': 'Not delivered via EDI'} |

**`fiscalization_status`** — {'en_US': 'Fiscalization status'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Successful'} |
| `1` | {'en_US': 'Unsuccessful'} |
| `2` | {'en_US': 'Pending'} |

**`mer_document_status`** — {'en_US': 'MojEracun document status'}

| Value | Label |
|---|---|
| `20` | {'en_US': 'In validation'} |
| `30` | {'en_US': 'Sent'} |
| `40` | {'en_US': 'Delivered'} |
| `45` | {'en_US': 'Canceled'} |
| `50` | {'en_US': 'Unsuccessful'} |
| `70` | {'en_US': 'Delivered (eReporting)'} |

**`payment_method_type`** — {'en_US': 'Payment Method Type'}

| Value | Label |
|---|---|
| `T` | {'en_US': 'Transakcijski račun'} |
| `O` | {'en_US': 'Obračunsko plaćanje'} |
| `Z` | {'en_US': 'Ostalo'} |

## `l10n_hr_edi.mojeracun_reject_wizard`

**`rejection_type`** — {'en_US': 'Rejection reason type'}

| Value | Label |
|---|---|
| `N` | {'en_US': "'N' - Data discrepancy that does not affect tax calculation"} |
| `U` | {'en_US': "'U' - Data discrepancy that affects tax calculation"} |
| `O` | {'en_US': "'O' - Other"} |

## `l10n_hu_edi.cancellation`

**`code`** — {'en_US': 'Annulment Code'}

| Value | Label |
|---|---|
| `ERRATIC_DATA` | {'en_US': 'ERRATIC_DATA - Erroneous data'} |
| `ERRATIC_INVOICE_NUMBER` | {'en_US': 'ERRATIC_INVOICE_NUMBER - Erroneous invoice number'} |
| `ERRATIC_INVOICE_ISSUE_DATE` | {'en_US': 'ERRATIC_INVOICE_ISSUE_DATE - Erroneous issue date'} |

## `l10n_hu_edi.tax_audit_export`

**`selection_mode`** — {'en_US': 'Selection mode'}

| Value | Label |
|---|---|
| `date` | {'en_US': 'By date'} |
| `name` | {'en_US': 'By serial number'} |

## `l10n_id_efaktur_coretax.document`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

## `l10n_in.pan.entity`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`msme_type`** — {'en_US': 'MSME/Udyam Registration Type'}

| Value | Label |
|---|---|
| `micro` | {'en_US': 'Micro'} |
| `small` | {'en_US': 'Small'} |
| `medium` | {'en_US': 'Medium'} |

**`tds_deduction`** — {'en_US': 'TDS Deduction'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Normal'} |
| `lower` | {'en_US': 'Lower'} |
| `higher` | {'en_US': 'Higher'} |
| `no` | {'en_US': 'No'} |

**`type`** — {'en_US': 'Type'}

| Value | Label |
|---|---|
| `a` | {'en_US': 'Association of Persons'} |
| `b` | {'en_US': 'Body of Individuals'} |
| `c` | {'en_US': 'Company'} |
| `f` | {'en_US': 'Firms'} |
| `g` | {'en_US': 'Government'} |
| `h` | {'en_US': 'Hindu Undivided Family'} |
| `j` | {'en_US': 'Artificial Judicial Person'} |
| `l` | {'en_US': 'Local Authority'} |
| `p` | {'en_US': 'Individual'} |
| `t` | {'en_US': 'Association of Persons for a Trust'} |
| `k` | {'en_US': 'Krish (Trust Krish)'} |

## `l10n_in.section.alert`

**`aggregate_period`** — {'en_US': 'Aggregate Period'}

| Value | Label |
|---|---|
| `monthly` | {'en_US': 'Monthly'} |
| `fiscal_yearly` | {'en_US': 'Financial Yearly'} |

**`consider_amount`** — {'en_US': 'Consider'}

| Value | Label |
|---|---|
| `untaxed_amount` | {'en_US': 'Untaxed Amount'} |
| `total_amount` | {'en_US': 'Total Amount'} |

**`tax_source_type`** — {'en_US': 'Tax Source Type'}

| Value | Label |
|---|---|
| `tds` | {'en_US': 'TDS'} |
| `tcs` | {'en_US': 'TCS'} |

## `l10n_in.withhold.wizard`

**`tds_deduction`** — {'en_US': 'TDS Deduction'} (not stored)

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Normal Deduction'} |
| `lower` | {'en_US': 'Lower Deduction'} |
| `higher` | {'en_US': 'Higher Deduction'} |
| `no` | {'en_US': 'No Deduction'} |

## `l10n_in_edi.cancel`

**`cancel_reason`** — {'en_US': 'Cancel Reason'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Duplicate'} |
| `2` | {'en_US': 'Data Entry Mistake'} |
| `3` | {'en_US': 'Order Cancelled'} |
| `4` | {'en_US': 'Others'} |

## `l10n_it.document.type`

**`type`** — {'en_US': 'Type'}

| Value | Label |
|---|---|
| `sale` | {'en_US': 'Sale'} |
| `purchase` | {'en_US': 'Purchase'} |

## `l10n_it_edi_doi.declaration_of_intent`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`state`** — {'en_US': 'State'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft'} |
| `active` | {'en_US': 'Active'} |
| `revoked` | {'en_US': 'Revoked'} |
| `terminated` | {'en_US': 'Terminated'} |

## `l10n_ke.item.code`

**`tax_rate`** — {'en_US': 'Tax Rate'}

| Value | Label |
|---|---|
| `C` | {'en_US': 'Zero Rated'} |
| `E` | {'en_US': 'Exempted'} |
| `B` | {'en_US': 'Taxable at 8%'} |

## `l10n_latam.check`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`issue_state`** — {'en_US': 'Issue State'}

| Value | Label |
|---|---|
| `handed` | {'en_US': 'Handed'} |
| `debited` | {'en_US': 'Debited'} |
| `voided` | {'en_US': 'Voided'} |

## `l10n_latam.document.type`

**`internal_type`** — {'en_US': 'Internal Type'}

| Value | Label |
|---|---|
| `invoice` | {'en_US': 'Invoices'} |
| `invoice_in` | {'en_US': 'Purchase Invoices'} |
| `debit_note` | {'en_US': 'Debit Notes'} |
| `credit_note` | {'en_US': 'Credit Notes'} |
| `all` | {'en_US': 'All Documents'} |
| `receipt_invoice` | {'en_US': 'Receipt Invoice'} |
| `stock_picking` | {'en_US': 'Stock Delivery'} |
| `purchase_liquidation` | {'en_US': 'Purchase Liquidation'} |
| `withhold` | {'en_US': 'Withhold'} |

**`purchase_aliquots`** — {'en_US': 'Purchase Aliquots'}

| Value | Label |
|---|---|
| `not_zero` | {'en_US': 'Not Zero'} |
| `zero` | {'en_US': 'Zero'} |

## `l10n_pl.bank.account.verification`

**`verification_status`** — {'en_US': 'Verification Status'}

| Value | Label |
|---|---|
| `valid` | {'en_US': 'Valid'} |
| `invalid` | {'en_US': 'Invalid'} |
| `incomplete_partner` | {'en_US': 'Incomplete partner'} |
| `not_found_partner` | {'en_US': 'Partner not found'} |
| `error` | {'en_US': 'An error occurred during check with Government API'} |

## `l10n_ro_edi.document`

**`state`** — {'en_US': 'E-Factura Status'}

| Value | Label |
|---|---|
| `invoice_sent` | {'en_US': 'Sent'} |
| `invoice_refused` | {'en_US': 'Error'} |
| `invoice_validated` | {'en_US': 'Validated'} |
| `stock_sent` | {'en_US': 'Sent'} |
| `stock_sending_failed` | {'en_US': 'Error'} |
| `stock_validated` | {'en_US': 'Validated'} |

## `l10n_tr.nilvera.trailer.plate`

**`plate_number_type`** — {'en_US': 'Plate Number', 'tr_TR': 'Plaka Numarası'}

| Value | Label |
|---|---|
| `vehicle` | {'en_US': 'Vehicle', 'tr_TR': 'Araç'} |
| `trailer` | {'en_US': 'Plate', 'tr_TR': 'Plaka'} |

## `l10n_tr_nilvera_einvoice_extended.account.tax.code`

**`code_type`** — {'en_US': 'Code Type', 'tr_TR': 'Kod Türü'}

| Value | Label |
|---|---|
| `withholding` | {'en_US': 'Withholding', 'tr_TR': 'Tevkifat'} |
| `exception` | {'en_US': 'Exception', 'tr_TR': 'İstisna'} |
| `export_exception` | {'en_US': 'Export Exception', 'tr_TR': 'İhracat İstisna'} |
| `export_registration` | {'en_US': 'Export Registration', 'tr_TR': 'İhracat Kayıt'} |

## `l10n_tw_edi.invoice.print`

**`print_format_b2b`** — {'en_US': 'Print Format (B2B)'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'A4 printing '} |
| `2` | {'en_US': 'A5 printing '} |

**`print_format_b2c`** — {'en_US': 'Print Format (B2C)'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'single-sided printing'} |
| `2` | {'en_US': 'double-sided printing'} |
| `3` | {'en_US': 'printing with thermal paper'} |

## `l10n_vn_edi_viettel.sinvoice.template`

**`template_invoice_type`** — {'en_US': 'Template Invoice Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': '1 - Value-added invoice'} |
| `2` | {'en_US': '2 - Sales invoice'} |
| `3` | {'en_US': '3 - Public assets sales'} |
| `4` | {'en_US': '4 - National reserve sales'} |
| `5` | {'en_US': '5 - Invoice for national reserve sales'} |
| `6` | {'en_US': '6 - Warehouse release note'} |

## `lot.label.layout`

**`label_quantity`** — {'en_US': 'Quantity to print', 'tr_TR': 'Bastırılacak miktar', 'ar_001': 'الكمية لطباعتها'}

| Value | Label |
|---|---|
| `lots` | {'en_US': 'One per lot/SN', 'tr_TR': 'Lot/SN başına bir', 'ar_001': 'واحدة لكل رقم مجموعة/رقم تسلسلي'} |
| `units` | {'en_US': 'One per unit', 'tr_TR': 'Birim başına 1', 'ar_001': 'واحدة لكل وحدة'} |

**`print_format`** — {'en_US': 'Format', 'tr_TR': 'Biçim', 'ar_001': 'التنسيق'}

| Value | Label |
|---|---|
| `4x12` | {'en_US': '4 x 12', 'tr_TR': '4 x 12', 'ar_001': '4 × 12'} |
| `zpl` | {'en_US': 'ZPL Labels', 'tr_TR': 'ZPL Etiketleri', 'ar_001': 'بطاقات عناوين ZPL'} |

## `loyalty.generate.wizard`

**`mode`** — {'en_US': 'For', 'tr_TR': 'İçin', 'ar_001': 'لـ'}

| Value | Label |
|---|---|
| `anonymous` | {'en_US': 'Anonymous Customers', 'tr_TR': 'Anonim Müşteriler', 'ar_001': 'العملاء مجهولو الهوية'} |
| `selected` | {'en_US': 'Selected Customers', 'tr_TR': 'Seçilmiş Müşteriler', 'ar_001': 'العملاء المختارون'} |

## `loyalty.mail`

**`trigger`** — {'en_US': 'When', 'tr_TR': 'Ne zaman', 'ar_001': 'الزمان'}

| Value | Label |
|---|---|
| `create` | {'en_US': 'At Creation', 'tr_TR': 'Oluşturulma Tarihi', 'ar_001': 'عند الإنشاء'} |
| `points_reach` | {'en_US': 'When Reaching', 'tr_TR': 'Ulaşırken', 'ar_001': 'عند الوصول'} |

## `loyalty.program`

**`applies_on`** — {'en_US': 'Applies On', 'tr_TR': 'Buna uygulanır', 'ar_001': 'ينطبق على'}

| Value | Label |
|---|---|
| `current` | {'en_US': 'Current order', 'tr_TR': 'Mevcut sipariş', 'ar_001': 'الطلب الحالي'} |
| `future` | {'en_US': 'Future orders', 'tr_TR': 'Gelecekteki siparişler', 'ar_001': 'الطلبات المستقبلية'} |
| `both` | {'en_US': 'Current & Future orders', 'tr_TR': 'Mevcut ve Gelecekteki siparişler', 'ar_001': 'الطلبات الحالية والمستقبلية'} |

**`program_type`** — {'en_US': 'Program Type', 'tr_TR': 'Kampanya Türü', 'ar_001': 'نوع البرنامج'}

| Value | Label |
|---|---|
| `coupons` | {'en_US': 'Coupons', 'tr_TR': 'Kuponlar', 'ar_001': 'الكوبونات'} |
| `gift_card` | {'en_US': 'Gift Card', 'tr_TR': 'Hediye Kartı', 'ar_001': 'بطاقة الهدايا'} |
| `loyalty` | {'en_US': 'Loyalty Cards', 'tr_TR': 'Sadakat Kartları', 'ar_001': 'بطاقات الولاء'} |
| `promotion` | {'en_US': 'Promotions', 'tr_TR': 'Promosyonlar', 'ar_001': 'العروض'} |
| `ewallet` | {'en_US': 'eWallet', 'tr_TR': 'e-Cüzdan', 'ar_001': 'المحفظة الإلكترونية'} |
| `promo_code` | {'en_US': 'Discount Code', 'tr_TR': 'İndirim Kodu', 'ar_001': 'كود الخصم'} |
| `buy_x_get_y` | {'en_US': 'Buy X Get Y', 'tr_TR': 'X Get Y Satın Alın', 'ar_001': 'اشترِ س، واحصل على ص'} |
| `next_order_coupons` | {'en_US': 'Next Order Coupons', 'tr_TR': 'Sonraki Sipariş Kuponları', 'ar_001': 'كوبونات الطلب التالي'} |

**`trigger`** — {'en_US': 'Trigger', 'tr_TR': 'Tetikleyici', 'ar_001': 'المشغّل'}

| Value | Label |
|---|---|
| `auto` | {'en_US': 'Automatic', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |
| `with_code` | {'en_US': 'Use a code', 'tr_TR': 'Bir kod kullan', 'ar_001': 'استخدام كود'} |

## `loyalty.reward`

**`discount_applicability`** — {'en_US': 'Discount Applicability', 'tr_TR': 'İndirim Uygulanabilirliği', 'ar_001': 'قابلية تطبيق الخصم'}

| Value | Label |
|---|---|
| `order` | {'en_US': 'Order', 'tr_TR': 'Sipariş', 'ar_001': 'الطلب'} |
| `cheapest` | {'en_US': 'Cheapest Product', 'tr_TR': 'En Ucuz Ürün', 'ar_001': 'أرخص منتج'} |
| `specific` | {'en_US': 'Specific Products', 'tr_TR': 'Özel Ürünler', 'ar_001': 'منتجات معينة'} |

**`reward_type`** — {'en_US': 'Reward Type', 'tr_TR': 'Ödül Türü', 'ar_001': 'نوع المكافأة'}

| Value | Label |
|---|---|
| `product` | {'en_US': 'Free Product', 'tr_TR': 'Ücretsiz Ürün', 'ar_001': 'منتج مجاني'} |
| `discount` | {'en_US': 'Discount', 'tr_TR': 'İndirim', 'ar_001': 'الخصم'} |
| `shipping` | {'en_US': 'Free Shipping', 'tr_TR': 'Ücretsiz Sevkiyat', 'ar_001': 'شحن مجاني'} |

## `loyalty.rule`

**`minimum_amount_tax_mode`** — {'en_US': 'Minimum Amount Tax Mode', 'tr_TR': 'Minimum Tutar Vergi Modu', 'ar_001': 'وضع ضريبة المبلغ الأدنى'}

| Value | Label |
|---|---|
| `incl` | {'en_US': 'tax included', 'tr_TR': 'vergi̇ dahi̇l', 'ar_001': 'شامل الضريبة'} |
| `excl` | {'en_US': 'tax excluded', 'tr_TR': 'vergi hariç', 'ar_001': 'غير شامل الضريبة'} |

**`mode`** — {'en_US': 'Application', 'tr_TR': 'Uygulama', 'ar_001': 'التطبيق'}

| Value | Label |
|---|---|
| `auto` | {'en_US': 'Automatic', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |
| `with_code` | {'en_US': 'With a promotion code', 'tr_TR': 'Promosyon koduyla', 'ar_001': 'مع كود ترويجي'} |

## `lunch.alert`

**`mode`** — {'en_US': 'Display', 'tr_TR': 'Ekran', 'ar_001': 'عرض'}

| Value | Label |
|---|---|
| `alert` | {'en_US': 'Alert in app', 'tr_TR': 'Uygulamada uyarı', 'ar_001': 'التنبيه في التطبيق'} |
| `chat` | {'en_US': 'Chat notification', 'tr_TR': 'Sohbet bildirimi', 'ar_001': 'إشعار الدردشة'} |

**`notification_moment`** — {'en_US': 'Notification Moment', 'tr_TR': 'Bildirim Anı', 'ar_001': 'لحظة الإشعار'}

| Value | Label |
|---|---|
| `am` | {'en_US': 'AM', 'tr_TR': 'AM', 'ar_001': 'صباحاً'} |
| `pm` | {'en_US': 'PM', 'tr_TR': 'PM', 'ar_001': 'مساءً'} |

**`recipients`** — {'en_US': 'Recipients', 'tr_TR': 'Alıcılar', 'ar_001': 'المستلمين'}

| Value | Label |
|---|---|
| `everyone` | {'en_US': 'Everyone', 'tr_TR': 'Herkes', 'ar_001': 'الجميع'} |
| `last_week` | {'en_US': 'Employee who ordered last week', 'tr_TR': 'Geçen hafta sipariş veren çalışan', 'ar_001': 'الموظف الذي طلب الأسبوع الماضي'} |
| `last_month` | {'en_US': 'Employee who ordered last month', 'tr_TR': 'Geçen ay sipariş veren çalışan', 'ar_001': 'الموظف الذي طلب الشهر الماضي'} |
| `last_year` | {'en_US': 'Employee who ordered last year', 'tr_TR': 'Geçen yıl sipariş veren çalışan', 'ar_001': 'الموظف الذي طلب السنة الماضية'} |

## `lunch.order`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'To Order', 'tr_TR': 'Sipariş Ver', 'ar_001': 'بحاجة إلى الطلب'} |
| `ordered` | {'en_US': 'Ordered', 'tr_TR': 'Sipariş edildi', 'ar_001': 'تم طلبه'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `confirmed` | {'en_US': 'Received', 'tr_TR': 'Alınan', 'ar_001': 'تم الاستلام'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `lunch.supplier`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`delivery`** — {'en_US': 'Delivery', 'tr_TR': 'Teslimat', 'ar_001': 'التوصيل'}

| Value | Label |
|---|---|
| `delivery` | {'en_US': 'Delivery', 'tr_TR': 'Teslimat', 'ar_001': 'التوصيل'} |
| `no_delivery` | {'en_US': 'No Delivery', 'tr_TR': 'Teslimat Yok', 'ar_001': 'لا يوجد توصيل'} |

**`moment`** — {'en_US': 'Moment', 'tr_TR': 'An', 'ar_001': 'وهلة'}

| Value | Label |
|---|---|
| `am` | {'en_US': 'AM', 'tr_TR': 'AM', 'ar_001': 'صباحاً'} |
| `pm` | {'en_US': 'PM', 'tr_TR': 'PM', 'ar_001': 'مساءً'} |

**`send_by`** — {'en_US': 'Send Order By', 'tr_TR': 'Siparişi Gönder', 'ar_001': 'إرسال الطلب عن طريق'}

| Value | Label |
|---|---|
| `phone` | {'en_US': 'Phone', 'tr_TR': 'Telefon', 'ar_001': 'رقم الهاتف'} |
| `mail` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |

**`topping_quantity_1`** — {'en_US': 'Extra 1 Quantity', 'tr_TR': 'Ekstra 1 Adet', 'ar_001': 'كمية 1 إضافية'}

| Value | Label |
|---|---|
| `0_more` | {'en_US': 'None or More', 'tr_TR': 'Hiçbiri veya Daha Fazlası', 'ar_001': 'لا شيء أو المزيد'} |
| `1_more` | {'en_US': 'One or More', 'tr_TR': 'Bir veya Daha Fazlası', 'ar_001': 'واحد أو أكثر'} |
| `1` | {'en_US': 'Only One', 'tr_TR': 'Sadece bir', 'ar_001': 'واحد فقط'} |

**`topping_quantity_2`** — {'en_US': 'Extra 2 Quantity', 'tr_TR': 'Ekstra 2 Adet', 'ar_001': '2 كمية إضافية'}

| Value | Label |
|---|---|
| `0_more` | {'en_US': 'None or More', 'tr_TR': 'Hiçbiri veya Daha Fazlası', 'ar_001': 'لا شيء أو المزيد'} |
| `1_more` | {'en_US': 'One or More', 'tr_TR': 'Bir veya Daha Fazlası', 'ar_001': 'واحد أو أكثر'} |
| `1` | {'en_US': 'Only One', 'tr_TR': 'Sadece bir', 'ar_001': 'واحد فقط'} |

**`topping_quantity_3`** — {'en_US': 'Extra 3 Quantity', 'tr_TR': 'Ekstra 3 Adet', 'ar_001': '3 كمية إضافية'}

| Value | Label |
|---|---|
| `0_more` | {'en_US': 'None or More', 'tr_TR': 'Hiçbiri veya Daha Fazlası', 'ar_001': 'لا شيء أو المزيد'} |
| `1_more` | {'en_US': 'One or More', 'tr_TR': 'Bir veya Daha Fazlası', 'ar_001': 'واحد أو أكثر'} |
| `1` | {'en_US': 'Only One', 'tr_TR': 'Sadece bir', 'ar_001': 'واحد فقط'} |

## `mail.activity`

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `mail.activity.mixin`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

## `mail.activity.plan.template`

**`delay_from`** — {'en_US': 'Trigger', 'tr_TR': 'Tetikleyici', 'ar_001': 'المشغّل'}

| Value | Label |
|---|---|
| `before_plan_date` | {'en_US': 'Before Plan Date', 'tr_TR': 'Plan Tarihinden Önce', 'ar_001': 'قبل تاريخ الخطة'} |
| `after_plan_date` | {'en_US': 'After Plan Date', 'tr_TR': 'Plan Tarihinden Sonra', 'ar_001': 'بعد تاريخ الخطة'} |

**`delay_unit`** — {'en_US': 'Delay units', 'tr_TR': 'Gecikme birimleri', 'ar_001': 'وحدات التأخير'}

| Value | Label |
|---|---|
| `days` | {'en_US': 'days', 'tr_TR': 'gün', 'ar_001': 'أيام'} |
| `weeks` | {'en_US': 'weeks', 'tr_TR': 'haftalar', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'months', 'tr_TR': 'ay', 'ar_001': 'شهور'} |

**`responsible_type`** — {'en_US': 'Assignment', 'tr_TR': 'Atama', 'ar_001': 'التعيين'}

| Value | Label |
|---|---|
| `on_demand` | {'en_US': 'Ask at launch', 'tr_TR': 'Başlangıçta sor', 'ar_001': 'السؤال عند الإطلاق'} |
| `other` | {'en_US': 'Default user', 'tr_TR': 'Varsayılan kullanıcı', 'ar_001': 'المستخدم الافتراضي'} |
| `coach` | {'en_US': 'Coach', 'tr_TR': 'Şef', 'ar_001': 'مدرب'} |
| `manager` | {'en_US': 'Manager', 'tr_TR': 'Yönetici', 'ar_001': 'المدير'} |
| `employee` | {'en_US': 'Employee', 'tr_TR': 'Çalışan', 'ar_001': 'الموظف'} |
| `fleet_manager` | {'en_US': 'Fleet Manager', 'tr_TR': 'Filo Yöneticisi', 'ar_001': 'مدير الأسطول'} |

## `mail.activity.type`

**`category`** — {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'}

| Value | Label |
|---|---|
| `default` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |
| `upload_file` | {'en_US': 'Upload Document', 'tr_TR': 'Belgeyi Yükle', 'ar_001': 'رفع المستند'} |
| `phonecall` | {'en_US': 'Phonecall', 'tr_TR': 'Telefon görüşmesi', 'ar_001': 'مكالمة هاتفية'} |
| `meeting` | {'en_US': 'Meeting', 'tr_TR': 'Toplantı', 'ar_001': 'الاجتماع'} |

**`chaining_type`** — {'en_US': 'Chaining Type', 'tr_TR': 'Zincirleme Tip', 'ar_001': 'نوع التسلسل'}

| Value | Label |
|---|---|
| `suggest` | {'en_US': 'Suggest Next Activity', 'tr_TR': 'Sonraki Etkinliği Öner', 'ar_001': 'اقتراح النشاط التالي'} |
| `trigger` | {'en_US': 'Trigger Next Activity', 'tr_TR': 'Sonraki Aktiviteyi Tetikle', 'ar_001': 'تشغيل النشاط التالي'} |

**`decoration_type`** — {'en_US': 'Decoration Type', 'tr_TR': 'Dekorasyon Tipi', 'ar_001': 'نوع الزخرفة'}

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`delay_from`** — {'en_US': 'Delay Type', 'tr_TR': 'Gecikme Türü', 'ar_001': 'نوع التأخير'}

| Value | Label |
|---|---|
| `current_date` | {'en_US': 'after previous activity completion date', 'tr_TR': 'önceki faaliyet tamamlanma tarihinden sonra', 'ar_001': 'بعد تاريخ إكمال النشاط السابق'} |
| `previous_activity` | {'en_US': 'after previous activity deadline', 'tr_TR': 'önceki aktivite süresinden sonra', 'ar_001': 'بعد الموعد النهائي للنشاط السابق'} |

**`delay_unit`** — {'en_US': 'Delay units', 'tr_TR': 'Gecikme birimleri', 'ar_001': 'وحدات التأخير'}

| Value | Label |
|---|---|
| `days` | {'en_US': 'days', 'tr_TR': 'gün', 'ar_001': 'أيام'} |
| `weeks` | {'en_US': 'weeks', 'tr_TR': 'haftalar', 'ar_001': 'أسابيع'} |
| `months` | {'en_US': 'months', 'tr_TR': 'ay', 'ar_001': 'شهور'} |

## `mail.alias`

**`alias_contact`** — {'en_US': 'Alias Contact Security', 'tr_TR': 'Rumuz Kontak Güvenliği', 'ar_001': 'أمان ألقاب جهات الاتصال'}

| Value | Label |
|---|---|
| `everyone` | {'en_US': 'Everyone', 'tr_TR': 'Herkes', 'ar_001': 'الجميع'} |
| `partners` | {'en_US': 'Authenticated Partners', 'tr_TR': 'Kimliği Doğrulanmış İş Ortakları', 'ar_001': 'الشركاء المعتمدون'} |
| `followers` | {'en_US': 'Followers only', 'tr_TR': 'Yalnızca takipçiler', 'ar_001': 'المتابعين فقط'} |
| `employees` | {'en_US': 'Authenticated Employees', 'tr_TR': 'Kimliği Doğrulanan Personel', 'ar_001': 'الموظفين المعتمدين'} |

**`alias_status`** — {'en_US': 'Alias Status', 'tr_TR': 'Takma Ad Durumu', 'ar_001': 'حالة اللقب'}

| Value | Label |
|---|---|
| `not_tested` | {'en_US': 'Not Tested', 'tr_TR': 'Test Edilmedi', 'ar_001': 'لم يتم اختباره'} |
| `valid` | {'en_US': 'Valid', 'tr_TR': 'Geçerli', 'ar_001': 'صالح'} |
| `invalid` | {'en_US': 'Invalid', 'tr_TR': 'Geçersiz', 'ar_001': 'غير صالح'} |

## `mail.compose.message`

**`composition_comment_option`** — {'en_US': 'Comment Options', 'tr_TR': 'Yorum Seçenekleri', 'ar_001': 'خيارات التعليق'}

| Value | Label |
|---|---|
| `reply_all` | {'en_US': 'Reply-All', 'tr_TR': 'Hepsini Yanıtla', 'ar_001': 'الرد على الكل'} |
| `forward` | {'en_US': 'Forward', 'tr_TR': 'İleri', 'ar_001': 'للأمام'} |

**`composition_mode`** — {'en_US': 'Composition mode', 'tr_TR': 'Kompozisyon modu', 'ar_001': 'وضع الإنشاء'}

| Value | Label |
|---|---|
| `comment` | {'en_US': 'Post on a document', 'tr_TR': 'Bir dokümanda yayınla', 'ar_001': 'النشر على مستند'} |
| `mass_mail` | {'en_US': 'Email Mass Mailing', 'tr_TR': 'Toplu E-posta Gönderimi', 'ar_001': 'إرسال بريد إلكتروني جماعي'} |

**`message_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `auto_comment` | {'en_US': 'Automated Targeted Notification', 'tr_TR': 'Otomatik Hedefli Bildirim', 'ar_001': 'الإشعارات المستهدفة المؤتمتة'} |
| `comment` | {'en_US': 'Comment', 'tr_TR': 'Yorum', 'ar_001': 'تعليق'} |
| `notification` | {'en_US': 'System notification', 'tr_TR': 'Sistem bildirimi', 'ar_001': 'إشعار من النظام'} |

**`reply_to_mode`** — {'en_US': 'Replies', 'tr_TR': 'Cevaplar', 'ar_001': 'الردود'} (not stored)

| Value | Label |
|---|---|
| `update` | {'en_US': 'Store email and replies in the chatter of each record', 'tr_TR': 'E-postayı ve yanıtları her kaydın mesajında saklayın', 'ar_001': 'قم بتجزين البريد والردود في الدردشة لكل سجل'} |
| `new` | {'en_US': 'Collect replies on a specific email address', 'tr_TR': 'Belirli bir e-posta adresindeki yanıtları toplama', 'ar_001': 'قم بجمع الردود في عنوان بريد محدد'} |

## `mail.followers.edit`

**`operation`** — {'en_US': 'Operation', 'tr_TR': 'Operasyon', 'ar_001': 'العملية'}

| Value | Label |
|---|---|
| `add` | {'en_US': 'Add', 'tr_TR': 'Ekleyin', 'ar_001': 'إضافة'} |
| `remove` | {'en_US': 'Remove', 'tr_TR': 'Kaldır', 'ar_001': 'إزالة'} |

## `mail.group`

**`access_mode`** — {'en_US': 'Privacy', 'tr_TR': 'Özel', 'ar_001': 'الخصوصية'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Everyone', 'tr_TR': 'Herkes', 'ar_001': 'الجميع'} |
| `members` | {'en_US': 'Members only', 'tr_TR': 'Sadece Üyeler', 'ar_001': 'الأعضاء فقط'} |
| `groups` | {'en_US': 'Selected group of users', 'tr_TR': 'Seçilen kullanıcı grubu', 'ar_001': 'مجموعة المستخدمين المحددة'} |

## `mail.group.message`

**`author_moderation`** — {'en_US': 'Author Moderation Status', 'tr_TR': 'Yazar Moderasyon Durumu', 'ar_001': 'حالة إشراف الكاتب'} (not stored)

| Value | Label |
|---|---|
| `ban` | {'en_US': 'Banned', 'tr_TR': 'Yasaklı', 'ar_001': 'محظور'} |
| `allow` | {'en_US': 'Whitelisted', 'tr_TR': 'Beyaz listeye alındı', 'ar_001': 'تم رفع الحظر'} |

**`moderation_status`** — {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `pending_moderation` | {'en_US': 'Pending Moderation', 'tr_TR': 'Denetleme Bekliyor', 'ar_001': 'بانتظار الإشراف'} |
| `accepted` | {'en_US': 'Accepted', 'tr_TR': 'Kabul Edildi', 'ar_001': 'تم القبول'} |
| `rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

## `mail.group.message.reject`

**`action`** — {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'}

| Value | Label |
|---|---|
| `reject` | {'en_US': 'Reject', 'tr_TR': 'Reddetme', 'ar_001': 'رفض'} |
| `ban` | {'en_US': 'Ban', 'tr_TR': 'Engel', 'ar_001': 'حظر'} |

## `mail.group.moderation`

**`status`** — {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `allow` | {'en_US': 'Always Allow', 'tr_TR': 'Daima İzin Ver', 'ar_001': 'السماح دائمًا'} |
| `ban` | {'en_US': 'Permanent Ban', 'tr_TR': 'Kalıcı Engel', 'ar_001': 'حظر دائم'} |

## `mail.ice.server`

**`server_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `stun` | {'en_US': 'stun:', 'tr_TR': 'stun:', 'ar_001': 'إيقاف:'} |
| `turn` | {'en_US': 'turn:', 'tr_TR': 'dönüş:', 'ar_001': 'الدوران:'} |

## `mail.mail`

**`failure_type`** — {'en_US': 'Failure type', 'tr_TR': 'Başarısızlık türü', 'ar_001': 'نوع الفشل'}

| Value | Label |
|---|---|
| `unknown` | {'en_US': 'Unknown error', 'tr_TR': 'Bilinmeyen hata', 'ar_001': 'خطأ غير معروف'} |
| `mail_spam` | {'en_US': 'Detected As Spam', 'tr_TR': 'Spam Olarak Algılandı', 'ar_001': 'تم رصده كبريد مزعج'} |
| `mail_email_invalid` | {'en_US': 'Invalid email address', 'tr_TR': 'Geçersiz e-posta adresi', 'ar_001': 'عنوان البريد الإلكتروني غير صالح'} |
| `mail_email_missing` | {'en_US': 'Missing email', 'tr_TR': 'Eksik e-posta', 'ar_001': 'البريد الإلكتروني المفقود'} |
| `mail_from_invalid` | {'en_US': 'Invalid from address', 'tr_TR': 'Geçersiz gönderen adresi', 'ar_001': 'عنوان "من" غير صالح'} |
| `mail_from_missing` | {'en_US': 'Missing from address', 'tr_TR': "'Kimden adresi' eksik", 'ar_001': 'عنوان "من" غير موجود'} |
| `mail_smtp` | {'en_US': 'Connection failed (outgoing mail server problem)', 'tr_TR': 'Bağlantı başarısız (giden posta sunucusu sorunu)', 'ar_001': 'فشل الاتصال (مشكلة في خادم رسائل البريد الإلكتروني الصادرة)'} |
| `mail_bl` | {'en_US': 'Blacklisted Address', 'tr_TR': 'Kara Listeye Alınmış Adres', 'ar_001': 'العنوان المدرج في القائمة السوداء'} |
| `mail_optout` | {'en_US': 'Opted Out', 'tr_TR': 'Devre Dışı Bırakıldı', 'ar_001': 'انسحب'} |
| `mail_dup` | {'en_US': 'Duplicated Email', 'tr_TR': 'Kopyalanmış E-posta', 'ar_001': 'البريد الإلكتروني المستنسخ'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `outgoing` | {'en_US': 'Outgoing', 'tr_TR': 'Giden', 'ar_001': 'الصادرة'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `received` | {'en_US': 'Received', 'tr_TR': 'Alınan', 'ar_001': 'تم الاستلام'} |
| `exception` | {'en_US': 'Delivery Failed', 'tr_TR': 'Gönderim Başarısız', 'ar_001': 'فشل التسليم'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `mail.message`

**`message_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `email` | {'en_US': 'Incoming Email', 'tr_TR': 'Gelen E-posta', 'ar_001': 'البريد الإلكتروني الوارد'} |
| `comment` | {'en_US': 'Comment', 'tr_TR': 'Yorum', 'ar_001': 'تعليق'} |
| `email_outgoing` | {'en_US': 'Outgoing Email', 'tr_TR': 'Giden E-posta', 'ar_001': 'البريد الإلكتروني الصادر'} |
| `notification` | {'en_US': 'System notification', 'tr_TR': 'Sistem bildirimi', 'ar_001': 'إشعار من النظام'} |
| `auto_comment` | {'en_US': 'Automated Targeted Notification', 'tr_TR': 'Otomatik Hedefli Bildirim', 'ar_001': 'الإشعارات المستهدفة المؤتمتة'} |
| `out_of_office` | {'en_US': 'Out-of-office Message', 'tr_TR': 'Ofis Dışı Mesaj', 'ar_001': 'رسالة خارج المكتب'} |
| `user_notification` | {'en_US': 'User Specific Notification', 'tr_TR': 'Kullanıcıya Özel Bildirim', 'ar_001': 'إشعار خاص بالمستخدم'} |
| `sms` | {'en_US': 'SMS', 'tr_TR': 'SMS', 'ar_001': 'الرسائل النصية القصيرة'} |
| `snailmail` | {'en_US': 'Snailmail', 'tr_TR': 'Snailmail', 'ar_001': 'البريد التقليدي'} |

## `mail.notification`

**`failure_type`** — {'en_US': 'Failure type', 'tr_TR': 'Başarısızlık türü', 'ar_001': 'نوع الفشل'}

| Value | Label |
|---|---|
| `unknown` | {'en_US': 'Unknown error', 'tr_TR': 'Bilinmeyen hata', 'ar_001': 'خطأ غير معروف'} |
| `mail_bounce` | {'en_US': 'Bounce', 'tr_TR': 'İletilmeyen E-Posta', 'ar_001': 'الارتداد'} |
| `mail_spam` | {'en_US': 'Detected As Spam', 'tr_TR': 'Spam Olarak Algılandı', 'ar_001': 'تم رصده كبريد مزعج'} |
| `mail_email_invalid` | {'en_US': 'Invalid email address', 'tr_TR': 'Geçersiz e-posta adresi', 'ar_001': 'عنوان البريد الإلكتروني غير صالح'} |
| `mail_email_missing` | {'en_US': 'Missing email address', 'tr_TR': 'Eksik e-posta adresi', 'ar_001': 'عنوان البريد الإلكتروني المفقود'} |
| `mail_from_invalid` | {'en_US': 'Invalid from address', 'tr_TR': 'Geçersiz gönderen adresi', 'ar_001': 'عنوان "من" غير صالح'} |
| `mail_from_missing` | {'en_US': 'Missing from address', 'tr_TR': "'Kimden adresi' eksik", 'ar_001': 'عنوان "من" غير موجود'} |
| `mail_smtp` | {'en_US': 'Connection failed (outgoing mail server problem)', 'tr_TR': 'Bağlantı başarısız (giden posta sunucusu sorunu)', 'ar_001': 'فشل الاتصال (مشكلة في خادم رسائل البريد الإلكتروني الصادرة)'} |
| `mail_bl` | {'en_US': 'Blacklisted Address', 'tr_TR': 'Kara Listeye Alınmış Adres', 'ar_001': 'العنوان المدرج في القائمة السوداء'} |
| `mail_optout` | {'en_US': 'Opted Out', 'tr_TR': 'Devre Dışı Bırakıldı', 'ar_001': 'انسحب'} |
| `mail_dup` | {'en_US': 'Duplicated Email', 'tr_TR': 'Kopyalanmış E-posta', 'ar_001': 'البريد الإلكتروني المستنسخ'} |
| `sms_number_missing` | {'en_US': 'Missing Number', 'tr_TR': 'Kayıp Numara', 'ar_001': 'الرقم مفقود'} |
| `sms_number_format` | {'en_US': 'Wrong Number Format', 'tr_TR': 'Yanlış Sayı Biçimi', 'ar_001': 'صيغة الرقم خطأ'} |
| `sms_credit` | {'en_US': 'Insufficient Credit', 'tr_TR': 'Yetersiz Kredi', 'ar_001': 'الرصيد غير كافٍ'} |
| `sms_country_not_supported` | {'en_US': 'Country Not Supported', 'tr_TR': 'Ülke Desteklenmiyor', 'ar_001': 'الدولة غير مدعومة'} |
| `sms_registration_needed` | {'en_US': 'Country-specific Registration Required', 'tr_TR': 'Ülkeye Özel Kayıt Gerekli', 'ar_001': 'مطلوب التسجيل الخاص بكل بلد'} |
| `sms_server` | {'en_US': 'Server Error', 'tr_TR': 'Sunucu Hatası', 'ar_001': 'خطأ في الخادم'} |
| `sms_acc` | {'en_US': 'Unregistered Account', 'tr_TR': 'Unregistered Account', 'ar_001': 'حساب غير مسجل'} |
| `sms_expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `sms_invalid_destination` | {'en_US': 'Invalid Destination', 'tr_TR': 'Geçersiz Hedef', 'ar_001': 'الوجهة غير صالحة'} |
| `sms_not_allowed` | {'en_US': 'Not Allowed', 'tr_TR': 'İzin verilmedi', 'ar_001': 'غير مسموح'} |
| `sms_not_delivered` | {'en_US': 'Not Delivered', 'tr_TR': 'Teslim Edilmedi', 'ar_001': 'لم يتم توصيلها'} |
| `sms_rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `sn_credit` | {'en_US': 'Snailmail Credit Error', 'tr_TR': 'Snailmail Kredi Hatası', 'ar_001': 'خطأ في رصيد البريد العادي'} |
| `sn_trial` | {'en_US': 'Snailmail Trial Error', 'tr_TR': 'Snailmail Deneme Hatası', 'ar_001': 'خطأ عند تجربة البريد العادي'} |
| `sn_price` | {'en_US': 'Snailmail No Price Available', 'tr_TR': 'Snailmail Fiyat Yok', 'ar_001': 'ليس هناك سعر متاح للبريد العادي'} |
| `sn_fields` | {'en_US': 'Snailmail Missing Required Fields', 'tr_TR': 'Snailmail Gerekli Alanları Eksik', 'ar_001': 'حقول مطلوبة مفقودة في البريد العادي'} |
| `sn_format` | {'en_US': 'Snailmail Format Error', 'tr_TR': 'Snailmail Format Hatası', 'ar_001': 'خطأ في تنسيق البريد العادي'} |
| `sn_error` | {'en_US': 'Snailmail Unknown Error', 'tr_TR': 'Snailmail Bilinmeyen Hata', 'ar_001': 'خطأ غير معروف في البريد العادي'} |
| `twilio_authentication` | {'en_US': 'Authentication Error"', 'tr_TR': 'Kimlik Doğrulama Hatası"'} |
| `twilio_callback` | {'en_US': 'Incorrect callback URL', 'tr_TR': "Yanlış geri arama URL'si"} |
| `twilio_from_missing` | {'en_US': 'Missing From Number', 'tr_TR': 'Numaradan Eksik'} |
| `twilio_from_to` | {'en_US': 'From / To identic', 'tr_TR': 'Kimden / Kime özdeş'} |

**`notification_status`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready to Send', 'tr_TR': 'Gönderime Hazır', 'ar_001': 'جاهز للإرسال'} |
| `process` | {'en_US': 'Processing', 'tr_TR': 'İşleniyor', 'ar_001': 'معالجة'} |
| `pending` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `sent` | {'en_US': 'Delivered', 'tr_TR': 'Teslim Edilen', 'ar_001': 'تم التوصيل'} |
| `bounce` | {'en_US': 'Bounced', 'tr_TR': 'İletilmeyen', 'ar_001': 'الرسائل المرتدة'} |
| `exception` | {'en_US': 'Exception', 'tr_TR': 'İstisna', 'ar_001': 'استثناء'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`notification_type`** — {'en_US': 'Notification Type', 'tr_TR': 'Bildirim türü', 'ar_001': 'نوع الإشعار'}

| Value | Label |
|---|---|
| `inbox` | {'en_US': 'Inbox', 'tr_TR': 'Gelen Kutusu', 'ar_001': 'صندوق الوارد'} |
| `email` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `sms` | {'en_US': 'SMS', 'tr_TR': 'SMS', 'ar_001': 'الرسائل النصية القصيرة'} |
| `snail` | {'en_US': 'Snailmail', 'tr_TR': 'Snailmail', 'ar_001': 'البريد التقليدي'} |

## `mail.presence`

**`status`** — {'en_US': 'IM Status', 'tr_TR': 'Anlık İleti Durumu', 'ar_001': 'حالة المحادثات الفورية'}

| Value | Label |
|---|---|
| `online` | {'en_US': 'Online', 'tr_TR': 'Çevrim içi', 'ar_001': 'عبر الإنترنت'} |
| `away` | {'en_US': 'Away', 'tr_TR': 'Dışarıda', 'ar_001': 'بعيد'} |
| `offline` | {'en_US': 'Offline', 'tr_TR': 'Çevrim dışı', 'ar_001': 'غير متصل بالإنترنت'} |

## `mail.scheduled.message`

**`composition_comment_option`** — {'en_US': 'Comment Options', 'tr_TR': 'Yorum Seçenekleri', 'ar_001': 'خيارات التعليق'}

| Value | Label |
|---|---|
| `reply_all` | {'en_US': 'Reply-All', 'tr_TR': 'Hepsini Yanıtla', 'ar_001': 'الرد على الكل'} |
| `forward` | {'en_US': 'Forward', 'tr_TR': 'İleri', 'ar_001': 'للأمام'} |

## `mail.template`

**`template_category`** — {'en_US': 'Template Category', 'tr_TR': 'Şablon Kategorisi', 'ar_001': 'فئة القالب'} (not stored)

| Value | Label |
|---|---|
| `base_template` | {'en_US': 'Base Template', 'tr_TR': 'Temel Şablon', 'ar_001': 'القالب الأساسي'} |
| `hidden_template` | {'en_US': 'Hidden Template', 'tr_TR': 'Gizli Şablon', 'ar_001': 'قالب مخفي'} |
| `custom_template` | {'en_US': 'Custom Template', 'tr_TR': 'Özel Şablon', 'ar_001': 'قالب مخصص'} |

## `mailing.list.merge`

**`merge_options`** — {'en_US': 'Merge Option', 'tr_TR': 'Birleştirme Seçeneği', 'ar_001': 'خيار الدمج'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'Merge into a new mailing list', 'tr_TR': 'Yeni bir posta listesiyle birleştir', 'ar_001': 'الدمج في قائمة بريدية جديدة'} |
| `existing` | {'en_US': 'Merge into an existing mailing list', 'tr_TR': 'Mevcut bir posta listesiyle birleştir', 'ar_001': 'الدمج في قائمة بريدية موجودة'} |

## `mailing.mailing`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`mailing_type`** — {'en_US': 'Mailing Type', 'tr_TR': 'Posta Türü', 'ar_001': 'نوع البريد'}

| Value | Label |
|---|---|
| `mail` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `sms` | {'en_US': 'SMS', 'tr_TR': 'SMS', 'ar_001': 'الرسائل النصية القصيرة'} |

**`reply_to_mode`** — {'en_US': 'Reply-To Mode', 'tr_TR': 'Yanıtlama Adresi', 'ar_001': 'وضع الرد على'}

| Value | Label |
|---|---|
| `update` | {'en_US': 'Recipient Followers', 'tr_TR': 'Alıcı Takipçileri', 'ar_001': 'المتابعين المستلمين'} |
| `new` | {'en_US': 'Specified Email Address', 'tr_TR': 'Belirtilen E-Posta Adresi', 'ar_001': 'عنوان بريد إلكتروني محدد'} |

**`schedule_type`** — {'en_US': 'Schedule', 'tr_TR': 'Planla', 'ar_001': 'جدولة'}

| Value | Label |
|---|---|
| `now` | {'en_US': 'Send now', 'tr_TR': 'Şimdi Gönder', 'ar_001': 'إرسال الآن'} |
| `scheduled` | {'en_US': 'Send on', 'tr_TR': 'Gönder', 'ar_001': 'إرسال في'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `in_queue` | {'en_US': 'In Queue', 'tr_TR': 'Sırada', 'ar_001': 'في قائمة الانتظار'} |
| `sending` | {'en_US': 'Sending', 'tr_TR': 'Gönderiyor', 'ar_001': 'جاري الإرسال'} |
| `done` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |

## `mailing.trace`

**`failure_type`** — {'en_US': 'Failure type', 'tr_TR': 'Başarısızlık türü', 'ar_001': 'نوع الفشل'}

| Value | Label |
|---|---|
| `unknown` | {'en_US': 'Unknown error', 'tr_TR': 'Bilinmeyen hata', 'ar_001': 'خطأ غير معروف'} |
| `mail_bounce` | {'en_US': 'Bounce', 'tr_TR': 'İletilmeyen E-Posta', 'ar_001': 'الارتداد'} |
| `mail_spam` | {'en_US': 'Detected As Spam', 'tr_TR': 'Spam Olarak Algılandı', 'ar_001': 'تم رصده كبريد مزعج'} |
| `mail_email_invalid` | {'en_US': 'Invalid email address', 'tr_TR': 'Geçersiz e-posta adresi', 'ar_001': 'عنوان البريد الإلكتروني غير صالح'} |
| `mail_email_missing` | {'en_US': 'Missing email address', 'tr_TR': 'Eksik e-posta adresi', 'ar_001': 'عنوان البريد الإلكتروني المفقود'} |
| `mail_from_invalid` | {'en_US': 'Invalid from address', 'tr_TR': 'Geçersiz gönderen adresi', 'ar_001': 'عنوان "من" غير صالح'} |
| `mail_from_missing` | {'en_US': 'Missing from address', 'tr_TR': "'Kimden adresi' eksik", 'ar_001': 'عنوان "من" غير موجود'} |
| `mail_smtp` | {'en_US': 'Connection failed (outgoing mail server problem)', 'tr_TR': 'Bağlantı başarısız (giden posta sunucusu sorunu)', 'ar_001': 'فشل الاتصال (مشكلة في خادم رسائل البريد الإلكتروني الصادرة)'} |
| `mail_bl` | {'en_US': 'Blacklisted Address', 'tr_TR': 'Kara Listeye Alınmış Adres', 'ar_001': 'العنوان المدرج في القائمة السوداء'} |
| `mail_dup` | {'en_US': 'Duplicated Email', 'tr_TR': 'Kopyalanmış E-posta', 'ar_001': 'البريد الإلكتروني المستنسخ'} |
| `mail_optout` | {'en_US': 'Opted Out', 'tr_TR': 'Devre Dışı Bırakıldı', 'ar_001': 'انسحب'} |
| `sms_number_missing` | {'en_US': 'Missing Number', 'tr_TR': 'Kayıp Numara', 'ar_001': 'الرقم مفقود'} |
| `sms_number_format` | {'en_US': 'Wrong Number Format', 'tr_TR': 'Yanlış Sayı Biçimi', 'ar_001': 'صيغة الرقم خطأ'} |
| `sms_credit` | {'en_US': 'Insufficient Credit', 'tr_TR': 'Yetersiz Kredi', 'ar_001': 'الرصيد غير كافٍ'} |
| `sms_country_not_supported` | {'en_US': 'Country Not Supported', 'tr_TR': 'Ülke Desteklenmiyor', 'ar_001': 'الدولة غير مدعومة'} |
| `sms_registration_needed` | {'en_US': 'Country-specific Registration Required', 'tr_TR': 'Ülkeye Özel Kayıt Gerekli', 'ar_001': 'مطلوب التسجيل الخاص بكل بلد'} |
| `sms_server` | {'en_US': 'Server Error', 'tr_TR': 'Sunucu Hatası', 'ar_001': 'خطأ في الخادم'} |
| `sms_acc` | {'en_US': 'Unregistered Account', 'tr_TR': 'Unregistered Account', 'ar_001': 'حساب غير مسجل'} |
| `sms_blacklist` | {'en_US': 'Blacklisted', 'tr_TR': 'Kara Liste', 'ar_001': 'مدرج في القائمة السوداء'} |
| `sms_duplicate` | {'en_US': 'Duplicate', 'tr_TR': 'Kopyala', 'ar_001': 'إنشاء نسخة مطابقة'} |
| `sms_optout` | {'en_US': 'Opted Out', 'tr_TR': 'Devre Dışı Bırakıldı', 'ar_001': 'انسحب'} |
| `sms_expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `sms_invalid_destination` | {'en_US': 'Invalid Destination', 'tr_TR': 'Geçersiz Hedef', 'ar_001': 'الوجهة غير صالحة'} |
| `sms_not_allowed` | {'en_US': 'Not Allowed', 'tr_TR': 'İzin verilmedi', 'ar_001': 'غير مسموح'} |
| `sms_not_delivered` | {'en_US': 'Not Delivered', 'tr_TR': 'Teslim Edilmedi', 'ar_001': 'لم يتم توصيلها'} |
| `sms_rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `twilio_authentication` | {'en_US': 'Authentication Error"', 'tr_TR': 'Doğrulama Hatası"'} |
| `twilio_callback` | {'en_US': 'Incorrect callback URL', 'tr_TR': "Yanlış geri arama URL'si"} |
| `twilio_from_missing` | {'en_US': 'Missing From Number', 'tr_TR': 'Numaradan Eksik'} |
| `twilio_from_to` | {'en_US': 'From / To identic', 'tr_TR': 'Kimden / Kime eş'} |

**`trace_status`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `outgoing` | {'en_US': 'Outgoing', 'tr_TR': 'Giden', 'ar_001': 'الصادرة'} |
| `process` | {'en_US': 'Processing', 'tr_TR': 'İşleniyor', 'ar_001': 'معالجة'} |
| `pending` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `sent` | {'en_US': 'Delivered', 'tr_TR': 'Teslim Edilen', 'ar_001': 'تم التوصيل'} |
| `open` | {'en_US': 'Opened', 'tr_TR': 'Açıldı', 'ar_001': 'مفتوحة'} |
| `reply` | {'en_US': 'Replied', 'tr_TR': 'Cevaplandı', 'ar_001': 'تم الرد'} |
| `bounce` | {'en_US': 'Bounced', 'tr_TR': 'İletilmeyen', 'ar_001': 'الرسائل المرتدة'} |
| `error` | {'en_US': 'Exception', 'tr_TR': 'İstisna', 'ar_001': 'استثناء'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`trace_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `mail` | {'en_US': 'Email', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |
| `sms` | {'en_US': 'SMS', 'tr_TR': 'SMS', 'ar_001': 'الرسائل النصية القصيرة'} |

## `mailing.trace.report`

**`mailing_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `mail` | {'en_US': 'Mail', 'tr_TR': 'E-Posta', 'ar_001': 'البريد الإلكتروني'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `test` | {'en_US': 'Tested', 'tr_TR': 'Test Edilmiş', 'ar_001': 'تم الاختبار'} |
| `done` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |

## `maintenance.equipment`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`equipment_assign_to`** — {'en_US': 'Used By', 'tr_TR': 'Kullanan', 'ar_001': 'تم استخدامه بواسطة'}

| Value | Label |
|---|---|
| `department` | {'en_US': 'Department', 'tr_TR': 'Departman', 'ar_001': 'القسم'} |
| `employee` | {'en_US': 'Employee', 'tr_TR': 'Çalışan', 'ar_001': 'الموظف'} |
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |

## `maintenance.request`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`instruction_type`** — {'en_US': 'Instruction', 'tr_TR': 'Açıklama', 'ar_001': 'تعليمات'}

| Value | Label |
|---|---|
| `pdf` | {'en_US': 'PDF', 'tr_TR': 'PDF', 'ar_001': 'PDF'} |
| `google_slide` | {'en_US': 'Google Slide', 'tr_TR': 'Google Slayt', 'ar_001': 'شرائح Google'} |
| `text` | {'en_US': 'Text', 'tr_TR': 'Metin', 'ar_001': 'النص'} |

**`kanban_state`** — {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `done` | {'en_US': 'Ready for next stage', 'tr_TR': 'Sonraki aşaması için hazır', 'ar_001': 'جاهز للمرحلة التالية'} |

**`maintenance_type`** — {'en_US': 'Maintenance Type', 'tr_TR': 'Bakım Tipi', 'ar_001': 'نوع الصيانة'}

| Value | Label |
|---|---|
| `corrective` | {'en_US': 'Corrective', 'tr_TR': 'Düzeltici', 'ar_001': 'تصحيحي'} |
| `preventive` | {'en_US': 'Preventive', 'tr_TR': 'Önleyici', 'ar_001': 'وقائية'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Very Low', 'tr_TR': 'Çok Düşük', 'ar_001': 'منخفض جدًا'} |
| `1` | {'en_US': 'Low', 'tr_TR': 'Düşük', 'ar_001': 'منخفض'} |
| `2` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `3` | {'en_US': 'High', 'tr_TR': 'Yüksek', 'ar_001': 'مرتفع'} |

**`repeat_type`** — {'en_US': 'Until', 'tr_TR': 'Bitiş', 'ar_001': 'حتى'}

| Value | Label |
|---|---|
| `forever` | {'en_US': 'Forever', 'tr_TR': 'Sonsuza dek', 'ar_001': 'للأبد'} |
| `until` | {'en_US': 'Until', 'tr_TR': 'Bitiş', 'ar_001': 'حتى'} |

**`repeat_unit`** — {'en_US': 'Repeat Unit', 'tr_TR': 'Tekrar Birimi', 'ar_001': 'تكرار الوحدة'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `week` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `month` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |
| `year` | {'en_US': 'Years', 'tr_TR': 'Yıllar', 'ar_001': 'سنوات'} |

## `microsoft.calendar.account.reset`

**`delete_policy`** — {'en_US': "User's Existing Events", 'tr_TR': 'Kullanıcının Mevcut Etkinlikleri', 'ar_001': 'فعاليات المستخدم الموجودة بالفعل'}

| Value | Label |
|---|---|
| `dont_delete` | {'en_US': 'Leave them untouched', 'tr_TR': 'Onları el değmeden bırakın', 'ar_001': 'اتركهم كما هم'} |
| `delete_microsoft` | {'en_US': 'Delete from the current Microsoft Calendar account', 'tr_TR': 'Geçerli Microsoft Takvim hesabından sil', 'ar_001': 'الحذف من حساب تقويم Microsoft الحالي'} |
| `delete_odoo` | {'en_US': 'Delete from Odoo', 'tr_TR': "Odoo'dan sil", 'ar_001': 'الحذف من أودو'} |
| `delete_both` | {'en_US': 'Delete from both', 'tr_TR': 'İkisinden de sil', 'ar_001': 'الحذف من كليهما'} |

**`sync_policy`** — {'en_US': 'Next Synchronization', 'tr_TR': 'Sonraki Senkronizasyon', 'ar_001': 'المزامنة التالية'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'Synchronize only new events', 'tr_TR': 'Yalnızca yeni etkinlikleri senkronize et', 'ar_001': 'مزامنة الفعاليات الجديدة فقط'} |
| `all` | {'en_US': 'Synchronize all existing events', 'tr_TR': 'Mevcut tüm etkinlikleri senkronize et', 'ar_001': 'مزامنة كافة الفعاليات الموجودة'} |

## `mrp.bom`

**`consumption`** — {'en_US': 'Flexible Consumption', 'tr_TR': 'Esnek Tüketim', 'ar_001': 'الاستهلاك المرن'}

| Value | Label |
|---|---|
| `flexible` | {'en_US': 'Allowed', 'tr_TR': 'İzin verildi', 'ar_001': 'مسموح به'} |
| `warning` | {'en_US': 'Allowed with warning', 'tr_TR': 'Uyarı ile izin verildi', 'ar_001': 'مسموح به مع التحذير'} |
| `strict` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |

**`ready_to_produce`** — {'en_US': 'Manufacturing Readiness', 'tr_TR': 'Üretim Hazırlığı', 'ar_001': 'جاهزية التصنيع'}

| Value | Label |
|---|---|
| `all_available` | {'en_US': ' When all components are available', 'tr_TR': ' Tüm bileşenler mevcut olduğunda', 'ar_001': ' عندما تصبح كافة المكونات متاحة'} |
| `asap` | {'en_US': 'When components for 1st operation are available', 'tr_TR': '1. operasyon için bileşenler mevcut olduğunda', 'ar_001': 'عندما تكون كافة المكونات المطلوبة للعملية الأولى متاحة'} |

**`type`** — {'en_US': 'BoM Type', 'tr_TR': 'BoM Türü', 'ar_001': 'نوع قائمة المواد'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Manufacture this product', 'tr_TR': 'Bu ürünü üretin', 'ar_001': 'تصنيع هذا المنتج'} |
| `phantom` | {'en_US': 'Kit', 'tr_TR': 'Kit', 'ar_001': 'عدة'} |
| `subcontract` | {'en_US': 'Subcontracting', 'tr_TR': 'Fason', 'ar_001': 'التعاقد من الباطن'} |

## `mrp.consumption.warning`

**`consumption`** — {'en_US': 'Consumption', 'tr_TR': 'Tüketim', 'ar_001': 'الاستهلاك'} (not stored)

| Value | Label |
|---|---|
| `flexible` | {'en_US': 'Allowed', 'tr_TR': 'İzin verildi', 'ar_001': 'مسموح به'} |
| `warning` | {'en_US': 'Allowed with warning', 'tr_TR': 'Uyarı ile izin verildi', 'ar_001': 'مسموح به مع التحذير'} |
| `strict` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |

## `mrp.production`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

**`components_availability_state`** — {'en_US': 'Components Availability State', 'tr_TR': 'Bileşenlerin Uygunluk Durumu', 'ar_001': 'حالة توافر المكونات'} (not stored)

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `expected` | {'en_US': 'Expected', 'tr_TR': 'Beklenen', 'ar_001': 'المتوقع'} |
| `late` | {'en_US': 'Late', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `unavailable` | {'en_US': 'Not Available', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غير متاح'} |

**`consumption`** — {'en_US': 'Consumption', 'tr_TR': 'Tüketim', 'ar_001': 'الاستهلاك'}

| Value | Label |
|---|---|
| `flexible` | {'en_US': 'Allowed', 'tr_TR': 'İzin verildi', 'ar_001': 'مسموح به'} |
| `warning` | {'en_US': 'Allowed with warning', 'tr_TR': 'Uyarı ile izin verildi', 'ar_001': 'مسموح به مع التحذير'} |
| `strict` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `1` | {'en_US': 'Urgent', 'tr_TR': 'Acil', 'ar_001': 'عاجل'} |

**`reservation_state`** — {'en_US': 'MO Readiness', 'tr_TR': 'MO Hazırlık', 'ar_001': 'جاهزية أمر التصنيع'}

| Value | Label |
|---|---|
| `confirmed` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `assigned` | {'en_US': 'Ready', 'tr_TR': 'Hazır', 'ar_001': 'جاهز'} |
| `waiting` | {'en_US': 'Waiting Another Operation', 'tr_TR': 'Başka Bir İşlem Bekliyor', 'ar_001': 'في انتظار عملية أخرى'} |

**`search_date_category`** — {'en_US': 'Date Category', 'tr_TR': 'Tarih Kategorisi', 'ar_001': 'فئة التاريخ'} (not stored)

| Value | Label |
|---|---|
| `before` | {'en_US': 'Before', 'tr_TR': 'Önce', 'ar_001': 'قبل'} |
| `yesterday` | {'en_US': 'Yesterday', 'tr_TR': 'Dün', 'ar_001': 'البارحة'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `day_1` | {'en_US': 'Tomorrow', 'tr_TR': 'Yarın', 'ar_001': 'غدًا'} |
| `day_2` | {'en_US': 'The day after tomorrow', 'tr_TR': 'Yarından sonraki gün', 'ar_001': 'بعد غد'} |
| `after` | {'en_US': 'After', 'tr_TR': 'Sonra', 'ar_001': 'بعد'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `confirmed` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `to_close` | {'en_US': 'To Close', 'tr_TR': 'Kapatılacak', 'ar_001': 'للإقفال'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `mrp.routing.workcenter`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

**`cost_mode`** — {'en_US': 'Cost based on', 'tr_TR': 'Maliyete dayalı'}

| Value | Label |
|---|---|
| `actual` | {'en_US': 'Actual time', 'tr_TR': 'Gerçek zaman'} |
| `estimated` | {'en_US': 'Theorical time', 'tr_TR': 'Teorik zaman'} |

**`time_mode`** — {'en_US': 'Duration Computation', 'tr_TR': 'Süre Hesaplama', 'ar_001': 'احتساب المدة'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Fixed', 'tr_TR': 'Sabit', 'ar_001': 'ثابت'} |
| `auto` | {'en_US': 'Computed', 'tr_TR': 'Hesaplanmış', 'ar_001': 'محسوب'} |

## `mrp.unbuild`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `mrp.workcenter`

**`working_state`** — {'en_US': 'Workcenter Status', 'tr_TR': 'İş Merkezi Durumu', 'ar_001': 'حالة مركز العمل'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `done` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |

## `mrp.workcenter.productivity.loss.type`

**`loss_type`** — {'en_US': 'Category', 'tr_TR': 'Kategori', 'ar_001': 'الفئة'}

| Value | Label |
|---|---|
| `availability` | {'en_US': 'Availability', 'tr_TR': 'Müsaitlik', 'ar_001': 'التوافر'} |
| `performance` | {'en_US': 'Performance', 'tr_TR': 'Performans', 'ar_001': 'الأداء'} |
| `quality` | {'en_US': 'Quality', 'tr_TR': 'Kalite', 'ar_001': 'الجودة'} |
| `productive` | {'en_US': 'Productive', 'tr_TR': 'Üretken', 'ar_001': 'إنتاجي'} |

## `mrp.workorder`

**`cost_mode`** — {'en_US': 'Cost Mode', 'tr_TR': 'Maliyet Modu', 'ar_001': 'وضع التكلفة'}

| Value | Label |
|---|---|
| `actual` | {'en_US': 'Actual', 'tr_TR': 'Gerçek', 'ar_001': 'الفعلي'} |
| `estimated` | {'en_US': 'Estimated', 'tr_TR': 'Tahmini', 'ar_001': 'متوقع'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `ready` | {'en_US': 'To Do', 'tr_TR': 'Yapılacak', 'ar_001': 'المهام المراد تنفيذها'} |
| `progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Finished', 'tr_TR': 'Bitmiş', 'ar_001': 'مُنتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `myinvois.consolidate.invoice.wizard`

**`consolidation_type`** — {'en_US': 'Consolidation Type'}

| Value | Label |
|---|---|
| `invoice` | {'en_US': 'Invoice'} |
| `pos` | {'en_US': 'PoS Order'} |

## `myinvois.document`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`myinvois_state`** — {'en_US': 'MyInvois State'}

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'Validation In Progress'} |
| `valid` | {'en_US': 'Valid'} |
| `rejected` | {'en_US': 'Rejected'} |
| `invalid` | {'en_US': 'Invalid'} |
| `cancelled` | {'en_US': 'Cancelled'} |

## `nemhandel.registration`

**`edi_mode`** — {'en_US': 'EDI mode'} (not stored)

| Value | Label |
|---|---|
| `demo` | {'en_US': 'Demo'} |
| `test` | {'en_US': 'Test'} |
| `prod` | {'en_US': 'Live'} |

## `nemhandel.response`

**`nemhandel_state`** — {'en_US': 'Nemhandel status'}

| Value | Label |
|---|---|
| `processing` | {'en_US': 'Pending Reception'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |
| `not_serviced` | {'en_US': 'Not Serviced'} |

**`response_code`** — {'en_US': 'Response Code'}

| Value | Label |
|---|---|
| `BusinessAccept` | {'en_US': 'Approval'} |
| `BusinessReject` | {'en_US': 'Rejection'} |

## `onboarding.onboarding`

**`current_onboarding_state`** — {'en_US': 'Completion State', 'tr_TR': 'Tamamlanma Durumu', 'ar_001': 'حالة الإنجاز'} (not stored)

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `onboarding.onboarding.step`

**`current_step_state`** — {'en_US': 'Completion State', 'tr_TR': 'Tamamlanma Durumu', 'ar_001': 'حالة الإنجاز'} (not stored)

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `onboarding.progress`

**`onboarding_state`** — {'en_US': 'Onboarding progress', 'tr_TR': 'Katılım ilerlemesi', 'ar_001': 'تقدم التأهيل'}

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `onboarding.progress.step`

**`step_state`** — {'en_US': 'Onboarding Step Progress', 'tr_TR': 'Katılım Adım İlerlemesi', 'ar_001': 'تقدم خطوة التأهيل'}

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `payment.method`

**`support_manual_capture`** — {'en_US': 'Manual Capture', 'tr_TR': 'Manuel Yakalama', 'ar_001': 'التحصيل يدوياً'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'Unsupported', 'tr_TR': 'Desteklenmiyor', 'ar_001': 'غير مدعوم'} |
| `full_only` | {'en_US': 'Full Only', 'tr_TR': 'Yalnızca Tam', 'ar_001': 'كامل فقط'} |
| `partial` | {'en_US': 'Full & Partial', 'tr_TR': 'Tam ve Kısmi', 'ar_001': 'كامل وجزئي'} |

**`support_refund`** — {'en_US': 'Refund', 'tr_TR': 'İade/Fiyat Farkı', 'ar_001': 'استرداد الأموال'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'Unsupported', 'tr_TR': 'Desteklenmiyor', 'ar_001': 'غير مدعوم'} |
| `full_only` | {'en_US': 'Full Only', 'tr_TR': 'Yalnızca Tam', 'ar_001': 'كامل فقط'} |
| `partial` | {'en_US': 'Full & Partial', 'tr_TR': 'Tam ve Kısmi', 'ar_001': 'كامل وجزئي'} |

## `payment.provider`

**`asiapay_brand`** — {'en_US': 'Asiapay Brand', 'tr_TR': 'Asiapay Markası', 'ar_001': 'علامة AsiaPay التجارية'}

| Value | Label |
|---|---|
| `paydollar` | {'en_US': 'PayDollar', 'tr_TR': 'PayDollar', 'ar_001': 'PayDollar'} |
| `pesopay` | {'en_US': 'PesoPay', 'tr_TR': 'PesoPay', 'ar_001': 'PesoPay'} |
| `siampay` | {'en_US': 'SiamPay', 'tr_TR': 'SiamPay', 'ar_001': 'SiamPay'} |
| `bimopay` | {'en_US': 'BimoPay', 'tr_TR': 'BimoPay', 'ar_001': 'BimoPay'} |

**`asiapay_secure_hash_function`** — {'en_US': 'AsiaPay Secure Hash Function', 'tr_TR': 'AsiaPay Güvenli Hash Fonksiyonu', 'ar_001': 'خاصية التشفير الآمن في AsiaPay'}

| Value | Label |
|---|---|
| `sha1` | {'en_US': 'SHA1', 'tr_TR': 'SHA1', 'ar_001': 'SHA1'} |
| `sha256` | {'en_US': 'SHA256', 'tr_TR': 'SHA256', 'ar_001': 'SHA256'} |
| `sha512` | {'en_US': 'SHA512', 'tr_TR': 'SHA512', 'ar_001': 'SHA512'} |

**`code`** — {'en_US': 'Code', 'tr_TR': 'Kod', 'ar_001': 'رمز'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'No Provider Set', 'tr_TR': 'Sağlayıcı Kümesi Yok', 'ar_001': 'لم يتم تعيين مزود'} |
| `adyen` | {'en_US': 'Adyen', 'tr_TR': 'Adyen', 'ar_001': 'Adyen'} |
| `aps` | {'en_US': 'Amazon Payment Services', 'tr_TR': 'Amazon Ödeme Hizmetleri', 'ar_001': 'خدمات دفع Amazon'} |
| `asiapay` | {'en_US': 'AsiaPay', 'tr_TR': 'AsiaPay', 'ar_001': 'AsiaPay'} |
| `authorize` | {'en_US': 'Authorize.Net', 'tr_TR': 'Authorize.Net', 'ar_001': 'Authorize.Net'} |
| `buckaroo` | {'en_US': 'Buckaroo', 'tr_TR': 'Buckaroo', 'ar_001': 'Buckaroo'} |
| `custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |
| `demo` | {'en_US': 'Demo', 'tr_TR': 'Demo', 'ar_001': 'النسخة التجريبية'} |
| `dpo` | {'en_US': 'DPO', 'tr_TR': 'DPO', 'ar_001': 'DPO'} |
| `ecpay` | {'en_US': 'ECPay'} |
| `flutterwave` | {'en_US': 'Flutterwave', 'tr_TR': 'Flutterwave', 'ar_001': 'Flutterwave'} |
| `iyzico` | {'en_US': 'Iyzico', 'tr_TR': 'Iyzico', 'ar_001': 'Iyzico'} |
| `mercado_pago` | {'en_US': 'Mercado Pago', 'tr_TR': 'Mercado Pago', 'ar_001': 'Mercado Pago'} |
| `mollie` | {'en_US': 'Mollie', 'tr_TR': 'Mollie', 'ar_001': 'Mollie'} |
| `nuvei` | {'en_US': 'Nuvei', 'tr_TR': 'Nuvei', 'ar_001': 'Nuvei'} |
| `paymob` | {'en_US': 'Paymob', 'tr_TR': 'Paymob', 'ar_001': 'Paymob'} |
| `paypal` | {'en_US': 'PayPal', 'tr_TR': 'PayPal', 'ar_001': 'PayPal'} |
| `payu` | {'en_US': 'PayU'} |
| `razorpay` | {'en_US': 'Razorpay', 'tr_TR': 'Razorpay', 'ar_001': 'Razorpay'} |
| `redsys` | {'en_US': 'Redsys', 'tr_TR': 'Redsys', 'ar_001': 'Redsys'} |
| `stripe` | {'en_US': 'Stripe', 'tr_TR': 'Stripe', 'ar_001': 'Stripe'} |
| `toss_payments` | {'en_US': 'Toss Payments'} |
| `worldline` | {'en_US': 'Worldline', 'tr_TR': 'Worldline', 'ar_001': 'Worldline'} |
| `xendit` | {'en_US': 'Xendit', 'tr_TR': 'Xendit', 'ar_001': 'Xendit'} |

**`custom_mode`** — {'en_US': 'Custom Mode', 'tr_TR': 'Özel mod', 'ar_001': 'الوضع المخصص'}

| Value | Label |
|---|---|
| `wire_transfer` | {'en_US': 'Wire Transfer', 'tr_TR': 'Manuel Transfer', 'ar_001': 'تحويل بنكي'} |
| `cash_on_delivery` | {'en_US': 'Cash On Delivery', 'tr_TR': 'Kapıda Ödeme', 'ar_001': 'الدفع عند الاستلام'} |
| `on_site` | {'en_US': 'Pay on site', 'tr_TR': 'Yerinde ödeme', 'ar_001': 'الدفع في الموقع'} |

**`so_reference_type`** — {'en_US': 'Communication', 'tr_TR': 'İletişim', 'ar_001': 'التواصل'}

| Value | Label |
|---|---|
| `so_name` | {'en_US': 'Based on Document Reference', 'tr_TR': 'Doküman Referansına Göre', 'ar_001': 'بناءً على مرجع المستند'} |
| `partner` | {'en_US': 'Based on Customer ID', 'tr_TR': 'Müşteri ID Bilgisine Göre', 'ar_001': 'حسب مُعرف العميل'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}

| Value | Label |
|---|---|
| `disabled` | {'en_US': 'Disabled', 'tr_TR': 'Devre Dışı', 'ar_001': 'معطل'} |
| `enabled` | {'en_US': 'Enabled', 'tr_TR': 'Etkin', 'ar_001': 'ممكن'} |
| `test` | {'en_US': 'Test Mode', 'tr_TR': 'Test Modu', 'ar_001': 'وضع الاختبار'} |

**`support_manual_capture`** — {'en_US': 'Manual Capture Supported', 'tr_TR': 'Manuel Yakalama Desteklenir', 'ar_001': 'يدعم عملية التحصيل اليدوية'} (not stored)

| Value | Label |
|---|---|
| `full_only` | {'en_US': 'Full Only', 'tr_TR': 'Yalnızca Tam', 'ar_001': 'كامل فقط'} |
| `partial` | {'en_US': 'Partial', 'tr_TR': 'Kısmi', 'ar_001': 'جزئي'} |

**`support_refund`** — {'en_US': 'Refund', 'tr_TR': 'İade/Fiyat Farkı', 'ar_001': 'استرداد الأموال'} (not stored)

| Value | Label |
|---|---|
| `none` | {'en_US': 'Unsupported', 'tr_TR': 'Desteklenmiyor', 'ar_001': 'غير مدعوم'} |
| `full_only` | {'en_US': 'Full Only', 'tr_TR': 'Yalnızca Tam', 'ar_001': 'كامل فقط'} |
| `partial` | {'en_US': 'Full & Partial', 'tr_TR': 'Tam ve Kısmi', 'ar_001': 'كامل وجزئي'} |

## `payment.refund.wizard`

**`support_refund`** — {'en_US': 'Refund', 'tr_TR': 'İade/Fiyat Farkı', 'ar_001': 'استرداد الأموال'} (not stored)

| Value | Label |
|---|---|
| `none` | {'en_US': 'Unsupported', 'tr_TR': 'Desteklenmiyor', 'ar_001': 'غير مدعوم'} |
| `full_only` | {'en_US': 'Full Only', 'tr_TR': 'Yalnızca Tamamı', 'ar_001': 'كامل فقط'} |
| `partial` | {'en_US': 'Partial', 'tr_TR': 'Kısmi', 'ar_001': 'جزئي'} |

## `payment.token`

**`demo_simulated_state`** — {'en_US': 'Simulated State', 'tr_TR': 'Simüle Edilmiş Durum', 'ar_001': 'الحالة التي تم إنشاؤها بالمحاكاة'}

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `done` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `cancel` | {'en_US': 'Canceled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

## `payment.transaction`

**`operation`** — {'en_US': 'Operation', 'tr_TR': 'Operasyon', 'ar_001': 'العملية'}

| Value | Label |
|---|---|
| `online_redirect` | {'en_US': 'Online payment with redirection', 'tr_TR': 'Yeniden yönlendirme ile online ödeme', 'ar_001': 'الدفع عبر الإنترنت مع إعادة التوجيه'} |
| `online_direct` | {'en_US': 'Online direct payment', 'tr_TR': 'Online doğrudan ödeme', 'ar_001': 'الدفع المباشر عبر الإنترنت'} |
| `online_token` | {'en_US': 'Online payment by token', 'tr_TR': 'Token ile online ödeme', 'ar_001': 'الدفع عبر الإنترنت عن طريق الرمز'} |
| `validation` | {'en_US': 'Validation of the payment method', 'tr_TR': 'Ödeme yönteminin doğrulanması', 'ar_001': 'تصديق طريقة الدفع'} |
| `offline` | {'en_US': 'Offline payment by token', 'tr_TR': 'Token ile çevrimdışı ödeme', 'ar_001': 'الدفع دون الاتصال بالإنترنت عن طريق الرمز'} |
| `refund` | {'en_US': 'Refund', 'tr_TR': 'İade/Fiyat Farkı', 'ar_001': 'استرداد الأموال'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Beklemede', 'ar_001': 'قيد الانتظار'} |
| `authorized` | {'en_US': 'Authorized', 'tr_TR': 'Yetkili', 'ar_001': 'مصرح به'} |
| `done` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `cancel` | {'en_US': 'Canceled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

## `pdp.registration`

**`edi_mode`** — {'en_US': 'EDI mode'} (not stored)

| Value | Label |
|---|---|
| `demo` | {'en_US': 'Demo'} |
| `test` | {'en_US': 'Test'} |
| `prod` | {'en_US': 'Live'} |

## `pdp.response.wizard`

**`reason_code`** — {'en_US': 'Reason Code'}

| Value | Label |
|---|---|
| `TX_TVA_ERR` | {'en_US': 'Incorrect VAT rate'} |
| `MONTANTTOTAL_ERR` | {'en_US': 'Incorrect Total Amount'} |
| `CALCUL_ERR` | {'en_US': 'Billing calculation error'} |
| `NON_CONFORME` | {'en_US': 'Legal information missing'} |
| `DEST_ERR` | {'en_US': 'Wrong recipient'} |
| `TRANSAC_INC` | {'en_US': 'Unknown transaction'} |
| `EMMET_INC` | {'en_US': 'Unknown sender'} |
| `CONTRAT_TERM` | {'en_US': 'Contract completed'} |
| `DOUBLE_FACT` | {'en_US': 'Duplicate Invoice'} |
| `CMD_ERR` | {'en_US': 'Order number is incorrect or missing'} |
| `ADR_ERR` | {'en_US': 'Incorrect electronic billing address'} |
| `REF_CT_ABSENT` | {'en_US': 'Contract reference required to process the missing invoice'} |
| `JUSTIF_ABS` | {'en_US': 'Missing or Insufficient Supporting Documentation'} |
| `COORD_BANC_ERR` | {'en_US': 'Bank Account Information Error'} |
| `SIRET_ERR` | {'en_US': 'Incorrect or missing SIRET number'} |
| `CODE_ROUTAGE_ERR` | {'en_US': 'Missing or incorrect CODE_ROUTAGE'} |
| `REF_ERR` | {'en_US': 'Incorrect reference'} |

**`status`** — {'en_US': 'Status'}

| Value | Label |
|---|---|
| `PD` | {'en_US': 'Paid'} |
| `cancelled` | {'en_US': 'Cancelled'} |
| `suspended` | {'en_US': 'Suspended'} |
| `refused` | {'en_US': 'Refused'} |
| `AP` | {'en_US': 'Approved'} |
| `in_hand` | {'en_US': 'In Hand'} |
| `completed` | {'en_US': 'Completed'} |

## `peppol.registration`

**`edi_mode`** — {'en_US': 'EDI mode', 'tr_TR': 'EDI modu', 'ar_001': 'EDI mode'} (not stored)

| Value | Label |
|---|---|
| `demo` | {'en_US': 'Demo', 'tr_TR': 'Demo', 'ar_001': 'النسخة التجريبية'} |
| `test` | {'en_US': 'Test', 'tr_TR': 'Test', 'ar_001': 'اختبار'} |
| `prod` | {'en_US': 'Live', 'tr_TR': 'Canlı', 'ar_001': 'حي'} |

**`use_parent_connection_selection`** — {'en_US': 'Use Parent Connection Selection', 'tr_TR': 'Ebeveyn Bağlantısı Seçimi Kullanın', 'ar_001': 'Use Parent Connection Selection'}

| Value | Label |
|---|---|
| `use_parent` | {'en_US': 'Send from parent company', 'tr_TR': 'Ana şirketten gönder', 'ar_001': 'Send from parent company'} |
| `use_self` | {'en_US': 'Register this company on peppol', 'tr_TR': "Bu şirketi peppol'a kaydettirin", 'ar_001': 'Register this company on peppol'} |

## `picking.label.type`

**`label_type`** — {'en_US': 'Labels to print', 'tr_TR': 'Yazdırılacak etiketler', 'ar_001': 'بطاقات العناوين لطباعتها'}

| Value | Label |
|---|---|
| `products` | {'en_US': 'Product Labels', 'tr_TR': 'Ürün Etiketleri', 'ar_001': 'بطاقات عناوين المنتجات'} |
| `lots` | {'en_US': 'Lot/SN Labels', 'tr_TR': 'Lot/SN Etiketleri', 'ar_001': 'بطاقات عناوين أرقام المجموعات/الأرقام التسلسلية'} |

## `portal.wizard.user`

**`email_state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'} (not stored)

| Value | Label |
|---|---|
| `ok` | {'en_US': 'Valid', 'tr_TR': 'Geçerli', 'ar_001': 'صالح'} |
| `ko` | {'en_US': 'Invalid', 'tr_TR': 'Geçersiz', 'ar_001': 'غير صالح'} |
| `exist` | {'en_US': 'Already Registered', 'tr_TR': 'Zaten Kayıtlı', 'ar_001': 'مسجل بالفعل'} |

## `pos.config`

**`default_screen`** — {'en_US': 'Default Screen', 'tr_TR': 'Varsayılan Ekran', 'ar_001': 'الشاشة الافتراضية'}

| Value | Label |
|---|---|
| `tables` | {'en_US': 'Tables', 'tr_TR': 'Tablolar', 'ar_001': 'الجداول'} |
| `register` | {'en_US': 'Register', 'tr_TR': 'Kayıt Ol', 'ar_001': 'تسجيل'} |

**`iface_tax_included`** — {'en_US': 'Tax Display', 'tr_TR': 'Vergi Görünümü', 'ar_001': 'عرض الضريبة'}

| Value | Label |
|---|---|
| `subtotal` | {'en_US': 'Tax-Excluded Price', 'tr_TR': 'Vergi Hariç Fiyat', 'ar_001': 'السعر غير شامل للضريبة'} |
| `total` | {'en_US': 'Tax-Included Price', 'tr_TR': 'Vergi Dahil Fiyat', 'ar_001': 'السعر شامل للضريبة'} |

**`picking_policy`** — {'en_US': 'Shipping Policy', 'tr_TR': 'Ürün Teslimat Politikası', 'ar_001': 'سياسة الشحن'}

| Value | Label |
|---|---|
| `direct` | {'en_US': 'As soon as possible', 'tr_TR': 'Mümkün olduğunca kısa sürede', 'ar_001': 'في أقرب وقت ممكن'} |
| `one` | {'en_US': 'When all products are ready', 'tr_TR': 'Tüm ürünler hazır olduğunda', 'ar_001': 'عندما تكون كافة المنتجات جاهزة'} |

**`self_ordering_mode`** — {'en_US': 'Self Ordering Mode', 'tr_TR': 'Self Ordering Modu', 'ar_001': 'وضع الطلب الذاتي'}

| Value | Label |
|---|---|
| `nothing` | {'en_US': 'Disable', 'tr_TR': 'Devre dışı', 'ar_001': 'تعطيل'} |
| `consultation` | {'en_US': 'QR menu', 'tr_TR': 'QR menu', 'ar_001': 'قائمة QR'} |
| `mobile` | {'en_US': 'QR menu + Ordering', 'tr_TR': 'QR menü + Sipariş', 'ar_001': 'قائمة QR + الطلب'} |
| `kiosk` | {'en_US': 'Kiosk', 'tr_TR': 'Dijital sipariş ekranı', 'ar_001': 'كشك'} |

**`self_ordering_service_mode`** — {'en_US': 'Self Ordering Service Mode', 'tr_TR': 'Self Ordering Servis Modu', 'ar_001': 'وضع خدمة الطلب الذاتي'}

| Value | Label |
|---|---|
| `counter` | {'en_US': 'Pickup zone', 'tr_TR': 'Teslim Alma alanı', 'ar_001': 'منطقة الاستلام'} |
| `table` | {'en_US': 'Table', 'tr_TR': 'Masa', 'ar_001': 'جدول'} |

**`status`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'} (not stored)

| Value | Label |
|---|---|
| `inactive` | {'en_US': 'Inactive', 'tr_TR': 'Pasif', 'ar_001': 'غير نشط'} |
| `active` | {'en_US': 'Active', 'tr_TR': 'Etkin', 'ar_001': 'نشط'} |

## `pos.order`

**`invoice_status`** — {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'} (not stored)

| Value | Label |
|---|---|
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to_invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |

**`l10n_es_edi_verifactu_refund_reason`** — {'en_US': 'Veri*Factu Refund Reason'}

| Value | Label |
|---|---|
| `R1` | {'en_US': 'R1: Art 80.1 and 80.2 and error of law'} |
| `R2` | {'en_US': 'R2: Art. 80.3'} |
| `R3` | {'en_US': 'R3: Art. 80.4'} |
| `R4` | {'en_US': 'R4: Rest'} |
| `R5` | {'en_US': 'R5: Corrective invoices concerning simplified invoices'} |

**`l10n_es_edi_verifactu_state`** — {'en_US': 'Veri*Factu Status'}

| Value | Label |
|---|---|
| `rejected` | {'en_US': 'Rejected'} |
| `registered_with_errors` | {'en_US': 'Registered with Errors'} |
| `accepted` | {'en_US': 'Accepted'} |
| `cancelled` | {'en_US': 'Cancelled'} |

**`l10n_es_tbai_refund_reason`** — {'en_US': 'Invoice Refund Reason Code (TicketBai)'}

| Value | Label |
|---|---|
| `R1` | {'en_US': 'R1: Art. 80.1, 80.2, 80.6 and rights founded error'} |
| `R2` | {'en_US': 'R2: Art. 80.3'} |
| `R3` | {'en_US': 'R3: Art. 80.4'} |
| `R4` | {'en_US': 'R4: Art. 80 - other'} |
| `R5` | {'en_US': 'R5: Factura rectificativa en facturas simplificadas'} |

**`l10n_es_tbai_state`** — {'en_US': 'TicketBAI status'} (not stored)

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |

**`l10n_jo_edi_pos_state`** — {'en_US': 'JoFotara State'}

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |
| `demo` | {'en_US': 'Sent (Demo)'} |

**`l10n_sa_reason`** — {'en_US': 'ZATCA Reason', 'ar_001': 'سبب هيئة الزكاة والضريبة والجمارك'}

| Value | Label |
|---|---|
| `BR-KSA-17-reason-1` | {'en_US': 'Cancellation or suspension of the supplies after its occurrence either wholly or partially', 'ar_001': 'إلغاء أو تعليق التوريدات بعد حدوثها سواء كليًا أو جزئيًا'} |
| `BR-KSA-17-reason-2` | {'en_US': 'In case of essential change or amendment in the supply, which leads to the change of the VAT due', 'ar_001': 'في حالة حدوث تغيير أو تعديل جوهري في التوريد، مما يؤدي إلى تغيير ضريبة القيمة المضافة المستحقة'} |
| `BR-KSA-17-reason-3` | {'en_US': 'Amendment of the supply value which is pre-agreed upon between the supplier and consumer', 'ar_001': 'تعديل قيمة التوريد المتفق عليها مسبقًا بين المورد والمستهلك'} |
| `BR-KSA-17-reason-4` | {'en_US': 'In case of goods or services refund', 'ar_001': 'في حالة رد سلع أو خدمات'} |
| `BR-KSA-17-reason-5` | {'en_US': "In case of change in Seller's or Buyer's information", 'ar_001': 'في حالة تغيير بيانات البائع أو المشتري'} |

**`l10n_tw_edi_carrier_type`** — {'en_US': 'Carrier Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'ECpay e-invoice carrier'} |
| `2` | {'en_US': 'Citizen Digital Certificate'} |
| `3` | {'en_US': 'Mobile Barcode'} |
| `4` | {'en_US': 'EasyCard'} |
| `5` | {'en_US': 'iPass'} |

**`source`** — {'en_US': 'Origin', 'tr_TR': 'Kaynak', 'ar_001': 'الأصل'}

| Value | Label |
|---|---|
| `pos` | {'en_US': 'Point of Sale', 'tr_TR': 'Satış Noktası', 'ar_001': 'نقطة البيع'} |
| `mobile` | {'en_US': 'Self-Order Mobile', 'tr_TR': 'Self Servis Sipariş Mobil'} |
| `kiosk` | {'en_US': 'Self-Order Kiosk', 'tr_TR': 'Self Servis Sipariş Kiosku'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |

## `pos.order.line`

**`price_type`** — {'en_US': 'Price Type', 'tr_TR': 'Fiyat Türü', 'ar_001': 'نوع السعر'}

| Value | Label |
|---|---|
| `original` | {'en_US': 'Original', 'tr_TR': 'Orijinal', 'ar_001': 'أصلي'} |
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `automatic` | {'en_US': 'Automatic', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |

## `pos.payment.method`

**`dpopay_payment_mode`** — {'en_US': 'Dpopay Payment Mode'}

| Value | Label |
|---|---|
| `card` | {'en_US': 'Card', 'ar_001': 'بطاقة'} |
| `momo` | {'en_US': 'Mobile Money'} |

**`pine_labs_allowed_payment_mode`** — {'en_US': 'Pine Labs Allowed Payment Modes', 'tr_TR': 'Pine Labs İçin İzin Verilen Ödeme Yöntemleri', 'ar_001': 'طرق الدفع المسموح بها في Pine Labs'}

| Value | Label |
|---|---|
| `all` | {'en_US': 'All', 'tr_TR': 'Tümü', 'ar_001': 'الكل'} |
| `card` | {'en_US': 'Card', 'tr_TR': 'Kart', 'ar_001': 'البطاقة'} |
| `upi` | {'en_US': 'Upi', 'tr_TR': 'Upi', 'ar_001': 'Upi'} |

**`qfpay_payment_type`** — {'en_US': 'QFPay Payment Type', 'tr_TR': 'QFPay Ödeme Tipi'}

| Value | Label |
|---|---|
| `card_payment` | {'en_US': 'Visa/Mastercard', 'tr_TR': 'Visa/Mastercard', 'ar_001': 'Visa/Mastercard'} |
| `wx` | {'en_US': 'WeChat Pay', 'tr_TR': 'WeChat Pay'} |
| `alipay` | {'en_US': 'Alipay', 'tr_TR': 'Alipay', 'ar_001': 'Alipay'} |
| `payme` | {'en_US': 'PayMe', 'tr_TR': 'PayMe', 'ar_001': 'PayMe'} |
| `union` | {'en_US': 'UnionPay QuickPass', 'tr_TR': 'UnionPay QuickPass'} |
| `fps` | {'en_US': 'FPS', 'tr_TR': 'FPS', 'ar_001': 'FPS'} |
| `octopus` | {'en_US': 'Octopus', 'tr_TR': 'Octopus', 'ar_001': 'Octopus'} |
| `unionpay_card` | {'en_US': 'Unionpay Card', 'tr_TR': 'Unionpay Kart'} |
| `amex_card` | {'en_US': 'American Express Card', 'tr_TR': 'American Express Kart', 'ar_001': 'بطاقة أمريكان إكسبريس'} |

**`razorpay_allowed_payment_modes`** — {'en_US': 'Razorpay Allowed Payment Modes', 'tr_TR': 'Razorpay İzin Verilen Ödeme Yöntemleri', 'ar_001': 'Razorpay Allowed Payment Modes'}

| Value | Label |
|---|---|
| `all` | {'en_US': 'All', 'tr_TR': 'Tümü', 'ar_001': 'الكل'} |
| `card` | {'en_US': 'Card', 'tr_TR': 'Kart', 'ar_001': 'البطاقة'} |
| `upi` | {'en_US': 'UPI', 'tr_TR': 'UPI', 'ar_001': 'UPI'} |
| `bharatqr` | {'en_US': 'BHARATQR', 'tr_TR': 'BHARATQR', 'ar_001': 'BHARATQR'} |

**`safaricom_payment_type`** — {'en_US': 'Payment Type', 'ar_001': 'نوع الدفع '}

| Value | Label |
|---|---|
| `mpesa_express` | {'en_US': 'M-PESA Express', 'ar_001': 'M-PESA Express'} |
| `lipa_na_mpesa` | {'en_US': 'Lipa na M-PESA', 'ar_001': 'Lipa na M-PESA'} |

**`type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'} (not stored)

| Value | Label |
|---|---|
| `cash` | {'en_US': 'Cash', 'tr_TR': 'Kasa', 'ar_001': 'نقدي'} |
| `bank` | {'en_US': 'Bank', 'tr_TR': 'Banka', 'ar_001': 'البنك'} |
| `pay_later` | {'en_US': 'Customer Account', 'tr_TR': 'Müşteri Hesabı', 'ar_001': 'حساب العميل'} |
| `online` | {'en_US': 'Online', 'tr_TR': 'Çevrimiçi', 'ar_001': 'عبر الإنترنت'} |

## `pos.preset`

**`identification`** — {'en_US': 'Identification', 'tr_TR': 'Tanımlama', 'ar_001': 'الهوية'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'Not required', 'tr_TR': 'Gerekli değil', 'ar_001': 'غير مطلوب'} |
| `address` | {'en_US': 'Address', 'tr_TR': 'Adres', 'ar_001': 'العنوان'} |
| `name` | {'en_US': 'Name', 'tr_TR': 'Adı', 'ar_001': 'الاسم'} |

**`service_at`** — {'en_US': 'Service at', 'tr_TR': 'Servis yeri', 'ar_001': 'الخدمة عند الطاولة'}

| Value | Label |
|---|---|
| `counter` | {'en_US': 'Pickup zone', 'tr_TR': 'Teslim Alma alanı', 'ar_001': 'منطقة الاستلام'} |
| `table` | {'en_US': 'Table', 'tr_TR': 'Masa', 'ar_001': 'جدول'} |
| `delivery` | {'en_US': 'Delivery', 'tr_TR': 'Teslimat', 'ar_001': 'التوصيل'} |

## `pos.printer`

**`printer_type`** — {'en_US': 'Printer Type', 'tr_TR': 'Yazıcı Türü', 'ar_001': 'نوع الطابعة'}

| Value | Label |
|---|---|
| `iot` | {'en_US': 'Use a printer connected to the IoT Box', 'tr_TR': "IoT Box'a bağlı bir yazıcı kullanın", 'ar_001': 'استخدم طابعة متصلة بجهاز IoT'} |
| `epson_epos` | {'en_US': 'Use an Epson printer', 'tr_TR': 'Bir Epson yazıcı kullanın', 'ar_001': 'استخدام طابعة Epson'} |

## `pos.session`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `opening_control` | {'en_US': 'Opening Control', 'tr_TR': 'Açılış Kontrolü', 'ar_001': 'التحكم في الفتح'} |
| `opened` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `closing_control` | {'en_US': 'Closing Control', 'tr_TR': 'Kapanış Kotrolü', 'ar_001': 'التحكم في الإغلاق'} |
| `closed` | {'en_US': 'Closed & Posted', 'tr_TR': 'Kapalı & Onaylı', 'ar_001': 'تم الإغلاق والترحيل'} |

## `pos_self_order.custom_link`

**`style`** — {'en_US': 'Style', 'tr_TR': 'Stil', 'ar_001': 'الشكل'}

| Value | Label |
|---|---|
| `primary` | {'en_US': 'Primary', 'tr_TR': 'Birincil', 'ar_001': 'الرئيسي'} |
| `secondary` | {'en_US': 'Secondary', 'tr_TR': 'İkincil', 'ar_001': 'ثانوي'} |
| `success` | {'en_US': 'Success', 'tr_TR': 'Başarılı', 'ar_001': 'النجاح'} |
| `warning` | {'en_US': 'Warning', 'tr_TR': 'Uyarı', 'ar_001': 'تحذير'} |
| `danger` | {'en_US': 'Danger', 'tr_TR': 'Tehlike', 'ar_001': 'خطر'} |
| `info` | {'en_US': 'Info', 'tr_TR': 'Info', 'ar_001': 'معلومات'} |
| `light` | {'en_US': 'Light', 'tr_TR': 'Açık', 'ar_001': 'فاتح'} |
| `dark` | {'en_US': 'Dark', 'tr_TR': 'Karanlık', 'ar_001': 'غامق'} |

## `product.attribute`

**`create_variant`** — {'en_US': 'Variant Creation', 'tr_TR': 'Varyant Oluşturma', 'ar_001': 'إنشاء المتغيِّر'}

| Value | Label |
|---|---|
| `always` | {'en_US': 'Instantly', 'tr_TR': 'Hemen', 'ar_001': 'فوراً'} |
| `dynamic` | {'en_US': 'Dynamically', 'tr_TR': 'Dinamik', 'ar_001': 'ديناميكياً'} |
| `no_variant` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقاً'} |

**`display_type`** — {'en_US': 'Display Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع العرض'}

| Value | Label |
|---|---|
| `radio` | {'en_US': 'Radio', 'tr_TR': 'Radio', 'ar_001': 'راديو'} |
| `pills` | {'en_US': 'Pills', 'tr_TR': 'Haplar', 'ar_001': 'حبوب'} |
| `select` | {'en_US': 'Select', 'tr_TR': 'Seçme', 'ar_001': 'تحديد'} |
| `color` | {'en_US': 'Color', 'tr_TR': 'Renk', 'ar_001': 'اللون'} |
| `multi` | {'en_US': 'Multi-checkbox', 'tr_TR': 'Çoklu-onaykutusu', 'ar_001': 'صناديق اختيار متعددة'} |
| `image` | {'en_US': 'Image', 'tr_TR': 'Görsel', 'ar_001': 'صورة'} |

**`preview_variants`** — {'en_US': 'On Product Cards', 'tr_TR': 'Ürün Kartlarında'}

| Value | Label |
|---|---|
| `visible` | {'en_US': 'Visible', 'tr_TR': 'Görünür', 'ar_001': 'مرئي'} |
| `hidden` | {'en_US': 'Hidden', 'tr_TR': 'Gizli', 'ar_001': 'مخفي'} |
| `hover` | {'en_US': 'Hover', 'tr_TR': 'Hover', 'ar_001': 'Hover'} |

**`visibility`** — {'en_US': 'Visibility', 'tr_TR': 'Görünürlük', 'ar_001': 'الظهور'}

| Value | Label |
|---|---|
| `visible` | {'en_US': 'Visible', 'tr_TR': 'Görünür', 'ar_001': 'مرئي'} |
| `hidden` | {'en_US': 'Hidden', 'tr_TR': 'Gizli', 'ar_001': 'مخفي'} |

## `product.category`

**`packaging_reserve_method`** — {'en_US': 'Reserve Packagings', 'tr_TR': 'Ambalajları Rezerve Et', 'ar_001': 'حجز التعبئات'}

| Value | Label |
|---|---|
| `full` | {'en_US': 'Reserve Only Full Packagings', 'tr_TR': 'Sadece Tam Ambalajları Rezerve Et', 'ar_001': 'حجز التعبئات الكاملة فقط'} |
| `partial` | {'en_US': 'Reserve Partial Packagings', 'tr_TR': 'Yarım Ambalajları Rezerve Et', 'ar_001': 'حجز التعبئات الجزئية'} |

**`property_cost_method`** — {'en_US': 'Costing Method', 'tr_TR': 'Maliyet Yöntemi', 'ar_001': 'طريقة حساب التكاليف'}

| Value | Label |
|---|---|
| `standard` | {'en_US': 'Standard Price', 'tr_TR': 'Standart Maliyet', 'ar_001': 'السعر القياسي'} |
| `fifo` | {'en_US': 'First In First Out (FIFO)', 'tr_TR': 'İlk Giren İlk Çıkar (FIFO)', 'ar_001': 'الوارد أولاً يخرج أولاً (FIFO)'} |
| `average` | {'en_US': 'Average Cost (AVCO)', 'tr_TR': 'Ortalama Maliyet (AVCO)', 'ar_001': 'متوسط التكلفة (AVCO)'} |

**`property_valuation`** — {'en_US': 'Inventory Valuation', 'tr_TR': 'Envanter Değerleme', 'ar_001': 'تقييم المخزون'}

| Value | Label |
|---|---|
| `periodic` | {'en_US': 'Periodic (at closing)', 'tr_TR': 'Periyodik (kapanışta)'} |
| `real_time` | {'en_US': 'Perpetual (at invoicing)', 'tr_TR': 'Sürekli (faturalandırmada)'} |

## `product.document`

**`attached_on_mrp`** — {'en_US': 'MRP : Visible at', 'tr_TR': 'MRP : Şu adreste görülebilir', 'ar_001': 'تخطيط متطلبات المواد: مرئي في'}

| Value | Label |
|---|---|
| `hidden` | {'en_US': 'Hidden', 'tr_TR': 'Gizli', 'ar_001': 'مخفي'} |
| `bom` | {'en_US': 'Bill of Materials', 'tr_TR': 'Ürün Reçeteleri', 'ar_001': 'قائمة المواد'} |

**`attached_on_sale`** — {'en_US': 'Sale : Visible at', 'tr_TR': 'Satış: Görünürlük Konumu', 'ar_001': 'البيع: مرئي في'}

| Value | Label |
|---|---|
| `hidden` | {'en_US': 'Hidden', 'tr_TR': 'Gizli', 'ar_001': 'مخفي'} |
| `quotation` | {'en_US': 'On quote', 'tr_TR': 'Teklifte', 'ar_001': 'في عرض السعر'} |
| `sale_order` | {'en_US': 'On confirmed order', 'tr_TR': 'Onaylanmış siparişte', 'ar_001': 'في الطلب المؤكد'} |
| `inside` | {'en_US': 'Inside quote pdf', 'tr_TR': 'Teklif İçi PDF’i', 'ar_001': 'داخل عرض السعر PDF'} |

## `product.feed`

**`target`** — {'en_US': 'Target', 'tr_TR': 'Hedef', 'ar_001': 'الهدف'}

| Value | Label |
|---|---|
| `gmc` | {'en_US': 'Google Merchant Center', 'tr_TR': 'Google Merchant Center', 'ar_001': 'Google Merchant Center'} |

## `product.label.layout`

**`move_quantity`** — {'en_US': 'Quantity to print', 'tr_TR': 'Bastırılacak miktar', 'ar_001': 'الكمية لطباعتها'}

| Value | Label |
|---|---|
| `move` | {'en_US': 'Operation Quantities', 'tr_TR': 'Operasyon Miktarları', 'ar_001': 'كميات العملية'} |
| `custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |

**`print_format`** — {'en_US': 'Format', 'tr_TR': 'Biçim', 'ar_001': 'التنسيق'}

| Value | Label |
|---|---|
| `dymo` | {'en_US': 'Dymo', 'tr_TR': 'Dymo', 'ar_001': 'Dymo'} |
| `2x7xprice` | {'en_US': '2 x 7 with price', 'tr_TR': '2x7 fiyat ile', 'ar_001': '2 × 7 بالسعر'} |
| `4x7xprice` | {'en_US': '4 x 7 with price', 'tr_TR': '4 x 7 fiyat ile', 'ar_001': '4 × 7 بالسعر'} |
| `4x12` | {'en_US': '4 x 12', 'tr_TR': '4 x 12', 'ar_001': '4 × 12'} |
| `4x12xprice` | {'en_US': '4 x 12 with price', 'tr_TR': '4 x 12 fiyat ile', 'ar_001': '4 × 12 بالسعر'} |
| `zpl` | {'en_US': 'ZPL Labels', 'tr_TR': 'ZPL Etiketleri', 'ar_001': 'بطاقات عناوين ZPL'} |
| `zplxprice` | {'en_US': 'ZPL Labels with price', 'tr_TR': 'ZPL Etiketleri, fiyat ile', 'ar_001': 'بطاقات عناوين ZPL مع السعر'} |

**`zpl_template`** — {'en_US': 'ZPL Template', 'tr_TR': 'ZPL Şablonu', 'ar_001': 'قالب ZPL'}

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Normal (2.25" x 1.25")', 'tr_TR': 'Normal (2,25" x 1,25")', 'ar_001': 'عادي (2.25" x 1.25")'} |
| `small` | {'en_US': 'Small (1.25" x 1.00")', 'tr_TR': 'Küçük (1,25" x 1,00")', 'ar_001': 'صغير (1.25" x 1.00")'} |
| `alternative` | {'en_US': 'Alternative (2.00" x 1.00")', 'tr_TR': 'Alternatif (2.00" x 1.00")', 'ar_001': 'البديل (2.00" x 1.00")'} |
| `jewelry` | {'en_US': 'Jewelry (2.20" x 0.50")', 'tr_TR': 'Takı (2,20" x 0,50")', 'ar_001': 'مجوهرات (2.20" x 0.50")'} |

## `product.margin`

**`invoice_state`** — {'en_US': 'Invoice State', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}

| Value | Label |
|---|---|
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `open_paid` | {'en_US': 'Open and Paid', 'tr_TR': 'Açık ve Ödendi', 'ar_001': 'مفتوحة ومدفوعة'} |
| `draft_open_paid` | {'en_US': 'Draft, Open and Paid', 'tr_TR': 'Taslak, Açık ve Ödenmiş', 'ar_001': 'مسودة، مفتوحة ومدفوعة'} |

## `product.pricelist`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

## `product.pricelist.item`

**`applied_on`** — {'en_US': 'Apply On', 'tr_TR': 'Şuna Uygula', 'ar_001': 'التطبيق على'}

| Value | Label |
|---|---|
| `3_global` | {'en_US': 'All Products', 'tr_TR': 'Tüm Ürünler', 'ar_001': 'كافة المنتجات'} |
| `2_product_category` | {'en_US': 'Product Category', 'tr_TR': 'Ürün Kategorisi', 'ar_001': 'فئة المنتج'} |
| `1_product` | {'en_US': 'Product', 'tr_TR': 'Ürün', 'ar_001': 'المنتج'} |
| `0_product_variant` | {'en_US': 'Product Variant', 'tr_TR': 'Ürün Varyantı', 'ar_001': 'متغير المنتج'} |

**`base`** — {'en_US': 'Based on', 'tr_TR': 'Buna göre', 'ar_001': 'بناءً على'}

| Value | Label |
|---|---|
| `list_price` | {'en_US': 'Sales Price', 'tr_TR': 'Satış Fiyatı', 'ar_001': 'سعر البيع'} |
| `standard_price` | {'en_US': 'Cost', 'tr_TR': 'Maliyet', 'ar_001': 'التكلفة'} |
| `pricelist` | {'en_US': 'Other Pricelist', 'tr_TR': 'Diğer Fiyat Listesi', 'ar_001': 'قوائم الأسعار الأخرى'} |

**`compute_price`** — {'en_US': 'Compute Price', 'tr_TR': 'Fiyat Hesaplama', 'ar_001': 'حساب السعر'}

| Value | Label |
|---|---|
| `percentage` | {'en_US': 'Discount', 'tr_TR': 'İndirim', 'ar_001': 'الخصم'} |
| `formula` | {'en_US': 'Formula', 'tr_TR': 'Formül', 'ar_001': 'الصيغة'} |
| `fixed` | {'en_US': 'Fixed Price', 'tr_TR': 'Sabit Fiyat', 'ar_001': 'سعر ثابت'} |

**`display_applied_on`** — {'en_US': 'Display Applied On', 'tr_TR': 'Ekran Açık Uygulandı', 'ar_001': 'عرض ما تم التطبيق عليه'}

| Value | Label |
|---|---|
| `1_product` | {'en_US': 'Product', 'tr_TR': 'Ürün', 'ar_001': 'المنتج'} |
| `2_product_category` | {'en_US': 'Category', 'tr_TR': 'Kategori', 'ar_001': 'الفئة'} |

## `product.product`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`invoice_state`** — {'en_US': 'Invoice State', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'} (not stored)

| Value | Label |
|---|---|
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `open_paid` | {'en_US': 'Open and Paid', 'tr_TR': 'Açık ve Ödendi', 'ar_001': 'مفتوحة ومدفوعة'} |
| `draft_open_paid` | {'en_US': 'Draft, Open and Paid', 'tr_TR': 'Taslak, Açık ve Ödenmiş', 'ar_001': 'مسودة، مفتوحة ومدفوعة'} |

## `product.ribbon`

**`assign`** — {'en_US': 'Assign', 'tr_TR': 'Ata', 'ar_001': 'تعيين'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manually', 'tr_TR': 'Manuel olarak', 'ar_001': 'يدويًا'} |
| `sale` | {'en_US': 'On Sale', 'tr_TR': 'Satışta'} |
| `new` | {'en_US': 'When New', 'tr_TR': 'Ne Zaman Yeni'} |
| `out_of_stock` | {'en_US': 'when out of stock', 'tr_TR': 'stokta kalmadığında'} |

**`position`** — {'en_US': 'Position', 'tr_TR': 'Pozisyon', 'ar_001': 'الموضع'}

| Value | Label |
|---|---|
| `left` | {'en_US': 'Left', 'tr_TR': 'Sol', 'ar_001': 'يسار'} |
| `right` | {'en_US': 'Right', 'tr_TR': 'Sağ', 'ar_001': 'يمين'} |

**`style`** — {'en_US': 'Style', 'tr_TR': 'Stil', 'ar_001': 'الشكل'}

| Value | Label |
|---|---|
| `ribbon` | {'en_US': 'Ribbon', 'tr_TR': 'Şerit', 'ar_001': 'شريط'} |
| `tag` | {'en_US': 'Badge', 'tr_TR': 'Rozet', 'ar_001': 'الشارة'} |

## `product.template`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`cost_method`** — {'en_US': 'Cost Method', 'tr_TR': 'Maliyet Yöntemi', 'ar_001': 'طريقة حساب التكلفة'} (not stored)

| Value | Label |
|---|---|
| `standard` | {'en_US': 'Standard Price', 'tr_TR': 'Standart Maliyet', 'ar_001': 'السعر القياسي'} |
| `fifo` | {'en_US': 'First In First Out (FIFO)', 'tr_TR': 'İlk Giren İlk Çıkar (FIFO)', 'ar_001': 'الوارد أولاً يخرج أولاً (FIFO)'} |
| `average` | {'en_US': 'Average Cost (AVCO)', 'tr_TR': 'Ortalama Maliyet (AVCO)', 'ar_001': 'متوسط التكلفة (AVCO)'} |

**`expense_policy`** — {'en_US': 'Re-Invoice Costs', 'tr_TR': 'Maliyetleri Yeniden Faturalandırın', 'ar_001': 'إعادة فوترة التكاليف'}

| Value | Label |
|---|---|
| `no` | {'en_US': 'No', 'tr_TR': 'Hayır', 'ar_001': 'لا'} |
| `cost` | {'en_US': 'At cost', 'tr_TR': 'Maliyet', 'ar_001': 'بالتكلفة'} |
| `sales_price` | {'en_US': 'Sales price', 'tr_TR': 'Satış Fiyatı', 'ar_001': 'سعر البيع'} |

**`invoice_policy`** — {'en_US': 'Invoicing Policy', 'tr_TR': 'Faturalama Kuralı', 'ar_001': 'سياسة الفوترة'}

| Value | Label |
|---|---|
| `order` | {'en_US': 'Ordered quantities', 'tr_TR': 'Siparişin miktarından', 'ar_001': 'الكميات المطلوبة'} |
| `delivery` | {'en_US': 'Delivered quantities', 'tr_TR': 'Teslim edilen miktar', 'ar_001': 'الكميات التي تم توصيلها'} |

**`l10n_hu_product_code_type`** — {'en_US': 'Product Code Type'}

| Value | Label |
|---|---|
| `VTSZ` | {'en_US': 'VTSZ - Customs Code'} |
| `SZJ` | {'en_US': 'SZJ - Service Registry Code'} |
| `TESZOR` | {'en_US': 'TESZOR - CPA 2.1 Code'} |
| `KN` | {'en_US': 'KN - Combined Nomenclature Code'} |
| `AHK` | {'en_US': 'AHK - e-TKO Excise Duty Code'} |
| `KT` | {'en_US': 'KT - Environmental Product Code'} |
| `CSK` | {'en_US': 'CSK - Packaging Catalogue Code'} |
| `EJ` | {'en_US': 'EJ - Building Registry Number'} |
| `OTHER` | {'en_US': 'Other'} |

**`l10n_my_edi_classification_code`** — {'en_US': 'Malaysian classification code'}

| Value | Label |
|---|---|
| `001` | {'en_US': '(001) Breastfeeding equipment '} |
| `002` | {'en_US': '(002) Child care centres and kindergartens fees'} |
| `003` | {'en_US': '(003) Computer, smartphone or tablet'} |
| `004` | {'en_US': '(004) Consolidated e-Invoice '} |
| `005` | {'en_US': '(005) Construction materials (as specified under Fourth Schedule of the Lembaga Pembangunan Industri Pembinaan Malaysia Act 1994)'} |
| `006` | {'en_US': '(006) Disbursement'} |
| `007` | {'en_US': '(007) Donation'} |
| `008` | {'en_US': '(008) -Commerce - e-Invoice to buyer / purchaser'} |
| `009` | {'en_US': '(009) e-Commerce - Self-billed e-Invoice to seller, logistics, etc. '} |
| `010` | {'en_US': '(010) Education fees'} |
| `011` | {'en_US': '(011) Goods on consignment (Consignor)'} |
| `012` | {'en_US': '(012) Goods on consignment (Consignee)'} |
| `013` | {'en_US': '(013) Gym membership'} |
| `014` | {'en_US': '(014) Insurance - Education and medical benefits'} |
| `015` | {'en_US': '(015) Insurance - Takaful or life insurance'} |
| `016` | {'en_US': '(016) Interest and financing expenses'} |
| `017` | {'en_US': '(017) Internet subscription'} |
| `018` | {'en_US': '(018) Land and building'} |
| `019` | {'en_US': '(019) Medical examination for learning disabilities and early intervention or rehabilitation treatments of learning disabilities'} |
| `020` | {'en_US': '(020) Medical examination or vaccination expenses'} |
| `021` | {'en_US': '(021) Medical expenses for serious diseases'} |
| `022` | {'en_US': '(022) Others'} |
| `023` | {'en_US': '(023) Petroleum operations (as defined in Petroleum (Income Tax) Act 1967)'} |
| `024` | {'en_US': '(024) Private retirement scheme or deferred annuity scheme'} |
| `025` | {'en_US': '(025) Motor vehicle'} |
| `026` | {'en_US': '(026) Subscription of books / journals / magazines / newspapers / other similar publications'} |
| `027` | {'en_US': '(027) Reimbursement'} |
| `028` | {'en_US': '(028) Rental of motor vehicle'} |
| `029` | {'en_US': '(029) EV charging facilities (Installation, rental, sale / purchase or subscription fees) '} |
| `030` | {'en_US': '(030) Repair and maintenance'} |
| `031` | {'en_US': '(031) Research and development'} |
| `032` | {'en_US': '(032) Foreign income'} |
| `033` | {'en_US': '(033) Self-billed - Betting and gaming'} |
| `034` | {'en_US': '(034) Self-billed - Importation of goods'} |
| `035` | {'en_US': '(035) Self-billed - Importation of services'} |
| `036` | {'en_US': '(036) Self-billed - Others'} |
| `037` | {'en_US': '(037) Self-billed - Monetary payment to agents, dealers or distributors'} |
| `038` | {'en_US': '(038) Fees related to sports equipment, facility rentals, competition registration, and training imposed by registered sports organizations under the Sports Development Act 1997'} |
| `039` | {'en_US': '(039) Supporting equipment for disabled person'} |
| `040` | {'en_US': '(040) Voluntary contribution to approved provident fund '} |
| `041` | {'en_US': '(041) Dental examination or treatment'} |
| `042` | {'en_US': '(042) Fertility treatment'} |
| `043` | {'en_US': '(043) Treatment and home care nursing, daycare centres and residential care centers'} |
| `044` | {'en_US': '(044) Vouchers, gift cards, loyalty points, etc'} |
| `045` | {'en_US': '(045) Self-billed - Non-monetary payment to agents, dealers or distributors'} |

**`l10n_pl_vat_gtu`** — {'en_US': 'GTU Codes'}

| Value | Label |
|---|---|
| `GTU_01` | {'en_US': 'GTU_01 - Alcoholic beverages'} |
| `GTU_02` | {'en_US': 'GTU_02 - Goods referred to under Art. 103 sec 5aa'} |
| `GTU_03` | {'en_US': 'GTU_03 - Fuel oil for excise duty, lubricating oils and other oils'} |
| `GTU_04` | {'en_US': 'GTU_04 - Tobacco products, tobacco, e-liquid'} |
| `GTU_05` | {'en_US': 'GTU_05 - Wastes'} |
| `GTU_06` | {'en_US': 'GTU_06 - Electronic devices, their parts and materials'} |
| `GTU_07` | {'en_US': 'GTU_07 - Vehicles and vehicle parts'} |
| `GTU_08` | {'en_US': 'GTU_08 - Precious metals and base metals'} |
| `GTU_09` | {'en_US': 'GTU_09 - Medicament and medical devices, medicinal products'} |
| `GTU_10` | {'en_US': 'GTU_10 - Buildings, structures and land'} |
| `GTU_11` | {'en_US': 'GTU_11 - Services related to the greenhouse gas emission allowance trading'} |
| `GTU_12` | {'en_US': 'GTU_12 - Intangible services'} |
| `GTU_13` | {'en_US': 'GTU_13 - Transport services and warehouse management services'} |

**`product_add_mode`** — {'en_US': 'Add product mode', 'tr_TR': 'Ürün modu ekle', 'ar_001': 'إضافة وضع المنتج'}

| Value | Label |
|---|---|
| `configurator` | {'en_US': 'Product Configurator', 'tr_TR': 'Ürün Yapılandırma', 'ar_001': 'مهيئ المنتج'} |
| `matrix` | {'en_US': 'Order Grid Entry', 'tr_TR': 'Sipariş Tablo Girişi', 'ar_001': 'قيد شبكة الطلب'} |

**`purchase_method`** — {'en_US': 'Control Policy', 'tr_TR': 'Kontrol Kuralı', 'ar_001': 'سياسة التحكم'}

| Value | Label |
|---|---|
| `purchase` | {'en_US': 'On ordered quantities', 'tr_TR': 'Sipariş edilen miktarlarda', 'ar_001': 'حسب الكميات المطلوبة'} |
| `receive` | {'en_US': 'On received quantities', 'tr_TR': 'Teslim alınmış miktardan', 'ar_001': 'حسب الكميات المستلمة'} |

**`rating_avg_text`** — {'en_US': 'Rating Avg Text', 'tr_TR': 'Değerlendirme Ort Metni', 'ar_001': 'متوسط نص التقييم'} (not stored)

| Value | Label |
|---|---|
| `top` | {'en_US': 'Happy'} |
| `ok` | {'en_US': 'Neutral'} |
| `ko` | {'en_US': 'Unhappy'} |
| `none` | {'en_US': 'Not Rated yet'} |

**`service_tracking`** — {'en_US': 'Create on Order', 'tr_TR': 'Sipariş Üzerine Oluştur', 'ar_001': 'الإنشاء عند الطلب'}

| Value | Label |
|---|---|
| `no` | {'en_US': 'Nothing', 'tr_TR': 'Hiçbir şey', 'ar_001': 'لا شيء'} |
| `event` | {'en_US': 'Event Registration', 'tr_TR': 'Etkinlik Kaydı', 'ar_001': 'التسجيل للفعالية'} |
| `partnership` | {'en_US': 'Membership / Partnership', 'tr_TR': 'Üyelik / Ortaklık'} |
| `repair` | {'en_US': 'Repair Order', 'tr_TR': 'Onarım Siparişi', 'ar_001': 'أمر الإصلاح'} |
| `event_booth` | {'en_US': 'Event Booth', 'tr_TR': 'Etkinlik Standı', 'ar_001': 'جناح الفعالية'} |
| `task_global_project` | {'en_US': 'Task', 'tr_TR': 'Görev', 'ar_001': 'المهمة'} |
| `task_in_project` | {'en_US': 'Project & Task', 'tr_TR': 'Proje & Görev', 'ar_001': 'المشروع والمهمة'} |
| `project_only` | {'en_US': 'Project', 'tr_TR': 'Proje', 'ar_001': 'المشروع'} |
| `course` | {'en_US': 'Course Access', 'tr_TR': 'Kurs Erişimi', 'ar_001': 'الوصول إلى الدورة'} |

**`service_type`** — {'en_US': 'Track Service', 'tr_TR': 'Servis Rotası', 'ar_001': 'تتبع الخدمة'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manually set quantities on order', 'tr_TR': 'Siparişte miktarı elle belirle', 'ar_001': 'تحديد الكميات في الطلب يدوياً'} |
| `milestones` | {'en_US': 'Project Milestones', 'tr_TR': 'Proje Kilometre Taşları', 'ar_001': 'مؤشرات تقدم المشروع'} |
| `timesheet` | {'en_US': 'Timesheets on project (one fare per SO/Project)', 'tr_TR': 'Projedeki çalışma çizelgesi (Satış Siparişi/Proje başına bir ücret)', 'ar_001': 'الجداول الزمنية للمشروع (أجرة واحدة لكل أمر بيع/مشروع)'} |

**`split_method_landed_cost`** — {'en_US': 'Default Split Method', 'tr_TR': 'Varsayılan Bölme Yöntemi', 'ar_001': 'طريقة التقسيم الافتراضية'}

| Value | Label |
|---|---|
| `equal` | {'en_US': 'Equal', 'tr_TR': 'Eşit', 'ar_001': 'بالتساوي'} |
| `by_quantity` | {'en_US': 'By Quantity', 'tr_TR': 'Miktara Göre', 'ar_001': 'حسب الكمية'} |
| `by_current_cost_price` | {'en_US': 'By Current Cost', 'tr_TR': 'Mevcut Maliyete Göre', 'ar_001': 'حسب التكلفة الحالية'} |
| `by_weight` | {'en_US': 'By Weight', 'tr_TR': 'Ağırlığa Göre', 'ar_001': 'حسب الوزن'} |
| `by_volume` | {'en_US': 'By Volume', 'tr_TR': 'Hacime Göre', 'ar_001': 'حسب الحجم'} |

**`tracking`** — {'en_US': 'Tracking', 'tr_TR': 'İzleme', 'ar_001': 'التتبع'}

| Value | Label |
|---|---|
| `serial` | {'en_US': 'By Unique Serial Number', 'tr_TR': "Seri No'lara Göre", 'ar_001': 'حسب الرقم التسلسلي الفريد'} |
| `lot` | {'en_US': 'By Lots', 'tr_TR': 'Lotlara Göre', 'ar_001': 'حسب أرقام الدفعات'} |
| `none` | {'en_US': 'By Quantity', 'tr_TR': 'Miktara Göre', 'ar_001': 'حسب الكمية'} |

**`type`** — {'en_US': 'Product Type', 'tr_TR': 'Ürün Türü', 'ar_001': 'نوع المنتج'}

| Value | Label |
|---|---|
| `consu` | {'en_US': 'Goods', 'tr_TR': 'Mallar', 'ar_001': 'البضائع'} |
| `service` | {'en_US': 'Service', 'tr_TR': 'Hizmet', 'ar_001': 'الخدمة'} |
| `combo` | {'en_US': 'Combo', 'tr_TR': 'Kombo', 'ar_001': 'مجموعة'} |

**`valuation`** — {'en_US': 'Valuation', 'tr_TR': 'Değerleme', 'ar_001': 'التقييم'} (not stored)

| Value | Label |
|---|---|
| `periodic` | {'en_US': 'Periodic (at closing)', 'tr_TR': 'Periyodik (kapanışta)'} |
| `real_time` | {'en_US': 'Perpetual (at invoicing)', 'tr_TR': 'Sürekli (faturalandırmada)'} |

## `project.project`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`billing_type`** — {'en_US': 'Billing Type', 'tr_TR': 'Faturalama Türü', 'ar_001': 'نوع الفوترة'}

| Value | Label |
|---|---|
| `not_billable` | {'en_US': 'not billable', 'tr_TR': 'faturalandırılamaz', 'ar_001': 'غير قابل للفوترة'} |
| `manually` | {'en_US': 'billed manually', 'tr_TR': 'manuel olarak faturalandırılır', 'ar_001': 'تتم الفوترة يدوياً'} |

**`last_update_status`** — {'en_US': 'Last Update Status'}

| Value | Label |
|---|---|
| `on_track` | {'en_US': 'On Track', 'tr_TR': 'İzleme Açık', 'ar_001': 'في المسار'} |
| `at_risk` | {'en_US': 'At Risk', 'tr_TR': 'Riskli', 'ar_001': 'في خطر'} |
| `off_track` | {'en_US': 'Off Track', 'tr_TR': 'İzleme Kapalı', 'ar_001': 'خارج المسار'} |
| `on_hold` | {'en_US': 'On Hold', 'tr_TR': 'Beklemede', 'ar_001': 'قيد الانتظار'} |
| `to_define` | {'en_US': 'Set Status', 'tr_TR': 'Durumu Ayarla', 'ar_001': 'قم بتحديد الحالة'} |
| `done` | {'en_US': 'Complete', 'tr_TR': 'Tamamlandı', 'ar_001': 'مكتمل'} |

**`pricing_type`** — {'en_US': 'Pricing', 'tr_TR': 'Fiyatlandırma', 'ar_001': 'الأسعار'} (not stored)

| Value | Label |
|---|---|
| `task_rate` | {'en_US': 'Task rate', 'tr_TR': 'Görev oranı', 'ar_001': 'سعر المهمة'} |
| `fixed_rate` | {'en_US': 'Project rate', 'tr_TR': 'proje oranı', 'ar_001': 'معدل المشروع'} |
| `employee_rate` | {'en_US': 'Employee rate', 'tr_TR': 'Çalışan oranı', 'ar_001': 'معدل الموظف'} |

**`privacy_visibility`** — {'en_US': 'Visibility', 'tr_TR': 'Görünürlük', 'ar_001': 'الظهور'}

| Value | Label |
|---|---|
| `followers` | {'en_US': 'Invited internal users', 'tr_TR': 'Davet edilen iç kullanıcılar'} |
| `invited_users` | {'en_US': 'Invited internal and portal users', 'tr_TR': 'Davet edilen iç ve portal kullanıcıları'} |
| `employees` | {'en_US': 'All internal users', 'tr_TR': 'Tüm dahili kullanıcılar', 'ar_001': 'جميع المستخدمين الداخليين'} |
| `portal` | {'en_US': 'All internal users and invited portal users'} |

## `project.share.collaborator.wizard`

**`access_mode`** — {'en_US': 'Access Mode', 'tr_TR': 'Erişim Modu', 'ar_001': 'وضع الوصول'}

| Value | Label |
|---|---|
| `read` | {'en_US': 'Read', 'tr_TR': 'Oku', 'ar_001': 'اقرأ'} |
| `edit_limited` | {'en_US': 'Edit with limited access', 'tr_TR': 'Sınırlı erişimle düzenle', 'ar_001': 'التحرير مع الوصول المحدود'} |
| `edit` | {'en_US': 'Edit', 'tr_TR': 'Düzenle', 'ar_001': 'تحرير'} |

## `project.task`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Low priority', 'tr_TR': 'Düşük öncelik', 'ar_001': 'أولوية منخفضة'} |
| `1` | {'en_US': 'Medium priority', 'tr_TR': 'Orta öncelik', 'ar_001': 'أولوية متوسطة'} |
| `2` | {'en_US': 'High priority', 'tr_TR': 'Yüksek öncelik', 'ar_001': 'أولوية عالية'} |
| `3` | {'en_US': 'Urgent', 'tr_TR': 'Acil', 'ar_001': 'عاجل'} |

**`rating_avg_text`** — {'en_US': 'Rating Avg Text', 'tr_TR': 'Değerlendirme Ort Metni', 'ar_001': 'متوسط نص التقييم'} (not stored)

| Value | Label |
|---|---|
| `top` | {'en_US': 'Happy', 'tr_TR': 'Mutlu', 'ar_001': 'سعيد'} |
| `ok` | {'en_US': 'Neutral', 'tr_TR': 'Nötr', 'ar_001': 'محايد'} |
| `ko` | {'en_US': 'Unhappy', 'tr_TR': 'Mutsuz', 'ar_001': 'غير سعيد'} |
| `none` | {'en_US': 'Not Rated yet', 'tr_TR': 'Henüz Değerlendirilmedi', 'ar_001': 'لم يتم التقييم بعد'} |

**`repeat_type`** — {'en_US': 'Until', 'tr_TR': 'Bitiş', 'ar_001': 'حتى'} (not stored)

| Value | Label |
|---|---|
| `forever` | {'en_US': 'Forever', 'tr_TR': 'Sonsuza dek', 'ar_001': 'للأبد'} |
| `until` | {'en_US': 'Until', 'tr_TR': 'Bitiş', 'ar_001': 'حتى'} |

**`repeat_unit`** — {'en_US': 'Repeat Unit', 'tr_TR': 'Tekrar Birimi', 'ar_001': 'تكرار الوحدة'} (not stored)

| Value | Label |
|---|---|
| `day` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `week` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `month` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |
| `year` | {'en_US': 'Years', 'tr_TR': 'Yıllar', 'ar_001': 'سنوات'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `01_in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `02_changes_requested` | {'en_US': 'Changes Requested', 'tr_TR': 'İstenen Değişiklikler', 'ar_001': 'التغييرات المطلوبة'} |
| `03_approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `1_done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `1_canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `04_waiting_normal` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |

## `project.task.burndown.chart.report`

**`is_closed`** — {'en_US': 'Closing Stage', 'tr_TR': 'Kapanış Aşaması', 'ar_001': 'مرحلة الإغلاق'}

| Value | Label |
|---|---|
| `closed` | {'en_US': 'Closed tasks', 'tr_TR': 'Kapatılmış görevler', 'ar_001': 'المهام المغلقة'} |
| `open` | {'en_US': 'Open tasks', 'tr_TR': 'Açık Görevler', 'ar_001': 'المهام المفتوحة'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `01_in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `1_done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `04_waiting_normal` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `03_approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `1_canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `02_changes_requested` | {'en_US': 'Changes Requested', 'tr_TR': 'İstenen Değişiklikler', 'ar_001': 'التغييرات المطلوبة'} |

## `project.task.recurrence`

**`repeat_type`** — {'en_US': 'Until', 'tr_TR': 'Bitiş', 'ar_001': 'حتى'}

| Value | Label |
|---|---|
| `forever` | {'en_US': 'Forever', 'tr_TR': 'Sonsuza dek', 'ar_001': 'للأبد'} |
| `until` | {'en_US': 'Until', 'tr_TR': 'Bitiş', 'ar_001': 'حتى'} |

**`repeat_unit`** — {'en_US': 'Repeat Unit'}

| Value | Label |
|---|---|
| `day` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `week` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |
| `month` | {'en_US': 'Months', 'tr_TR': 'Aylar', 'ar_001': 'شهور'} |
| `year` | {'en_US': 'Years', 'tr_TR': 'Yıllar', 'ar_001': 'سنوات'} |

## `project.task.type`

**`rating_status`** — {'en_US': 'Customer Ratings Status', 'tr_TR': 'Müşteri Puanları Durumu', 'ar_001': 'حالة تقييمات العملاء'}

| Value | Label |
|---|---|
| `stage` | {'en_US': 'when reaching this stage', 'tr_TR': 'bu aşamaya ulaşıldığında'} |
| `periodic` | {'en_US': 'on a periodic basis', 'tr_TR': 'belirli aralıklarla', 'ar_001': 'على نحو دوري'} |

**`rating_status_period`** — {'en_US': 'Rating Frequency', 'tr_TR': 'Değerlendirme Frekansı', 'ar_001': 'تواتر التقييم'}

| Value | Label |
|---|---|
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `weekly` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `bimonthly` | {'en_US': 'Twice a Month', 'tr_TR': 'Ayda 2 Kez', 'ar_001': 'مرتان شهريًا'} |
| `monthly` | {'en_US': 'Once a Month', 'tr_TR': 'Ayda Bir Kez', 'ar_001': 'مرة في الشهر'} |
| `quarterly` | {'en_US': 'Quarterly', 'tr_TR': 'Üç Aylık', 'ar_001': 'ربع سنوي'} |
| `yearly` | {'en_US': 'Yearly', 'tr_TR': 'Yıllık', 'ar_001': 'سنويًا'} |

## `project.update`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`status`** — {'en_US': 'Status'}

| Value | Label |
|---|---|
| `on_track` | {'en_US': 'On Track', 'tr_TR': 'İzleme Açık', 'ar_001': 'في المسار'} |
| `at_risk` | {'en_US': 'At Risk', 'tr_TR': 'Riskli', 'ar_001': 'في خطر'} |
| `off_track` | {'en_US': 'Off Track', 'tr_TR': 'İzleme Kapalı', 'ar_001': 'خارج المسار'} |
| `on_hold` | {'en_US': 'On Hold', 'tr_TR': 'Beklemede', 'ar_001': 'قيد الانتظار'} |
| `done` | {'en_US': 'Complete', 'tr_TR': 'Tamamlandı', 'ar_001': 'مكتمل'} |

## `purchase.order`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

**`invoice_status`** — {'en_US': 'Billing Status', 'tr_TR': 'Faturalama Durumu', 'ar_001': 'حالة الفاتورة'}

| Value | Label |
|---|---|
| `no` | {'en_US': 'Nothing to Bill', 'tr_TR': 'Faturalanacak Bir Şey Yok', 'ar_001': 'لا يوجد شي لفوترته'} |
| `to invoice` | {'en_US': 'Waiting Bills', 'tr_TR': 'Bekleyen Faturalar', 'ar_001': 'الفواتير قيد الانتظار'} |
| `invoiced` | {'en_US': 'Fully Billed', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `1` | {'en_US': 'Urgent', 'tr_TR': 'Acil', 'ar_001': 'عاجل'} |

**`receipt_status`** — {'en_US': 'Receipt Status', 'tr_TR': 'Alım Durumu', 'ar_001': 'حالة الإيصال'}

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Not Received', 'tr_TR': 'Alınmadı', 'ar_001': 'لم يتم الاستلام'} |
| `partial` | {'en_US': 'Partially Received', 'tr_TR': 'Kısmen Alındı', 'ar_001': 'تم الاستلام جزئياً'} |
| `full` | {'en_US': 'Fully Received', 'tr_TR': 'Tamamen Alındı', 'ar_001': 'تم الاستلام كلياً'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'RFQ', 'tr_TR': 'Teklif Talebi', 'ar_001': 'طلب عرض سعر'} |
| `sent` | {'en_US': 'RFQ Sent', 'tr_TR': 'Teklif Talebi Gönderildi', 'ar_001': 'تم إرسال طلب عرض السعر'} |
| `to approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `purchase` | {'en_US': 'Purchase Order', 'tr_TR': 'Satınalma Siparişi', 'ar_001': 'أمر شراء'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `purchase.order.line`

**`display_type`** — {'en_US': 'Display Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع العرض'}

| Value | Label |
|---|---|
| `line_section` | {'en_US': 'Section', 'tr_TR': 'Bölüm', 'ar_001': 'القسم'} |
| `line_subsection` | {'en_US': 'Subsection', 'tr_TR': 'Alt bölüm', 'ar_001': 'القسم الفرعي'} |
| `line_note` | {'en_US': 'Note', 'tr_TR': 'Not', 'ar_001': 'الملاحظات'} |

**`qty_received_method`** — {'en_US': 'Received Qty Method', 'tr_TR': 'Alınan Miktar Yöntemi', 'ar_001': 'طريقة الكمية المستلمة'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `stock_moves` | {'en_US': 'Stock Moves', 'tr_TR': 'Stok Hareketleri', 'ar_001': 'حركات المخزون'} |

## `purchase.report`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft RFQ', 'tr_TR': 'Taslak Teklif Talebi', 'ar_001': 'مسودة طلب عرض سعر'} |
| `sent` | {'en_US': 'RFQ Sent', 'tr_TR': 'Teklif Talebi Gönderildi', 'ar_001': 'تم إرسال طلب عرض السعر'} |
| `to approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `purchase` | {'en_US': 'Purchase Order', 'tr_TR': 'Satınalma Siparişi', 'ar_001': 'أمر شراء'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `purchase.requisition`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`requisition_type`** — {'en_US': 'Agreement Type', 'tr_TR': 'Sözleşme Türü', 'ar_001': 'نوع الاتفاقية'}

| Value | Label |
|---|---|
| `blanket_order` | {'en_US': 'Blanket Order', 'tr_TR': 'Kapsamlı Sipariş', 'ar_001': 'طلب شامل'} |
| `purchase_template` | {'en_US': 'Purchase Template', 'tr_TR': 'Satın Alma Şablonu', 'ar_001': 'قالب الشراء'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `confirmed` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `done` | {'en_US': 'Closed', 'tr_TR': 'Kapanmış', 'ar_001': 'مغلق'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `quotation.document`

**`document_type`** — {'en_US': 'Document Type', 'tr_TR': 'Belge Tipi', 'ar_001': 'نوع المستند'}

| Value | Label |
|---|---|
| `header` | {'en_US': 'Header', 'tr_TR': 'Üstbilgi', 'ar_001': 'الترويسة'} |
| `footer` | {'en_US': 'Footer', 'tr_TR': 'Alt Bilgi', 'ar_001': 'التذييل'} |

## `rating.mixin`

**`rating_avg_text`** — {'en_US': 'Rating Avg Text', 'tr_TR': 'Değerlendirme Ort Metni', 'ar_001': 'متوسط نص التقييم'} (not stored)

| Value | Label |
|---|---|
| `top` | {'en_US': 'Happy', 'tr_TR': 'Mutlu', 'ar_001': 'سعيد'} |
| `ok` | {'en_US': 'Neutral', 'tr_TR': 'Nötr', 'ar_001': 'محايد'} |
| `ko` | {'en_US': 'Unhappy', 'tr_TR': 'Mutsuz', 'ar_001': 'غير سعيد'} |
| `none` | {'en_US': 'Not Rated yet', 'tr_TR': 'Henüz Değerlendirilmedi', 'ar_001': 'لم يتم التقييم بعد'} |

## `rating.rating`

**`rating_text`** — {'en_US': 'Rating', 'tr_TR': 'Değerlendirme', 'ar_001': 'التقييم'}

| Value | Label |
|---|---|
| `top` | {'en_US': 'Happy', 'tr_TR': 'Mutlu', 'ar_001': 'سعيد'} |
| `ok` | {'en_US': 'Neutral', 'tr_TR': 'Nötr', 'ar_001': 'محايد'} |
| `ko` | {'en_US': 'Unhappy', 'tr_TR': 'Mutsuz', 'ar_001': 'غير سعيد'} |
| `none` | {'en_US': 'Not Rated yet', 'tr_TR': 'Henüz Değerlendirilmedi', 'ar_001': 'لم يتم التقييم بعد'} |

## `repair.order`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`parts_availability_state`** — {'en_US': 'Parts Availability State', 'tr_TR': 'Parça Uygunluk Durumu', 'ar_001': 'حالة توافر الأجزاء'} (not stored)

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `expected` | {'en_US': 'Expected', 'tr_TR': 'Beklenen', 'ar_001': 'المتوقع'} |
| `late` | {'en_US': 'Late', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `1` | {'en_US': 'Urgent', 'tr_TR': 'Acil', 'ar_001': 'عاجل'} |

**`search_date_category`** — {'en_US': 'Date Category', 'tr_TR': 'Tarih Kategorisi', 'ar_001': 'فئة التاريخ'} (not stored)

| Value | Label |
|---|---|
| `before` | {'en_US': 'Before', 'tr_TR': 'Önce', 'ar_001': 'قبل'} |
| `yesterday` | {'en_US': 'Yesterday', 'tr_TR': 'Dün', 'ar_001': 'البارحة'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `day_1` | {'en_US': 'Tomorrow', 'tr_TR': 'Yarın', 'ar_001': 'غدًا'} |
| `day_2` | {'en_US': 'The day after tomorrow', 'tr_TR': 'Yarından sonraki gün', 'ar_001': 'بعد غد'} |
| `after` | {'en_US': 'After', 'tr_TR': 'Sonra', 'ar_001': 'بعد'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `confirmed` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `under_repair` | {'en_US': 'Under Repair', 'tr_TR': 'Onarımda', 'ar_001': 'قيد الإصلاح'} |
| `done` | {'en_US': 'Repaired', 'tr_TR': 'Onarıldı', 'ar_001': 'تم الإصلاح'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `report.paperformat`

**`format`** — {'en_US': 'Paper size', 'tr_TR': 'Sayfa Boyutu', 'ar_001': 'مقاس الورقة'}

| Value | Label |
|---|---|
| `A0` | {'en_US': 'A0  5   841 x 1189 mm', 'tr_TR': 'A0  5   841 x 1189 mm', 'ar_001': 'A0  5   841 x 1189 mm'} |
| `A1` | {'en_US': 'A1  6   594 x 841 mm', 'tr_TR': 'A1  6   594 x 841 mm', 'ar_001': 'A1  6   594 x 841 mm'} |
| `A2` | {'en_US': 'A2  7   420 x 594 mm', 'tr_TR': 'A2  7   420 x 594 mm', 'ar_001': 'A2  7   420 x 594 mm'} |
| `A3` | {'en_US': 'A3  8   297 x 420 mm', 'tr_TR': 'A3  8   297 x 420 mm', 'ar_001': 'A3  8   297 x 420 mm'} |
| `A4` | {'en_US': 'A4  0   210 x 297 mm, 8.26 x 11.69 inches', 'tr_TR': 'A4 0 210 x 297 mm, 8.26 x 11.69 inç', 'ar_001': 'A4  0   210 x 297 mm, 8.26 x 11.69 بوصة'} |
| `A5` | {'en_US': 'A5  9   148 x 210 mm', 'tr_TR': 'A5  9   148 x 210 mm', 'ar_001': 'A5  9   148 x 210 mm'} |
| `A6` | {'en_US': 'A6  10  105 x 148 mm', 'tr_TR': 'A6  10  105 x 148 mm', 'ar_001': 'A6  10  105 x 148 mm'} |
| `A7` | {'en_US': 'A7  11  74 x 105 mm', 'tr_TR': 'A7  11  74 x 105 mm', 'ar_001': 'A7  11  74 x 105 mm'} |
| `A8` | {'en_US': 'A8  12  52 x 74 mm', 'tr_TR': 'A8  12  52 x 74 mm', 'ar_001': 'A8  12  52 x 74 mm'} |
| `A9` | {'en_US': 'A9  13  37 x 52 mm', 'tr_TR': 'A9  13  37 x 52 mm', 'ar_001': 'A9  13  37 x 52 mm'} |
| `B0` | {'en_US': 'B0  14  1000 x 1414 mm', 'tr_TR': 'B0  14  1000 x 1414 mm', 'ar_001': 'B0  14  1000 x 1414 مم'} |
| `B1` | {'en_US': 'B1  15  707 x 1000 mm', 'tr_TR': 'B1  15  707 x 1000 mm', 'ar_001': 'B1  15  707 x 1000 مم'} |
| `B2` | {'en_US': 'B2  17  500 x 707 mm', 'tr_TR': 'B2 17 500 x 707 mm', 'ar_001': 'B2  17  500 x 707 مم'} |
| `B3` | {'en_US': 'B3  18  353 x 500 mm', 'tr_TR': 'B3  18  353 x 500 mm', 'ar_001': 'B3  18  353 x 500 مم'} |
| `B4` | {'en_US': 'B4  19  250 x 353 mm', 'tr_TR': 'B4  19  250 x 353 mm', 'ar_001': 'B4  19  250 x 353 مم'} |
| `B5` | {'en_US': 'B5  1   176 x 250 mm, 6.93 x 9.84 inches', 'tr_TR': 'B5  1   176 x 250 mm, 6.93 x 9.84 inç', 'ar_001': 'B5  1   176 x 250 مم، 6.93 x 9.84 إنش'} |
| `B6` | {'en_US': 'B6  20  125 x 176 mm', 'tr_TR': 'B6  20  125 x 176 mm', 'ar_001': 'B6  20  125 x 176 مم'} |
| `B7` | {'en_US': 'B7  21  88 x 125 mm', 'tr_TR': 'B7  21  88 x 125 mm', 'ar_001': 'B7  21  88 x 125 مم'} |
| `B8` | {'en_US': 'B8  22  62 x 88 mm', 'tr_TR': 'B8  22  62 x 88 mm', 'ar_001': 'B8  22  62 x 88 مم'} |
| `B9` | {'en_US': 'B9  23  33 x 62 mm', 'tr_TR': 'B9  23  33 x 62 mm', 'ar_001': 'B9  23  33 x 62 مم'} |
| `B10` | {'en_US': 'B10    16  31 x 44 mm', 'tr_TR': 'B10    16  31 x 44 mm', 'ar_001': 'B10    16  31 x 44 mm'} |
| `C5E` | {'en_US': 'C5E 24  163 x 229 mm', 'tr_TR': 'C5E 24  163 x 229 mm', 'ar_001': 'C5E 24  163 x 229 مم'} |
| `Comm10E` | {'en_US': 'Comm10E 25  105 x 241 mm, U.S. Common 10 Envelope', 'tr_TR': 'Comm10E 25  105 x 241 mm, U.S. Common 10 Envelope', 'ar_001': 'Comm10E 25  105 x 241 mm, U.S. Common 10 Envelope'} |
| `DLE` | {'en_US': 'DLE 26 110 x 220 mm', 'tr_TR': 'DLE 26 110 x 220 mm', 'ar_001': 'DLE 26 110 x 220 مم'} |
| `Executive` | {'en_US': 'Executive 4   7.5 x 10 inches, 190.5 x 254 mm', 'tr_TR': 'Executive 4 7.5 x 10 inç, 190.5 x 254 mm', 'ar_001': 'Executive 4   7.5 x 10 إنش، 190.5 x 254 مم'} |
| `Folio` | {'en_US': 'Folio 27  210 x 330 mm', 'tr_TR': 'Folio 27  210 x 330 mm', 'ar_001': 'Folio 27  210 x 330 mm'} |
| `Ledger` | {'en_US': 'Ledger  28  431.8 x 279.4 mm', 'tr_TR': 'Ledger 28 431.8 x 279.4 mm', 'ar_001': 'دفتر  28  431.8 * 279.4 مم'} |
| `Legal` | {'en_US': 'Legal    3   8.5 x 14 inches, 215.9 x 355.6 mm', 'tr_TR': 'Legal 3 8.5 x 14 inç, 215.9 x 355.6 mm', 'ar_001': 'القانوني    3   8.5 * 14 إنش، 215.9 x 355.6 مم'} |
| `Letter` | {'en_US': 'Letter 2 8.5 x 11 inches, 215.9 x 279.4 mm', 'tr_TR': 'Letter 2 8.5 x 11 inç, 215.9 x 279.4 mm', 'ar_001': 'رسالة 2 8.5 * 11 إنش، 215.9 x 279.4 مم'} |
| `Tabloid` | {'en_US': 'Tabloid 29 279.4 x 431.8 mm', 'tr_TR': 'Tabloid 29 279.4 x 431.8 mm', 'ar_001': 'Tabloid 29 279.4 x 431.8 mm'} |
| `custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |

**`orientation`** — {'en_US': 'Orientation', 'tr_TR': 'Sayfa Yönü', 'ar_001': 'الاتجاه'}

| Value | Label |
|---|---|
| `Landscape` | {'en_US': 'Landscape', 'tr_TR': 'Yatay', 'ar_001': 'بالعرض'} |
| `Portrait` | {'en_US': 'Portrait', 'tr_TR': 'Dikey', 'ar_001': 'بالطول'} |

## `report.pos.order`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `report.project.task.user`

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Low priority', 'tr_TR': 'Düşük öncelik', 'ar_001': 'أولوية منخفضة'} |
| `1` | {'en_US': 'Medium priority', 'tr_TR': 'Orta öncelik', 'ar_001': 'أولوية متوسطة'} |
| `2` | {'en_US': 'High priority', 'tr_TR': 'Yüksek öncelik', 'ar_001': 'أولوية عالية'} |
| `3` | {'en_US': 'Urgent', 'tr_TR': 'Acil', 'ar_001': 'عاجل'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `01_in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `1_done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `04_waiting_normal` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `03_approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `1_canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `02_changes_requested` | {'en_US': 'Changes Requested', 'tr_TR': 'İstenen Değişiklikler', 'ar_001': 'التغييرات المطلوبة'} |

## `report.stock.quantity`

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `forecast` | {'en_US': 'Forecasted Stock', 'tr_TR': 'Öngörülen Stok', 'ar_001': 'المخزون المتوقع'} |
| `in` | {'en_US': 'Forecasted Receipts', 'tr_TR': 'Öngörülen Alım', 'ar_001': 'الإيصالات المتوقعة'} |
| `out` | {'en_US': 'Forecasted Deliveries', 'tr_TR': 'Öngörülen Teslimatlar', 'ar_001': 'التوصيلات المتوقعة'} |

## `res.company`

**`account_check_printing_layout`** — {'en_US': 'Check Layout', 'tr_TR': 'Çek çıktı düzeni', 'ar_001': 'مخطط الشيك'}

| Value | Label |
|---|---|
| `disabled` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |

**`account_peppol_proxy_state`** — {'en_US': 'PEPPOL status', 'tr_TR': 'PEPPOL status', 'ar_001': 'PEPPOL status'}

| Value | Label |
|---|---|
| `not_registered` | {'en_US': 'Not registered', 'tr_TR': 'Kayıtlı değil', 'ar_001': 'غير مسجل'} |
| `sender` | {'en_US': 'Can send but not receive', 'tr_TR': 'Gönderebilir ama alamaz', 'ar_001': 'Can send but not receive'} |
| `smp_registration` | {'en_US': 'Can send, pending registration to receive', 'tr_TR': 'Gönderebilir, almak için kayıt bekleniyor', 'ar_001': 'Can send, pending registration to receive'} |
| `receiver` | {'en_US': 'Can send and receive', 'tr_TR': 'Gönderip alabilir', 'ar_001': 'Can send and receive'} |
| `rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

**`account_price_include`** — {'en_US': 'Default Sales Price Include', 'tr_TR': 'Varsayılan Satış Fiyatı Dahil', 'ar_001': 'يشمل سعر البيع الافتراضي'}

| Value | Label |
|---|---|
| `tax_included` | {'en_US': 'Tax Included', 'tr_TR': 'Vergi Dahil', 'ar_001': 'شامل الضريبة'} |
| `tax_excluded` | {'en_US': 'Tax Excluded', 'tr_TR': 'Vergi Hariç', 'ar_001': 'غير شامل الضريبة'} |

**`annual_inventory_month`** — {'en_US': 'Annual Inventory Month', 'tr_TR': 'Yıllık Envanter Ayı', 'ar_001': 'شهر المخزون السنوي'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'January', 'tr_TR': 'Ocak', 'ar_001': 'يناير'} |
| `2` | {'en_US': 'February', 'tr_TR': 'Şubat', 'ar_001': 'فبراير'} |
| `3` | {'en_US': 'March', 'tr_TR': 'Mart', 'ar_001': 'مارس'} |
| `4` | {'en_US': 'April', 'tr_TR': 'Nisan', 'ar_001': 'أبريل'} |
| `5` | {'en_US': 'May', 'tr_TR': 'Mayıs', 'ar_001': 'مايو'} |
| `6` | {'en_US': 'June', 'tr_TR': 'Haziran', 'ar_001': 'يونيو'} |
| `7` | {'en_US': 'July', 'tr_TR': 'Temmuz', 'ar_001': 'يوليو'} |
| `8` | {'en_US': 'August', 'tr_TR': 'Ağustos', 'ar_001': 'أغسطس'} |
| `9` | {'en_US': 'September', 'tr_TR': 'Eylül', 'ar_001': 'سبتمبر'} |
| `10` | {'en_US': 'October', 'tr_TR': 'Ekim', 'ar_001': 'أكتوبر'} |
| `11` | {'en_US': 'November', 'tr_TR': 'Kasım', 'ar_001': 'نوفمبر'} |
| `12` | {'en_US': 'December', 'tr_TR': 'Aralık', 'ar_001': 'ديسمبر'} |

**`attendance_barcode_source`** — {'en_US': 'Barcode Source', 'tr_TR': 'Barkod Kaynağı', 'ar_001': 'مصدر الباركود'}

| Value | Label |
|---|---|
| `scanner` | {'en_US': 'Scanner', 'tr_TR': 'Tarayıcı', 'ar_001': 'الماسح الضوئي'} |
| `front` | {'en_US': 'Front Camera', 'tr_TR': 'Ön Kamera', 'ar_001': 'الكاميرا الأمامية'} |
| `back` | {'en_US': 'Back Camera', 'tr_TR': 'Arka Kamera', 'ar_001': 'الكاميرا الخلفية'} |

**`attendance_kiosk_mode`** — {'en_US': 'Attendance Mode', 'tr_TR': 'Katılım Modu', 'ar_001': 'وضع الحضور'}

| Value | Label |
|---|---|
| `barcode` | {'en_US': 'Barcode / RFID', 'tr_TR': 'Barkod / RFID', 'ar_001': 'الباركود / رقاقات RFID'} |
| `barcode_manual` | {'en_US': 'Barcode / RFID and Manual Selection', 'tr_TR': 'Barkod/RFID ve Manuel Seçim', 'ar_001': 'الباركود / رقاقات RFID والتحديد التلقائي'} |
| `manual` | {'en_US': 'Manual Selection', 'tr_TR': 'Manuel Seçim', 'ar_001': 'اختيار يدوي'} |

**`attendance_overtime_validation`** — {'en_US': 'Extra Hours Validation', 'tr_TR': 'Ekstra Saat Doğrulaması', 'ar_001': 'تصديق الساعات الإضافية'}

| Value | Label |
|---|---|
| `no_validation` | {'en_US': 'Automatically Approved', 'tr_TR': 'Otomatik Olarak Onaylandı', 'ar_001': 'تمت الموافقة تلقائياً'} |
| `by_manager` | {'en_US': 'Approved by Manager', 'tr_TR': 'Yönetici tarafından Onaylandı', 'ar_001': 'تمت الموافقة من قِبَل المدير'} |

**`cost_method`** — {'en_US': 'Cost Method', 'tr_TR': 'Maliyet Yöntemi', 'ar_001': 'طريقة حساب التكلفة'}

| Value | Label |
|---|---|
| `standard` | {'en_US': 'Standard Price', 'tr_TR': 'Standart Maliyet', 'ar_001': 'السعر القياسي'} |
| `fifo` | {'en_US': 'First In First Out (FIFO)', 'tr_TR': 'İlk Giren İlk Çıkar (FIFO)', 'ar_001': 'الوارد أولاً يخرج أولاً (FIFO)'} |
| `average` | {'en_US': 'Average Cost (AVCO)', 'tr_TR': 'Ortalama Maliyet (AVCO)', 'ar_001': 'متوسط التكلفة (AVCO)'} |

**`fiscalyear_last_month`** — {'en_US': 'Fiscalyear Last Month', 'tr_TR': 'Mali Yıl Son Ay', 'ar_001': 'آخر شهور السنة المالية'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'January', 'tr_TR': 'Ocak', 'ar_001': 'يناير'} |
| `2` | {'en_US': 'February', 'tr_TR': 'Şubat', 'ar_001': 'فبراير'} |
| `3` | {'en_US': 'March', 'tr_TR': 'Mart', 'ar_001': 'مارس'} |
| `4` | {'en_US': 'April', 'tr_TR': 'Nisan', 'ar_001': 'أبريل'} |
| `5` | {'en_US': 'May', 'tr_TR': 'Mayıs', 'ar_001': 'مايو'} |
| `6` | {'en_US': 'June', 'tr_TR': 'Haziran', 'ar_001': 'يونيو'} |
| `7` | {'en_US': 'July', 'tr_TR': 'Temmuz', 'ar_001': 'يوليو'} |
| `8` | {'en_US': 'August', 'tr_TR': 'Ağustos', 'ar_001': 'أغسطس'} |
| `9` | {'en_US': 'September', 'tr_TR': 'Eylül', 'ar_001': 'سبتمبر'} |
| `10` | {'en_US': 'October', 'tr_TR': 'Ekim', 'ar_001': 'أكتوبر'} |
| `11` | {'en_US': 'November', 'tr_TR': 'Kasım', 'ar_001': 'نوفمبر'} |
| `12` | {'en_US': 'December', 'tr_TR': 'Aralık', 'ar_001': 'ديسمبر'} |

**`font`** — {'en_US': 'Font', 'tr_TR': 'Font', 'ar_001': 'الخط'}

| Value | Label |
|---|---|
| `Lato` | {'en_US': 'Lato', 'tr_TR': 'Lato', 'ar_001': 'Lato'} |
| `Roboto` | {'en_US': 'Roboto', 'tr_TR': 'Roboto', 'ar_001': 'Roboto'} |
| `Open_Sans` | {'en_US': 'Open Sans', 'tr_TR': 'Açık Sans', 'ar_001': 'Open Sans'} |
| `Montserrat` | {'en_US': 'Montserrat', 'tr_TR': 'Montserrat', 'ar_001': 'مونتسيرات'} |
| `Oswald` | {'en_US': 'Oswald', 'tr_TR': 'Oswald', 'ar_001': 'Oswald'} |
| `Raleway` | {'en_US': 'Raleway', 'tr_TR': 'Raleway', 'ar_001': 'Raleway'} |
| `Tajawal` | {'en_US': 'Tajawal', 'tr_TR': 'Tajawal', 'ar_001': 'Tajawal'} |
| `Fira_Mono` | {'en_US': 'Fira Mono', 'tr_TR': 'Fira Mono', 'ar_001': 'Fira Mono'} |

**`inventory_period`** — {'en_US': 'Inventory Period', 'tr_TR': 'Envanter Dönemi'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `daily` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `monthly` | {'en_US': 'Monthly', 'tr_TR': 'Aylık', 'ar_001': 'شهرياً'} |

**`inventory_valuation`** — {'en_US': 'Valuation', 'tr_TR': 'Değerleme', 'ar_001': 'التقييم'}

| Value | Label |
|---|---|
| `periodic` | {'en_US': 'Periodic (at closing)', 'tr_TR': 'Periyodik (kapanışta)'} |
| `real_time` | {'en_US': 'Perpetual (at invoicing)', 'tr_TR': 'Sürekli (faturalandırmada)'} |

**`l10n_dk_nemhandel_proxy_state`** — {'en_US': 'Nemhandel status'}

| Value | Label |
|---|---|
| `not_registered` | {'en_US': 'Not registered'} |
| `in_verification` | {'en_US': 'In verification'} |
| `receiver` | {'en_US': 'Can send and receive'} |
| `rejected` | {'en_US': 'Rejected'} |

**`l10n_es_edi_verifactu_special_vat_regime`** — {'en_US': 'Veri*Factu VAT Regime'}

| Value | Label |
|---|---|
| `simplified` | {'en_US': 'Simplified Regime'} |
| `reagyp` | {'en_US': 'REAGYP (Special Regime for Agriculture, Livestock and Fisheries)'} |
| `recargo` | {'en_US': 'Recargo de Equivalencia'} |

**`l10n_es_sii_tax_agency`** — {'en_US': 'Tax Agency for SII'}

| Value | Label |
|---|---|
| `aeat` | {'en_US': 'Agencia Tributaria española'} |
| `gipuzkoa` | {'en_US': 'Hacienda Foral de Gipuzkoa'} |
| `bizkaia` | {'en_US': 'Hacienda Foral de Bizkaia'} |
| `navarra` | {'en_US': 'Hacienda Foral de Navarra'} |

**`l10n_es_tbai_tax_agency`** — {'en_US': 'Tax Agency for TBAI'}

| Value | Label |
|---|---|
| `araba` | {'en_US': 'Hacienda Foral de Araba'} |
| `bizkaia` | {'en_US': 'Hacienda Foral de Bizkaia'} |
| `gipuzkoa` | {'en_US': 'Hacienda Foral de Gipuzkoa'} |

**`l10n_fr_pdp_periodicity`** — {'en_US': 'Flow 10 Report Periodicity'}

| Value | Label |
|---|---|
| `normal_monthly` | {'en_US': 'Real Monthly Normal Regime'} |
| `normal_quarterly` | {'en_US': 'Real Normal Quarterly Regime'} |
| `simplified_monthly` | {'en_US': 'Simplified VAT Regime (Monthly)'} |
| `simplified_bimonthly` | {'en_US': 'Franchised VAT Regime (Bimonthly)'} |

**`l10n_hr_mer_connection_mode`** — {'en_US': 'MojEracun Operating mode'}

| Value | Label |
|---|---|
| `prod` | {'en_US': 'Production'} |
| `test` | {'en_US': 'Test'} |
| `demo` | {'en_US': 'Demo'} |

**`l10n_hr_mer_connection_state`** — {'en_US': 'MojEracun connection status'}

| Value | Label |
|---|---|
| `inactive` | {'en_US': 'Inactive'} |
| `active` | {'en_US': 'Active'} |

**`l10n_hu_edi_server_mode`** — {'en_US': 'Server Mode'}

| Value | Label |
|---|---|
| `production` | {'en_US': 'Production'} |
| `test` | {'en_US': 'Test'} |
| `demo` | {'en_US': 'Demo'} |

**`l10n_hu_tax_regime`** — {'en_US': 'NAV Tax Regime'}

| Value | Label |
|---|---|
| `ie` | {'en_US': 'Individual Exemption'} |
| `ca` | {'en_US': 'Cash Accounting'} |
| `sb` | {'en_US': 'Small Business'} |

**`l10n_in_hsn_code_digit`** — {'en_US': 'HSN Code Digit'}

| Value | Label |
|---|---|
| `4` | {'en_US': '4 Digits (turnover < 5 CR.)'} |
| `6` | {'en_US': '6 Digits (turnover > 5 CR.)'} |
| `8` | {'en_US': '8 Digits'} |

**`l10n_it_eco_index_liquidation_state`** — {'en_US': 'Liquidation state'}

| Value | Label |
|---|---|
| `LS` | {'en_US': 'The company is in a state of liquidation'} |
| `LN` | {'en_US': 'The company is not in a state of liquidation'} |

**`l10n_it_eco_index_sole_shareholder`** — {'en_US': 'Shareholder'}

| Value | Label |
|---|---|
| `NO` | {'en_US': 'Not a limited liability company'} |
| `SU` | {'en_US': 'Socio unico'} |
| `SM` | {'en_US': 'Più soci'} |

**`l10n_it_tax_system`** — {'en_US': 'Tax System'}

| Value | Label |
|---|---|
| `RF01` | {'en_US': '[RF01] Ordinario'} |
| `RF02` | {'en_US': '[RF02] Contribuenti minimi (art.1, c.96-117, L. 244/07)'} |
| `RF04` | {'en_US': '[RF04] Agricoltura e attività connesse e pesca (artt.34 e 34-bis, DPR 633/72)'} |
| `RF05` | {'en_US': '[RF05] Vendita sali e tabacchi (art.74, c.1, DPR. 633/72)'} |
| `RF06` | {'en_US': '[RF06] Commercio fiammiferi (art.74, c.1, DPR  633/72)'} |
| `RF07` | {'en_US': '[RF07] Editoria (art.74, c.1, DPR  633/72)'} |
| `RF08` | {'en_US': '[RF08] Gestione servizi telefonia pubblica (art.74, c.1, DPR 633/72)'} |
| `RF09` | {'en_US': '[RF09] Rivendita documenti di trasporto pubblico e di sosta (art.74, c.1, DPR  633/72)'} |
| `RF10` | {'en_US': '[RF10] Intrattenimenti, giochi e altre attività di cui alla tariffa allegata al DPR 640/72 (art.74, c.6, DPR 633/72)'} |
| `RF11` | {'en_US': '[RF11] Agenzie viaggi e turismo (art.74-ter, DPR 633/72)'} |
| `RF12` | {'en_US': '[RF12] Agriturismo (art.5, c.2, L. 413/91)'} |
| `RF13` | {'en_US': '[RF13] Vendite a domicilio (art.25-bis, c.6, DPR  600/73)'} |
| `RF14` | {'en_US': '[RF14] Rivendita beni usati, oggetti d’arte, d’antiquariato o da collezione (art.36, DL 41/95)'} |
| `RF15` | {'en_US': '[RF15] Agenzie di vendite all’asta di oggetti d’arte, antiquariato o da collezione (art.40-bis, DL 41/95)'} |
| `RF16` | {'en_US': '[RF16] IVA per cassa P.A. (art.6, c.5, DPR 633/72)'} |
| `RF17` | {'en_US': '[RF17] IVA per cassa (art. 32-bis, DL 83/2012)'} |
| `RF18` | {'en_US': '[RF18] Altro'} |
| `RF19` | {'en_US': '[RF19] Regime forfettario (art.1, c.54-89, L. 190/2014)'} |

**`l10n_jo_edi_taxpayer_type`** — {'en_US': 'JoFotara Taxpayer Type'}

| Value | Label |
|---|---|
| `income` | {'en_US': 'Unregistered in the sales tax'} |
| `sales` | {'en_US': 'Registered in the sales tax'} |
| `special` | {'en_US': 'Registered in the special sales tax'} |

**`l10n_my_edi_mode`** — {'en_US': 'L10N My Edi Mode'}

| Value | Label |
|---|---|
| `test` | {'en_US': 'Pre-Production'} |
| `prod` | {'en_US': 'Production'} |

**`l10n_sa_api_mode`** — {'en_US': 'L10N Sa Api Mode', 'ar_001': 'L10N Sa Api Mode'}

| Value | Label |
|---|---|
| `sandbox` | {'en_US': 'Sandbox', 'ar_001': 'Sandbox'} |
| `preprod` | {'en_US': 'Simulation (Pre-Production)', 'ar_001': 'المحاكاة (قبل الإنتاج)'} |
| `prod` | {'en_US': 'Production', 'ar_001': 'الإنتاج'} |

**`layout_background`** — {'en_US': 'Layout Background', 'tr_TR': 'Serilim Arka Planı', 'ar_001': 'خلفية المخطط'}

| Value | Label |
|---|---|
| `Blank` | {'en_US': 'Blank', 'tr_TR': 'Boş', 'ar_001': 'فارغ'} |
| `Demo logo` | {'en_US': 'Demo logo', 'tr_TR': 'Demo logosu', 'ar_001': 'الشعار التجريبي'} |
| `Custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |

**`pdp_kyc_status`** — {'en_US': 'Pdp Kyc Status'}

| Value | Label |
|---|---|
| `processing` | {'en_US': 'Processing'} |
| `success` | {'en_US': 'Success'} |
| `fail` | {'en_US': 'Fail'} |

**`po_double_validation`** — {'en_US': 'Levels of Approvals', 'tr_TR': 'Onay Seviyeleri', 'ar_001': 'مستويات الموافقات'}

| Value | Label |
|---|---|
| `one_step` | {'en_US': 'Confirm purchase orders in one step', 'tr_TR': 'Satınalma siparişlerini bir adımda onayla', 'ar_001': 'تأكيد أوامر الشراء في خطوة واحدة'} |
| `two_step` | {'en_US': 'Get 2 levels of approvals to confirm a purchase order', 'tr_TR': 'Bir satın alma siparişinin onaylanması için 2 onay seviyesi almak gerekir.', 'ar_001': 'احصل على مستويين من الموافقة لتأكيد أمر شراء'} |

**`po_lock`** — {'en_US': 'Purchase Order Modification', 'tr_TR': 'Satınalma Siparişi Revizyonu', 'ar_001': 'تعديل أمر الشراء'}

| Value | Label |
|---|---|
| `edit` | {'en_US': 'Allow to edit purchase orders', 'tr_TR': 'Satınalma siparişlerini düzenlemeye izin ver', 'ar_001': 'السماح بتحرير أوامر الشراء'} |
| `lock` | {'en_US': 'Confirmed purchase orders are not editable', 'tr_TR': 'Onaylanan satınalma siparişleri düzenlenemez', 'ar_001': 'أوامر الشراء المؤكّدة غير قابلة للتحرير'} |

**`point_of_sale_ticket_portal_url_display_mode`** — {'en_US': 'Print', 'tr_TR': 'Yazdır', 'ar_001': 'طباعة'}

| Value | Label |
|---|---|
| `qr_code` | {'en_US': 'QR code', 'tr_TR': 'QR kodu', 'ar_001': 'QR Code'} |
| `url` | {'en_US': 'URL', 'tr_TR': 'URL', 'ar_001': 'رابط URL'} |
| `qr_code_and_url` | {'en_US': 'QR code + URL', 'tr_TR': 'QR kodu + URL', 'ar_001': 'QR code + URL'} |

**`point_of_sale_update_stock_quantities`** — {'en_US': 'Update quantities in stock', 'tr_TR': 'Update quantities in stock', 'ar_001': 'تحديث الكميات في المخزون'}

| Value | Label |
|---|---|
| `closing` | {'en_US': 'At the session closing', 'tr_TR': 'Oturum kapanışında', 'ar_001': 'عند إقفال الجلسة'} |
| `real` | {'en_US': 'In real time', 'tr_TR': 'Gerçek zamanlı', 'ar_001': 'في الوقت الفعلي'} |

**`quick_edit_mode`** — {'en_US': 'Quick encoding', 'tr_TR': 'Hızlı kodlama', 'ar_001': 'الترميز السريع'}

| Value | Label |
|---|---|
| `out_invoices` | {'en_US': 'Customer Invoices', 'tr_TR': 'Müşteri Faturaları', 'ar_001': 'فواتير العملاء'} |
| `in_invoices` | {'en_US': 'Vendor Bills', 'tr_TR': 'Tedarikçi  Faturaları', 'ar_001': 'فواتير المورد'} |
| `out_and_in_invoices` | {'en_US': 'Customer Invoices and Vendor Bills', 'tr_TR': 'Müşteri Faturaları ve Tedarikçi Faturaları', 'ar_001': 'فواتير العملاء والموردين'} |

**`sale_onboarding_payment_method`** — {'en_US': 'Sale onboarding selected payment method', 'tr_TR': 'Satışa girişte seçilen ödeme yöntemi', 'ar_001': 'طريقة تهيئة الدفع المحددة للمبيعات'}

| Value | Label |
|---|---|
| `digital_signature` | {'en_US': 'Sign online', 'tr_TR': 'Çevrimiçi imza', 'ar_001': 'التوقيع عبر الإنترنت'} |
| `paypal` | {'en_US': 'PayPal', 'tr_TR': 'PayPal', 'ar_001': 'PayPal'} |
| `stripe` | {'en_US': 'Stripe', 'tr_TR': 'Stripe', 'ar_001': 'Stripe'} |
| `other` | {'en_US': 'Pay with another payment provider', 'tr_TR': 'Başka bir ödeme sağlayıcısı ile ödeme yapın', 'ar_001': 'الدفع باستخدام مزود دفع آخر'} |
| `manual` | {'en_US': 'Manual Payment', 'tr_TR': 'Manuel Ödeme', 'ar_001': 'الدفع اليدوي'} |

**`sms_provider`** — {'en_US': 'SMS Provider', 'tr_TR': 'SMS Sağlayıcı', 'ar_001': 'مزود خدمة الرسائل النصية القصيرة'}

| Value | Label |
|---|---|
| `iap` | {'en_US': 'Send via Odoo', 'tr_TR': 'Odoo üzerinden gönder'} |
| `twilio` | {'en_US': 'Send via Twilio', 'tr_TR': 'Twilio ile gönder'} |

**`stock_confirmation_type`** — {'en_US': 'Stock Confirmation Type', 'tr_TR': 'Stok Onay Türü'}

| Value | Label |
|---|---|
| `sms` | {'en_US': 'SMS', 'tr_TR': 'SMS', 'ar_001': 'الرسائل النصية القصيرة'} |

**`tax_calculation_rounding_method`** — {'en_US': 'Tax Calculation Rounding Method', 'tr_TR': 'Vergi Hesaplaması Yuvarlama Yöntemi', 'ar_001': 'طريقة التقريب لحساب الضريبة'}

| Value | Label |
|---|---|
| `round_globally` | {'en_US': 'Round per Tax', 'tr_TR': 'Vergi Başına Yuvarlak', 'ar_001': 'التقريب حسب الضريبة'} |
| `round_per_line` | {'en_US': 'Round per Line', 'tr_TR': 'Satır Satır Yuvarla', 'ar_001': 'التقريب لكل بند'} |

**`terms_type`** — {'en_US': 'Terms & Conditions format', 'tr_TR': 'Satış Koşulları formatı', 'ar_001': 'صيغة الشروط والأحكام'}

| Value | Label |
|---|---|
| `plain` | {'en_US': 'Add a Note', 'tr_TR': 'Not Ekle', 'ar_001': 'إضافة ملاحظة'} |
| `html` | {'en_US': 'Add a link to a Web Page', 'tr_TR': 'Web Sayfasına bağlantı ekleme', 'ar_001': 'أضف رابطاً لصفحة ويب'} |

## `res.config.settings`

**`account_on_checkout`** — {'en_US': 'Customer Accounts', 'tr_TR': 'Müşteri Hesapları', 'ar_001': 'حسابات العملاء'} (not stored)

| Value | Label |
|---|---|
| `optional` | {'en_US': 'Optional', 'tr_TR': 'İsteğe bağlı', 'ar_001': 'اختياري'} |
| `disabled` | {'en_US': 'Disabled', 'tr_TR': 'Devre Dışı', 'ar_001': 'معطل'} |
| `mandatory` | {'en_US': 'Mandatory', 'tr_TR': 'Zorunlu', 'ar_001': 'إلزامي'} |

**`auth_signup_uninvited`** — {'en_US': 'Customer Account', 'tr_TR': 'Müşteri Hesabı', 'ar_001': 'حساب العميل'} (not stored)

| Value | Label |
|---|---|
| `b2b` | {'en_US': 'On invitation', 'tr_TR': 'Davetiye ile', 'ar_001': 'بدعوة'} |
| `b2c` | {'en_US': 'Free sign up', 'tr_TR': 'Ücretsiz kaydolma', 'ar_001': 'تسجيل مجاني'} |

**`auth_totp_policy`** — {'en_US': 'Two-factor authentication enforcing policy', 'tr_TR': 'İki faktörlü kimlik doğrulama uygulama politikası', 'ar_001': 'سياية تنفيذ المصادقة ثنائية العوامل'}

| Value | Label |
|---|---|
| `employee_required` | {'en_US': 'Employees only', 'tr_TR': 'Yalnızca çalışanlar', 'ar_001': 'الموظفين فقط'} |
| `all_required` | {'en_US': 'All users', 'tr_TR': 'Tüm kullanıcılar', 'ar_001': 'كافة المستخدمين'} |

**`cloud_storage_provider`** — {'en_US': 'Cloud Storage Provider for new attachments', 'tr_TR': 'Yeni ekler için Bulut Depolama Sağlayıcısı', 'ar_001': 'مزود مساحة التخزين السحابية للمرفقات الجديدة'}

| Value | Label |
|---|---|
| `azure` | {'en_US': 'Azure Cloud Storage', 'tr_TR': 'Azure Bulut Depolama', 'ar_001': 'مساحة التخزين السحابية لدى Azure'} |
| `google` | {'en_US': 'Google Cloud Storage', 'tr_TR': 'Google Bulut Depolama', 'ar_001': 'مساحة التخزين السحابية لـ Google'} |

**`crm_auto_assignment_action`** — {'en_US': 'Auto Assignment Action', 'tr_TR': 'Otomatik Atama Aksiyonu', 'ar_001': 'إجراء التعيين التلقائي'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manually', 'tr_TR': 'Manuel olarak', 'ar_001': 'يدويًا'} |
| `auto` | {'en_US': 'Repeatedly', 'tr_TR': 'Defalarca', 'ar_001': 'بشكل متكرر'} |

**`crm_auto_assignment_interval_type`** — {'en_US': 'Auto Assignment Interval Unit', 'tr_TR': 'Otomatik Atama Aralığı Birimi', 'ar_001': 'وحدة الفترة الزمنية الفاصلة للتعيين التلقائي'}

| Value | Label |
|---|---|
| `minutes` | {'en_US': 'Minutes', 'tr_TR': 'Dakika', 'ar_001': 'الدقائق'} |
| `hours` | {'en_US': 'Hours', 'tr_TR': 'Saat', 'ar_001': 'ساعات'} |
| `days` | {'en_US': 'Days', 'tr_TR': 'Gün', 'ar_001': 'الأيام'} |
| `weeks` | {'en_US': 'Weeks', 'tr_TR': 'Hafta', 'ar_001': 'أسابيع'} |

**`default_invoice_policy`** — {'en_US': 'Invoicing Policy', 'tr_TR': 'Faturalama Kuralı', 'ar_001': 'سياسة الفوترة'}

| Value | Label |
|---|---|
| `order` | {'en_US': 'Invoice what is ordered', 'tr_TR': 'Sipariş miktarını faturala', 'ar_001': 'فوترة الكميات المطلوبة'} |
| `delivery` | {'en_US': 'Invoice what is delivered', 'tr_TR': 'Teslim edilenden faturala', 'ar_001': 'فوترة الكميات التي تم توصيلها'} |

**`default_picking_policy`** — {'en_US': 'Picking Policy', 'tr_TR': 'Toplama Politikası', 'ar_001': 'سياسة الاستلام'}

| Value | Label |
|---|---|
| `direct` | {'en_US': 'Ship products as soon as available, with back orders', 'tr_TR': 'Hazır olan ürünleri teslimatı bölerek hemen gönder', 'ar_001': 'شحن كل منتج على حدة عند توافره، مع الطلبات المؤجلة'} |
| `one` | {'en_US': 'Ship all products at once', 'tr_TR': 'Tüm ürünleri tek seferde gönder', 'ar_001': 'شحن كافة المنتجات دفعة واحدة'} |

**`l10n_in_gsp`** — {'en_US': 'GSP'} (not stored)

| Value | Label |
|---|---|
| `bvm` | {'en_US': 'BVM IT Consulting'} |
| `tera` | {'en_US': 'Tera Software (Deprecated)'} |

**`lead_enrich_auto`** — {'en_US': 'Enrich lead automatically', 'tr_TR': 'Adayı otomatik olarak zenginleştirin', 'ar_001': 'إثراء العميل المهتم تلقائياً'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Enrich leads on demand only', 'tr_TR': 'Sadece talep üzerine adayları zenginleştirin.', 'ar_001': 'إثراء العملاء المهتمين عند الطلب فقط'} |
| `auto` | {'en_US': 'Enrich all leads automatically', 'tr_TR': 'Tüm adayları otomatik olarak zenginleştirin.', 'ar_001': 'قم بإثراء كافة العملاء المهتمين تلقائياً'} |

**`onboarding_payment_module`** — {'en_US': 'Onboarding Payment Module', 'tr_TR': 'Onboarding Ödeme Modülü'} (not stored)

| Value | Label |
|---|---|
| `mercado_pago` | {'en_US': 'Mercado Pago', 'tr_TR': 'Mercado Pago', 'ar_001': 'Mercado Pago'} |
| `razorpay` | {'en_US': 'Razorpay', 'tr_TR': 'Razorpay', 'ar_001': 'Razorpay'} |
| `stripe` | {'en_US': 'Stripe', 'tr_TR': 'Stripe', 'ar_001': 'Stripe'} |

**`peppol_participation_role`** — {'en_US': 'Peppol Participation Role', 'ar_001': 'دور المشاركة في Peppol'} (not stored)

| Value | Label |
|---|---|
| `sending_and_receiving` | {'en_US': 'Sending & Receiving', 'ar_001': 'الإرسال والاستلام'} |
| `sending_only` | {'en_US': 'Sending Only', 'ar_001': 'إرسال فقط'} |

**`product_volume_volume_in_cubic_feet`** — {'en_US': 'Volume unit of measure', 'tr_TR': 'Hacim ölçü birimi', 'ar_001': 'وحدة قياس الحجم'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Cubic Meters (m³)', 'tr_TR': 'Metreküp (m³)', 'ar_001': 'متر مكعب (m³)'} |
| `1` | {'en_US': 'Cubic Feet (ft³)', 'tr_TR': 'Fit Küp (ft³)', 'ar_001': 'قدم مكعب (ft³)'} |

**`product_weight_in_lbs`** — {'en_US': 'Weight unit of measure', 'tr_TR': 'Ağırlık ölçü birimi', 'ar_001': 'وحدة قياس الوزن'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Kilograms (kg)', 'tr_TR': 'Kilogram (kg)', 'ar_001': 'كيلوجرام (كجم)'} |
| `1` | {'en_US': 'Pounds (lb)', 'tr_TR': 'Pound (lb)', 'ar_001': 'رطل (lb)'} |

**`timesheet_encode_method`** — {'en_US': 'Encoding Method', 'tr_TR': 'Kodlama Yöntemi', 'ar_001': 'طريقة التشفير'} (not stored)

| Value | Label |
|---|---|
| `hours` | {'en_US': 'Hours / Minutes', 'tr_TR': 'Saat / Dakika', 'ar_001': 'الساعات / الدقائق'} |
| `days` | {'en_US': 'Days / Half-Days', 'tr_TR': 'Gün / Yarım Gün', 'ar_001': 'الأيام / أنصاف الأيام'} |

## `res.country`

**`name_position`** — {'en_US': 'Customer Name Position', 'tr_TR': 'Müşteri Adı Pozisyonu', 'ar_001': 'موضع اسم العميل'}

| Value | Label |
|---|---|
| `before` | {'en_US': 'Before Address', 'tr_TR': 'Adresten Önce', 'ar_001': 'قبل العنوان'} |
| `after` | {'en_US': 'After Address', 'tr_TR': 'Adres sonrası', 'ar_001': 'بعد العنوان'} |

## `res.currency`

**`position`** — {'en_US': 'Symbol Position', 'tr_TR': 'Sembol Pozisyonu', 'ar_001': 'موضع الرمز'}

| Value | Label |
|---|---|
| `after` | {'en_US': 'After Amount', 'tr_TR': 'Tutardan Sonra', 'ar_001': 'بعد المبلغ'} |
| `before` | {'en_US': 'Before Amount', 'tr_TR': 'Tutar Önce', 'ar_001': 'قبل المبلغ'} |

## `res.device`

**`device_type`** — {'en_US': 'Device Type', 'tr_TR': 'Cihaz Türü', 'ar_001': 'نوع الجهاز'}

| Value | Label |
|---|---|
| `computer` | {'en_US': 'Computer', 'tr_TR': 'Bilgisayar', 'ar_001': 'الحاسوب'} |
| `mobile` | {'en_US': 'Mobile', 'tr_TR': 'Mobil', 'ar_001': 'الهاتف المحمول'} |

## `res.device.log`

**`device_type`** — {'en_US': 'Device Type', 'tr_TR': 'Cihaz Türü', 'ar_001': 'نوع الجهاز'}

| Value | Label |
|---|---|
| `computer` | {'en_US': 'Computer', 'tr_TR': 'Bilgisayar', 'ar_001': 'الحاسوب'} |
| `mobile` | {'en_US': 'Mobile', 'tr_TR': 'Mobil', 'ar_001': 'الهاتف المحمول'} |

## `res.groups`

**`lock_timeout_2fa_selection`** — {'en_US': 'Lock Timeout 2Fa Selection', 'tr_TR': 'Kilit Zaman Aşımı 2Fa Seçimi', 'ar_001': 'اختيار مهلة القفل 2Fa'} (not stored)

| Value | Label |
|---|---|
| `without_2fa` | {'en_US': 'Logout', 'tr_TR': 'Çıkış', 'ar_001': 'تسجيل الخروج'} |
| `with_2fa` | {'en_US': 'Logout with two-factor authentication', 'tr_TR': 'İki faktörlü kimlik doğrulama ile oturum kapatma', 'ar_001': 'تسجيل الخروج باستخدام المصادقة الثنائية'} |

**`lock_timeout_delay_unit`** — {'en_US': 'Lock Timeout Delay Unit', 'tr_TR': 'Kilit Zaman Aşımı Gecikme Birimi', 'ar_001': 'وحدة تأخير انتهاء مهلة القفل'} (not stored)

| Value | Label |
|---|---|
| `minutes` | {'en_US': 'minutes', 'tr_TR': 'dakika', 'ar_001': 'دقائق'} |
| `hours` | {'en_US': 'hours', 'tr_TR': 'saat', 'ar_001': 'ساعات'} |
| `days` | {'en_US': 'days', 'tr_TR': 'gün', 'ar_001': 'أيام'} |

**`lock_timeout_inactivity_2fa_selection`** — {'en_US': 'Lock Timeout Inactivity 2Fa Selection', 'tr_TR': 'Kilit Zaman Aşımı Hareketsizlik 2Fa Seçimi', 'ar_001': 'اختيار المصادقة الثنائية لوقت انتهاء صلاحية القفل بسبب عدم النشاط'} (not stored)

| Value | Label |
|---|---|
| `without_2fa` | {'en_US': 'Screen lock', 'tr_TR': 'Ekran kilidi', 'ar_001': 'قفل الشاشة'} |
| `with_2fa` | {'en_US': 'Screen lock with two-factor authentication', 'tr_TR': 'İki faktörlü kimlik doğrulama ile ekran kilidi', 'ar_001': 'قفل الشاشة باستخدام المصادقة الثنائية'} |

**`lock_timeout_inactivity_delay_unit`** — {'en_US': 'Lock Timeout Inactivity Delay Unit', 'tr_TR': 'Kilit Zaman Aşımı Hareketsizlik Gecikme Birimi', 'ar_001': 'وحدة تأخير مهلة القفل بسبب عدم النشاط'} (not stored)

| Value | Label |
|---|---|
| `minutes` | {'en_US': 'minutes', 'tr_TR': 'dakika', 'ar_001': 'دقائق'} |
| `hours` | {'en_US': 'hours', 'tr_TR': 'saat', 'ar_001': 'ساعات'} |
| `days` | {'en_US': 'days', 'tr_TR': 'gün', 'ar_001': 'أيام'} |

## `res.lang`

**`direction`** — {'en_US': 'Direction', 'tr_TR': 'Yön', 'ar_001': 'الاتجاه'}

| Value | Label |
|---|---|
| `ltr` | {'en_US': 'Left-to-Right', 'tr_TR': 'Soldan Sağa', 'ar_001': 'من اليسار إلى اليمين'} |
| `rtl` | {'en_US': 'Right-to-Left', 'tr_TR': 'Sağdan Sola', 'ar_001': 'من اليمين إلى اليسار'} |

**`grouping`** — {'en_US': 'Separator Format', 'tr_TR': 'Ayırıcı Biçimi', 'ar_001': 'تنسيق الفاصل'}

| Value | Label |
|---|---|
| `[3,0]` | {'en_US': 'International Grouping', 'tr_TR': 'Uluslararası Gruplandırma', 'ar_001': 'التجمع الدولي'} |
| `[3,2,0]` | {'en_US': 'Indian Grouping', 'tr_TR': 'Hint Gruplaması', 'ar_001': 'Indian Grouping'} |

**`time_format`** — {'en_US': 'Time Format', 'tr_TR': 'Saat Biçimi', 'ar_001': 'تنسيق الوقت'}

| Value | Label |
|---|---|
| `%H:%M:%S` | {'en_US': '13:00:00', 'tr_TR': '13:00:00', 'ar_001': '13:00:00'} |
| `%I:%M:%S %p` | {'en_US': ' 1:00:00 PM', 'tr_TR': ' 13:00', 'ar_001': ' 1:00:00 مساءً'} |

**`week_start`** — {'en_US': 'First Day of Week', 'tr_TR': 'Haftanın ilk Günü', 'ar_001': 'أول أيام الأسبوع'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `2` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `3` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `4` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `5` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `6` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |
| `7` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |

## `res.partner`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`autopost_bills`** — {'en_US': 'Auto-post bills', 'tr_TR': 'Faturaları otomatik-onayla', 'ar_001': 'ترحيل فواتير المورّدين تلقائياً'}

| Value | Label |
|---|---|
| `always` | {'en_US': 'Always', 'tr_TR': 'Daima', 'ar_001': 'دائمًا'} |
| `ask` | {'en_US': 'Ask after 3 validations without edits', 'tr_TR': 'Düzenleme olmadan 3 doğrulamadan sonra sor', 'ar_001': 'السؤال بعد 3 عمليات تصديق دون تعديلات'} |
| `never` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقًا'} |

**`company_type`** — {'en_US': 'Company Type', 'tr_TR': 'Şirket Türü', 'ar_001': 'نوع الشركة'} (not stored)

| Value | Label |
|---|---|
| `person` | {'en_US': 'Person', 'tr_TR': 'Kişi', 'ar_001': 'شخص'} |
| `company` | {'en_US': 'Company', 'tr_TR': 'Firma', 'ar_001': 'الشركة'} |

**`group_on`** — {'en_US': 'Week Day', 'tr_TR': 'Haftanın Günü', 'ar_001': 'يوم من الأسبوع'}

| Value | Label |
|---|---|
| `default` | {'en_US': 'Expected Date', 'tr_TR': 'Beklenen Tarih', 'ar_001': 'التاريخ المتوقع'} |
| `1` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `2` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `3` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `4` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `5` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `6` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |
| `7` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |

**`group_rfq`** — {'en_US': 'Group RFQ', 'tr_TR': 'Grup Teklif Talebi'}

| Value | Label |
|---|---|
| `default` | {'en_US': 'On Order', 'tr_TR': 'Sipariş Üzerine'} |
| `day` | {'en_US': 'Daily', 'tr_TR': 'Günlük', 'ar_001': 'يوميًا'} |
| `week` | {'en_US': 'Weekly', 'tr_TR': 'Haftalık', 'ar_001': 'أسبوعيًا'} |
| `all` | {'en_US': 'Always', 'tr_TR': 'Daima', 'ar_001': 'دائمًا'} |

**`invoice_edi_format`** — {'en_US': 'eInvoice format', 'tr_TR': 'efatura formatı', 'ar_001': 'تنسيق الفاتورة الإلكترونية'} (not stored)

| Value | Label |
|---|---|
| `facturx` | {'en_US': 'France (FacturX)', 'tr_TR': 'Fransa (FacturX)', 'ar_001': 'France (FacturX)'} |
| `ubl_bis3` | {'en_US': 'EU Standard (Peppol Bis 3.0)', 'tr_TR': 'AB Standardı (Peppol Bis 3.0)', 'ar_001': 'EU Standard (Peppol Bis 3.0)'} |
| `zugferd` | {'en_US': 'Germany (ZUGFeRD)', 'ar_001': 'ألمانيا (ZUGFeRD)'} |
| `xrechnung` | {'en_US': 'Germany (XRechnung)', 'tr_TR': 'Almanya (XRechnung)', 'ar_001': 'Germany (XRechnung)'} |
| `nlcius` | {'en_US': 'Netherlands (NLCIUS)', 'tr_TR': 'Hollanda (NLCIUS)', 'ar_001': 'Netherlands (NLCIUS)'} |
| `ubl_a_nz` | {'en_US': 'Australia (BIS Billing 3.0 A-NZ)', 'tr_TR': 'Avustralya (BIS Billing 3.0 A-NZ)', 'ar_001': 'Australia (BIS Billing 3.0 A-NZ)'} |
| `ubl_sg` | {'en_US': 'Singapore (BIS Billing 3.0 SG)', 'tr_TR': 'Singapur (BIS Billing 3.0 SG)', 'ar_001': 'Singapore (BIS Billing 3.0 SG)'} |
| `pint_anz` | {'en_US': 'Australia (Peppol Pint AU)'} |
| `pint_jp` | {'en_US': 'Japan (Peppol PINT JP)'} |
| `pint_my` | {'en_US': 'Malaysia (Peppol PINT MY)'} |
| `pint_sg` | {'en_US': 'Singapore (Peppol PINT SG)'} |
| `ubl_tr` | {'en_US': 'Türkiye (UBL TR 1.2)'} |
| `tw_ecpay` | {'en_US': 'ECPay'} |
| `oioubl_21` | {'en_US': 'OIOUBL 2.1'} |
| `oioubl_201` | {'en_US': 'Denmark (Oioubl)'} |
| `es_facturae` | {'en_US': 'Spain (FacturaE)'} |
| `ubl_hr` | {'en_US': 'CIUS HR'} |
| `it_edi_xml` | {'en_US': 'Italy (Factura PA)'} |
| `fa3_pl` | {'en_US': 'Polish FA3'} |
| `ciusro` | {'en_US': 'Romania (CIUS RO)'} |
| `vn_sinvoice` | {'en_US': 'Vietnam (SInvoice)'} |
| `ubl_21_fr` | {'en_US': 'France E-Invoicing (UBL 2.1)'} |

**`invoice_sending_method`** — {'en_US': 'Invoice sending', 'tr_TR': 'Fatura gönderiliyor', 'ar_001': 'جاري إرسال الفاتورة'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `email` | {'en_US': 'by Email', 'tr_TR': 'Eposta ile', 'ar_001': 'عن طريق البريد الإلكتروني'} |
| `snailmail` | {'en_US': 'by Post', 'tr_TR': 'tarafından Post', 'ar_001': 'عن طريق البريد'} |
| `peppol` | {'en_US': 'by Peppol', 'tr_TR': 'by Peppol', 'ar_001': 'by Peppol'} |
| `nemhandel` | {'en_US': 'By Nemhandel'} |
| `mojeracun` | {'en_US': 'by MojEracun'} |

**`l10n_ar_gross_income_type`** — {'en_US': 'Gross Income Type'}

| Value | Label |
|---|---|
| `multilateral` | {'en_US': 'Multilateral'} |
| `local` | {'en_US': 'Local'} |
| `exempt` | {'en_US': 'Exempt'} |

**`l10n_cl_sii_taxpayer_type`** — {'en_US': 'Taxpayer Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'VAT Affected (1st Category)'} |
| `2` | {'en_US': 'Fees Receipt Issuer (2nd category)'} |
| `3` | {'en_US': 'End Consumer'} |
| `4` | {'en_US': 'Foreigner'} |

**`l10n_id_buyer_document_type`** — {'en_US': 'Document Type'}

| Value | Label |
|---|---|
| `TIN` | {'en_US': 'TIN'} |
| `NIK` | {'en_US': 'NIK'} |
| `Passport` | {'en_US': 'Passport'} |
| `Other` | {'en_US': 'Others'} |

**`l10n_id_kode_transaksi`** — {'en_US': 'Invoice Transaction Code'}

| Value | Label |
|---|---|
| `01` | {'en_US': '01 To the Parties that is not VAT Collector (Regular Customers)'} |
| `02` | {'en_US': '02 To the Treasurer'} |
| `03` | {'en_US': '03 To other VAT Collectors other than the Treasurer'} |
| `04` | {'en_US': '04 Other Value of VAT Imposition Base'} |
| `05` | {'en_US': '05 Specified Amount (Article 9A Paragraph (1) VAT Law)'} |
| `06` | {'en_US': '06 to individuals holding foreign passports'} |
| `07` | {'en_US': '07 Deliveries that the VAT is not Collected'} |
| `08` | {'en_US': '08 Deliveries that the VAT is Exempted'} |
| `09` | {'en_US': '09 Deliveries of Assets (Article 16D of VAT Law)'} |
| `10` | {'en_US': '10 Other deliveries'} |

**`l10n_in_gst_treatment`** — {'en_US': 'GST Treatment'}

| Value | Label |
|---|---|
| `regular` | {'en_US': 'Registered Business - Regular'} |
| `composition` | {'en_US': 'Registered Business - Composition'} |
| `unregistered` | {'en_US': 'Unregistered Business'} |
| `consumer` | {'en_US': 'Consumer'} |
| `overseas` | {'en_US': 'Overseas'} |
| `special_economic_zone` | {'en_US': 'Special Economic Zone'} |
| `deemed_export` | {'en_US': 'Deemed Export'} |
| `uin_holders` | {'en_US': 'UIN Holders'} |

**`l10n_my_identification_type`** — {'en_US': 'ID Type'}

| Value | Label |
|---|---|
| `NRIC` | {'en_US': 'MyKad/MyTentera/MyPR/MyKAS'} |
| `BRN` | {'en_US': 'Business Registration Number'} |
| `PASSPORT` | {'en_US': 'Passport'} |
| `ARMY` | {'en_US': 'Army'} |

**`l10n_my_tin_validation_state`** — {'en_US': 'Tin Validation State'}

| Value | Label |
|---|---|
| `valid` | {'en_US': 'Valid'} |
| `invalid` | {'en_US': 'Invalid'} |

**`l10n_sa_edi_additional_identification_scheme`** — {'en_US': 'Identification Scheme', 'ar_001': 'خطة التعريف'}

| Value | Label |
|---|---|
| `TIN` | {'en_US': 'Tax Identification Number', 'ar_001': 'رقم التعريف الضريبي'} |
| `CRN` | {'en_US': 'Commercial Registration Number', 'ar_001': 'رقم التسجيل التجاري'} |
| `MOM` | {'en_US': 'Momra License', 'ar_001': 'رخصة وزارة الشؤون البلدية و القروية والإسكان'} |
| `MLS` | {'en_US': 'MLSD License', 'ar_001': 'رخصة وزارة الموارد البشرية و التنمية الاجتماعية'} |
| `700` | {'en_US': '700 Number', 'ar_001': '700 رقم'} |
| `SAG` | {'en_US': 'Sagia License', 'ar_001': 'رخصة وزارة الاستثمار'} |
| `NAT` | {'en_US': 'National ID', 'ar_001': 'الهوية الوطنية'} |
| `GCC` | {'en_US': 'GCC ID', 'ar_001': 'معرّف مجلس التعاون الخليجي'} |
| `IQA` | {'en_US': 'Iqama Number', 'ar_001': 'رقم إقامة '} |
| `PAS` | {'en_US': 'Passport ID', 'ar_001': 'معرف جواز السفر'} |
| `OTH` | {'en_US': 'Other ID', 'ar_001': 'معرف آخر'} |

**`l10n_tr_nilvera_customer_status`** — {'en_US': 'Nilvera Status'}

| Value | Label |
|---|---|
| `not_checked` | {'en_US': 'Not Verified'} |
| `earchive` | {'en_US': 'E-Archive'} |
| `einvoice` | {'en_US': 'E-Invoice'} |

**`nemhandel_identifier_type`** — {'en_US': 'Nemhandel Endpoint Type'}

| Value | Label |
|---|---|
| `0088` | {'en_US': 'EAN/GLN'} |
| `0184` | {'en_US': 'CVR'} |
| `9918` | {'en_US': 'IBAN'} |
| `0198` | {'en_US': 'SE'} |

**`nemhandel_verification_state`** — {'en_US': 'Nemhandel endpoint verification'}

| Value | Label |
|---|---|
| `not_verified` | {'en_US': 'Not verified yet'} |
| `not_valid` | {'en_US': 'Not on Nemhandel'} |
| `valid` | {'en_US': 'Valid'} |

**`pdp_verification_display_state`** — {'en_US': 'E-Invoicing State'} (not stored)

| Value | Label |
|---|---|
| `not_verified` | {'en_US': 'Not verified yet'} |
| `pdp_not_valid` | {'en_US': 'Partner is not in the annuaire'} |
| `pdp_not_valid_format` | {'en_US': 'Partner cannot receive format'} |
| `pdp_valid` | {'en_US': 'Partner is in the annuaire'} |
| `peppol_not_valid` | {'en_US': 'Partner is not on Peppol'} |
| `peppol_not_valid_format` | {'en_US': 'Partner cannot receive format'} |
| `peppol_valid` | {'en_US': 'Partner is on Peppol'} |

**`peppol_eas`** — {'en_US': 'Peppol e-address (EAS)', 'tr_TR': 'Peppol e-address (EAS)', 'ar_001': 'Peppol e-address (EAS)'}

| Value | Label |
|---|---|
| `9923` | {'en_US': 'Albania VAT', 'tr_TR': 'Arnavutluk KDV', 'ar_001': 'Albania VAT'} |
| `9922` | {'en_US': 'Andorra VAT', 'tr_TR': 'Andorra KDV', 'ar_001': 'Andorra VAT'} |
| `0151` | {'en_US': 'Australia ABN', 'tr_TR': 'Avustralya ABN', 'ar_001': 'Australia ABN'} |
| `9914` | {'en_US': 'Austria UID', 'tr_TR': 'Avusturya UID', 'ar_001': 'Austria UID'} |
| `9915` | {'en_US': 'Austria VOKZ', 'tr_TR': 'Avusturya VOKZ', 'ar_001': 'Austria VOKZ'} |
| `0208` | {'en_US': 'Belgian Company Registry', 'tr_TR': 'Belçika Şirket Sicili', 'ar_001': 'Belgian Company Registry'} |
| `9925` | {'en_US': 'Belgian VAT', 'tr_TR': 'Belçika KDV', 'ar_001': 'Belgian VAT'} |
| `9924` | {'en_US': 'Bosnia and Herzegovina VAT', 'tr_TR': 'Bsna-Hersek KDV', 'ar_001': 'Bosnia and Herzegovina VAT'} |
| `9926` | {'en_US': 'Bulgaria VAT', 'tr_TR': 'Bulgaristan KDV', 'ar_001': 'Bulgaria VAT'} |
| `9934` | {'en_US': 'Croatia VAT', 'tr_TR': 'Hırvatistan KDV', 'ar_001': 'Croatia VAT'} |
| `9928` | {'en_US': 'Cyprus VAT', 'tr_TR': 'Kıbrıs KDV', 'ar_001': 'Cyprus VAT'} |
| `9929` | {'en_US': 'Czech Republic VAT', 'tr_TR': 'Çek Cumhuriyeti KDV', 'ar_001': 'Czech Republic VAT'} |
| `0096` | {'en_US': 'Denmark P', 'tr_TR': 'Danimarka P', 'ar_001': 'Denmark P'} |
| `0184` | {'en_US': 'Denmark CVR', 'tr_TR': 'Danimarka CVR', 'ar_001': 'Denmark CVR'} |
| `0198` | {'en_US': 'Denmark SE', 'tr_TR': 'Danimarka SE', 'ar_001': 'Denmark SE'} |
| `0191` | {'en_US': 'Estonia Company code', 'tr_TR': 'Estonya Şirket Kodu', 'ar_001': 'Estonia Company code'} |
| `9931` | {'en_US': 'Estonia VAT', 'tr_TR': 'Estonya KDV', 'ar_001': 'Estonia VAT'} |
| `0037` | {'en_US': 'Finland LY-tunnus', 'tr_TR': 'Finlandiya LY-tunnus', 'ar_001': 'Finland LY-tunnus'} |
| `0216` | {'en_US': 'Finland OVT code', 'tr_TR': 'Finlandiya OVT kodu', 'ar_001': 'Finland OVT code'} |
| `0213` | {'en_US': 'Finland VAT', 'tr_TR': 'Finlandiya KDV', 'ar_001': 'Finland VAT'} |
| `0002` | {'en_US': 'France SIRENE', 'tr_TR': 'Fransa SIRENE', 'ar_001': 'France SIRENE'} |
| `0009` | {'en_US': 'France SIRET', 'tr_TR': 'Fransa SIRET', 'ar_001': 'France SIRET'} |
| `9957` | {'en_US': 'France VAT', 'tr_TR': 'Fransa KDV', 'ar_001': 'France VAT'} |
| `0225` | {'en_US': 'France FRCTC Electronic Address', 'tr_TR': 'Fransa FRCTC Elektronik Adresi', 'ar_001': 'France FRCTC Electronic Address'} |
| `0240` | {'en_US': 'France Register of legal persons', 'tr_TR': 'Fransa Tüzel Kişiler Sicili', 'ar_001': 'France Register of legal persons'} |
| `0246` | {'en_US': 'German Electronic Business Address', 'ar_001': 'German Electronic Business Address'} |
| `0204` | {'en_US': 'Germany Leitweg-ID', 'tr_TR': 'Almanya Leitweg Kimliği', 'ar_001': 'Germany Leitweg-ID'} |
| `9930` | {'en_US': 'Germany VAT', 'tr_TR': 'Almanya KDV Numarası', 'ar_001': 'Germany VAT'} |
| `9933` | {'en_US': 'Greece VAT', 'tr_TR': 'Yunanistan KDV Numarası', 'ar_001': 'Greece VAT'} |
| `9910` | {'en_US': 'Hungary VAT', 'tr_TR': 'Macaristan KDV Numarası', 'ar_001': 'Hungary VAT'} |
| `0196` | {'en_US': 'Iceland Kennitala', 'tr_TR': 'İzlanda Kennitala', 'ar_001': 'Iceland Kennitala'} |
| `9935` | {'en_US': 'Ireland VAT', 'tr_TR': 'İrlanda KDV Numarası', 'ar_001': 'Ireland VAT'} |
| `0211` | {'en_US': 'Italia Partita IVA', 'tr_TR': 'İtalya Partita IVA (KDV Numarası)', 'ar_001': 'Italia Partita IVA'} |
| `0097` | {'en_US': 'Italia FTI', 'tr_TR': 'İtalya FTI', 'ar_001': 'Italia FTI'} |
| `0188` | {'en_US': 'Japan SST', 'tr_TR': 'Japonya SST', 'ar_001': 'Japan SST'} |
| `0221` | {'en_US': 'Japan IIN', 'tr_TR': 'Japonya IIN', 'ar_001': 'Japan IIN'} |
| `0218` | {'en_US': 'Latvia Unified registration number', 'tr_TR': 'Letonya Birleşik Kayıt Numarası', 'ar_001': 'Latvia Unified registration number'} |
| `9939` | {'en_US': 'Latvia VAT', 'tr_TR': 'Litvanya JAK', 'ar_001': 'Latvia VAT'} |
| `9936` | {'en_US': 'Liechtenstein VAT', 'tr_TR': 'Lihtenştayn KDV Numarası', 'ar_001': 'Liechtenstein VAT'} |
| `0200` | {'en_US': 'Lithuania JAK', 'tr_TR': 'Litvanya JAK', 'ar_001': 'Lithuania JAK'} |
| `9937` | {'en_US': 'Lithuania VAT', 'tr_TR': 'Litvanya KDV Numarası', 'ar_001': 'Lithuania VAT'} |
| `9938` | {'en_US': 'Luxembourg VAT', 'tr_TR': 'Lüksemburg KDV Numarası', 'ar_001': 'Luxembourg VAT'} |
| `9942` | {'en_US': 'Macedonia VAT', 'tr_TR': 'Makedonya KDV Numarası', 'ar_001': 'Macedonia VAT'} |
| `0230` | {'en_US': 'Malaysia', 'tr_TR': 'Malezya', 'ar_001': 'ماليزيا'} |
| `9943` | {'en_US': 'Malta VAT', 'tr_TR': 'Malta KDV Numarası', 'ar_001': 'Malta VAT'} |
| `9940` | {'en_US': 'Monaco VAT', 'tr_TR': 'Monako KDV Numarası', 'ar_001': 'Monaco VAT'} |
| `9941` | {'en_US': 'Montenegro VAT', 'tr_TR': 'Karadağ KDV Numarası', 'ar_001': 'Montenegro VAT'} |
| `0106` | {'en_US': 'Netherlands KvK', 'tr_TR': 'Hollanda KvK', 'ar_001': 'Netherlands KvK'} |
| `0190` | {'en_US': 'Netherlands OIN', 'tr_TR': 'Hollanda OIN', 'ar_001': 'Netherlands OIN'} |
| `9944` | {'en_US': 'Netherlands VAT', 'tr_TR': 'Hollanda KDV Numarası', 'ar_001': 'Netherlands VAT'} |
| `0244` | {'en_US': 'Nigeria Tax Identification', 'ar_001': 'Nigeria Tax Identification'} |
| `0192` | {'en_US': 'Norway Org.nr.', 'tr_TR': 'Norveç Kurum No.', 'ar_001': 'Norway Org.nr.'} |
| `9945` | {'en_US': 'Poland VAT', 'tr_TR': 'Polonya KDV Numarası', 'ar_001': 'Poland VAT'} |
| `9946` | {'en_US': 'Portugal VAT', 'tr_TR': 'Portekiz KDV Numarası', 'ar_001': 'Portugal VAT'} |
| `9947` | {'en_US': 'Romania VAT', 'tr_TR': 'Romanya KDV Numarası', 'ar_001': 'Romania VAT'} |
| `9948` | {'en_US': 'Serbia VAT', 'tr_TR': 'Sırbistan KDV Numarası', 'ar_001': 'Serbia VAT'} |
| `0195` | {'en_US': 'Singapore UEN', 'tr_TR': 'Singapur UEN', 'ar_001': 'Singapore UEN'} |
| `0245` | {'en_US': 'SK Tax identification number (DIČ)', 'ar_001': 'SK Tax identification number (DIČ)'} |
| `9949` | {'en_US': 'Slovenia VAT', 'tr_TR': 'Slovenya KDV Numarası', 'ar_001': 'Slovenia VAT'} |
| `9950` | {'en_US': 'Slovakia VAT', 'tr_TR': 'Slovakya KDV Numarası', 'ar_001': 'Slovakia VAT'} |
| `9920` | {'en_US': 'Spain VAT', 'tr_TR': 'İspanya KDV Numarası', 'ar_001': 'Spain VAT'} |
| `0007` | {'en_US': 'Sweden Org.nr.', 'tr_TR': 'İsveç Kurum No.', 'ar_001': 'Sweden Org.nr.'} |
| `9955` | {'en_US': 'Sweden VAT', 'tr_TR': 'İsveç KDV Numarası', 'ar_001': 'Sweden VAT'} |
| `9927` | {'en_US': 'Swiss VAT', 'tr_TR': 'İsviçre KDV Numarası', 'ar_001': 'Swiss VAT'} |
| `0183` | {'en_US': 'Swiss UIDB', 'tr_TR': 'İsviçre UIDB', 'ar_001': 'Swiss UIDB'} |
| `9952` | {'en_US': 'Turkey VAT', 'tr_TR': 'Türkiye KDV Numarası', 'ar_001': 'Turkey VAT'} |
| `0235` | {'en_US': 'UAE Tax Identification Number (TIN)', 'tr_TR': 'BAE Vergi Kimlik Numarası (TIN)', 'ar_001': 'رقم التعريف الضريبي الإماراتي (TIN)'} |
| `9932` | {'en_US': 'United Kingdom VAT', 'tr_TR': 'Birleşik Krallık KDV', 'ar_001': 'United Kingdom VAT'} |
| `9959` | {'en_US': 'USA EIN', 'tr_TR': 'ABD EIN', 'ar_001': 'USA EIN'} |
| `0060` | {'en_US': 'DUNS Number', 'tr_TR': 'DUNS Numarası', 'ar_001': 'DUNS Number'} |
| `0088` | {'en_US': 'EAN Location Code', 'tr_TR': 'EAN Konum Kodu', 'ar_001': 'EAN Location Code'} |
| `0130` | {'en_US': 'Directorates of the European Commission', 'tr_TR': 'Avrupa Komisyonu Genel Müdürlükleri', 'ar_001': 'Directorates of the European Commission'} |
| `0135` | {'en_US': 'SIA Object Identifiers', 'tr_TR': 'SIA Nesne Tanımlayıcıları', 'ar_001': 'SIA Object Identifiers'} |
| `0142` | {'en_US': 'SECETI Object Identifiers', 'tr_TR': 'SECETI Nesne Tanımlayıcıları', 'ar_001': 'SECETI Object Identifiers'} |
| `0193` | {'en_US': 'UBL.BE party identifier', 'tr_TR': 'UBL.BE taraf tanımlayıcı', 'ar_001': 'UBL.BE party identifier'} |
| `0199` | {'en_US': 'Legal Entity Identifier (LEI)', 'tr_TR': 'Tüzel Kişilik Tanımlayıcısı (LEI)', 'ar_001': 'Legal Entity Identifier (LEI)'} |
| `0201` | {'en_US': 'Codice Univoco Unità Organizzativa iPA', 'tr_TR': 'Codice Univoco Unità Organizzativa iPA', 'ar_001': 'Codice Univoco Unità Organizzativa iPA'} |
| `0202` | {'en_US': 'Indirizzo di Posta Elettronica Certificata', 'tr_TR': 'Indirizzo di Posta Elettronica Certificata', 'ar_001': 'Indirizzo di Posta Elettronica Certificata'} |
| `0209` | {'en_US': 'GS1 identification keys', 'tr_TR': 'GS1 tanımlama anahtarları', 'ar_001': 'GS1 identification keys'} |
| `0210` | {'en_US': 'Codice Fiscale', 'tr_TR': 'Codice Fiscale', 'ar_001': 'Codice Fiscale'} |
| `9913` | {'en_US': 'Business Registers Network', 'tr_TR': 'Ticaret Sicil Ağı', 'ar_001': 'Business Registers Network'} |
| `9918` | {'en_US': 'S.W.I.F.T', 'tr_TR': 'S.W.I.F.T', 'ar_001': 'S.W.I.F.T'} |
| `9919` | {'en_US': 'Kennziffer des Unternehmensregisters', 'tr_TR': 'Kennziffer des Unternehmensregisters', 'ar_001': 'Kennziffer des Unternehmensregisters'} |
| `9951` | {'en_US': 'San Marino VAT', 'tr_TR': 'San Marino KDV', 'ar_001': 'San Marino VAT'} |
| `9953` | {'en_US': 'Vatican VAT', 'tr_TR': 'Vatikan KDV Numarası', 'ar_001': 'Vatican VAT'} |
| `AN` | {'en_US': 'O.F.T.P. (ODETTE File Transfer Protocol)', 'tr_TR': 'O.F.T.P. (ODETTE Dosya Aktarım Protokolü)', 'ar_001': 'O.F.T.P. (ODETTE File Transfer Protocol)'} |
| `AQ` | {'en_US': 'X.400 address for mail text', 'tr_TR': 'E-posta metni için X.400 adresi', 'ar_001': 'X.400 address for mail text'} |
| `AS` | {'en_US': 'AS2 exchange', 'tr_TR': 'AS2 aktarımı', 'ar_001': 'AS2 exchange'} |
| `AU` | {'en_US': 'File Transfer Protocol', 'tr_TR': 'Dosya Aktarım Protokolü (FTP)', 'ar_001': 'File Transfer Protocol'} |
| `EM` | {'en_US': 'Electronic mail', 'tr_TR': 'Elektronik posta', 'ar_001': 'Electronic mail'} |
| `odemo` | {'en_US': 'Odoo Demo ID', 'tr_TR': 'Odoo Demo Kimliği', 'ar_001': 'Odoo Demo ID'} |

**`peppol_verification_state`** — {'en_US': 'Peppol status', 'tr_TR': 'Peppol status', 'ar_001': 'Peppol status'}

| Value | Label |
|---|---|
| `not_verified` | {'en_US': 'Unchecked', 'ar_001': 'غير محدد'} |
| `not_valid` | {'en_US': 'Partner is not on Peppol', 'ar_001': 'الشريك غير مسجل في Peppol'} |
| `not_valid_format` | {'en_US': 'Partner cannot receive format', 'ar_001': 'لا يمكن للشريك تلقي التنسيق'} |
| `valid` | {'en_US': 'Partner is on Peppol', 'ar_001': 'الشريك موجود على Peppol'} |

**`trust`** — {'en_US': 'Degree of trust you have in this debtor', 'tr_TR': 'Bu borçlu için güven seviyesi', 'ar_001': 'درجة ثقتكم بهذا المدين'}

| Value | Label |
|---|---|
| `good` | {'en_US': 'Good Debtor', 'tr_TR': 'İyi Borçlu', 'ar_001': 'مدين ملتزم'} |
| `normal` | {'en_US': 'Normal Debtor', 'tr_TR': 'Normal Borçlu', 'ar_001': 'المدين العادي'} |
| `bad` | {'en_US': 'Bad Debtor', 'tr_TR': 'Kötü Borçlu', 'ar_001': 'مدين معسر'} |

**`type`** — {'en_US': 'Address Type', 'tr_TR': 'Adres Türü', 'ar_001': 'نوع العنوان'}

| Value | Label |
|---|---|
| `contact` | {'en_US': 'Contact', 'tr_TR': 'Kontak', 'ar_001': 'جهة الاتصال'} |
| `invoice` | {'en_US': 'Invoice', 'tr_TR': 'Fatura', 'ar_001': 'الفاتورة'} |
| `delivery` | {'en_US': 'Delivery', 'tr_TR': 'Teslimat', 'ar_001': 'التوصيل'} |
| `facturae_ac` | {'en_US': 'FACe Center'} |
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |

**`tz`** — {'en_US': 'Timezone', 'tr_TR': 'Saat Dilimi', 'ar_001': 'المنطقة الزمنية'}

| Value | Label |
|---|---|
| `Africa/Abidjan` | {'en_US': 'Africa/Abidjan', 'tr_TR': 'Afrika/Abidjan', 'ar_001': 'Africa/Abidjan'} |
| `Africa/Accra` | {'en_US': 'Africa/Accra', 'tr_TR': 'Afrika/Accra', 'ar_001': 'Africa/Accra'} |
| `Africa/Addis_Ababa` | {'en_US': 'Africa/Addis_Ababa', 'tr_TR': 'Afrika/Addis_Ababa', 'ar_001': 'Africa/Addis_Ababa'} |
| `Africa/Algiers` | {'en_US': 'Africa/Algiers', 'tr_TR': 'Afrika/Cezayir', 'ar_001': 'Africa/Algiers'} |
| `Africa/Asmara` | {'en_US': 'Africa/Asmara', 'tr_TR': 'Afrika/Asmara', 'ar_001': 'Africa/Asmara'} |
| `Africa/Asmera` | {'en_US': 'Africa/Asmera', 'tr_TR': 'Afrika/Asmera', 'ar_001': 'Africa/Asmera'} |
| `Africa/Bamako` | {'en_US': 'Africa/Bamako', 'tr_TR': 'Afrika/Bamako', 'ar_001': 'Africa/Bamako'} |
| `Africa/Bangui` | {'en_US': 'Africa/Bangui', 'tr_TR': 'Afrika/Bangui', 'ar_001': 'Africa/Bangui'} |
| `Africa/Banjul` | {'en_US': 'Africa/Banjul', 'tr_TR': 'Afrika/Banjul', 'ar_001': 'Africa/Banjul'} |
| `Africa/Bissau` | {'en_US': 'Africa/Bissau', 'tr_TR': 'Afrika/Bissau', 'ar_001': 'Africa/Bissau'} |
| `Africa/Blantyre` | {'en_US': 'Africa/Blantyre', 'tr_TR': 'Afrika/Blantyre', 'ar_001': 'Africa/Blantyre'} |
| `Africa/Brazzaville` | {'en_US': 'Africa/Brazzaville', 'tr_TR': 'Afrika/Brazzaville', 'ar_001': 'Africa/Brazzaville'} |
| `Africa/Bujumbura` | {'en_US': 'Africa/Bujumbura', 'tr_TR': 'Afrika/Bujumbura', 'ar_001': 'Africa/Bujumbura'} |
| `Africa/Cairo` | {'en_US': 'Africa/Cairo', 'tr_TR': 'Afrika/Kahire', 'ar_001': 'Africa/Cairo'} |
| `Africa/Casablanca` | {'en_US': 'Africa/Casablanca', 'tr_TR': 'Afrika/Kazablanka', 'ar_001': 'Africa/Casablanca'} |
| `Africa/Ceuta` | {'en_US': 'Africa/Ceuta', 'tr_TR': 'Afrika/Ceuta', 'ar_001': 'Africa/Ceuta'} |
| `Africa/Conakry` | {'en_US': 'Africa/Conakry', 'tr_TR': 'Afrika/Konakri', 'ar_001': 'Africa/Conakry'} |
| `Africa/Dakar` | {'en_US': 'Africa/Dakar', 'tr_TR': 'Afrika/Dakar', 'ar_001': 'Africa/Dakar'} |
| `Africa/Dar_es_Salaam` | {'en_US': 'Africa/Dar_es_Salaam', 'tr_TR': 'Afrika/Dar_es_Salaam', 'ar_001': 'Africa/Dar_es_Salaam'} |
| `Africa/Djibouti` | {'en_US': 'Africa/Djibouti', 'tr_TR': 'Afrika/Cibuti', 'ar_001': 'Africa/Djibouti'} |
| `Africa/Douala` | {'en_US': 'Africa/Douala', 'tr_TR': 'Afrika/Douala', 'ar_001': 'Africa/Douala'} |
| `Africa/El_Aaiun` | {'en_US': 'Africa/El_Aaiun', 'tr_TR': 'Afrika/El_Aaiun', 'ar_001': 'Africa/El_Aaiun'} |
| `Africa/Freetown` | {'en_US': 'Africa/Freetown', 'tr_TR': 'Afrika/Freetown', 'ar_001': 'Africa/Freetown'} |
| `Africa/Gaborone` | {'en_US': 'Africa/Gaborone', 'tr_TR': 'Afrika/Gaborone', 'ar_001': 'Africa/Gaborone'} |
| `Africa/Harare` | {'en_US': 'Africa/Harare', 'tr_TR': 'Afrika/Harare', 'ar_001': 'Africa/Harare'} |
| `Africa/Johannesburg` | {'en_US': 'Africa/Johannesburg', 'tr_TR': 'Afrika/Johannesburg', 'ar_001': 'Africa/Johannesburg'} |
| `Africa/Juba` | {'en_US': 'Africa/Juba', 'tr_TR': 'Afrika/Cuba', 'ar_001': 'Africa/Juba'} |
| `Africa/Kampala` | {'en_US': 'Africa/Kampala', 'tr_TR': 'Afrika/Kampala', 'ar_001': 'Africa/Kampala'} |
| `Africa/Khartoum` | {'en_US': 'Africa/Khartoum', 'tr_TR': 'Afrika/Hartum', 'ar_001': 'Africa/Khartoum'} |
| `Africa/Kigali` | {'en_US': 'Africa/Kigali', 'tr_TR': 'Afrika/Kigali', 'ar_001': 'Africa/Kigali'} |
| `Africa/Kinshasa` | {'en_US': 'Africa/Kinshasa', 'tr_TR': 'Afrika/Kinşasa', 'ar_001': 'Africa/Kinshasa'} |
| `Africa/Lagos` | {'en_US': 'Africa/Lagos', 'tr_TR': 'Afrika/Lagos', 'ar_001': 'Africa/Lagos'} |
| `Africa/Libreville` | {'en_US': 'Africa/Libreville', 'tr_TR': 'Afrika/Librevil', 'ar_001': 'Africa/Libreville'} |
| `Africa/Lome` | {'en_US': 'Africa/Lome', 'tr_TR': 'Afrika/Lome', 'ar_001': 'Africa/Lome'} |
| `Africa/Luanda` | {'en_US': 'Africa/Luanda', 'tr_TR': 'Afrika/Luanda', 'ar_001': 'Africa/Luanda'} |
| `Africa/Lubumbashi` | {'en_US': 'Africa/Lubumbashi', 'tr_TR': 'Afrika/Lubumbaşi', 'ar_001': 'Africa/Lubumbashi'} |
| `Africa/Lusaka` | {'en_US': 'Africa/Lusaka', 'tr_TR': 'Afrika/Lusaka', 'ar_001': 'Africa/Lusaka'} |
| `Africa/Malabo` | {'en_US': 'Africa/Malabo', 'tr_TR': 'Afrika/Malabo', 'ar_001': 'Africa/Malabo'} |
| `Africa/Maputo` | {'en_US': 'Africa/Maputo', 'tr_TR': 'Afrika/Maputo', 'ar_001': 'Africa/Maputo'} |
| `Africa/Maseru` | {'en_US': 'Africa/Maseru', 'tr_TR': 'Afrika/Maseru', 'ar_001': 'Africa/Maseru'} |
| `Africa/Mbabane` | {'en_US': 'Africa/Mbabane', 'tr_TR': 'Afrika/Mbabane', 'ar_001': 'Africa/Mbabane'} |
| `Africa/Mogadishu` | {'en_US': 'Africa/Mogadishu', 'tr_TR': 'Afrika/Mogadişu', 'ar_001': 'Africa/Mogadishu'} |
| `Africa/Monrovia` | {'en_US': 'Africa/Monrovia', 'tr_TR': 'Afrika/Monrovia', 'ar_001': 'Africa/Monrovia'} |
| `Africa/Nairobi` | {'en_US': 'Africa/Nairobi', 'tr_TR': 'Afrika/Nairobi', 'ar_001': 'Africa/Nairobi'} |
| `Africa/Ndjamena` | {'en_US': 'Africa/Ndjamena', 'tr_TR': 'Afrika/Ndjamena', 'ar_001': 'Africa/Ndjamena'} |
| `Africa/Niamey` | {'en_US': 'Africa/Niamey', 'tr_TR': 'Afrika/Niamey', 'ar_001': 'Africa/Niamey'} |
| `Africa/Nouakchott` | {'en_US': 'Africa/Nouakchott', 'tr_TR': 'Afrika/Nuakşot', 'ar_001': 'Africa/Nouakchott'} |
| `Africa/Ouagadougou` | {'en_US': 'Africa/Ouagadougou', 'tr_TR': 'Afrika/Ouagadougou', 'ar_001': 'Africa/Ouagadougou'} |
| `Africa/Porto-Novo` | {'en_US': 'Africa/Porto-Novo', 'tr_TR': 'Afrika/Porto-Novo', 'ar_001': 'Africa/Porto-Novo'} |
| `Africa/Sao_Tome` | {'en_US': 'Africa/Sao_Tome', 'tr_TR': 'Afrika/Sao_Tome', 'ar_001': 'Africa/Sao_Tome'} |
| `Africa/Timbuktu` | {'en_US': 'Africa/Timbuktu', 'tr_TR': 'Africa/Timbuktu', 'ar_001': 'Africa/Timbuktu'} |
| `Africa/Tripoli` | {'en_US': 'Africa/Tripoli', 'tr_TR': 'Afrika/Trablus', 'ar_001': 'Africa/Tripoli'} |
| `Africa/Tunis` | {'en_US': 'Africa/Tunis', 'tr_TR': 'Afrika/Tunus', 'ar_001': 'Africa/Tunis'} |
| `Africa/Windhoek` | {'en_US': 'Africa/Windhoek', 'tr_TR': 'Afrika/Windhoek', 'ar_001': 'Africa/Windhoek'} |
| `America/Adak` | {'en_US': 'America/Adak', 'tr_TR': 'Amerika/Adak', 'ar_001': 'America/Adak'} |
| `America/Anchorage` | {'en_US': 'America/Anchorage', 'tr_TR': 'Amerika/Anchorage', 'ar_001': 'America/Anchorage'} |
| `America/Anguilla` | {'en_US': 'America/Anguilla', 'tr_TR': 'Amerika/Anguilla', 'ar_001': 'America/Anguilla'} |
| `America/Antigua` | {'en_US': 'America/Antigua', 'tr_TR': 'Amerika/Antigua', 'ar_001': 'America/Antigua'} |
| `America/Araguaina` | {'en_US': 'America/Araguaina', 'tr_TR': 'Amerika/Araguaina', 'ar_001': 'America/Araguaina'} |
| `America/Argentina/Buenos_Aires` | {'en_US': 'America/Argentina/Buenos_Aires', 'tr_TR': 'Amerika/Arjantin/Buenos_Aires', 'ar_001': 'America/Argentina/Buenos_Aires'} |
| `America/Argentina/Catamarca` | {'en_US': 'America/Argentina/Catamarca', 'tr_TR': 'Amerika/Arjantin/Catamarca', 'ar_001': 'America/Argentina/Catamarca'} |
| `America/Argentina/ComodRivadavia` | {'en_US': 'America/Argentina/ComodRivadavia', 'tr_TR': 'Amerika/Arjantin/ComodRivadavia', 'ar_001': 'America/Argentina/ComodRivadavia'} |
| `America/Argentina/Cordoba` | {'en_US': 'America/Argentina/Cordoba', 'tr_TR': 'Amerika/Arjantin/Cordoba', 'ar_001': 'America/Argentina/Cordoba'} |
| `America/Argentina/Jujuy` | {'en_US': 'America/Argentina/Jujuy', 'tr_TR': 'Amerika/Arjantin/Jujuy', 'ar_001': 'America/Argentina/Jujuy'} |
| `America/Argentina/La_Rioja` | {'en_US': 'America/Argentina/La_Rioja', 'tr_TR': 'Amerika/Arjantin/La_Rioja', 'ar_001': 'America/Argentina/La_Rioja'} |
| `America/Argentina/Mendoza` | {'en_US': 'America/Argentina/Mendoza', 'tr_TR': 'Amerika/Arjantin/Mendoza', 'ar_001': 'America/Argentina/Mendoza'} |
| `America/Argentina/Rio_Gallegos` | {'en_US': 'America/Argentina/Rio_Gallegos', 'tr_TR': 'Amerika/Arjantin/Rio_Gallegos', 'ar_001': 'America/Argentina/Rio_Gallegos'} |
| `America/Argentina/Salta` | {'en_US': 'America/Argentina/Salta', 'tr_TR': 'Amerika/Arjantin/Salta', 'ar_001': 'America/Argentina/Salta'} |
| `America/Argentina/San_Juan` | {'en_US': 'America/Argentina/San_Juan', 'tr_TR': 'Amerika/Arjantin/San_Juan', 'ar_001': 'America/Argentina/San_Juan'} |
| `America/Argentina/San_Luis` | {'en_US': 'America/Argentina/San_Luis', 'tr_TR': 'Amerika/Arjantin/San_Luis', 'ar_001': 'America/Argentina/San_Luis'} |
| `America/Argentina/Tucuman` | {'en_US': 'America/Argentina/Tucuman', 'tr_TR': 'Amerika/Arjantin/Tucuman', 'ar_001': 'America/Argentina/Tucuman'} |
| `America/Argentina/Ushuaia` | {'en_US': 'America/Argentina/Ushuaia', 'tr_TR': 'Amerika/Arjantin/Ushuaia', 'ar_001': 'America/Argentina/Ushuaia'} |
| `America/Aruba` | {'en_US': 'America/Aruba', 'tr_TR': 'Amerika/Aruba', 'ar_001': 'America/Aruba'} |
| `America/Asuncion` | {'en_US': 'America/Asuncion', 'tr_TR': 'Amerika/Asunción', 'ar_001': 'America/Asuncion'} |
| `America/Atikokan` | {'en_US': 'America/Atikokan', 'tr_TR': 'Amerika/Atikokan', 'ar_001': 'America/Atikokan'} |
| `America/Atka` | {'en_US': 'America/Atka', 'tr_TR': 'Amerika/Atka', 'ar_001': 'America/Atka'} |
| `America/Bahia` | {'en_US': 'America/Bahia', 'tr_TR': 'Amerika/Bahia', 'ar_001': 'America/Bahia'} |
| `America/Bahia_Banderas` | {'en_US': 'America/Bahia_Banderas', 'tr_TR': 'Amerika/Bahia Banderas', 'ar_001': 'America/Bahia_Banderas'} |
| `America/Barbados` | {'en_US': 'America/Barbados', 'tr_TR': 'Amerika/Barbados', 'ar_001': 'America/Barbados'} |
| `America/Belem` | {'en_US': 'America/Belem', 'tr_TR': 'Amerika/Belém', 'ar_001': 'America/Belem'} |
| `America/Belize` | {'en_US': 'America/Belize', 'tr_TR': 'Amerika/Belize', 'ar_001': 'America/Belize'} |
| `America/Blanc-Sablon` | {'en_US': 'America/Blanc-Sablon', 'tr_TR': 'Amerika/Blanc-Sablon', 'ar_001': 'America/Blanc-Sablon'} |
| `America/Boa_Vista` | {'en_US': 'America/Boa_Vista', 'tr_TR': 'Amerika/Boa Vista', 'ar_001': 'America/Boa_Vista'} |
| `America/Bogota` | {'en_US': 'America/Bogota', 'tr_TR': 'Amerika/Bogotá', 'ar_001': 'America/Bogota'} |
| `America/Boise` | {'en_US': 'America/Boise', 'tr_TR': 'Amerika/Boise', 'ar_001': 'America/Boise'} |
| `America/Buenos_Aires` | {'en_US': 'America/Buenos_Aires', 'tr_TR': 'Amerika/Buenos Aires', 'ar_001': 'America/Buenos_Aires'} |
| `America/Cambridge_Bay` | {'en_US': 'America/Cambridge_Bay', 'tr_TR': 'Amerika/Cambridge Bay', 'ar_001': 'America/Cambridge_Bay'} |
| `America/Campo_Grande` | {'en_US': 'America/Campo_Grande', 'tr_TR': 'Amerika/Campo Grande', 'ar_001': 'America/Campo_Grande'} |
| `America/Cancun` | {'en_US': 'America/Cancun', 'tr_TR': 'Amerika/Cancún', 'ar_001': 'America/Cancun'} |
| `America/Caracas` | {'en_US': 'America/Caracas', 'tr_TR': 'Amerika/Karakas', 'ar_001': 'America/Caracas'} |
| `America/Catamarca` | {'en_US': 'America/Catamarca', 'tr_TR': 'Amerika/Catamarca', 'ar_001': 'America/Catamarca'} |
| `America/Cayenne` | {'en_US': 'America/Cayenne', 'tr_TR': 'Amerika/Cayenne', 'ar_001': 'America/Cayenne'} |
| `America/Cayman` | {'en_US': 'America/Cayman', 'tr_TR': 'Amerika/Cayman', 'ar_001': 'America/Cayman'} |
| `America/Chicago` | {'en_US': 'America/Chicago', 'tr_TR': 'Amerika/Chicago', 'ar_001': 'America/Chicago'} |
| `America/Chihuahua` | {'en_US': 'America/Chihuahua', 'tr_TR': 'Amerika/Chihuahua', 'ar_001': 'America/Chihuahua'} |
| `America/Ciudad_Juarez` | {'en_US': 'America/Ciudad_Juarez', 'tr_TR': 'Amerika/Ciudad Juárez', 'ar_001': 'America/Ciudad_Juarez'} |
| `America/Coral_Harbour` | {'en_US': 'America/Coral_Harbour', 'tr_TR': 'Amerika/Coral Harbour', 'ar_001': 'America/Coral_Harbour'} |
| `America/Cordoba` | {'en_US': 'America/Cordoba', 'tr_TR': 'Amerika/Córdoba', 'ar_001': 'America/Cordoba'} |
| `America/Costa_Rica` | {'en_US': 'America/Costa_Rica', 'tr_TR': 'Amerika/Kosta Rika', 'ar_001': 'America/Costa_Rica'} |
| `America/Coyhaique` | {'en_US': 'America/Coyhaique', 'tr_TR': 'Amerika/Coyhaique', 'ar_001': 'America/Coyhaique'} |
| `America/Creston` | {'en_US': 'America/Creston', 'tr_TR': 'Amerika/Creston', 'ar_001': 'America/Creston'} |
| `America/Cuiaba` | {'en_US': 'America/Cuiaba', 'tr_TR': 'Amerika/Cuiabá', 'ar_001': 'America/Cuiaba'} |
| `America/Curacao` | {'en_US': 'America/Curacao', 'tr_TR': 'Amerika/Curaçao', 'ar_001': 'America/Curacao'} |
| `America/Danmarkshavn` | {'en_US': 'America/Danmarkshavn', 'tr_TR': 'Amerika/Danmarkshavn', 'ar_001': 'America/Danmarkshavn'} |
| `America/Dawson` | {'en_US': 'America/Dawson', 'tr_TR': 'Amerika/Dawson', 'ar_001': 'America/Dawson'} |
| `America/Dawson_Creek` | {'en_US': 'America/Dawson_Creek', 'tr_TR': 'Amerika/Dawson Creek', 'ar_001': 'America/Dawson_Creek'} |
| `America/Denver` | {'en_US': 'America/Denver', 'tr_TR': 'Amerika/Denver', 'ar_001': 'America/Denver'} |
| `America/Detroit` | {'en_US': 'America/Detroit', 'tr_TR': 'Amerika/Detroit', 'ar_001': 'America/Detroit'} |
| `America/Dominica` | {'en_US': 'America/Dominica', 'tr_TR': 'Amerika/Dominika', 'ar_001': 'America/Dominica'} |
| `America/Edmonton` | {'en_US': 'America/Edmonton', 'tr_TR': 'Amerika/Edmonton', 'ar_001': 'America/Edmonton'} |
| `America/Eirunepe` | {'en_US': 'America/Eirunepe', 'tr_TR': 'Amerika/Eirunepé', 'ar_001': 'America/Eirunepe'} |
| `America/El_Salvador` | {'en_US': 'America/El_Salvador', 'tr_TR': 'Amerika/El Salvador', 'ar_001': 'America/El_Salvador'} |
| `America/Ensenada` | {'en_US': 'America/Ensenada', 'tr_TR': 'Amerika/Ensenada', 'ar_001': 'America/Ensenada'} |
| `America/Fort_Nelson` | {'en_US': 'America/Fort_Nelson', 'tr_TR': 'Amerika/Fort Nelson', 'ar_001': 'America/Fort_Nelson'} |
| `America/Fort_Wayne` | {'en_US': 'America/Fort_Wayne', 'tr_TR': 'Amerika/Fort Wayne', 'ar_001': 'America/Fort_Wayne'} |
| `America/Fortaleza` | {'en_US': 'America/Fortaleza', 'tr_TR': 'Amerika/Fortaleza', 'ar_001': 'America/Fortaleza'} |
| `America/Glace_Bay` | {'en_US': 'America/Glace_Bay', 'tr_TR': 'Amerika/Glace Bay', 'ar_001': 'America/Glace_Bay'} |
| `America/Godthab` | {'en_US': 'America/Godthab', 'tr_TR': 'Amerika/Godthåb (Nuuk)', 'ar_001': 'America/Godthab'} |
| `America/Goose_Bay` | {'en_US': 'America/Goose_Bay', 'tr_TR': 'Amerika/Goose Bay', 'ar_001': 'America/Goose_Bay'} |
| `America/Grand_Turk` | {'en_US': 'America/Grand_Turk', 'tr_TR': 'Amerika/Grand Turk', 'ar_001': 'America/Grand_Turk'} |
| `America/Grenada` | {'en_US': 'America/Grenada', 'tr_TR': 'Amerika/Grenada', 'ar_001': 'America/Grenada'} |
| `America/Guadeloupe` | {'en_US': 'America/Guadeloupe', 'tr_TR': 'Amerika/Guadeloupe', 'ar_001': 'America/Guadeloupe'} |
| `America/Guatemala` | {'en_US': 'America/Guatemala', 'tr_TR': 'Amerika/Guatemala', 'ar_001': 'America/Guatemala'} |
| `America/Guayaquil` | {'en_US': 'America/Guayaquil', 'tr_TR': 'Amerika/Guayaquil', 'ar_001': 'America/Guayaquil'} |
| `America/Guyana` | {'en_US': 'America/Guyana', 'tr_TR': 'Amerika/Guyana', 'ar_001': 'America/Guyana'} |
| `America/Halifax` | {'en_US': 'America/Halifax', 'tr_TR': 'Amerika/Halifax', 'ar_001': 'America/Halifax'} |
| `America/Havana` | {'en_US': 'America/Havana', 'tr_TR': 'Amerika/Havana', 'ar_001': 'America/Havana'} |
| `America/Hermosillo` | {'en_US': 'America/Hermosillo', 'tr_TR': 'Amerika/Hermosillo', 'ar_001': 'America/Hermosillo'} |
| `America/Indiana/Indianapolis` | {'en_US': 'America/Indiana/Indianapolis', 'tr_TR': 'Amerika/Indiana/Indianapolis', 'ar_001': 'America/Indiana/Indianapolis'} |
| `America/Indiana/Knox` | {'en_US': 'America/Indiana/Knox', 'tr_TR': 'Amerika/Indiana/Knox', 'ar_001': 'America/Indiana/Knox'} |
| `America/Indiana/Marengo` | {'en_US': 'America/Indiana/Marengo', 'tr_TR': 'Amerika/Indiana/Marengo', 'ar_001': 'America/Indiana/Marengo'} |
| `America/Indiana/Petersburg` | {'en_US': 'America/Indiana/Petersburg', 'tr_TR': 'Amerika/Indiana/Petersburg', 'ar_001': 'America/Indiana/Petersburg'} |
| `America/Indiana/Tell_City` | {'en_US': 'America/Indiana/Tell_City', 'tr_TR': 'Amerika/Indiana/Tell_City', 'ar_001': 'America/Indiana/Tell_City'} |
| `America/Indiana/Vevay` | {'en_US': 'America/Indiana/Vevay', 'tr_TR': 'Amerika/Indiana/Vevay', 'ar_001': 'America/Indiana/Vevay'} |
| `America/Indiana/Vincennes` | {'en_US': 'America/Indiana/Vincennes', 'tr_TR': 'Amerika/Indiana/Vincennes', 'ar_001': 'America/Indiana/Vincennes'} |
| `America/Indiana/Winamac` | {'en_US': 'America/Indiana/Winamac', 'tr_TR': 'Amerika/Indiana/Winamac', 'ar_001': 'America/Indiana/Winamac'} |
| `America/Indianapolis` | {'en_US': 'America/Indianapolis', 'tr_TR': 'Amerika/Indianapolis', 'ar_001': 'America/Indianapolis'} |
| `America/Inuvik` | {'en_US': 'America/Inuvik', 'tr_TR': 'Amerika/Inuvik', 'ar_001': 'America/Inuvik'} |
| `America/Iqaluit` | {'en_US': 'America/Iqaluit', 'tr_TR': 'Amerika/Iqaluit', 'ar_001': 'America/Iqaluit'} |
| `America/Jamaica` | {'en_US': 'America/Jamaica', 'tr_TR': 'Amerika/Jamaika', 'ar_001': 'America/Jamaica'} |
| `America/Jujuy` | {'en_US': 'America/Jujuy', 'tr_TR': 'Amerika/Jujuy', 'ar_001': 'America/Jujuy'} |
| `America/Juneau` | {'en_US': 'America/Juneau', 'tr_TR': 'Amerika/Juneau', 'ar_001': 'America/Juneau'} |
| `America/Kentucky/Louisville` | {'en_US': 'America/Kentucky/Louisville', 'tr_TR': 'Amerika/Kentucky/Louisville', 'ar_001': 'America/Kentucky/Louisville'} |
| `America/Kentucky/Monticello` | {'en_US': 'America/Kentucky/Monticello', 'tr_TR': 'Amerika/Kentucky/Monticello', 'ar_001': 'America/Kentucky/Monticello'} |
| `America/Knox_IN` | {'en_US': 'America/Knox_IN', 'tr_TR': 'Amerika/Knox_IN', 'ar_001': 'America/Knox_IN'} |
| `America/Kralendijk` | {'en_US': 'America/Kralendijk', 'tr_TR': 'Amerika/Kralendijk', 'ar_001': 'America/Kralendijk'} |
| `America/La_Paz` | {'en_US': 'America/La_Paz', 'tr_TR': 'Amerika/La_Paz', 'ar_001': 'America/La_Paz'} |
| `America/Lima` | {'en_US': 'America/Lima', 'tr_TR': 'Amerika/Lima', 'ar_001': 'America/Lima'} |
| `America/Los_Angeles` | {'en_US': 'America/Los_Angeles', 'tr_TR': 'Amerika/Los_Angeles', 'ar_001': 'America/Los_Angeles'} |
| `America/Louisville` | {'en_US': 'America/Louisville', 'tr_TR': 'Amerika/Louisville', 'ar_001': 'America/Louisville'} |
| `America/Lower_Princes` | {'en_US': 'America/Lower_Princes', 'tr_TR': 'Amerika/Lower_Princes', 'ar_001': 'America/Lower_Princes'} |
| `America/Maceio` | {'en_US': 'America/Maceio', 'tr_TR': 'Amerika/Maceio', 'ar_001': 'America/Maceio'} |
| `America/Managua` | {'en_US': 'America/Managua', 'tr_TR': 'Amerika/Managua', 'ar_001': 'America/Managua'} |
| `America/Manaus` | {'en_US': 'America/Manaus', 'tr_TR': 'Amerika/Manaus', 'ar_001': 'America/Manaus'} |
| `America/Marigot` | {'en_US': 'America/Marigot', 'tr_TR': 'Amerika/Marigot', 'ar_001': 'America/Marigot'} |
| `America/Martinique` | {'en_US': 'America/Martinique', 'tr_TR': 'Amerika/Martinik', 'ar_001': 'America/Martinique'} |
| `America/Matamoros` | {'en_US': 'America/Matamoros', 'tr_TR': 'Amerika/Matamoros', 'ar_001': 'America/Matamoros'} |
| `America/Mazatlan` | {'en_US': 'America/Mazatlan', 'tr_TR': 'Amerika/Mazatlan', 'ar_001': 'America/Mazatlan'} |
| `America/Mendoza` | {'en_US': 'America/Mendoza', 'tr_TR': 'Amerika/Mendoza', 'ar_001': 'America/Mendoza'} |
| `America/Menominee` | {'en_US': 'America/Menominee', 'tr_TR': 'Amerika/Menominee', 'ar_001': 'America/Menominee'} |
| `America/Merida` | {'en_US': 'America/Merida', 'tr_TR': 'Amerika/Merida', 'ar_001': 'America/Merida'} |
| `America/Metlakatla` | {'en_US': 'America/Metlakatla', 'tr_TR': 'Amerika/Metlakatla', 'ar_001': 'America/Metlakatla'} |
| `America/Mexico_City` | {'en_US': 'America/Mexico_City', 'tr_TR': 'Amerika/Meksiko_City', 'ar_001': 'America/Mexico_City'} |
| `America/Miquelon` | {'en_US': 'America/Miquelon', 'tr_TR': 'Amerika/Miquelon', 'ar_001': 'America/Miquelon'} |
| `America/Moncton` | {'en_US': 'America/Moncton', 'tr_TR': 'Amerika/Moncton', 'ar_001': 'America/Moncton'} |
| `America/Monterrey` | {'en_US': 'America/Monterrey', 'tr_TR': 'Amerika/Monterrey', 'ar_001': 'America/Monterrey'} |
| `America/Montevideo` | {'en_US': 'America/Montevideo', 'tr_TR': 'Amerika/Montevideo', 'ar_001': 'America/Montevideo'} |
| `America/Montreal` | {'en_US': 'America/Montreal', 'tr_TR': 'Amerika/Montreal', 'ar_001': 'America/Montreal'} |
| `America/Montserrat` | {'en_US': 'America/Montserrat', 'tr_TR': 'Amerika/Montserrat', 'ar_001': 'America/Montserrat'} |
| `America/Nassau` | {'en_US': 'America/Nassau', 'tr_TR': 'Amerika/Nassau', 'ar_001': 'America/Nassau'} |
| `America/New_York` | {'en_US': 'America/New_York', 'tr_TR': 'Amerika/New York', 'ar_001': 'America/New_York'} |
| `America/Nipigon` | {'en_US': 'America/Nipigon', 'tr_TR': 'Amerika/Nipigon', 'ar_001': 'America/Nipigon'} |
| `America/Nome` | {'en_US': 'America/Nome', 'tr_TR': 'Amerika/Nome', 'ar_001': 'America/Nome'} |
| `America/Noronha` | {'en_US': 'America/Noronha', 'tr_TR': 'Amerika/Noronha', 'ar_001': 'America/Noronha'} |
| `America/North_Dakota/Beulah` | {'en_US': 'America/North_Dakota/Beulah', 'tr_TR': 'Amerika/Kuzey Dakota/Beulah', 'ar_001': 'America/North_Dakota/Beulah'} |
| `America/North_Dakota/Center` | {'en_US': 'America/North_Dakota/Center', 'tr_TR': 'Amerika/Kuzey Dakota/Merkez', 'ar_001': 'America/North_Dakota/Center'} |
| `America/North_Dakota/New_Salem` | {'en_US': 'America/North_Dakota/New_Salem', 'tr_TR': 'Amerika/Kuzey Dakota/New Salem', 'ar_001': 'America/North_Dakota/New_Salem'} |
| `America/Nuuk` | {'en_US': 'America/Nuuk', 'tr_TR': 'America/Nuuk', 'ar_001': 'America/Nuuk'} |
| `America/Ojinaga` | {'en_US': 'America/Ojinaga', 'tr_TR': 'Amerika/Ojinaga', 'ar_001': 'America/Ojinaga'} |
| `America/Panama` | {'en_US': 'America/Panama', 'tr_TR': 'Amerika/Panama', 'ar_001': 'America/Panama'} |
| `America/Pangnirtung` | {'en_US': 'America/Pangnirtung', 'tr_TR': 'Amerika/Pangnirtung', 'ar_001': 'America/Pangnirtung'} |
| `America/Paramaribo` | {'en_US': 'America/Paramaribo', 'tr_TR': 'Amerika/Paramaribo', 'ar_001': 'America/Paramaribo'} |
| `America/Phoenix` | {'en_US': 'America/Phoenix', 'tr_TR': 'Amerika/Phoenix', 'ar_001': 'America/Phoenix'} |
| `America/Port-au-Prince` | {'en_US': 'America/Port-au-Prince', 'tr_TR': 'Amerika/Port-au-Prince', 'ar_001': 'America/Port-au-Prince'} |
| `America/Port_of_Spain` | {'en_US': 'America/Port_of_Spain', 'tr_TR': 'Amerika/Port of Spain', 'ar_001': 'America/Port_of_Spain'} |
| `America/Porto_Acre` | {'en_US': 'America/Porto_Acre', 'tr_TR': 'Amerika/Porto Acre', 'ar_001': 'America/Porto_Acre'} |
| `America/Porto_Velho` | {'en_US': 'America/Porto_Velho', 'tr_TR': 'Amerika/Porto Velho', 'ar_001': 'America/Porto_Velho'} |
| `America/Puerto_Rico` | {'en_US': 'America/Puerto_Rico', 'tr_TR': 'Amerika/Porto Riko', 'ar_001': 'America/Puerto_Rico'} |
| `America/Punta_Arenas` | {'en_US': 'America/Punta_Arenas', 'tr_TR': 'Amerika/Punta Arenas', 'ar_001': 'America/Punta_Arenas'} |
| `America/Rainy_River` | {'en_US': 'America/Rainy_River', 'tr_TR': 'Amerika/Rainy River', 'ar_001': 'America/Rainy_River'} |
| `America/Rankin_Inlet` | {'en_US': 'America/Rankin_Inlet', 'tr_TR': 'Amerika/Rankin Inlet', 'ar_001': 'America/Rankin_Inlet'} |
| `America/Recife` | {'en_US': 'America/Recife', 'tr_TR': 'Amerika/Recife', 'ar_001': 'America/Recife'} |
| `America/Regina` | {'en_US': 'America/Regina', 'tr_TR': 'Amerika/Regina', 'ar_001': 'America/Regina'} |
| `America/Resolute` | {'en_US': 'America/Resolute', 'tr_TR': 'Amerika/Resolute', 'ar_001': 'America/Resolute'} |
| `America/Rio_Branco` | {'en_US': 'America/Rio_Branco', 'tr_TR': 'Amerika/Rio Branco', 'ar_001': 'America/Rio_Branco'} |
| `America/Rosario` | {'en_US': 'America/Rosario', 'tr_TR': 'Amerika/Rosario', 'ar_001': 'America/Rosario'} |
| `America/Santa_Isabel` | {'en_US': 'America/Santa_Isabel', 'tr_TR': 'Amerika/Santa Isabel', 'ar_001': 'America/Santa_Isabel'} |
| `America/Santarem` | {'en_US': 'America/Santarem', 'tr_TR': 'Amerika/Santarem', 'ar_001': 'America/Santarem'} |
| `America/Santiago` | {'en_US': 'America/Santiago', 'tr_TR': 'Amerika/Santiago', 'ar_001': 'America/Santiago'} |
| `America/Santo_Domingo` | {'en_US': 'America/Santo_Domingo', 'tr_TR': 'Amerika/Santo Domingo', 'ar_001': 'America/Santo_Domingo'} |
| `America/Sao_Paulo` | {'en_US': 'America/Sao_Paulo', 'tr_TR': 'Amerika/Sao Paulo', 'ar_001': 'America/Sao_Paulo'} |
| `America/Scoresbysund` | {'en_US': 'America/Scoresbysund', 'tr_TR': 'Amerika/Scoresbysund', 'ar_001': 'America/Scoresbysund'} |
| `America/Shiprock` | {'en_US': 'America/Shiprock', 'tr_TR': 'Amerika/Shiprock', 'ar_001': 'America/Shiprock'} |
| `America/Sitka` | {'en_US': 'America/Sitka', 'tr_TR': 'Amerika/Sitka', 'ar_001': 'America/Sitka'} |
| `America/St_Barthelemy` | {'en_US': 'America/St_Barthelemy', 'tr_TR': 'Amerika/St. Barthelemy', 'ar_001': 'America/St_Barthelemy'} |
| `America/St_Johns` | {'en_US': 'America/St_Johns', 'tr_TR': 'Amerika/St. Johns', 'ar_001': 'America/St_Johns'} |
| `America/St_Kitts` | {'en_US': 'America/St_Kitts', 'tr_TR': 'Amerika/St. Kitts', 'ar_001': 'America/St_Kitts'} |
| `America/St_Lucia` | {'en_US': 'America/St_Lucia', 'tr_TR': 'Amerika/St. Lucia', 'ar_001': 'America/St_Lucia'} |
| `America/St_Thomas` | {'en_US': 'America/St_Thomas', 'tr_TR': 'Amerika/St. Thomas', 'ar_001': 'America/St_Thomas'} |
| `America/St_Vincent` | {'en_US': 'America/St_Vincent', 'tr_TR': 'Amerika/St. Vincent', 'ar_001': 'America/St_Vincent'} |
| `America/Swift_Current` | {'en_US': 'America/Swift_Current', 'tr_TR': 'Amerika/Swift Current', 'ar_001': 'America/Swift_Current'} |
| `America/Tegucigalpa` | {'en_US': 'America/Tegucigalpa', 'tr_TR': 'Amerika/Tegucigalpa', 'ar_001': 'America/Tegucigalpa'} |
| `America/Thule` | {'en_US': 'America/Thule', 'tr_TR': 'Amerika/Thule', 'ar_001': 'America/Thule'} |
| `America/Thunder_Bay` | {'en_US': 'America/Thunder_Bay', 'tr_TR': 'Amerika/Thunder Bay', 'ar_001': 'America/Thunder_Bay'} |
| `America/Tijuana` | {'en_US': 'America/Tijuana', 'tr_TR': 'Amerika/Tijuana', 'ar_001': 'America/Tijuana'} |
| `America/Toronto` | {'en_US': 'America/Toronto', 'tr_TR': 'Amerika/Toronto', 'ar_001': 'America/Toronto'} |
| `America/Tortola` | {'en_US': 'America/Tortola', 'tr_TR': 'Amerika/Tortola', 'ar_001': 'America/Tortola'} |
| `America/Vancouver` | {'en_US': 'America/Vancouver', 'tr_TR': 'Amerika/Vancouver', 'ar_001': 'America/Vancouver'} |
| `America/Virgin` | {'en_US': 'America/Virgin', 'tr_TR': 'Amerika/Virgin', 'ar_001': 'America/Virgin'} |
| `America/Whitehorse` | {'en_US': 'America/Whitehorse', 'tr_TR': 'Amerika/Whitehorse', 'ar_001': 'America/Whitehorse'} |
| `America/Winnipeg` | {'en_US': 'America/Winnipeg', 'tr_TR': 'Amerika/Winnipeg', 'ar_001': 'America/Winnipeg'} |
| `America/Yakutat` | {'en_US': 'America/Yakutat', 'tr_TR': 'Amerika/Yakutat', 'ar_001': 'America/Yakutat'} |
| `America/Yellowknife` | {'en_US': 'America/Yellowknife', 'tr_TR': 'Amerika/Yellowknife', 'ar_001': 'America/Yellowknife'} |
| `Antarctica/Casey` | {'en_US': 'Antarctica/Casey', 'tr_TR': 'Antarktika/Casey', 'ar_001': 'Antarctica/Casey'} |
| `Antarctica/Davis` | {'en_US': 'Antarctica/Davis', 'tr_TR': 'Antarktika/Davis', 'ar_001': 'Antarctica/Davis'} |
| `Antarctica/DumontDUrville` | {'en_US': 'Antarctica/DumontDUrville', 'tr_TR': 'Antarktika/DumontDUrville', 'ar_001': 'Antarctica/DumontDUrville'} |
| `Antarctica/Macquarie` | {'en_US': 'Antarctica/Macquarie', 'tr_TR': 'Antarktika/Macquarie', 'ar_001': 'Antarctica/Macquarie'} |
| `Antarctica/Mawson` | {'en_US': 'Antarctica/Mawson', 'tr_TR': 'Antarktika/Mawson', 'ar_001': 'Antarctica/Mawson'} |
| `Antarctica/McMurdo` | {'en_US': 'Antarctica/McMurdo', 'tr_TR': 'Antarktika/McMurdo', 'ar_001': 'Antarctica/McMurdo'} |
| `Antarctica/Palmer` | {'en_US': 'Antarctica/Palmer', 'tr_TR': 'Antarktika/Palmer', 'ar_001': 'Antarctica/Palmer'} |
| `Antarctica/Rothera` | {'en_US': 'Antarctica/Rothera', 'tr_TR': 'Antarktika/Rothera', 'ar_001': 'Antarctica/Rothera'} |
| `Antarctica/South_Pole` | {'en_US': 'Antarctica/South_Pole', 'tr_TR': 'Antarktika/Güney Kutbu', 'ar_001': 'Antarctica/South_Pole'} |
| `Antarctica/Syowa` | {'en_US': 'Antarctica/Syowa', 'tr_TR': 'Antarktika/Syowa', 'ar_001': 'Antarctica/Syowa'} |
| `Antarctica/Troll` | {'en_US': 'Antarctica/Troll', 'tr_TR': 'Antarktika/Troll', 'ar_001': 'Antarctica/Troll'} |
| `Antarctica/Vostok` | {'en_US': 'Antarctica/Vostok', 'tr_TR': 'Antarktika/Vostok', 'ar_001': 'Antarctica/Vostok'} |
| `Arctic/Longyearbyen` | {'en_US': 'Arctic/Longyearbyen', 'tr_TR': 'Arktik/Longyearbyen', 'ar_001': 'Arctic/Longyearbyen'} |
| `Asia/Aden` | {'en_US': 'Asia/Aden', 'tr_TR': 'Asya/Aden', 'ar_001': 'Asia/Aden'} |
| `Asia/Almaty` | {'en_US': 'Asia/Almaty', 'tr_TR': 'Asya/Almatı', 'ar_001': 'Asia/Almaty'} |
| `Asia/Amman` | {'en_US': 'Asia/Amman', 'tr_TR': 'Asya/Amman', 'ar_001': 'Asia/Amman'} |
| `Asia/Anadyr` | {'en_US': 'Asia/Anadyr', 'tr_TR': 'Asya/Anadır', 'ar_001': 'Asia/Anadyr'} |
| `Asia/Aqtau` | {'en_US': 'Asia/Aqtau', 'tr_TR': 'Asya/Aktav', 'ar_001': 'Asia/Aqtau'} |
| `Asia/Aqtobe` | {'en_US': 'Asia/Aqtobe', 'tr_TR': 'Asya/Aktobe', 'ar_001': 'Asia/Aqtobe'} |
| `Asia/Ashgabat` | {'en_US': 'Asia/Ashgabat', 'tr_TR': 'Asya/Aşkabat', 'ar_001': 'Asia/Ashgabat'} |
| `Asia/Ashkhabad` | {'en_US': 'Asia/Ashkhabad', 'tr_TR': 'Asya/Aşkabat', 'ar_001': 'Asia/Ashkhabad'} |
| `Asia/Atyrau` | {'en_US': 'Asia/Atyrau', 'tr_TR': 'Asya/Atırav', 'ar_001': 'Asia/Atyrau'} |
| `Asia/Baghdad` | {'en_US': 'Asia/Baghdad', 'tr_TR': 'Asya/Bağdat', 'ar_001': 'Asia/Baghdad'} |
| `Asia/Bahrain` | {'en_US': 'Asia/Bahrain', 'tr_TR': 'Asya/Bahreyn', 'ar_001': 'Asia/Bahrain'} |
| `Asia/Baku` | {'en_US': 'Asia/Baku', 'tr_TR': 'Asya/Bakü', 'ar_001': 'Asia/Baku'} |
| `Asia/Bangkok` | {'en_US': 'Asia/Bangkok', 'tr_TR': 'Asya/Bangkok', 'ar_001': 'Asia/Bangkok'} |
| `Asia/Barnaul` | {'en_US': 'Asia/Barnaul', 'tr_TR': 'Asya/Barnaul', 'ar_001': 'Asia/Barnaul'} |
| `Asia/Beirut` | {'en_US': 'Asia/Beirut', 'tr_TR': 'Asya/Beyrut', 'ar_001': 'Asia/Beirut'} |
| `Asia/Bishkek` | {'en_US': 'Asia/Bishkek', 'tr_TR': 'Asya/Bişkek', 'ar_001': 'Asia/Bishkek'} |
| `Asia/Brunei` | {'en_US': 'Asia/Brunei', 'tr_TR': 'Asya/Brunei', 'ar_001': 'Asia/Brunei'} |
| `Asia/Calcutta` | {'en_US': 'Asia/Calcutta', 'tr_TR': 'Asya/Kalküta', 'ar_001': 'Asia/Calcutta'} |
| `Asia/Chita` | {'en_US': 'Asia/Chita', 'tr_TR': 'Asya/Çita', 'ar_001': 'Asia/Chita'} |
| `Asia/Choibalsan` | {'en_US': 'Asia/Choibalsan', 'tr_TR': 'Asya/Çoybalsan', 'ar_001': 'Asia/Choibalsan'} |
| `Asia/Chongqing` | {'en_US': 'Asia/Chongqing', 'tr_TR': 'Asya/Çongçing', 'ar_001': 'Asia/Chongqing'} |
| `Asia/Chungking` | {'en_US': 'Asia/Chungking', 'tr_TR': 'Asya/Çungking', 'ar_001': 'Asia/Chungking'} |
| `Asia/Colombo` | {'en_US': 'Asia/Colombo', 'tr_TR': 'Asya/Kolombo', 'ar_001': 'Asia/Colombo'} |
| `Asia/Dacca` | {'en_US': 'Asia/Dacca', 'tr_TR': 'Asya/Dakka', 'ar_001': 'Asia/Dacca'} |
| `Asia/Damascus` | {'en_US': 'Asia/Damascus', 'tr_TR': 'Asya/Şam', 'ar_001': 'Asia/Damascus'} |
| `Asia/Dhaka` | {'en_US': 'Asia/Dhaka', 'tr_TR': 'Asya/Dakka', 'ar_001': 'Asia/Dhaka'} |
| `Asia/Dili` | {'en_US': 'Asia/Dili', 'tr_TR': 'Asya/Dili', 'ar_001': 'Asia/Dili'} |
| `Asia/Dubai` | {'en_US': 'Asia/Dubai', 'tr_TR': 'Asya/Dubai', 'ar_001': 'Asia/Dubai'} |
| `Asia/Dushanbe` | {'en_US': 'Asia/Dushanbe', 'tr_TR': 'Asya/Duşanbe', 'ar_001': 'Asia/Dushanbe'} |
| `Asia/Famagusta` | {'en_US': 'Asia/Famagusta', 'tr_TR': 'Asya/Mağusa', 'ar_001': 'Asia/Famagusta'} |
| `Asia/Gaza` | {'en_US': 'Asia/Gaza', 'tr_TR': 'Asya/Gazze', 'ar_001': 'Asia/Gaza'} |
| `Asia/Harbin` | {'en_US': 'Asia/Harbin', 'tr_TR': 'Asya/Harbin', 'ar_001': 'Asia/Harbin'} |
| `Asia/Hebron` | {'en_US': 'Asia/Hebron', 'tr_TR': 'Asya/Hebron', 'ar_001': 'Asia/Hebron'} |
| `Asia/Ho_Chi_Minh` | {'en_US': 'Asia/Ho_Chi_Minh', 'tr_TR': 'Asya/Ho Şi Min', 'ar_001': 'Asia/Ho_Chi_Minh'} |
| `Asia/Hong_Kong` | {'en_US': 'Asia/Hong_Kong', 'tr_TR': 'Asya/Hong Kong', 'ar_001': 'Asia/Hong_Kong'} |
| `Asia/Hovd` | {'en_US': 'Asia/Hovd', 'tr_TR': 'Asya/Hovd', 'ar_001': 'Asia/Hovd'} |
| `Asia/Irkutsk` | {'en_US': 'Asia/Irkutsk', 'tr_TR': 'Asya/İrkutsk', 'ar_001': 'Asia/Irkutsk'} |
| `Asia/Istanbul` | {'en_US': 'Asia/Istanbul', 'tr_TR': 'Asya/İstanbul', 'ar_001': 'Asia/Istanbul'} |
| `Asia/Jakarta` | {'en_US': 'Asia/Jakarta', 'tr_TR': 'Asya/Cakarta', 'ar_001': 'Asia/Jakarta'} |
| `Asia/Jayapura` | {'en_US': 'Asia/Jayapura', 'tr_TR': 'Asya/Cayapura', 'ar_001': 'Asia/Jayapura'} |
| `Asia/Jerusalem` | {'en_US': 'Asia/Jerusalem', 'tr_TR': 'Asya/Kudüs', 'ar_001': 'Asia/Jerusalem'} |
| `Asia/Kabul` | {'en_US': 'Asia/Kabul', 'tr_TR': 'Asya/Kabil', 'ar_001': 'Asia/Kabul'} |
| `Asia/Kamchatka` | {'en_US': 'Asia/Kamchatka', 'tr_TR': 'Asya/Kamçatka', 'ar_001': 'Asia/Kamchatka'} |
| `Asia/Karachi` | {'en_US': 'Asia/Karachi', 'tr_TR': 'Asya/Karaçi', 'ar_001': 'Asia/Karachi'} |
| `Asia/Kashgar` | {'en_US': 'Asia/Kashgar', 'tr_TR': 'Asya/Kaşgar', 'ar_001': 'Asia/Kashgar'} |
| `Asia/Kathmandu` | {'en_US': 'Asia/Kathmandu', 'tr_TR': 'Asya/Katmandu', 'ar_001': 'Asia/Kathmandu'} |
| `Asia/Katmandu` | {'en_US': 'Asia/Katmandu', 'tr_TR': 'Asya/Katmandu', 'ar_001': 'Asia/Katmandu'} |
| `Asia/Khandyga` | {'en_US': 'Asia/Khandyga', 'tr_TR': 'Asya/Handıga', 'ar_001': 'Asia/Khandyga'} |
| `Asia/Kolkata` | {'en_US': 'Asia/Kolkata', 'tr_TR': 'Asya/Kalküta', 'ar_001': 'Asia/Kolkata'} |
| `Asia/Krasnoyarsk` | {'en_US': 'Asia/Krasnoyarsk', 'tr_TR': 'Asya/Krasnoyarsk', 'ar_001': 'Asia/Krasnoyarsk'} |
| `Asia/Kuala_Lumpur` | {'en_US': 'Asia/Kuala_Lumpur', 'tr_TR': 'Asya/Kuala Lumpur', 'ar_001': 'Asia/Kuala_Lumpur'} |
| `Asia/Kuching` | {'en_US': 'Asia/Kuching', 'tr_TR': 'Asya/Kuçing', 'ar_001': 'Asia/Kuching'} |
| `Asia/Kuwait` | {'en_US': 'Asia/Kuwait', 'tr_TR': 'Asya/Kuveyt', 'ar_001': 'Asia/Kuwait'} |
| `Asia/Macao` | {'en_US': 'Asia/Macao', 'tr_TR': 'Asya/Makao', 'ar_001': 'Asia/Macao'} |
| `Asia/Macau` | {'en_US': 'Asia/Macau', 'tr_TR': 'Asya/Makao', 'ar_001': 'Asia/Macau'} |
| `Asia/Magadan` | {'en_US': 'Asia/Magadan', 'tr_TR': 'Asya/Magadan', 'ar_001': 'Asia/Magadan'} |
| `Asia/Makassar` | {'en_US': 'Asia/Makassar', 'tr_TR': 'Asya/Makassar', 'ar_001': 'Asia/Makassar'} |
| `Asia/Manila` | {'en_US': 'Asia/Manila', 'tr_TR': 'Asya/Manila', 'ar_001': 'Asia/Manila'} |
| `Asia/Muscat` | {'en_US': 'Asia/Muscat', 'tr_TR': 'Asya/Maskat', 'ar_001': 'Asia/Muscat'} |
| `Asia/Nicosia` | {'en_US': 'Asia/Nicosia', 'tr_TR': 'Asya/Lefkoşa', 'ar_001': 'Asia/Nicosia'} |
| `Asia/Novokuznetsk` | {'en_US': 'Asia/Novokuznetsk', 'tr_TR': 'Asya/Novokuznetsk', 'ar_001': 'Asia/Novokuznetsk'} |
| `Asia/Novosibirsk` | {'en_US': 'Asia/Novosibirsk', 'tr_TR': 'Asya/Novosibirsk', 'ar_001': 'Asia/Novosibirsk'} |
| `Asia/Omsk` | {'en_US': 'Asia/Omsk', 'tr_TR': 'Asya/Omsk', 'ar_001': 'Asia/Omsk'} |
| `Asia/Oral` | {'en_US': 'Asia/Oral', 'tr_TR': 'Asya/Oral', 'ar_001': 'Asia/Oral'} |
| `Asia/Phnom_Penh` | {'en_US': 'Asia/Phnom_Penh', 'tr_TR': 'Asya/Phnom Penh', 'ar_001': 'Asia/Phnom_Penh'} |
| `Asia/Pontianak` | {'en_US': 'Asia/Pontianak', 'tr_TR': 'Asya/Pontianak', 'ar_001': 'Asia/Pontianak'} |
| `Asia/Pyongyang` | {'en_US': 'Asia/Pyongyang', 'tr_TR': 'Asya/Pyongyang', 'ar_001': 'Asia/Pyongyang'} |
| `Asia/Qatar` | {'en_US': 'Asia/Qatar', 'tr_TR': 'Asya/Katar', 'ar_001': 'Asia/Qatar'} |
| `Asia/Qostanay` | {'en_US': 'Asia/Qostanay', 'tr_TR': 'Asya/Kostanay', 'ar_001': 'Asia/Qostanay'} |
| `Asia/Qyzylorda` | {'en_US': 'Asia/Qyzylorda', 'tr_TR': 'Asya/Kızılorda', 'ar_001': 'Asia/Qyzylorda'} |
| `Asia/Rangoon` | {'en_US': 'Asia/Rangoon', 'tr_TR': 'Asya/Yangon', 'ar_001': 'Asia/Rangoon'} |
| `Asia/Riyadh` | {'en_US': 'Asia/Riyadh', 'tr_TR': 'Asya/Riyad', 'ar_001': 'Asia/Riyadh'} |
| `Asia/Saigon` | {'en_US': 'Asia/Saigon', 'tr_TR': 'Asya/Saygon', 'ar_001': 'Asia/Saigon'} |
| `Asia/Sakhalin` | {'en_US': 'Asia/Sakhalin', 'tr_TR': 'Asya/Sahalin', 'ar_001': 'Asia/Sakhalin'} |
| `Asia/Samarkand` | {'en_US': 'Asia/Samarkand', 'tr_TR': 'Asya/Semerkant', 'ar_001': 'Asia/Samarkand'} |
| `Asia/Seoul` | {'en_US': 'Asia/Seoul', 'tr_TR': 'Asya/Seul', 'ar_001': 'Asia/Seoul'} |
| `Asia/Shanghai` | {'en_US': 'Asia/Shanghai', 'tr_TR': 'Asya/Şangay', 'ar_001': 'Asia/Shanghai'} |
| `Asia/Singapore` | {'en_US': 'Asia/Singapore', 'tr_TR': 'Asya/Singapur', 'ar_001': 'Asia/Singapore'} |
| `Asia/Srednekolymsk` | {'en_US': 'Asia/Srednekolymsk', 'tr_TR': 'Asya/Srednekolimsk', 'ar_001': 'Asia/Srednekolymsk'} |
| `Asia/Taipei` | {'en_US': 'Asia/Taipei', 'tr_TR': 'Asya/Taipei', 'ar_001': 'Asia/Taipei'} |
| `Asia/Tashkent` | {'en_US': 'Asia/Tashkent', 'tr_TR': 'Asya/Taşkent', 'ar_001': 'Asia/Tashkent'} |
| `Asia/Tbilisi` | {'en_US': 'Asia/Tbilisi', 'tr_TR': 'Asya/Tiflis', 'ar_001': 'Asia/Tbilisi'} |
| `Asia/Tehran` | {'en_US': 'Asia/Tehran', 'tr_TR': 'AsyalTahran', 'ar_001': 'Asia/Tehran'} |
| `Asia/Tel_Aviv` | {'en_US': 'Asia/Tel_Aviv', 'tr_TR': 'Asya/Tel Aviv', 'ar_001': 'Asia/Tel_Aviv'} |
| `Asia/Thimbu` | {'en_US': 'Asia/Thimbu', 'tr_TR': 'Asya/Thimbu', 'ar_001': 'Asia/Thimbu'} |
| `Asia/Thimphu` | {'en_US': 'Asia/Thimphu', 'tr_TR': 'Asya/Thimphu', 'ar_001': 'Asia/Thimphu'} |
| `Asia/Tokyo` | {'en_US': 'Asia/Tokyo', 'tr_TR': 'Asya/Tokyo', 'ar_001': 'Asia/Tokyo'} |
| `Asia/Tomsk` | {'en_US': 'Asia/Tomsk', 'tr_TR': 'Asya/Tomsk', 'ar_001': 'Asia/Tomsk'} |
| `Asia/Ujung_Pandang` | {'en_US': 'Asia/Ujung_Pandang', 'tr_TR': 'Asya/Ujung Pandang', 'ar_001': 'Asia/Ujung_Pandang'} |
| `Asia/Ulaanbaatar` | {'en_US': 'Asia/Ulaanbaatar', 'tr_TR': 'Asya/Ulan Batur', 'ar_001': 'Asia/Ulaanbaatar'} |
| `Asia/Ulan_Bator` | {'en_US': 'Asia/Ulan_Bator', 'tr_TR': 'Asya/Ulan Batur', 'ar_001': 'Asia/Ulan_Bator'} |
| `Asia/Urumqi` | {'en_US': 'Asia/Urumqi', 'tr_TR': 'Asya/Urumçi', 'ar_001': 'Asia/Urumqi'} |
| `Asia/Ust-Nera` | {'en_US': 'Asia/Ust-Nera', 'tr_TR': 'Asya/Ust-Nera', 'ar_001': 'Asia/Ust-Nera'} |
| `Asia/Vientiane` | {'en_US': 'Asia/Vientiane', 'tr_TR': 'Asya/Vientiane', 'ar_001': 'Asia/Vientiane'} |
| `Asia/Vladivostok` | {'en_US': 'Asia/Vladivostok', 'tr_TR': 'Asya/Vladivostok', 'ar_001': 'Asia/Vladivostok'} |
| `Asia/Yakutsk` | {'en_US': 'Asia/Yakutsk', 'tr_TR': 'Asya/Yakutsk', 'ar_001': 'Asia/Yakutsk'} |
| `Asia/Yangon` | {'en_US': 'Asia/Yangon', 'tr_TR': 'Asya/Yangon', 'ar_001': 'Asia/Yangon'} |
| `Asia/Yekaterinburg` | {'en_US': 'Asia/Yekaterinburg', 'tr_TR': 'Asya/Yekaterinburg', 'ar_001': 'Asia/Yekaterinburg'} |
| `Asia/Yerevan` | {'en_US': 'Asia/Yerevan', 'tr_TR': 'Asya/Erivan', 'ar_001': 'Asia/Yerevan'} |
| `Atlantic/Azores` | {'en_US': 'Atlantic/Azores', 'tr_TR': 'Atlantik/Azorlar', 'ar_001': 'Atlantic/Azores'} |
| `Atlantic/Bermuda` | {'en_US': 'Atlantic/Bermuda', 'tr_TR': 'Atlantik/Bermuda', 'ar_001': 'Atlantic/Bermuda'} |
| `Atlantic/Canary` | {'en_US': 'Atlantic/Canary', 'tr_TR': 'Atlantik/Kanarya', 'ar_001': 'Atlantic/Canary'} |
| `Atlantic/Cape_Verde` | {'en_US': 'Atlantic/Cape_Verde', 'tr_TR': 'Atlantik/Yeşil Burun Adaları', 'ar_001': 'Atlantic/Cape_Verde'} |
| `Atlantic/Faeroe` | {'en_US': 'Atlantic/Faeroe', 'tr_TR': 'Atlantik/Faroe', 'ar_001': 'Atlantic/Faeroe'} |
| `Atlantic/Faroe` | {'en_US': 'Atlantic/Faroe', 'tr_TR': 'Atlantik/Faroe', 'ar_001': 'Atlantic/Faroe'} |
| `Atlantic/Jan_Mayen` | {'en_US': 'Atlantic/Jan_Mayen', 'tr_TR': 'Atlantik/Jan Mayen', 'ar_001': 'Atlantic/Jan_Mayen'} |
| `Atlantic/Madeira` | {'en_US': 'Atlantic/Madeira', 'tr_TR': 'Atlantik/Madeira', 'ar_001': 'Atlantic/Madeira'} |
| `Atlantic/Reykjavik` | {'en_US': 'Atlantic/Reykjavik', 'tr_TR': 'Atlantik/Reykjavik', 'ar_001': 'Atlantic/Reykjavik'} |
| `Atlantic/South_Georgia` | {'en_US': 'Atlantic/South_Georgia', 'tr_TR': 'Atlantik/Güney Georgia', 'ar_001': 'Atlantic/South_Georgia'} |
| `Atlantic/St_Helena` | {'en_US': 'Atlantic/St_Helena', 'tr_TR': 'Atlantik/St. Helena', 'ar_001': 'Atlantic/St_Helena'} |
| `Atlantic/Stanley` | {'en_US': 'Atlantic/Stanley', 'tr_TR': 'Atlantik/Stanley', 'ar_001': 'Atlantic/Stanley'} |
| `Australia/ACT` | {'en_US': 'Australia/ACT', 'tr_TR': 'Avustralya/ACT', 'ar_001': 'Australia/ACT'} |
| `Australia/Adelaide` | {'en_US': 'Australia/Adelaide', 'tr_TR': 'Avustralya/Adelaide', 'ar_001': 'Australia/Adelaide'} |
| `Australia/Brisbane` | {'en_US': 'Australia/Brisbane', 'tr_TR': 'Avustralya/Brisbane', 'ar_001': 'Australia/Brisbane'} |
| `Australia/Broken_Hill` | {'en_US': 'Australia/Broken_Hill', 'tr_TR': 'Avustralya/Broken Hill', 'ar_001': 'Australia/Broken_Hill'} |
| `Australia/Canberra` | {'en_US': 'Australia/Canberra', 'tr_TR': 'Avustralya/Kanberra', 'ar_001': 'Australia/Canberra'} |
| `Australia/Currie` | {'en_US': 'Australia/Currie', 'tr_TR': 'Avustralya/Currie', 'ar_001': 'Australia/Currie'} |
| `Australia/Darwin` | {'en_US': 'Australia/Darwin', 'tr_TR': 'Avustralya/Darwin', 'ar_001': 'Australia/Darwin'} |
| `Australia/Eucla` | {'en_US': 'Australia/Eucla', 'tr_TR': 'Avustralya/Eucla', 'ar_001': 'Australia/Eucla'} |
| `Australia/Hobart` | {'en_US': 'Australia/Hobart', 'tr_TR': 'Avustralya/Hobart', 'ar_001': 'Australia/Hobart'} |
| `Australia/LHI` | {'en_US': 'Australia/LHI', 'tr_TR': 'Avustralya/LHI', 'ar_001': 'Australia/LHI'} |
| `Australia/Lindeman` | {'en_US': 'Australia/Lindeman', 'tr_TR': 'Avustralya/Lindeman', 'ar_001': 'Australia/Lindeman'} |
| `Australia/Lord_Howe` | {'en_US': 'Australia/Lord_Howe', 'tr_TR': 'Avustralya/Lord Howe', 'ar_001': 'Australia/Lord_Howe'} |
| `Australia/Melbourne` | {'en_US': 'Australia/Melbourne', 'tr_TR': 'Avustralya/Melbourne', 'ar_001': 'Australia/Melbourne'} |
| `Australia/NSW` | {'en_US': 'Australia/NSW', 'tr_TR': 'Avustralya/NSW', 'ar_001': 'Australia/NSW'} |
| `Australia/North` | {'en_US': 'Australia/North', 'tr_TR': 'Avustralya/Kuzey', 'ar_001': 'Australia/North'} |
| `Australia/Perth` | {'en_US': 'Australia/Perth', 'tr_TR': 'Avustralya/Perth', 'ar_001': 'Australia/Perth'} |
| `Australia/Queensland` | {'en_US': 'Australia/Queensland', 'tr_TR': 'Avustralya/Queensland', 'ar_001': 'Australia/Queensland'} |
| `Australia/South` | {'en_US': 'Australia/South', 'tr_TR': 'Avustralya/Güney', 'ar_001': 'Australia/South'} |
| `Australia/Sydney` | {'en_US': 'Australia/Sydney', 'tr_TR': 'Avustralya/Sidney', 'ar_001': 'Australia/Sydney'} |
| `Australia/Tasmania` | {'en_US': 'Australia/Tasmania', 'tr_TR': 'Avustralya/Tasmanya', 'ar_001': 'Australia/Tasmania'} |
| `Australia/Victoria` | {'en_US': 'Australia/Victoria', 'tr_TR': 'Avustralya/Victoria', 'ar_001': 'Australia/Victoria'} |
| `Australia/West` | {'en_US': 'Australia/West', 'tr_TR': 'Avustralya/Batı', 'ar_001': 'Australia/West'} |
| `Australia/Yancowinna` | {'en_US': 'Australia/Yancowinna', 'tr_TR': 'Avustralya/Yancowinna', 'ar_001': 'Australia/Yancowinna'} |
| `Brazil/Acre` | {'en_US': 'Brazil/Acre', 'tr_TR': 'Brezilya/Acre', 'ar_001': 'Brazil/Acre'} |
| `Brazil/DeNoronha` | {'en_US': 'Brazil/DeNoronha', 'tr_TR': 'Brezilya/De Noronya', 'ar_001': 'Brazil/DeNoronha'} |
| `Brazil/East` | {'en_US': 'Brazil/East', 'tr_TR': 'Brezilya/Doğu', 'ar_001': 'Brazil/East'} |
| `Brazil/West` | {'en_US': 'Brazil/West', 'tr_TR': 'Brezilya/Batı', 'ar_001': 'Brazil/West'} |
| `CET` | {'en_US': 'CET', 'tr_TR': 'OZE (Orta Avrupa Saati)', 'ar_001': 'CET'} |
| `CST6CDT` | {'en_US': 'CST6CDT', 'tr_TR': 'CST6CDT', 'ar_001': 'CST6CDT'} |
| `Canada/Atlantic` | {'en_US': 'Canada/Atlantic', 'tr_TR': 'Kanada/Atlantik', 'ar_001': 'Canada/Atlantic'} |
| `Canada/Central` | {'en_US': 'Canada/Central', 'tr_TR': 'Kanada/Merkez', 'ar_001': 'Canada/Central'} |
| `Canada/Eastern` | {'en_US': 'Canada/Eastern', 'tr_TR': 'Kanada/Doğu', 'ar_001': 'Canada/Eastern'} |
| `Canada/Mountain` | {'en_US': 'Canada/Mountain', 'tr_TR': 'Kanada/Dağ', 'ar_001': 'Canada/Mountain'} |
| `Canada/Newfoundland` | {'en_US': 'Canada/Newfoundland', 'tr_TR': 'Kanada/Newfoundland', 'ar_001': 'Canada/Newfoundland'} |
| `Canada/Pacific` | {'en_US': 'Canada/Pacific', 'tr_TR': 'Kanada/Pasifik', 'ar_001': 'Canada/Pacific'} |
| `Canada/Saskatchewan` | {'en_US': 'Canada/Saskatchewan', 'tr_TR': 'Kanada/Saskatchewan', 'ar_001': 'Canada/Saskatchewan'} |
| `Canada/Yukon` | {'en_US': 'Canada/Yukon', 'tr_TR': 'Canada/Yukon', 'ar_001': 'Canada/Yukon'} |
| `Chile/Continental` | {'en_US': 'Chile/Continental', 'tr_TR': 'Şili/Kıtasal', 'ar_001': 'Chile/Continental'} |
| `Chile/EasterIsland` | {'en_US': 'Chile/EasterIsland', 'tr_TR': 'Şili/Paskalya Adası', 'ar_001': 'Chile/EasterIsland'} |
| `Cuba` | {'en_US': 'Cuba', 'tr_TR': 'Küba', 'ar_001': 'كوبا'} |
| `EET` | {'en_US': 'EET', 'tr_TR': 'EET', 'ar_001': 'EET'} |
| `EST` | {'en_US': 'EST', 'tr_TR': 'EST', 'ar_001': 'EST'} |
| `EST5EDT` | {'en_US': 'EST5EDT', 'tr_TR': 'EST5EDT', 'ar_001': 'EST5EDT'} |
| `Egypt` | {'en_US': 'Egypt', 'tr_TR': 'Mısır', 'ar_001': 'مصر'} |
| `Eire` | {'en_US': 'Eire', 'tr_TR': 'Eire', 'ar_001': 'Eire'} |
| `Europe/Amsterdam` | {'en_US': 'Europe/Amsterdam', 'tr_TR': 'Avrupa/Amsterdam', 'ar_001': 'Europe/Amsterdam'} |
| `Europe/Andorra` | {'en_US': 'Europe/Andorra', 'tr_TR': 'Avrupa/Andorra', 'ar_001': 'Europe/Andorra'} |
| `Europe/Astrakhan` | {'en_US': 'Europe/Astrakhan', 'tr_TR': 'Avrupa/Astrahan', 'ar_001': 'Europe/Astrakhan'} |
| `Europe/Athens` | {'en_US': 'Europe/Athens', 'tr_TR': 'Avrupa/Atina', 'ar_001': 'Europe/Athens'} |
| `Europe/Belfast` | {'en_US': 'Europe/Belfast', 'tr_TR': 'Avrupa/Belfast', 'ar_001': 'Europe/Belfast'} |
| `Europe/Belgrade` | {'en_US': 'Europe/Belgrade', 'tr_TR': 'Avrupa/Belgrad', 'ar_001': 'Europe/Belgrade'} |
| `Europe/Berlin` | {'en_US': 'Europe/Berlin', 'tr_TR': 'Avrupa/Berlin', 'ar_001': 'Europe/Berlin'} |
| `Europe/Bratislava` | {'en_US': 'Europe/Bratislava', 'tr_TR': 'Avrupa/Bratislava', 'ar_001': 'Europe/Bratislava'} |
| `Europe/Brussels` | {'en_US': 'Europe/Brussels', 'tr_TR': 'Avrupa/Brüksel', 'ar_001': 'Europe/Brussels'} |
| `Europe/Bucharest` | {'en_US': 'Europe/Bucharest', 'tr_TR': 'Avrupa/Bükreş', 'ar_001': 'Europe/Bucharest'} |
| `Europe/Budapest` | {'en_US': 'Europe/Budapest', 'tr_TR': 'Avrupa/Budapeşte', 'ar_001': 'Europe/Budapest'} |
| `Europe/Busingen` | {'en_US': 'Europe/Busingen', 'tr_TR': 'Avrupa/Büsingen', 'ar_001': 'Europe/Busingen'} |
| `Europe/Chisinau` | {'en_US': 'Europe/Chisinau', 'tr_TR': 'Avrupa/Kişinev', 'ar_001': 'Europe/Chisinau'} |
| `Europe/Copenhagen` | {'en_US': 'Europe/Copenhagen', 'tr_TR': 'Avrupa/Kopenhag', 'ar_001': 'Europe/Copenhagen'} |
| `Europe/Dublin` | {'en_US': 'Europe/Dublin', 'tr_TR': 'Avrupa/Dublin', 'ar_001': 'Europe/Dublin'} |
| `Europe/Gibraltar` | {'en_US': 'Europe/Gibraltar', 'tr_TR': 'Avrupa/Cebelitarık', 'ar_001': 'Europe/Gibraltar'} |
| `Europe/Guernsey` | {'en_US': 'Europe/Guernsey', 'tr_TR': 'Avrupa/Guernsey', 'ar_001': 'Europe/Guernsey'} |
| `Europe/Helsinki` | {'en_US': 'Europe/Helsinki', 'tr_TR': 'Avrupa/Helsinki', 'ar_001': 'Europe/Helsinki'} |
| `Europe/Isle_of_Man` | {'en_US': 'Europe/Isle_of_Man', 'tr_TR': 'Avrupa/Man Adası', 'ar_001': 'Europe/Isle_of_Man'} |
| `Europe/Istanbul` | {'en_US': 'Europe/Istanbul', 'tr_TR': 'Avrupa/İstanbul', 'ar_001': 'Europe/Istanbul'} |
| `Europe/Jersey` | {'en_US': 'Europe/Jersey', 'tr_TR': 'Avrupa/Jersey', 'ar_001': 'Europe/Jersey'} |
| `Europe/Kaliningrad` | {'en_US': 'Europe/Kaliningrad', 'tr_TR': 'Avrupa/Kaliningrad', 'ar_001': 'Europe/Kaliningrad'} |
| `Europe/Kiev` | {'en_US': 'Europe/Kiev', 'tr_TR': 'Avrupa/Kiev', 'ar_001': 'Europe/Kiev'} |
| `Europe/Kirov` | {'en_US': 'Europe/Kirov', 'tr_TR': 'Avrupa/Kirov', 'ar_001': 'Europe/Kirov'} |
| `Europe/Kyiv` | {'en_US': 'Europe/Kyiv', 'tr_TR': 'Avrupa/Kiev', 'ar_001': 'Europe/Kyiv'} |
| `Europe/Lisbon` | {'en_US': 'Europe/Lisbon', 'tr_TR': 'Avrupa/Lizbon', 'ar_001': 'Europe/Lisbon'} |
| `Europe/Ljubljana` | {'en_US': 'Europe/Ljubljana', 'tr_TR': 'Avrupa/Ljubljana', 'ar_001': 'Europe/Ljubljana'} |
| `Europe/London` | {'en_US': 'Europe/London', 'tr_TR': 'Avrupa/Londra', 'ar_001': 'Europe/London'} |
| `Europe/Luxembourg` | {'en_US': 'Europe/Luxembourg', 'tr_TR': 'Avrupa/Lüksemburg', 'ar_001': 'Europe/Luxembourg'} |
| `Europe/Madrid` | {'en_US': 'Europe/Madrid', 'tr_TR': 'Avrupa/Madrid', 'ar_001': 'Europe/Madrid'} |
| `Europe/Malta` | {'en_US': 'Europe/Malta', 'tr_TR': 'Avrupa/Malta', 'ar_001': 'Europe/Malta'} |
| `Europe/Mariehamn` | {'en_US': 'Europe/Mariehamn', 'tr_TR': 'Avrupa/Mariehamn', 'ar_001': 'Europe/Mariehamn'} |
| `Europe/Minsk` | {'en_US': 'Europe/Minsk', 'tr_TR': 'Avrupa/Minsk', 'ar_001': 'Europe/Minsk'} |
| `Europe/Monaco` | {'en_US': 'Europe/Monaco', 'tr_TR': 'Avrupa/Monako', 'ar_001': 'Europe/Monaco'} |
| `Europe/Moscow` | {'en_US': 'Europe/Moscow', 'tr_TR': 'Avrupa/Moskova', 'ar_001': 'Europe/Moscow'} |
| `Europe/Nicosia` | {'en_US': 'Europe/Nicosia', 'tr_TR': 'Avrupa/Lefkoşa', 'ar_001': 'Europe/Nicosia'} |
| `Europe/Oslo` | {'en_US': 'Europe/Oslo', 'tr_TR': 'Avrupa/Oslo', 'ar_001': 'Europe/Oslo'} |
| `Europe/Paris` | {'en_US': 'Europe/Paris', 'tr_TR': 'Avrupa/Paris', 'ar_001': 'Europe/Paris'} |
| `Europe/Podgorica` | {'en_US': 'Europe/Podgorica', 'tr_TR': 'Avrupa/Podgoritsa', 'ar_001': 'Europe/Podgorica'} |
| `Europe/Prague` | {'en_US': 'Europe/Prague', 'tr_TR': 'Avrupa/Prag', 'ar_001': 'Europe/Prague'} |
| `Europe/Riga` | {'en_US': 'Europe/Riga', 'tr_TR': 'Avrupa/Riga', 'ar_001': 'Europe/Riga'} |
| `Europe/Rome` | {'en_US': 'Europe/Rome', 'tr_TR': 'Avrupa/Roma', 'ar_001': 'Europe/Rome'} |
| `Europe/Samara` | {'en_US': 'Europe/Samara', 'tr_TR': 'Avrupa/Samara', 'ar_001': 'Europe/Samara'} |
| `Europe/San_Marino` | {'en_US': 'Europe/San_Marino', 'tr_TR': 'Avrupa/San Marino', 'ar_001': 'Europe/San_Marino'} |
| `Europe/Sarajevo` | {'en_US': 'Europe/Sarajevo', 'tr_TR': 'Avrupa/Saraybosna', 'ar_001': 'Europe/Sarajevo'} |
| `Europe/Saratov` | {'en_US': 'Europe/Saratov', 'tr_TR': 'Avrupa/Saratov', 'ar_001': 'Europe/Saratov'} |
| `Europe/Simferopol` | {'en_US': 'Europe/Simferopol', 'tr_TR': 'Avrupa/Simferopol', 'ar_001': 'Europe/Simferopol'} |
| `Europe/Skopje` | {'en_US': 'Europe/Skopje', 'tr_TR': 'Avrupa/Üsküp', 'ar_001': 'Europe/Skopje'} |
| `Europe/Sofia` | {'en_US': 'Europe/Sofia', 'tr_TR': 'Avrupa/Sofya', 'ar_001': 'Europe/Sofia'} |
| `Europe/Stockholm` | {'en_US': 'Europe/Stockholm', 'tr_TR': 'Avrupa/Stockholm', 'ar_001': 'Europe/Stockholm'} |
| `Europe/Tallinn` | {'en_US': 'Europe/Tallinn', 'tr_TR': 'Avrupa/Tallinn', 'ar_001': 'Europe/Tallinn'} |
| `Europe/Tirane` | {'en_US': 'Europe/Tirane', 'tr_TR': 'Avrupa/Tiran', 'ar_001': 'Europe/Tirane'} |
| `Europe/Tiraspol` | {'en_US': 'Europe/Tiraspol', 'tr_TR': 'Avrupa/Tiraspol', 'ar_001': 'Europe/Tiraspol'} |
| `Europe/Ulyanovsk` | {'en_US': 'Europe/Ulyanovsk', 'tr_TR': 'Avrupa/Ulyanovsk', 'ar_001': 'Europe/Ulyanovsk'} |
| `Europe/Uzhgorod` | {'en_US': 'Europe/Uzhgorod', 'tr_TR': 'Avrupa/Ujgorod', 'ar_001': 'Europe/Uzhgorod'} |
| `Europe/Vaduz` | {'en_US': 'Europe/Vaduz', 'tr_TR': 'Avrupa/Vaduz', 'ar_001': 'Europe/Vaduz'} |
| `Europe/Vatican` | {'en_US': 'Europe/Vatican', 'tr_TR': 'Avrupa/Vatikan', 'ar_001': 'Europe/Vatican'} |
| `Europe/Vienna` | {'en_US': 'Europe/Vienna', 'tr_TR': 'Avrupa/Viyana', 'ar_001': 'Europe/Vienna'} |
| `Europe/Vilnius` | {'en_US': 'Europe/Vilnius', 'tr_TR': 'Avrupa/Vilnius', 'ar_001': 'Europe/Vilnius'} |
| `Europe/Volgograd` | {'en_US': 'Europe/Volgograd', 'tr_TR': 'Avrupa/Volgograd', 'ar_001': 'Europe/Volgograd'} |
| `Europe/Warsaw` | {'en_US': 'Europe/Warsaw', 'tr_TR': 'Avrupa/Varşova', 'ar_001': 'Europe/Warsaw'} |
| `Europe/Zagreb` | {'en_US': 'Europe/Zagreb', 'tr_TR': 'Avrupa/Zagreb', 'ar_001': 'Europe/Zagreb'} |
| `Europe/Zaporozhye` | {'en_US': 'Europe/Zaporozhye', 'tr_TR': 'Avrupa/Zaporojya', 'ar_001': 'Europe/Zaporozhye'} |
| `Europe/Zurich` | {'en_US': 'Europe/Zurich', 'tr_TR': 'Avrupa/Zürih', 'ar_001': 'Europe/Zurich'} |
| `GB` | {'en_US': 'GB', 'tr_TR': 'GB', 'ar_001': 'GB'} |
| `GB-Eire` | {'en_US': 'GB-Eire', 'tr_TR': 'GB-Eire', 'ar_001': 'GB-Eire'} |
| `GMT` | {'en_US': 'GMT', 'tr_TR': 'GMT', 'ar_001': 'GMT'} |
| `GMT+0` | {'en_US': 'GMT+0', 'tr_TR': 'GMT+0', 'ar_001': 'GMT+0'} |
| `GMT-0` | {'en_US': 'GMT-0', 'tr_TR': 'GMT-0', 'ar_001': 'GMT-0'} |
| `GMT0` | {'en_US': 'GMT0', 'tr_TR': 'GMT0', 'ar_001': 'GMT0'} |
| `Greenwich` | {'en_US': 'Greenwich', 'tr_TR': 'Greenwich', 'ar_001': 'غرينتش'} |
| `HST` | {'en_US': 'HST', 'tr_TR': 'HST', 'ar_001': 'ضريبة المبيعات المنسقة (HST)'} |
| `Hongkong` | {'en_US': 'Hongkong', 'tr_TR': 'Hong Kong', 'ar_001': 'Hongkong'} |
| `Iceland` | {'en_US': 'Iceland', 'tr_TR': 'İzlanda', 'ar_001': 'آيسلندا'} |
| `Indian/Antananarivo` | {'en_US': 'Indian/Antananarivo', 'tr_TR': 'Hint/Antananarivo', 'ar_001': 'Indian/Antananarivo'} |
| `Indian/Chagos` | {'en_US': 'Indian/Chagos', 'tr_TR': 'Hint/Chagos', 'ar_001': 'Indian/Chagos'} |
| `Indian/Christmas` | {'en_US': 'Indian/Christmas', 'tr_TR': 'Hintli/Noel', 'ar_001': 'Indian/Christmas'} |
| `Indian/Cocos` | {'en_US': 'Indian/Cocos', 'tr_TR': 'Hint/Cocos', 'ar_001': 'Indian/Cocos'} |
| `Indian/Comoro` | {'en_US': 'Indian/Comoro', 'tr_TR': 'Hint/Komoro', 'ar_001': 'Indian/Comoro'} |
| `Indian/Kerguelen` | {'en_US': 'Indian/Kerguelen', 'tr_TR': 'Hint/Kerguelen', 'ar_001': 'Indian/Kerguelen'} |
| `Indian/Mahe` | {'en_US': 'Indian/Mahe', 'tr_TR': 'Hint/Mahe', 'ar_001': 'Indian/Mahe'} |
| `Indian/Maldives` | {'en_US': 'Indian/Maldives', 'tr_TR': 'Hint/Maldivler', 'ar_001': 'Indian/Maldives'} |
| `Indian/Mauritius` | {'en_US': 'Indian/Mauritius', 'tr_TR': 'Hint/Mauritius', 'ar_001': 'Indian/Mauritius'} |
| `Indian/Mayotte` | {'en_US': 'Indian/Mayotte', 'tr_TR': 'Hint/Mayotte', 'ar_001': 'Indian/Mayotte'} |
| `Indian/Reunion` | {'en_US': 'Indian/Reunion', 'tr_TR': 'Hint/Reunion', 'ar_001': 'Indian/Reunion'} |
| `Iran` | {'en_US': 'Iran', 'tr_TR': 'İran', 'ar_001': 'إيران'} |
| `Israel` | {'en_US': 'Israel', 'tr_TR': 'İsrail', 'ar_001': 'إسرائيل'} |
| `Jamaica` | {'en_US': 'Jamaica', 'tr_TR': 'Jamaika', 'ar_001': 'جامايكا'} |
| `Japan` | {'en_US': 'Japan', 'tr_TR': 'Japonya', 'ar_001': 'اليابان'} |
| `Kwajalein` | {'en_US': 'Kwajalein', 'tr_TR': 'Kwajalein', 'ar_001': 'Kwajalein'} |
| `Libya` | {'en_US': 'Libya', 'tr_TR': 'Libya', 'ar_001': 'ليبيا'} |
| `MET` | {'en_US': 'MET', 'tr_TR': 'MET', 'ar_001': 'MET'} |
| `MST` | {'en_US': 'MST', 'tr_TR': 'MST', 'ar_001': 'MST'} |
| `MST7MDT` | {'en_US': 'MST7MDT', 'tr_TR': 'MST7MDT', 'ar_001': 'MST7MDT'} |
| `Mexico/BajaNorte` | {'en_US': 'Mexico/BajaNorte', 'tr_TR': 'Meksika/BajaNorte', 'ar_001': 'Mexico/BajaNorte'} |
| `Mexico/BajaSur` | {'en_US': 'Mexico/BajaSur', 'tr_TR': 'Meksika/BajaSur', 'ar_001': 'Mexico/BajaSur'} |
| `Mexico/General` | {'en_US': 'Mexico/General', 'tr_TR': 'Meksika/Genel', 'ar_001': 'Mexico/General'} |
| `NZ` | {'en_US': 'NZ', 'tr_TR': 'NZ', 'ar_001': 'NZ'} |
| `NZ-CHAT` | {'en_US': 'NZ-CHAT', 'tr_TR': 'NZ-CHAT', 'ar_001': 'NZ-CHAT'} |
| `Navajo` | {'en_US': 'Navajo', 'tr_TR': 'Navajo', 'ar_001': 'Navajo'} |
| `PRC` | {'en_US': 'PRC', 'tr_TR': 'ÇHC', 'ar_001': 'PRC'} |
| `PST8PDT` | {'en_US': 'PST8PDT', 'tr_TR': 'PST8PDT', 'ar_001': 'PST8PDT'} |
| `Pacific/Apia` | {'en_US': 'Pacific/Apia', 'tr_TR': 'Pasifik/Apia', 'ar_001': 'Pacific/Apia'} |
| `Pacific/Auckland` | {'en_US': 'Pacific/Auckland', 'tr_TR': 'Pasifik/Auckland', 'ar_001': 'Pacific/Auckland'} |
| `Pacific/Bougainville` | {'en_US': 'Pacific/Bougainville', 'tr_TR': 'Pasifik/Bougainville', 'ar_001': 'Pacific/Bougainville'} |
| `Pacific/Chatham` | {'en_US': 'Pacific/Chatham', 'tr_TR': 'Pasifik/Chatham', 'ar_001': 'Pacific/Chatham'} |
| `Pacific/Chuuk` | {'en_US': 'Pacific/Chuuk', 'tr_TR': 'Pacific/Chuuk', 'ar_001': 'Pacific/Chuuk'} |
| `Pacific/Easter` | {'en_US': 'Pacific/Easter', 'tr_TR': 'Pasifik/Easter', 'ar_001': 'Pacific/Easter'} |
| `Pacific/Efate` | {'en_US': 'Pacific/Efate', 'tr_TR': 'Pasifik/Efate', 'ar_001': 'Pacific/Efate'} |
| `Pacific/Enderbury` | {'en_US': 'Pacific/Enderbury', 'tr_TR': 'Pasifik/Enderbury', 'ar_001': 'Pacific/Enderbury'} |
| `Pacific/Fakaofo` | {'en_US': 'Pacific/Fakaofo', 'tr_TR': 'Pasifik/Fakaofo', 'ar_001': 'Pacific/Fakaofo'} |
| `Pacific/Fiji` | {'en_US': 'Pacific/Fiji', 'tr_TR': 'Pasifik/Fiji', 'ar_001': 'Pacific/Fiji'} |
| `Pacific/Funafuti` | {'en_US': 'Pacific/Funafuti', 'tr_TR': 'Pasifik/Funafuti', 'ar_001': 'Pacific/Funafuti'} |
| `Pacific/Galapagos` | {'en_US': 'Pacific/Galapagos', 'tr_TR': 'Pasifik/Galapagos', 'ar_001': 'Pacific/Galapagos'} |
| `Pacific/Gambier` | {'en_US': 'Pacific/Gambier', 'tr_TR': 'Pasifik/Gambier', 'ar_001': 'Pacific/Gambier'} |
| `Pacific/Guadalcanal` | {'en_US': 'Pacific/Guadalcanal', 'tr_TR': 'Pasifik/Guadalcanal', 'ar_001': 'Pacific/Guadalcanal'} |
| `Pacific/Guam` | {'en_US': 'Pacific/Guam', 'tr_TR': 'Pasifik/Guam', 'ar_001': 'Pacific/Guam'} |
| `Pacific/Honolulu` | {'en_US': 'Pacific/Honolulu', 'tr_TR': 'Pasifik/Honolulu', 'ar_001': 'Pacific/Honolulu'} |
| `Pacific/Johnston` | {'en_US': 'Pacific/Johnston', 'tr_TR': 'Pasifik/Johnston', 'ar_001': 'Pacific/Johnston'} |
| `Pacific/Kanton` | {'en_US': 'Pacific/Kanton', 'tr_TR': 'Pasifik/Kanton', 'ar_001': 'Pacific/Kanton'} |
| `Pacific/Kiritimati` | {'en_US': 'Pacific/Kiritimati', 'tr_TR': 'Pasifik/Kiritimati', 'ar_001': 'Pacific/Kiritimati'} |
| `Pacific/Kosrae` | {'en_US': 'Pacific/Kosrae', 'tr_TR': 'Pasifik/Kosrae', 'ar_001': 'Pacific/Kosrae'} |
| `Pacific/Kwajalein` | {'en_US': 'Pacific/Kwajalein', 'tr_TR': 'Pasifik/Kwajalein', 'ar_001': 'Pacific/Kwajalein'} |
| `Pacific/Majuro` | {'en_US': 'Pacific/Majuro', 'tr_TR': 'Pasifik/Majuro', 'ar_001': 'Pacific/Majuro'} |
| `Pacific/Marquesas` | {'en_US': 'Pacific/Marquesas', 'tr_TR': 'Pasifik/Marquesas', 'ar_001': 'Pacific/Marquesas'} |
| `Pacific/Midway` | {'en_US': 'Pacific/Midway', 'tr_TR': 'Pasifik/Midway', 'ar_001': 'Pacific/Midway'} |
| `Pacific/Nauru` | {'en_US': 'Pacific/Nauru', 'tr_TR': 'Pasifik/Nauru', 'ar_001': 'Pacific/Nauru'} |
| `Pacific/Niue` | {'en_US': 'Pacific/Niue', 'tr_TR': 'Pasifik/Niue', 'ar_001': 'Pacific/Niue'} |
| `Pacific/Norfolk` | {'en_US': 'Pacific/Norfolk', 'tr_TR': 'Pasifik/Norfolk', 'ar_001': 'Pacific/Norfolk'} |
| `Pacific/Noumea` | {'en_US': 'Pacific/Noumea', 'tr_TR': 'Pasifik/Noumea', 'ar_001': 'Pacific/Noumea'} |
| `Pacific/Pago_Pago` | {'en_US': 'Pacific/Pago_Pago', 'tr_TR': 'Pasifik/Pago_Pago', 'ar_001': 'Pacific/Pago_Pago'} |
| `Pacific/Palau` | {'en_US': 'Pacific/Palau', 'tr_TR': 'Pasifik/Palau', 'ar_001': 'Pacific/Palau'} |
| `Pacific/Pitcairn` | {'en_US': 'Pacific/Pitcairn', 'tr_TR': 'Pasifik/Pitcairn', 'ar_001': 'Pacific/Pitcairn'} |
| `Pacific/Pohnpei` | {'en_US': 'Pacific/Pohnpei', 'tr_TR': 'Pasifik/Pohnpei', 'ar_001': 'Pacific/Pohnpei'} |
| `Pacific/Ponape` | {'en_US': 'Pacific/Ponape', 'tr_TR': 'Pasifik/Ponape', 'ar_001': 'Pacific/Ponape'} |
| `Pacific/Port_Moresby` | {'en_US': 'Pacific/Port_Moresby', 'tr_TR': 'Pasifik/Port_Moresby', 'ar_001': 'Pacific/Port_Moresby'} |
| `Pacific/Rarotonga` | {'en_US': 'Pacific/Rarotonga', 'tr_TR': 'Pasifik/Rarotonga', 'ar_001': 'Pacific/Rarotonga'} |
| `Pacific/Saipan` | {'en_US': 'Pacific/Saipan', 'tr_TR': 'Pasifik/Saipan', 'ar_001': 'Pacific/Saipan'} |
| `Pacific/Samoa` | {'en_US': 'Pacific/Samoa', 'tr_TR': 'Pasifik/Samoa', 'ar_001': 'Pacific/Samoa'} |
| `Pacific/Tahiti` | {'en_US': 'Pacific/Tahiti', 'tr_TR': 'Pasifik/Tahiti', 'ar_001': 'Pacific/Tahiti'} |
| `Pacific/Tarawa` | {'en_US': 'Pacific/Tarawa', 'tr_TR': 'Pasifik/Tarawa', 'ar_001': 'Pacific/Tarawa'} |
| `Pacific/Tongatapu` | {'en_US': 'Pacific/Tongatapu', 'tr_TR': 'Pasifik/Tongatapu', 'ar_001': 'Pacific/Tongatapu'} |
| `Pacific/Truk` | {'en_US': 'Pacific/Truk', 'tr_TR': 'Pacific/Truk', 'ar_001': 'Pacific/Truk'} |
| `Pacific/Wake` | {'en_US': 'Pacific/Wake', 'tr_TR': 'Pasifik/Wake', 'ar_001': 'Pacific/Wake'} |
| `Pacific/Wallis` | {'en_US': 'Pacific/Wallis', 'tr_TR': 'Pasifik/Wallis', 'ar_001': 'Pacific/Wallis'} |
| `Pacific/Yap` | {'en_US': 'Pacific/Yap', 'tr_TR': 'Pasifik/Yap', 'ar_001': 'Pacific/Yap'} |
| `Poland` | {'en_US': 'Poland', 'tr_TR': 'Polonya', 'ar_001': 'بولندا'} |
| `Portugal` | {'en_US': 'Portugal', 'tr_TR': 'Portekiz', 'ar_001': 'البرتغال'} |
| `ROC` | {'en_US': 'ROC', 'tr_TR': 'ROC', 'ar_001': 'ROC'} |
| `ROK` | {'en_US': 'ROK', 'tr_TR': 'ROK', 'ar_001': 'ROK'} |
| `Singapore` | {'en_US': 'Singapore', 'tr_TR': 'Singapur', 'ar_001': 'سنغافورة'} |
| `Turkey` | {'en_US': 'Turkey', 'tr_TR': 'Türkiye', 'ar_001': 'تركيا'} |
| `UCT` | {'en_US': 'UCT', 'tr_TR': 'UCT', 'ar_001': 'UCT'} |
| `US/Alaska` | {'en_US': 'US/Alaska', 'tr_TR': 'ABD/Alaska', 'ar_001': 'US/Alaska'} |
| `US/Aleutian` | {'en_US': 'US/Aleutian', 'tr_TR': 'ABD/Aleutian', 'ar_001': 'US/Aleutian'} |
| `US/Arizona` | {'en_US': 'US/Arizona', 'tr_TR': 'ABD/Arizona', 'ar_001': 'US/Arizona'} |
| `US/Central` | {'en_US': 'US/Central', 'tr_TR': 'ABD/Merkez', 'ar_001': 'US/Central'} |
| `US/East-Indiana` | {'en_US': 'US/East-Indiana', 'tr_TR': 'ABD/Doğu-Indiana', 'ar_001': 'US/East-Indiana'} |
| `US/Eastern` | {'en_US': 'US/Eastern', 'tr_TR': 'ABD/Doğu', 'ar_001': 'US/Eastern'} |
| `US/Hawaii` | {'en_US': 'US/Hawaii', 'tr_TR': 'ABD/Hawaii', 'ar_001': 'US/Hawaii'} |
| `US/Indiana-Starke` | {'en_US': 'US/Indiana-Starke', 'tr_TR': 'ABD/Indiana-Starke', 'ar_001': 'US/Indiana-Starke'} |
| `US/Michigan` | {'en_US': 'US/Michigan', 'tr_TR': 'ABD/Michigan', 'ar_001': 'US/Michigan'} |
| `US/Mountain` | {'en_US': 'US/Mountain', 'tr_TR': 'ABD/Dağ', 'ar_001': 'US/Mountain'} |
| `US/Pacific` | {'en_US': 'US/Pacific', 'tr_TR': 'ABD/Pasifik', 'ar_001': 'US/Pacific'} |
| `US/Samoa` | {'en_US': 'US/Samoa', 'tr_TR': 'ABD/Samoa', 'ar_001': 'US/Samoa'} |
| `UTC` | {'en_US': 'UTC', 'tr_TR': 'UTC', 'ar_001': 'التوقيت العالمي المنسق'} |
| `Universal` | {'en_US': 'Universal', 'tr_TR': 'Evrensel', 'ar_001': 'عالمي'} |
| `W-SU` | {'en_US': 'W-SU', 'tr_TR': 'W-SU', 'ar_001': 'W-SU'} |
| `WET` | {'en_US': 'WET', 'tr_TR': 'WET', 'ar_001': 'WET'} |
| `Zulu` | {'en_US': 'Zulu', 'tr_TR': 'Zulu', 'ar_001': 'Zulu'} |
| `Etc/GMT` | {'en_US': 'Etc/GMT', 'tr_TR': 'Etc/GMT', 'ar_001': 'Etc/GMT'} |
| `Etc/GMT+0` | {'en_US': 'Etc/GMT+0', 'tr_TR': 'Etc/GMT+0', 'ar_001': 'Etc/GMT+0'} |
| `Etc/GMT+1` | {'en_US': 'Etc/GMT+1', 'tr_TR': 'Etc/GMT+1', 'ar_001': 'Etc/GMT+1'} |
| `Etc/GMT+10` | {'en_US': 'Etc/GMT+10', 'tr_TR': 'Etc/GMT+10', 'ar_001': 'Etc/GMT+10'} |
| `Etc/GMT+11` | {'en_US': 'Etc/GMT+11', 'tr_TR': 'Etc/GMT+11', 'ar_001': 'Etc/GMT+11'} |
| `Etc/GMT+12` | {'en_US': 'Etc/GMT+12', 'tr_TR': 'Etc/GMT+12', 'ar_001': 'Etc/GMT+12'} |
| `Etc/GMT+2` | {'en_US': 'Etc/GMT+2', 'tr_TR': 'Etc/GMT+2', 'ar_001': 'Etc/GMT+2'} |
| `Etc/GMT+3` | {'en_US': 'Etc/GMT+3', 'tr_TR': 'Etc/GMT+3', 'ar_001': 'Etc/GMT+3'} |
| `Etc/GMT+4` | {'en_US': 'Etc/GMT+4', 'tr_TR': 'Etc/GMT+4', 'ar_001': 'Etc/GMT+4'} |
| `Etc/GMT+5` | {'en_US': 'Etc/GMT+5', 'tr_TR': 'Etc/GMT+5', 'ar_001': 'Etc/GMT+5'} |
| `Etc/GMT+6` | {'en_US': 'Etc/GMT+6', 'tr_TR': 'Etc/GMT+6', 'ar_001': 'Etc/GMT+6'} |
| `Etc/GMT+7` | {'en_US': 'Etc/GMT+7', 'tr_TR': 'Etc/GMT+7', 'ar_001': 'Etc/GMT+7'} |
| `Etc/GMT+8` | {'en_US': 'Etc/GMT+8', 'tr_TR': 'Etc/GMT+8', 'ar_001': 'Etc/GMT+8'} |
| `Etc/GMT+9` | {'en_US': 'Etc/GMT+9', 'tr_TR': 'Etc/GMT+9', 'ar_001': 'Etc/GMT+9'} |
| `Etc/GMT-0` | {'en_US': 'Etc/GMT-0', 'tr_TR': 'Etc/GMT-0', 'ar_001': 'Etc/GMT-0'} |
| `Etc/GMT-1` | {'en_US': 'Etc/GMT-1', 'tr_TR': 'Etc/GMT-1', 'ar_001': 'Etc/GMT-1'} |
| `Etc/GMT-10` | {'en_US': 'Etc/GMT-10', 'tr_TR': 'Etc/GMT-10', 'ar_001': 'Etc/GMT-10'} |
| `Etc/GMT-11` | {'en_US': 'Etc/GMT-11', 'tr_TR': 'Etc/GMT-11', 'ar_001': 'Etc/GMT-11'} |
| `Etc/GMT-12` | {'en_US': 'Etc/GMT-12', 'tr_TR': 'Etc/GMT-12', 'ar_001': 'Etc/GMT-12'} |
| `Etc/GMT-13` | {'en_US': 'Etc/GMT-13', 'tr_TR': 'Etc/GMT-13', 'ar_001': 'Etc/GMT-13'} |
| `Etc/GMT-14` | {'en_US': 'Etc/GMT-14', 'tr_TR': 'Etc/GMT-14', 'ar_001': 'Etc/GMT-14'} |
| `Etc/GMT-2` | {'en_US': 'Etc/GMT-2', 'tr_TR': 'Etc/GMT-2', 'ar_001': 'Etc/GMT-2'} |
| `Etc/GMT-3` | {'en_US': 'Etc/GMT-3', 'tr_TR': 'Etc/GMT-3', 'ar_001': 'Etc/GMT-3'} |
| `Etc/GMT-4` | {'en_US': 'Etc/GMT-4', 'tr_TR': 'Etc/GMT-4', 'ar_001': 'Etc/GMT-4'} |
| `Etc/GMT-5` | {'en_US': 'Etc/GMT-5', 'tr_TR': 'Etc/GMT-5', 'ar_001': 'Etc/GMT-5'} |
| `Etc/GMT-6` | {'en_US': 'Etc/GMT-6', 'tr_TR': 'Etc/GMT-6', 'ar_001': 'Etc/GMT-6'} |
| `Etc/GMT-7` | {'en_US': 'Etc/GMT-7', 'tr_TR': 'Etc/GMT-7', 'ar_001': 'Etc/GMT-7'} |
| `Etc/GMT-8` | {'en_US': 'Etc/GMT-8', 'tr_TR': 'Etc/GMT-8', 'ar_001': 'Etc/GMT-8'} |
| `Etc/GMT-9` | {'en_US': 'Etc/GMT-9', 'tr_TR': 'Etc/GMT-9', 'ar_001': 'Etc/GMT-9'} |
| `Etc/GMT0` | {'en_US': 'Etc/GMT0', 'tr_TR': 'Etc/GMT0', 'ar_001': 'Etc/GMT0'} |
| `Etc/Greenwich` | {'en_US': 'Etc/Greenwich', 'tr_TR': 'Etc/Greenwich', 'ar_001': 'Etc/Greenwich'} |
| `Etc/UCT` | {'en_US': 'Etc/UCT', 'tr_TR': 'Etc/UCT', 'ar_001': 'Etc/UCT'} |
| `Etc/UTC` | {'en_US': 'Etc/UTC', 'tr_TR': 'Etc/UTC', 'ar_001': 'Etc/UTC'} |
| `Etc/Universal` | {'en_US': 'Etc/Universal', 'tr_TR': 'Etc/Evrensel', 'ar_001': 'Etc/Universal'} |
| `Etc/Zulu` | {'en_US': 'Etc/Zulu', 'tr_TR': 'Etc/Zulu', 'ar_001': 'Etc/Zulu'} |

## `res.partner.bank`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`l10n_us_bank_account_type`** — {'en_US': 'Bank Account Type'}

| Value | Label |
|---|---|
| `checking` | {'en_US': 'Checking'} |
| `savings` | {'en_US': 'Savings'} |

**`proxy_type`** — {'en_US': 'Proxy Type', 'tr_TR': 'Vekil Türü', 'ar_001': 'نوع الوكيل'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |
| `id` | {'en_US': 'FPS ID'} |
| `ewallet_id` | {'en_US': 'Ewallet ID'} |
| `merchant_tax_id` | {'en_US': 'Merchant Tax ID'} |
| `email` | {'en_US': 'Email Address'} |
| `mobile` | {'en_US': 'Mobile Number'} |
| `bakong_id_solo` | {'en_US': 'Bakong Account ID (Solo Merchant)'} |
| `bakong_id_merchant` | {'en_US': 'Bakong Account ID (Corporate Merchant)'} |
| `uen` | {'en_US': 'UEN'} |
| `merchant_id` | {'en_US': 'Merchant ID'} |
| `payment_service` | {'en_US': 'Payment Service'} |
| `atm_card` | {'en_US': 'ATM Card Number'} |
| `bank_acc` | {'en_US': 'Bank Account'} |
| `br_cpf_cnpj` | {'en_US': 'CPF/CNPJ (BR)'} |
| `br_random` | {'en_US': 'Random Key (BR)'} |

## `res.users`

**`calendar_default_privacy`** — {'en_US': 'Calendar Default Privacy', 'tr_TR': 'Takvim Varsayılan Gizlilik', 'ar_001': 'الخصوصية الافتراضية للتقويم'} (not stored)

| Value | Label |
|---|---|
| `public` | {'en_US': 'Public by default', 'tr_TR': 'Varsayılan olarak halka açık', 'ar_001': 'عام بشكل افتراضي'} |
| `private` | {'en_US': 'Private by default', 'tr_TR': 'Varsayılan olarak özel', 'ar_001': 'خاص بشكل افتراضي'} |
| `confidential` | {'en_US': 'Internal users only', 'tr_TR': 'Yalnızca dahili kullanıcılar', 'ar_001': 'للمستخدمين الداخليين فقط'} |

**`manual_im_status`** — {'en_US': 'IM status manually set by the user', 'tr_TR': 'Kullanıcı tarafından manuel olarak ayarlanan IM durumu', 'ar_001': 'تم ضبط حالة المراسلة الفورية يدويًا من قِبَل المستخدم'}

| Value | Label |
|---|---|
| `away` | {'en_US': 'Away', 'tr_TR': 'Dışarıda', 'ar_001': 'بعيد'} |
| `busy` | {'en_US': 'Do Not Disturb', 'tr_TR': 'Rahatsız Etmeyin', 'ar_001': 'عدم الإزعاج'} |
| `offline` | {'en_US': 'Offline', 'tr_TR': 'Çevrim dışı', 'ar_001': 'غير متصل بالإنترنت'} |

**`notification_type`** — {'en_US': 'Notification', 'tr_TR': 'Bildirimler', 'ar_001': 'إشعار'}

| Value | Label |
|---|---|
| `email` | {'en_US': 'By Emails', 'tr_TR': 'E-postalar ile', 'ar_001': 'حسب البريد الإلكتروني'} |
| `inbox` | {'en_US': 'In Odoo', 'tr_TR': "Odoo'da", 'ar_001': 'في أودو'} |

**`odoobot_state`** — {'en_US': 'OdooBot Status', 'tr_TR': 'OdooBot Durumu', 'ar_001': 'حالة OdooBot'}

| Value | Label |
|---|---|
| `not_initialized` | {'en_US': 'Not initialized', 'tr_TR': 'Başlatılmadı', 'ar_001': 'لم يبدأ'} |
| `onboarding_emoji` | {'en_US': 'Onboarding emoji', 'tr_TR': 'Yeni başlayan emoji', 'ar_001': 'الوجه الباسم للتهيئة للعمل'} |
| `onboarding_attachement` | {'en_US': 'Onboarding attachment', 'tr_TR': 'İlk katılım eki', 'ar_001': 'مرفق التهيئة للعمل'} |
| `onboarding_command` | {'en_US': 'Onboarding command', 'tr_TR': 'Yerleştirme komutu', 'ar_001': 'أمر التهيئة للعمل'} |
| `onboarding_ping` | {'en_US': 'Onboarding ping', 'tr_TR': 'Yeni ping', 'ar_001': 'Ping التهيئة للعمل'} |
| `onboarding_canned` | {'en_US': 'Onboarding canned', 'tr_TR': 'İşe alım kaydedildi', 'ar_001': 'الرد الجاهز للتهيئة'} |
| `idle` | {'en_US': 'Idle', 'tr_TR': 'Idle', 'ar_001': 'خامل'} |
| `disabled` | {'en_US': 'Disabled', 'tr_TR': 'Devre Dışı', 'ar_001': 'معطل'} |

**`outgoing_mail_server_type`** — {'en_US': 'Outgoing Mail Server Type', 'tr_TR': 'Giden Posta Sunucusu Türü', 'ar_001': 'نوع خادم البريد الصادر'} (not stored)

| Value | Label |
|---|---|
| `default` | {'en_US': 'Default', 'tr_TR': 'Öntanımlı', 'ar_001': 'افتراضي'} |
| `gmail` | {'en_US': 'Gmail', 'tr_TR': 'Gmail', 'ar_001': 'Gmail'} |
| `outlook` | {'en_US': 'Outlook', 'tr_TR': 'Outlook', 'ar_001': 'Outlook'} |

**`role`** — {'en_US': 'Role', 'tr_TR': 'Rol', 'ar_001': 'الدور'} (not stored)

| Value | Label |
|---|---|
| `group_user` | {'en_US': 'User', 'tr_TR': 'Kullanıcı', 'ar_001': 'المستخدم'} |
| `group_system` | {'en_US': 'Administrator', 'tr_TR': 'Yönetici', 'ar_001': 'المدير'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'} (not stored)

| Value | Label |
|---|---|
| `new` | {'en_US': 'Invited', 'tr_TR': 'Davetli', 'ar_001': 'تمت دعوته'} |
| `active` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |

## `res.users.deletion`

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}

| Value | Label |
|---|---|
| `todo` | {'en_US': 'To Do', 'tr_TR': 'Yapılacak', 'ar_001': 'المهام المراد تنفيذها'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `fail` | {'en_US': 'Failed', 'tr_TR': 'Başarısız', 'ar_001': 'فشل'} |

## `res.users.identitycheck`

**`auth_method`** — {'en_US': 'Auth Method', 'tr_TR': 'Kimlik Doğrulama Yöntemi', 'ar_001': 'طريقة المصادقة'}

| Value | Label |
|---|---|
| `password` | {'en_US': 'Password', 'tr_TR': 'Parola', 'ar_001': 'كلمة المرور'} |
| `webauthn` | {'en_US': 'Passkey', 'tr_TR': 'Anahtar', 'ar_001': 'مفتاح المرور'} |

## `res.users.settings`

**`calendar_default_privacy`** — {'en_US': 'Calendar Default Privacy', 'tr_TR': 'Takvim Varsayılan Gizlilik', 'ar_001': 'الخصوصية الافتراضية للتقويم'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Public', 'tr_TR': 'Genel', 'ar_001': 'عام'} |
| `private` | {'en_US': 'Private', 'tr_TR': 'Özel', 'ar_001': 'خاص'} |
| `confidential` | {'en_US': 'Only internal users', 'tr_TR': 'Sadece dahili kullanıcılar', 'ar_001': 'المستخدمين الداخليين فقط'} |

**`channel_notifications`** — {'en_US': 'Channel Notifications', 'tr_TR': 'Kanal Bildirimleri', 'ar_001': 'إشعارات القناة'}

| Value | Label |
|---|---|
| `all` | {'en_US': 'All Messages', 'tr_TR': 'Tüm Mesajlar', 'ar_001': 'كافة الرسائل'} |
| `no_notif` | {'en_US': 'Nothing', 'tr_TR': 'Hiçbir şey', 'ar_001': 'لا شيء'} |

## `reset.view.arch.wizard`

**`reset_mode`** — {'en_US': 'Reset Mode', 'tr_TR': 'Sıfırlama Modu', 'ar_001': 'إعادة تعيين الوضع'}

| Value | Label |
|---|---|
| `soft` | {'en_US': 'Restore previous version (soft reset).', 'tr_TR': 'Önceki sürümü geri yükle (yazılımdan sıfırlama).', 'ar_001': 'استعادة النسخة السابقة (إعادة تشغيل).'} |
| `hard` | {'en_US': 'Reset to file version (hard reset).', 'tr_TR': 'Dosya sürümüne sıfırla (donanımdan sıfırlama).', 'ar_001': 'إعادة تعيين إلى نسخة الملف (إعادة الضبط حسب إعدادات المصنع).'} |
| `other_view` | {'en_US': 'Reset to another view.', 'tr_TR': 'Başka bir görünüme sıfırlayın.', 'ar_001': 'إعادة التعيين لعرض آخر.'} |

## `resource.calendar`

**`schedule_type`** — {'en_US': 'Schedule Type', 'tr_TR': 'Planlama Türü'}

| Value | Label |
|---|---|
| `flexible` | {'en_US': 'Flexible', 'tr_TR': 'Esnek', 'ar_001': 'مرن'} |
| `fully_fixed` | {'en_US': 'Fully Fixed', 'tr_TR': 'Tamamen Sabit'} |

## `resource.calendar.attendance`

**`day_period`** — {'en_US': 'Day Period', 'tr_TR': 'Gün Periyodu', 'ar_001': 'فترة اليوم'}

| Value | Label |
|---|---|
| `morning` | {'en_US': 'Morning', 'tr_TR': 'Sabah', 'ar_001': 'صباحاً'} |
| `lunch` | {'en_US': 'Break', 'tr_TR': 'Bozulan', 'ar_001': 'فاصل'} |
| `afternoon` | {'en_US': 'Afternoon', 'tr_TR': 'Öğleden Sonra', 'ar_001': 'بعد الظهر'} |
| `full_day` | {'en_US': 'Full Day', 'tr_TR': 'Tüm Gün', 'ar_001': 'يوم كامل'} |

**`dayofweek`** — {'en_US': 'Day of Week', 'tr_TR': 'Haftanın Günü', 'ar_001': 'اليوم من الأسبوع'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Monday', 'tr_TR': 'Pazartesi', 'ar_001': 'الاثنين'} |
| `1` | {'en_US': 'Tuesday', 'tr_TR': 'Salı', 'ar_001': 'الثلاثاء'} |
| `2` | {'en_US': 'Wednesday', 'tr_TR': 'Çarşamba', 'ar_001': 'الأربعاء'} |
| `3` | {'en_US': 'Thursday', 'tr_TR': 'Perşembe', 'ar_001': 'الخميس'} |
| `4` | {'en_US': 'Friday', 'tr_TR': 'Cuma', 'ar_001': 'الجمعة'} |
| `5` | {'en_US': 'Saturday', 'tr_TR': 'Cumartesi', 'ar_001': 'السبت'} |
| `6` | {'en_US': 'Sunday', 'tr_TR': 'Pazar', 'ar_001': 'الأحد'} |

**`display_type`** — {'en_US': 'Display Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع العرض'}

| Value | Label |
|---|---|
| `line_section` | {'en_US': 'Section', 'tr_TR': 'Bölüm', 'ar_001': 'القسم'} |

**`week_type`** — {'en_US': 'Week Number', 'tr_TR': 'Hafta Numarası', 'ar_001': 'رقم الأسبوع'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Second', 'tr_TR': 'Saniye', 'ar_001': 'الثاني'} |
| `0` | {'en_US': 'First', 'tr_TR': 'İlk', 'ar_001': 'الأول'} |

## `resource.calendar.leaves`

**`time_type`** — {'en_US': 'Time Type', 'tr_TR': 'Zaman Tipi', 'ar_001': 'نوع الوقت'}

| Value | Label |
|---|---|
| `leave` | {'en_US': 'Time Off', 'tr_TR': 'İzin', 'ar_001': 'الإجازات'} |
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |

## `resource.resource`

**`resource_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `user` | {'en_US': 'Human', 'tr_TR': 'İnsan', 'ar_001': 'إنسان'} |
| `material` | {'en_US': 'Material', 'tr_TR': 'Malzeme', 'ar_001': 'المادة'} |

## `restaurant.table`

**`shape`** — {'en_US': 'Shape', 'tr_TR': 'Şekil', 'ar_001': 'الشكل'}

| Value | Label |
|---|---|
| `square` | {'en_US': 'Square', 'tr_TR': 'Kare', 'ar_001': 'مربع'} |
| `round` | {'en_US': 'Round', 'tr_TR': 'Yuvarlak', 'ar_001': 'دائري'} |

## `sale.advance.payment.inv`

**`advance_payment_method`** — {'en_US': 'Create Invoice', 'tr_TR': 'Fatura Oluştur', 'ar_001': 'إنشاء فاتورة'}

| Value | Label |
|---|---|
| `delivered` | {'en_US': 'Regular invoice', 'tr_TR': 'Düzenli fatura', 'ar_001': 'فاتورة دورية'} |
| `percentage` | {'en_US': 'Down payment (percentage)', 'tr_TR': 'Peşinat (yüzde)', 'ar_001': 'دفعة مقدّمة (نسبة)'} |
| `fixed` | {'en_US': 'Down payment (fixed amount)', 'tr_TR': 'Peşinat (sabit miktar)', 'ar_001': 'دفعة مقدّمة (مبلغ ثابت)'} |

## `sale.order`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`delivery_status`** — {'en_US': 'Delivery Status', 'tr_TR': 'Teslim Durumu', 'ar_001': 'حالة التوصيل'}

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Not Delivered', 'tr_TR': 'Teslim Edilmedi', 'ar_001': 'لم يتم توصيلها'} |
| `started` | {'en_US': 'Started', 'tr_TR': 'Başladı', 'ar_001': 'بدأ'} |
| `partial` | {'en_US': 'Partially Delivered', 'tr_TR': 'Kısmen Teslim Edildi', 'ar_001': 'تم التوصيل جزئياً'} |
| `full` | {'en_US': 'Fully Delivered', 'tr_TR': 'Tam Teslim', 'ar_001': 'تم التوصيل بالكامل'} |

**`invoice_status`** — {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

**`l10n_it_origin_document_type`** — {'en_US': 'Origin Document Type'}

| Value | Label |
|---|---|
| `purchase_order` | {'en_US': 'Purchase Order'} |
| `contract` | {'en_US': 'Contract'} |
| `agreement` | {'en_US': 'Agreement'} |

**`l10n_tw_edi_carrier_type`** — {'en_US': 'Carrier Type'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Member Account'} |
| `2` | {'en_US': 'Citizen Digital Certificate'} |
| `3` | {'en_US': 'Mobile Barcode'} |
| `4` | {'en_US': 'EasyCard'} |
| `5` | {'en_US': 'iPass'} |

**`picking_policy`** — {'en_US': 'Shipping Policy', 'tr_TR': 'Ürün Teslimat Politikası', 'ar_001': 'سياسة الشحن'}

| Value | Label |
|---|---|
| `direct` | {'en_US': 'As soon as possible', 'tr_TR': 'Mümkün olduğunca kısa sürede', 'ar_001': 'في أقرب وقت ممكن'} |
| `one` | {'en_US': 'When all products are ready', 'tr_TR': 'Tüm ürünler hazır olduğunda', 'ar_001': 'عندما تكون كافة المنتجات جاهزة'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Quotation', 'tr_TR': 'Teklif', 'ar_001': 'عرض سعر'} |
| `sent` | {'en_US': 'Quotation Sent', 'tr_TR': 'Gönderilen', 'ar_001': 'تم إرسال عرض السعر'} |
| `sale` | {'en_US': 'Sales Order', 'tr_TR': 'Satış Siparişi', 'ar_001': 'أمر البيع'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `sale.order.discount`

**`discount_type`** — {'en_US': 'Discount Type', 'tr_TR': 'İndirim Türü', 'ar_001': 'نوع الخصم'}

| Value | Label |
|---|---|
| `sol_discount` | {'en_US': 'On All Order Lines', 'tr_TR': 'Tüm Satırlarda', 'ar_001': 'في كافة بنود الطلب'} |
| `so_discount` | {'en_US': 'Global Discount', 'tr_TR': 'Global İndirim', 'ar_001': 'خصم شامل'} |
| `amount` | {'en_US': 'Fixed Amount', 'tr_TR': 'Sabit Tutar', 'ar_001': 'مبلغ ثابت'} |

## `sale.order.line`

**`display_type`** — {'en_US': 'Display Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع العرض'}

| Value | Label |
|---|---|
| `line_section` | {'en_US': 'Section', 'tr_TR': 'Bölüm', 'ar_001': 'القسم'} |
| `line_subsection` | {'en_US': 'Subsection', 'tr_TR': 'Alt bölüm', 'ar_001': 'القسم الفرعي'} |
| `line_note` | {'en_US': 'Note', 'tr_TR': 'Not', 'ar_001': 'الملاحظات'} |

**`invoice_status`** — {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

**`qty_delivered_method`** — {'en_US': 'Method to update delivered qty', 'tr_TR': 'Teslim edilen Miktarı güncelleme yöntemi', 'ar_001': 'طريقة تحديث الكمية التي قد تم توصيلها'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `analytic` | {'en_US': 'Analytic From Expenses', 'tr_TR': 'Gider Analitiği', 'ar_001': 'تحليلياً من النفقات'} |
| `stock_move` | {'en_US': 'Stock Moves', 'tr_TR': 'Stok Hareketleri', 'ar_001': 'حركات المخزون'} |
| `milestones` | {'en_US': 'Milestones', 'tr_TR': 'Kilometre taşları', 'ar_001': 'مؤشرات التقدم'} |
| `timesheet` | {'en_US': 'Timesheets', 'tr_TR': 'Çalışma Çizelgeleri', 'ar_001': 'الجداول الزمنية'} |

## `sale.order.template.line`

**`display_type`** — {'en_US': 'Display Type', 'tr_TR': 'Görünüm Türü', 'ar_001': 'نوع العرض'}

| Value | Label |
|---|---|
| `line_section` | {'en_US': 'Section', 'tr_TR': 'Bölüm', 'ar_001': 'القسم'} |
| `line_subsection` | {'en_US': 'Subsection', 'tr_TR': 'Alt bölüm', 'ar_001': 'القسم الفرعي'} |
| `line_note` | {'en_US': 'Note', 'tr_TR': 'Not', 'ar_001': 'الملاحظات'} |

## `sale.pdf.form.field`

**`document_type`** — {'en_US': 'Document Type', 'tr_TR': 'Belge Tipi', 'ar_001': 'نوع المستند'}

| Value | Label |
|---|---|
| `quotation_document` | {'en_US': 'Header/Footer', 'tr_TR': 'Üstbilgi/Altbilgi', 'ar_001': 'الترويسة/التذييل'} |
| `product_document` | {'en_US': 'Product Document', 'tr_TR': 'Ürün Dokümanı', 'ar_001': 'مستند المنتج'} |

## `sale.report`

**`invoice_status`** — {'en_US': 'Order Invoice Status', 'tr_TR': 'Sipariş Fatura Durumu', 'ar_001': 'حالة فاتورة الطلب'}

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

**`line_invoice_status`** — {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Quotation', 'tr_TR': 'Teklif', 'ar_001': 'عرض سعر'} |
| `sent` | {'en_US': 'Quotation Sent', 'tr_TR': 'Gönderilen', 'ar_001': 'تم إرسال عرض السعر'} |
| `sale` | {'en_US': 'Sales Order', 'tr_TR': 'Satış Siparişi', 'ar_001': 'أمر البيع'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `invoiced` | {'en_US': 'Invoiced', 'tr_TR': 'Faturalanan', 'ar_001': 'مفوتر'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |

## `slide.channel`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`channel_type`** — {'en_US': 'Course type', 'tr_TR': 'Kurs türü', 'ar_001': 'نوع الدورة'}

| Value | Label |
|---|---|
| `training` | {'en_US': 'Training', 'tr_TR': 'Eğitim', 'ar_001': 'التدريب'} |
| `documentation` | {'en_US': 'Documentation', 'tr_TR': 'Belgeleme', 'ar_001': 'التوثيق'} |

**`enroll`** — {'en_US': 'Enroll Policy', 'tr_TR': 'Kayıt Politikası', 'ar_001': 'سياسة التسجيل'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Open', 'tr_TR': 'Açık', 'ar_001': 'فتح'} |
| `invite` | {'en_US': 'On Invitation', 'tr_TR': 'Davetiyede', 'ar_001': 'عن طريق الدعوة'} |
| `payment` | {'en_US': 'On payment', 'tr_TR': 'Ödeme üzerine', 'ar_001': 'عند الدفع'} |

**`promote_strategy`** — {'en_US': 'Featured Content', 'tr_TR': 'Öne Çıkan İçerik', 'ar_001': 'المحتوى المُبرز'}

| Value | Label |
|---|---|
| `latest` | {'en_US': 'Latest Created', 'tr_TR': 'En Son Oluşturulanlar', 'ar_001': 'الشرائح التي تم إنشاؤها مؤخراً'} |
| `most_voted` | {'en_US': 'Most Voted', 'tr_TR': 'En fazla oylanan', 'ar_001': 'الأكثر تصويتًا'} |
| `most_viewed` | {'en_US': 'Most Viewed', 'tr_TR': 'En Çok Görüntülenen', 'ar_001': 'الأكثر عرضًا'} |
| `specific` | {'en_US': 'Select Manually', 'tr_TR': 'Manuel olarak Seç', 'ar_001': 'التحديد يدوياً'} |
| `none` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |

**`rating_avg_text`** — {'en_US': 'Rating Avg Text', 'tr_TR': 'Değerlendirme Ort Metni', 'ar_001': 'متوسط نص التقييم'} (not stored)

| Value | Label |
|---|---|
| `top` | {'en_US': 'Happy'} |
| `ok` | {'en_US': 'Neutral'} |
| `ko` | {'en_US': 'Unhappy'} |
| `none` | {'en_US': 'Not Rated yet'} |

**`visibility`** — {'en_US': 'Show Course To', 'tr_TR': 'Kursu Göster:', 'ar_001': 'إظهار الدورة لـ'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Everyone', 'tr_TR': 'Herkes', 'ar_001': 'الجميع'} |
| `connected` | {'en_US': 'Signed In', 'tr_TR': 'Giriş', 'ar_001': 'تم تسجيل الدخول'} |
| `members` | {'en_US': 'Course Attendees', 'tr_TR': 'Kurs Katılımcıları', 'ar_001': 'حاضرو الدورة'} |
| `link` | {'en_US': 'Anyone with the link', 'tr_TR': 'Bağlantıya sahip olan herkes', 'ar_001': 'أي شخص لديه الرابط'} |

## `slide.channel.partner`

**`member_status`** — {'en_US': 'Attendee Status', 'tr_TR': 'Katılımcı Durumu', 'ar_001': 'حالة الحاضر'}

| Value | Label |
|---|---|
| `invited` | {'en_US': 'Invite Sent', 'tr_TR': 'Davet Gönderildi', 'ar_001': 'تم إرسال الدعوة'} |
| `joined` | {'en_US': 'Joined', 'tr_TR': 'Katıldı', 'ar_001': 'تم الانضمام'} |
| `ongoing` | {'en_US': 'Ongoing', 'tr_TR': 'Devam eden', 'ar_001': 'جاري'} |
| `completed` | {'en_US': 'Finished', 'tr_TR': 'Bitmiş', 'ar_001': 'مُنتهي'} |

## `slide.slide`

**`slide_category`** — {'en_US': 'Category', 'tr_TR': 'Kategori', 'ar_001': 'الفئة'}

| Value | Label |
|---|---|
| `infographic` | {'en_US': 'Image', 'tr_TR': 'Görsel', 'ar_001': 'صورة'} |
| `article` | {'en_US': 'Article', 'tr_TR': 'Yazı', 'ar_001': 'مقال'} |
| `document` | {'en_US': 'Document', 'tr_TR': 'Belge', 'ar_001': 'المستند'} |
| `video` | {'en_US': 'Video', 'tr_TR': 'Video', 'ar_001': 'الفيديو'} |
| `quiz` | {'en_US': 'Quiz', 'tr_TR': 'Sınav', 'ar_001': 'الاختبار القصير'} |
| `certification` | {'en_US': 'Certification', 'tr_TR': 'Sertifikasyon', 'ar_001': 'شهادة'} |

**`slide_type`** — {'en_US': 'Slide Type', 'tr_TR': 'Slayt Tipi', 'ar_001': 'نوع الشريحة'}

| Value | Label |
|---|---|
| `image` | {'en_US': 'Image', 'tr_TR': 'Görsel', 'ar_001': 'صورة'} |
| `article` | {'en_US': 'Article', 'tr_TR': 'Yazı', 'ar_001': 'مقال'} |
| `quiz` | {'en_US': 'Quiz', 'tr_TR': 'Sınav', 'ar_001': 'الاختبار القصير'} |
| `pdf` | {'en_US': 'PDF', 'tr_TR': 'PDF', 'ar_001': 'PDF'} |
| `sheet` | {'en_US': 'Sheet (Excel, Google Sheet, ...)', 'tr_TR': 'E-Tablo (Excel, Google E-Tablo, ...)', 'ar_001': 'ورقة (Excel، Google Sheet، ...)'} |
| `doc` | {'en_US': 'Document (Word, Google Doc, ...)', 'tr_TR': 'Doküman (Word, Google Doküman, ...)', 'ar_001': 'مستند (Word، Google Doc، ...)'} |
| `slides` | {'en_US': 'Slides (PowerPoint, Google Slides, ...)', 'tr_TR': 'Slaytlar (PowerPoint, Google Slaytlar, ...)', 'ar_001': 'الشرائح (PowerPoint، Google Slides، ...)'} |
| `youtube_video` | {'en_US': 'YouTube Video', 'tr_TR': 'YouTube Video', 'ar_001': 'فيديو YouTube'} |
| `google_drive_video` | {'en_US': 'Google Drive Video', 'tr_TR': 'Google Drive Video', 'ar_001': 'فيديو Google Drive'} |
| `vimeo_video` | {'en_US': 'Vimeo Video', 'tr_TR': 'Vimeo Video', 'ar_001': 'فيديو Vimeo'} |
| `certification` | {'en_US': 'Certification', 'tr_TR': 'Sertifikasyon', 'ar_001': 'شهادة'} |

**`source_type`** — {'en_US': 'Source Type', 'tr_TR': 'Kaynak Tipi', 'ar_001': 'نوع المصدر'}

| Value | Label |
|---|---|
| `local_file` | {'en_US': 'Upload from Device', 'tr_TR': 'Cihazdan Yükleme', 'ar_001': 'الرفع من الجهاز'} |
| `external` | {'en_US': 'Retrieve from Google Drive', 'tr_TR': "Google Drive'dan alma", 'ar_001': 'الإحضار من Google Drive'} |

**`video_source_type`** — {'en_US': 'Video Source', 'tr_TR': 'Video Kaynağı', 'ar_001': 'مصدر الفيديو'} (not stored)

| Value | Label |
|---|---|
| `youtube` | {'en_US': 'YouTube', 'tr_TR': 'YouTube', 'ar_001': 'YouTube'} |
| `google_drive` | {'en_US': 'Google Drive', 'tr_TR': 'Google Drive', 'ar_001': 'Google Drive'} |
| `vimeo` | {'en_US': 'Vimeo', 'tr_TR': 'Vimeo', 'ar_001': 'Vimeo'} |

## `slide.slide.resource`

**`resource_type`** — {'en_US': 'Resource Type', 'tr_TR': 'Kaynak Tipi', 'ar_001': 'نوع المورد'}

| Value | Label |
|---|---|
| `file` | {'en_US': 'File', 'tr_TR': 'Dosya', 'ar_001': 'الملف'} |
| `url` | {'en_US': 'Link', 'tr_TR': 'Bağlantı', 'ar_001': 'الرابط'} |

## `sms.composer`

**`composition_mode`** — {'en_US': 'Composition Mode', 'tr_TR': 'Kompozisyon Modu', 'ar_001': 'وضع الإنشاء'}

| Value | Label |
|---|---|
| `numbers` | {'en_US': 'Send to numbers', 'tr_TR': 'Numaralara gönder', 'ar_001': 'الإرسال إلى الأرقام'} |
| `comment` | {'en_US': 'Post on a document', 'tr_TR': 'Bir dokümanda yayınla', 'ar_001': 'النشر على مستند'} |
| `mass` | {'en_US': 'Send SMS in batch', 'tr_TR': "SMS'i toplu olarak gönder", 'ar_001': 'إرسال الرسائل النصية القصيرة على دفعات'} |

## `sms.sms`

**`failure_type`** — {'en_US': 'Failure Type', 'tr_TR': 'Başarısızlık türü', 'ar_001': 'نوع الفشل'}

| Value | Label |
|---|---|
| `unknown` | {'en_US': 'Unknown error', 'tr_TR': 'Bilinmeyen hata', 'ar_001': 'خطأ غير معروف'} |
| `sms_number_missing` | {'en_US': 'Missing Number', 'tr_TR': 'Kayıp Numara', 'ar_001': 'الرقم مفقود'} |
| `sms_number_format` | {'en_US': 'Wrong Number Format', 'tr_TR': 'Yanlış Sayı Biçimi', 'ar_001': 'صيغة الرقم خطأ'} |
| `sms_country_not_supported` | {'en_US': 'Country Not Supported', 'tr_TR': 'Ülke Desteklenmiyor', 'ar_001': 'الدولة غير مدعومة'} |
| `sms_registration_needed` | {'en_US': 'Country-specific Registration Required', 'tr_TR': 'Ülkeye Özel Kayıt Gerekli', 'ar_001': 'مطلوب التسجيل الخاص بكل بلد'} |
| `sms_credit` | {'en_US': 'Insufficient Credit', 'tr_TR': 'Yetersiz Kredi', 'ar_001': 'الرصيد غير كافٍ'} |
| `sms_server` | {'en_US': 'Server Error', 'tr_TR': 'Sunucu Hatası', 'ar_001': 'خطأ في الخادم'} |
| `sms_acc` | {'en_US': 'Unregistered Account', 'tr_TR': 'Unregistered Account', 'ar_001': 'حساب غير مسجل'} |
| `sms_blacklist` | {'en_US': 'Blacklisted', 'tr_TR': 'Kara Liste', 'ar_001': 'مدرج في القائمة السوداء'} |
| `sms_duplicate` | {'en_US': 'Duplicate', 'tr_TR': 'Kopyala', 'ar_001': 'إنشاء نسخة مطابقة'} |
| `sms_optout` | {'en_US': 'Opted Out', 'tr_TR': 'Devre Dışı Bırakıldı', 'ar_001': 'انسحب'} |
| `twilio_authentication` | {'en_US': 'Authentication Error"', 'tr_TR': 'Kimlik Doğrulama Hatası"'} |
| `twilio_callback` | {'en_US': 'Incorrect callback URL', 'tr_TR': "Yanlış geri arama URL'si"} |
| `twilio_from_missing` | {'en_US': 'Missing From Number', 'tr_TR': 'Numaradan Eksik'} |
| `twilio_from_to` | {'en_US': 'From / To identic', 'tr_TR': 'Kimden / Kime özdeş'} |

**`state`** — {'en_US': 'SMS Status', 'tr_TR': 'SMS Durumu', 'ar_001': 'حالة الرسائل النصية القصيرة'}

| Value | Label |
|---|---|
| `outgoing` | {'en_US': 'In Queue', 'tr_TR': 'Sırada', 'ar_001': 'في قائمة الانتظار'} |
| `process` | {'en_US': 'Processing', 'tr_TR': 'İşleniyor', 'ar_001': 'معالجة'} |
| `pending` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `sent` | {'en_US': 'Delivered', 'tr_TR': 'Teslim Edilen', 'ar_001': 'تم التوصيل'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `snailmail.letter`

**`error_code`** — {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'}

| Value | Label |
|---|---|
| `MISSING_REQUIRED_FIELDS` | {'en_US': 'MISSING_REQUIRED_FIELDS', 'tr_TR': 'MISSING_REQUIRED_FIELDS', 'ar_001': 'MISSING_REQUIRED_FIELDS'} |
| `CREDIT_ERROR` | {'en_US': 'CREDIT_ERROR', 'tr_TR': 'CREDIT_ERROR', 'ar_001': 'CREDIT_ERROR'} |
| `TRIAL_ERROR` | {'en_US': 'TRIAL_ERROR', 'tr_TR': 'TRIAL_ERROR', 'ar_001': 'TRIAL_ERROR'} |
| `NO_PRICE_AVAILABLE` | {'en_US': 'NO_PRICE_AVAILABLE', 'tr_TR': 'NO_PRICE_AVAILABLE', 'ar_001': 'NO_PRICE_AVAILABLE'} |
| `FORMAT_ERROR` | {'en_US': 'FORMAT_ERROR', 'tr_TR': 'FORMAT_ERROR', 'ar_001': 'FORMAT_ERROR'} |
| `UNKNOWN_ERROR` | {'en_US': 'UNKNOWN_ERROR', 'tr_TR': 'UNKNOWN_ERROR', 'ar_001': 'UNKNOWN_ERROR'} |
| `ATTACHMENT_ERROR` | {'en_US': 'ATTACHMENT_ERROR', 'tr_TR': 'EK HATASI', 'ar_001': 'ATTACHMENT_ERROR'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `pending` | {'en_US': 'In Queue', 'tr_TR': 'Sırada', 'ar_001': 'في قائمة الانتظار'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `sparse_fields.test`

**`selection`** — {'en_US': 'Selection', 'tr_TR': 'Seçim', 'ar_001': 'قائمة خيارات'} (not stored)

| Value | Label |
|---|---|
| `one` | {'en_US': 'One', 'tr_TR': 'Bir', 'ar_001': 'واحد'} |
| `two` | {'en_US': 'Two', 'tr_TR': 'İki', 'ar_001': 'اثنان'} |

## `stock.add.to.wave`

**`mode`** — {'en_US': 'Mode', 'tr_TR': 'Şekli', 'ar_001': 'الوضع'}

| Value | Label |
|---|---|
| `existing` | {'en_US': 'an existing wave transfer', 'tr_TR': 'mevcut bir grup transfer', 'ar_001': 'عملية جمع العمليات من شحنات مختلفة موجودة بالفعل'} |
| `new` | {'en_US': 'a new wave transfer', 'tr_TR': 'yeni bir grup transfer', 'ar_001': 'عملية جمع العمليات من شحنات مختلفة'} |

## `stock.avco.report`

**`res_model_name`** — {'en_US': 'Resource Model Name', 'tr_TR': 'Kaynak Model Adı'}

| Value | Label |
|---|---|
| `stock.move` | {'en_US': 'Stock Move', 'tr_TR': 'Stok Hareketi', 'ar_001': 'حركة المخزون'} |
| `product.value` | {'en_US': 'Product Value', 'tr_TR': 'Ürün Değeri'} |

## `stock.landed.cost`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert'} |
| `danger` | {'en_US': 'Error'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'İşlenmiş', 'ar_001': 'مُرحّل'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

**`target_model`** — {'en_US': 'Apply On', 'tr_TR': 'Şuna Uygula', 'ar_001': 'التطبيق على'}

| Value | Label |
|---|---|
| `picking` | {'en_US': 'Transfers', 'tr_TR': 'Transferler', 'ar_001': 'التحويلات'} |
| `manufacturing` | {'en_US': 'Manufacturing Orders', 'tr_TR': 'Üretim Emirleri', 'ar_001': 'أوامر التصنيع'} |

## `stock.landed.cost.lines`

**`split_method`** — {'en_US': 'Split Method', 'tr_TR': 'Bölme Yöntemi', 'ar_001': 'طريقة التقسيم'}

| Value | Label |
|---|---|
| `equal` | {'en_US': 'Equal', 'tr_TR': 'Eşit', 'ar_001': 'بالتساوي'} |
| `by_quantity` | {'en_US': 'By Quantity', 'tr_TR': 'Miktara Göre', 'ar_001': 'حسب الكمية'} |
| `by_current_cost_price` | {'en_US': 'By Current Cost', 'tr_TR': 'Mevcut Maliyete Göre', 'ar_001': 'حسب التكلفة الحالية'} |
| `by_weight` | {'en_US': 'By Weight', 'tr_TR': 'Ağırlığa Göre', 'ar_001': 'حسب الوزن'} |
| `by_volume` | {'en_US': 'By Volume', 'tr_TR': 'Hacime Göre', 'ar_001': 'حسب الحجم'} |

## `stock.location`

**`usage`** — {'en_US': 'Location Type', 'tr_TR': 'Konum Türü', 'ar_001': 'نوع الموقع'}

| Value | Label |
|---|---|
| `supplier` | {'en_US': 'Vendor', 'tr_TR': 'Tedarikçi', 'ar_001': 'المورد'} |
| `view` | {'en_US': 'Virtual', 'tr_TR': 'Sanal', 'ar_001': 'افتراضي'} |
| `internal` | {'en_US': 'Internal', 'tr_TR': 'İç', 'ar_001': 'داخلي'} |
| `customer` | {'en_US': 'Customer', 'tr_TR': 'Müşteri', 'ar_001': 'العميل'} |
| `inventory` | {'en_US': 'Inventory Loss', 'tr_TR': 'Envanter Zararı', 'ar_001': 'خسارة المخزون'} |
| `production` | {'en_US': 'Production', 'tr_TR': 'Üretim', 'ar_001': 'الإنتاج'} |
| `transit` | {'en_US': 'Transit', 'tr_TR': 'Transit', 'ar_001': 'العبور'} |

## `stock.lot`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

## `stock.move`

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `1` | {'en_US': 'Urgent', 'tr_TR': 'Acil', 'ar_001': 'عاجل'} |

**`procure_method`** — {'en_US': 'Supply Method', 'tr_TR': 'Tedarik Yöntemi', 'ar_001': 'طريقة  التزويد'}

| Value | Label |
|---|---|
| `make_to_stock` | {'en_US': 'Default: Take From Stock', 'tr_TR': 'Öntanımlı: Stok Alım Konumu', 'ar_001': 'الافتراضي : الأخذ من المخزون'} |
| `make_to_order` | {'en_US': 'Advanced: Apply Procurement Rules', 'tr_TR': 'Gelişmiş: Tedarik Kuralları Uygula', 'ar_001': 'متقدم: تطبيق قواعد الشراء'} |

**`repair_line_type`** — {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}

| Value | Label |
|---|---|
| `add` | {'en_US': 'Add', 'tr_TR': 'Ekle', 'ar_001': 'إضافة'} |
| `remove` | {'en_US': 'Remove', 'tr_TR': 'Kaldır', 'ar_001': 'إزالة'} |
| `recycle` | {'en_US': 'Recycle', 'tr_TR': 'Geri Dönüşüm', 'ar_001': 'إعادة التدوير'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `waiting` | {'en_US': 'Waiting Another Move', 'tr_TR': 'Başka Bir Hareketi Bekliyor', 'ar_001': 'في انتظار حركة أخرى'} |
| `confirmed` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `partially_available` | {'en_US': 'Partially Available', 'tr_TR': 'Kısmi Uygunluk', 'ar_001': 'متوفر جزئياً'} |
| `assigned` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `stock.orderpoint.snooze`

**`predefined_date`** — {'en_US': 'Snooze for', 'tr_TR': 'Ertele', 'ar_001': 'التأجيل لـ'}

| Value | Label |
|---|---|
| `day` | {'en_US': '1 Day', 'tr_TR': '1 Gün', 'ar_001': 'يوم 1'} |
| `week` | {'en_US': '1 Week', 'tr_TR': '1 Hafta', 'ar_001': 'أسبوع 1'} |
| `month` | {'en_US': '1 Month', 'tr_TR': '1 Ay', 'ar_001': 'شهر 1'} |
| `custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |

## `stock.package.type`

**`package_carrier_type`** — {'en_US': 'Carrier', 'tr_TR': 'Nakliyeci', 'ar_001': 'شركة الشحن'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'No carrier integration', 'tr_TR': 'Nakliyeci entegrasyonu yok', 'ar_001': 'لم يتم اختيار شركة شحن'} |

**`package_use`** — {'en_US': 'Package Use', 'tr_TR': 'Paket Kullanımı', 'ar_001': 'استخدام الطرد'}

| Value | Label |
|---|---|
| `disposable` | {'en_US': 'Disposable Box', 'tr_TR': 'İmha edilebilir Kutu', 'ar_001': 'صندوق للاستخدام مرة واحدة'} |
| `reusable` | {'en_US': 'Reusable Box (totes)', 'tr_TR': 'Yeniden Kullanılabilir Kutu (totes)'} |

## `stock.picking`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`l10n_it_transport_method`** — {'en_US': 'Transport Method'}

| Value | Label |
|---|---|
| `sender` | {'en_US': 'Sender'} |
| `recipient` | {'en_US': 'Recipient'} |
| `courier` | {'en_US': 'Courier service'} |

**`l10n_it_transport_reason`** — {'en_US': 'Transport Reason'}

| Value | Label |
|---|---|
| `sale` | {'en_US': 'Sale'} |
| `outsourcing` | {'en_US': 'Outsourcing'} |
| `evaluation` | {'en_US': 'Evaluation'} |
| `gift` | {'en_US': 'Gift'} |
| `transfer` | {'en_US': 'Transfer'} |
| `substitution` | {'en_US': 'Returned goods'} |
| `attemped_sale` | {'en_US': 'Attempted Sale'} |
| `loaned_use` | {'en_US': 'Loaned for Use'} |
| `repair` | {'en_US': 'Repair'} |

**`l10n_ro_edi_stock_end_bcp`** — {'en_US': 'End Border Crossing Point'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Petea (HU)'} |
| `2` | {'en_US': 'Borș(HU)'} |
| `3` | {'en_US': 'Vărșand(HU)'} |
| `4` | {'en_US': 'Nădlac(HU)'} |
| `5` | {'en_US': 'Calafat (BG)'} |
| `6` | {'en_US': 'Bechet(BG)'} |
| `7` | {'en_US': 'Turnu Măgurele(BG)'} |
| `8` | {'en_US': 'Zimnicea(BG)'} |
| `9` | {'en_US': 'Giurgiu(BG)'} |
| `10` | {'en_US': 'Ostrov(BG)'} |
| `11` | {'en_US': 'Negru Vodă(BG)'} |
| `12` | {'en_US': 'Vama Veche(BG)'} |
| `13` | {'en_US': 'Călărași(BG)'} |
| `14` | {'en_US': 'Corabia(BG)'} |
| `15` | {'en_US': 'Oltenița(BG)'} |
| `16` | {'en_US': 'Carei  (HU)'} |
| `17` | {'en_US': 'Cenad (HU)'} |
| `18` | {'en_US': 'Episcopia Bihor (HU)'} |
| `19` | {'en_US': 'Salonta (HU)'} |
| `20` | {'en_US': 'Săcuieni (HU)'} |
| `21` | {'en_US': 'Turnu (HU)'} |
| `22` | {'en_US': 'Urziceni (HU)'} |
| `23` | {'en_US': 'Valea lui Mihai (HU)'} |
| `24` | {'en_US': 'Vladimirescu (HU)'} |
| `25` | {'en_US': 'Porțile de Fier 1 (RS)'} |
| `26` | {'en_US': 'Naidăș(RS)'} |
| `27` | {'en_US': 'Stamora Moravița(RS)'} |
| `28` | {'en_US': 'Jimbolia(RS)'} |
| `29` | {'en_US': 'Halmeu (UA)'} |
| `30` | {'en_US': 'Stânca Costești (MD)'} |
| `31` | {'en_US': 'Sculeni(MD)'} |
| `32` | {'en_US': 'Albița(MD)'} |
| `33` | {'en_US': 'Oancea(MD)'} |
| `34` | {'en_US': 'Galați Giurgiulești(MD)'} |
| `35` | {'en_US': 'Constanța Sud Agigea'} |
| `36` | {'en_US': 'Siret  (UA)'} |
| `37` | {'en_US': 'Nădlac 2 - A1 (HU)'} |
| `38` | {'en_US': 'Borș 2 - A3 (HU)'} |

**`l10n_ro_edi_stock_end_customs_office`** — {'en_US': 'End Customs Office'}

| Value | Label |
|---|---|
| `12801` | {'en_US': 'BVI Alba Iulia (ROBV0300)'} |
| `22801` | {'en_US': 'BVI Arad (ROTM0200)'} |
| `22901` | {'en_US': 'BVF Arad Aeroport (ROTM0230)'} |
| `22902` | {'en_US': 'BVF Zona Liberă Curtici (ROTM2300)'} |
| `32801` | {'en_US': 'BVI Pitești (ROCR7000)'} |
| `42801` | {'en_US': 'BVI Bacău (ROIS0600)'} |
| `42901` | {'en_US': 'BVF Bacău Aeroport (ROIS0620)'} |
| `52801` | {'en_US': 'BVI Oradea (ROCJ6570)'} |
| `52901` | {'en_US': 'BVF Oradea Aeroport (ROCJ6580)'} |
| `62801` | {'en_US': 'BVI Bistriţa-Năsăud (ROCJ0400)'} |
| `72801` | {'en_US': 'BVI Botoşani (ROIS1600)'} |
| `72901` | {'en_US': 'BVF Stanca Costeşti (ROIS1610)'} |
| `72902` | {'en_US': 'BVF Rădăuţi Prut (ROIS1620)'} |
| `82801` | {'en_US': 'BVI Braşov (ROBV0900)'} |
| `92901` | {'en_US': 'BVF Zona Liberă Brăila (ROGL0710)'} |
| `92902` | {'en_US': 'BVF Brăila (ROGL0700)'} |
| `102801` | {'en_US': 'BVI Buzău (ROGL1500)'} |
| `112801` | {'en_US': 'BVI Reșița (ROTM7600)'} |
| `112901` | {'en_US': 'BVF Naidăș (ROTM6100)'} |
| `122801` | {'en_US': 'BVI Cluj Napoca (ROCJ1800)'} |
| `122901` | {'en_US': 'BVF Cluj Napoca Aero (ROCJ1810)'} |
| `132901` | {'en_US': 'BVF Constanţa Sud Agigea (ROCT1900)'} |
| `132902` | {'en_US': 'BVF Mihail Kogălniceanu (ROCT5100)'} |
| `132903` | {'en_US': 'BVF Mangalia (ROCT5400)'} |
| `132904` | {'en_US': 'BVF Constanţa Port (ROCT1970)'} |
| `142801` | {'en_US': 'BVI Sfântu Gheorghe (ROBV7820)'} |
| `152801` | {'en_US': 'BVI Târgoviște (ROBU8600)'} |
| `162801` | {'en_US': 'BVI Craiova (ROCR2100)'} |
| `162901` | {'en_US': 'BVF Craiova Aeroport (ROCR2110)'} |
| `162902` | {'en_US': 'BVF Bechet (ROCR1720)'} |
| `162903` | {'en_US': 'BVF Calafat (ROCR1700)'} |
| `172901` | {'en_US': 'BVF Zona Liberă Galaţi (ROGL3810)'} |
| `172902` | {'en_US': 'BVF Giurgiuleşti (ROGL3850)'} |
| `172903` | {'en_US': 'BVF Oancea (ROGL3610)'} |
| `172904` | {'en_US': 'BVF Galaţi (ROGL3800)'} |
| `182801` | {'en_US': 'BVI Târgu Jiu (ROCR8810)'} |
| `192801` | {'en_US': 'BVI Miercurea Ciuc (ROBV5600)'} |
| `202801` | {'en_US': 'BVI Deva (ROTM8100)'} |
| `212801` | {'en_US': 'BVI Slobozia (ROCT8220)'} |
| `222901` | {'en_US': 'BVF Iaşi Aero (ROIS4660)'} |
| `222902` | {'en_US': 'BVF Sculeni (ROIS4990)'} |
| `222903` | {'en_US': 'BVF Iaşi (ROIS4650)'} |
| `232801` | {'en_US': 'BVI Antrepozite/Ilfov (ROBU1200)'} |
| `232901` | {'en_US': 'BVF Otopeni Călători (ROBU1030)'} |
| `242801` | {'en_US': 'BVI Baia Mare (ROCJ0500)'} |
| `242901` | {'en_US': 'BVF Aero Baia Mare (ROCJ0510)'} |
| `242902` | {'en_US': 'BVF Sighet (ROCJ8000)'} |
| `252901` | {'en_US': 'BVF Orşova (ROCR7280)'} |
| `252902` | {'en_US': 'BVF Porţile De Fier I (ROCR7270)'} |
| `252903` | {'en_US': 'BVF Porţile De Fier II (ROCR7200)'} |
| `252904` | {'en_US': 'BVF Drobeta Turnu Severin (ROCR9000)'} |
| `262801` | {'en_US': 'BVI Târgu Mureş (ROBV8800)'} |
| `262901` | {'en_US': 'BVF Târgu Mureş Aeroport (ROBV8820)'} |
| `272801` | {'en_US': 'BVI Piatra Neamţ (ROIS7400)'} |
| `282801` | {'en_US': 'BVI Corabia (ROCR2000)'} |
| `282802` | {'en_US': 'BVI Olt (ROCR8210)'} |
| `292801` | {'en_US': 'BVI Ploiești (ROBU7100)'} |
| `302801` | {'en_US': 'BVI Satu-Mare (ROCJ7810)'} |
| `302901` | {'en_US': 'BVF Halmeu (ROCJ4310)'} |
| `302902` | {'en_US': 'BVF Aeroport Satu Mare (ROCJ7830)'} |
| `312801` | {'en_US': 'BVI Zalău (ROCJ9700)'} |
| `322801` | {'en_US': 'BVI Sibiu (ROBV7900)'} |
| `322901` | {'en_US': 'BVF Sibiu Aeroport (ROBV7910)'} |
| `332801` | {'en_US': 'BVI Suceava (ROIS8230)'} |
| `332901` | {'en_US': 'BVF Dorneşti (ROIS2700)'} |
| `332902` | {'en_US': 'BVF Siret (ROIS8200)'} |
| `332903` | {'en_US': 'BVF Suceava Aero (ROIS8250)'} |
| `332904` | {'en_US': 'BVF Vicovu De Sus (ROIS9620)'} |
| `342801` | {'en_US': 'BVI Alexandria (ROCR0310)'} |
| `342901` | {'en_US': 'BVF Turnu Măgurele (ROCR9100)'} |
| `342902` | {'en_US': 'BVF Zimnicea (ROCR5800)'} |
| `352802` | {'en_US': 'BVI Timişoara Bază (ROTM8720)'} |
| `352901` | {'en_US': 'BVF Jimbolia (ROTM5010)'} |
| `352902` | {'en_US': 'BVF Moraviţa (ROTM5510)'} |
| `352903` | {'en_US': 'BVF Timişoara Aeroport (ROTM8730)'} |
| `362901` | {'en_US': 'BVF Sulina (ROCT8300)'} |
| `362902` | {'en_US': 'BVF Aeroport Delta Dunării Tulcea (ROGL8910)'} |
| `362903` | {'en_US': 'BVF Tulcea (ROGL8900)'} |
| `362904` | {'en_US': 'BVF Isaccea (ROGL8920)'} |
| `372801` | {'en_US': 'BVI Vaslui (ROIS9610)'} |
| `372901` | {'en_US': 'BVF Fălciu (-)'} |
| `372902` | {'en_US': 'BVF Albiţa (ROIS0100)'} |
| `382801` | {'en_US': 'BVI Râmnicu Vâlcea (ROCR7700)'} |
| `392801` | {'en_US': 'BVI Focșani (ROGL3600)'} |
| `402801` | {'en_US': 'BVI Bucureşti Poştă (ROBU1380)'} |
| `402802` | {'en_US': 'BVI Târguri și Expoziții (ROBU1400)'} |
| `402901` | {'en_US': 'BVF Băneasa (ROBU1040)'} |
| `512801` | {'en_US': 'BVI Călăraşi (ROCT1710)'} |
| `522801` | {'en_US': 'BVI Giurgiu (ROBU3910)'} |
| `522901` | {'en_US': 'BVF Zona Liberă Giurgiu (ROBU3980)'} |

**`l10n_ro_edi_stock_end_loc_type`** — {'en_US': 'End Location Type'}

| Value | Label |
|---|---|
| `location` | {'en_US': 'Location'} |
| `bcp` | {'en_US': 'Border Crossing Point'} |
| `customs` | {'en_US': 'Customs Office'} |

**`l10n_ro_edi_stock_operation_scope`** — {'en_US': 'Operation Scope'}

| Value | Label |
|---|---|
| `101` | {'en_US': 'Marketing'} |
| `201` | {'en_US': 'Output'} |
| `301` | {'en_US': 'Gratuities'} |
| `401` | {'en_US': 'Commercial equipment'} |
| `501` | {'en_US': 'Fixed assets'} |
| `601` | {'en_US': 'Own consumption'} |
| `703` | {'en_US': 'Delivery operations with installation'} |
| `704` | {'en_US': 'Transfer between managements'} |
| `705` | {'en_US': 'Goods made available to the customer'} |
| `801` | {'en_US': 'Financial/operational leasing'} |
| `802` | {'en_US': 'Goods under warranty'} |
| `901` | {'en_US': 'Exempt operations'} |
| `1001` | {'en_US': 'Investment in progress'} |
| `1101` | {'en_US': 'Donations, help'} |
| `9901` | {'en_US': 'Other'} |
| `9999` | {'en_US': 'Same with operation'} |

**`l10n_ro_edi_stock_operation_type`** — {'en_US': 'eTransport Operation Type'}

| Value | Label |
|---|---|
| `10` | {'en_US': 'Intra-community purchase'} |
| `12` | {'en_US': 'Operations in lohn system (EU) - input'} |
| `14` | {'en_US': 'Stocks available to the customer (Call-off stock) - entry'} |
| `20` | {'en_US': 'Intra-Community delivery'} |
| `22` | {'en_US': 'Operations in lohn system (EU) - exit'} |
| `24` | {'en_US': 'Stocks available to the customer (Call-off stock) - exit'} |
| `30` | {'en_US': 'Transport on the national territory'} |
| `40` | {'en_US': 'Import'} |
| `50` | {'en_US': 'Export'} |
| `60` | {'en_US': 'Intra-community transaction - Entry for storage/formation of new transport'} |
| `70` | {'en_US': 'Intra-community transaction - Exit after storage/formation of new transport'} |

**`l10n_ro_edi_stock_start_bcp`** — {'en_US': 'Start Border Crossing Point'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Petea (HU)'} |
| `2` | {'en_US': 'Borș(HU)'} |
| `3` | {'en_US': 'Vărșand(HU)'} |
| `4` | {'en_US': 'Nădlac(HU)'} |
| `5` | {'en_US': 'Calafat (BG)'} |
| `6` | {'en_US': 'Bechet(BG)'} |
| `7` | {'en_US': 'Turnu Măgurele(BG)'} |
| `8` | {'en_US': 'Zimnicea(BG)'} |
| `9` | {'en_US': 'Giurgiu(BG)'} |
| `10` | {'en_US': 'Ostrov(BG)'} |
| `11` | {'en_US': 'Negru Vodă(BG)'} |
| `12` | {'en_US': 'Vama Veche(BG)'} |
| `13` | {'en_US': 'Călărași(BG)'} |
| `14` | {'en_US': 'Corabia(BG)'} |
| `15` | {'en_US': 'Oltenița(BG)'} |
| `16` | {'en_US': 'Carei  (HU)'} |
| `17` | {'en_US': 'Cenad (HU)'} |
| `18` | {'en_US': 'Episcopia Bihor (HU)'} |
| `19` | {'en_US': 'Salonta (HU)'} |
| `20` | {'en_US': 'Săcuieni (HU)'} |
| `21` | {'en_US': 'Turnu (HU)'} |
| `22` | {'en_US': 'Urziceni (HU)'} |
| `23` | {'en_US': 'Valea lui Mihai (HU)'} |
| `24` | {'en_US': 'Vladimirescu (HU)'} |
| `25` | {'en_US': 'Porțile de Fier 1 (RS)'} |
| `26` | {'en_US': 'Naidăș(RS)'} |
| `27` | {'en_US': 'Stamora Moravița(RS)'} |
| `28` | {'en_US': 'Jimbolia(RS)'} |
| `29` | {'en_US': 'Halmeu (UA)'} |
| `30` | {'en_US': 'Stânca Costești (MD)'} |
| `31` | {'en_US': 'Sculeni(MD)'} |
| `32` | {'en_US': 'Albița(MD)'} |
| `33` | {'en_US': 'Oancea(MD)'} |
| `34` | {'en_US': 'Galați Giurgiulești(MD)'} |
| `35` | {'en_US': 'Constanța Sud Agigea'} |
| `36` | {'en_US': 'Siret  (UA)'} |
| `37` | {'en_US': 'Nădlac 2 - A1 (HU)'} |
| `38` | {'en_US': 'Borș 2 - A3 (HU)'} |

**`l10n_ro_edi_stock_start_customs_office`** — {'en_US': 'Start Customs Office'}

| Value | Label |
|---|---|
| `12801` | {'en_US': 'BVI Alba Iulia (ROBV0300)'} |
| `22801` | {'en_US': 'BVI Arad (ROTM0200)'} |
| `22901` | {'en_US': 'BVF Arad Aeroport (ROTM0230)'} |
| `22902` | {'en_US': 'BVF Zona Liberă Curtici (ROTM2300)'} |
| `32801` | {'en_US': 'BVI Pitești (ROCR7000)'} |
| `42801` | {'en_US': 'BVI Bacău (ROIS0600)'} |
| `42901` | {'en_US': 'BVF Bacău Aeroport (ROIS0620)'} |
| `52801` | {'en_US': 'BVI Oradea (ROCJ6570)'} |
| `52901` | {'en_US': 'BVF Oradea Aeroport (ROCJ6580)'} |
| `62801` | {'en_US': 'BVI Bistriţa-Năsăud (ROCJ0400)'} |
| `72801` | {'en_US': 'BVI Botoşani (ROIS1600)'} |
| `72901` | {'en_US': 'BVF Stanca Costeşti (ROIS1610)'} |
| `72902` | {'en_US': 'BVF Rădăuţi Prut (ROIS1620)'} |
| `82801` | {'en_US': 'BVI Braşov (ROBV0900)'} |
| `92901` | {'en_US': 'BVF Zona Liberă Brăila (ROGL0710)'} |
| `92902` | {'en_US': 'BVF Brăila (ROGL0700)'} |
| `102801` | {'en_US': 'BVI Buzău (ROGL1500)'} |
| `112801` | {'en_US': 'BVI Reșița (ROTM7600)'} |
| `112901` | {'en_US': 'BVF Naidăș (ROTM6100)'} |
| `122801` | {'en_US': 'BVI Cluj Napoca (ROCJ1800)'} |
| `122901` | {'en_US': 'BVF Cluj Napoca Aero (ROCJ1810)'} |
| `132901` | {'en_US': 'BVF Constanţa Sud Agigea (ROCT1900)'} |
| `132902` | {'en_US': 'BVF Mihail Kogălniceanu (ROCT5100)'} |
| `132903` | {'en_US': 'BVF Mangalia (ROCT5400)'} |
| `132904` | {'en_US': 'BVF Constanţa Port (ROCT1970)'} |
| `142801` | {'en_US': 'BVI Sfântu Gheorghe (ROBV7820)'} |
| `152801` | {'en_US': 'BVI Târgoviște (ROBU8600)'} |
| `162801` | {'en_US': 'BVI Craiova (ROCR2100)'} |
| `162901` | {'en_US': 'BVF Craiova Aeroport (ROCR2110)'} |
| `162902` | {'en_US': 'BVF Bechet (ROCR1720)'} |
| `162903` | {'en_US': 'BVF Calafat (ROCR1700)'} |
| `172901` | {'en_US': 'BVF Zona Liberă Galaţi (ROGL3810)'} |
| `172902` | {'en_US': 'BVF Giurgiuleşti (ROGL3850)'} |
| `172903` | {'en_US': 'BVF Oancea (ROGL3610)'} |
| `172904` | {'en_US': 'BVF Galaţi (ROGL3800)'} |
| `182801` | {'en_US': 'BVI Târgu Jiu (ROCR8810)'} |
| `192801` | {'en_US': 'BVI Miercurea Ciuc (ROBV5600)'} |
| `202801` | {'en_US': 'BVI Deva (ROTM8100)'} |
| `212801` | {'en_US': 'BVI Slobozia (ROCT8220)'} |
| `222901` | {'en_US': 'BVF Iaşi Aero (ROIS4660)'} |
| `222902` | {'en_US': 'BVF Sculeni (ROIS4990)'} |
| `222903` | {'en_US': 'BVF Iaşi (ROIS4650)'} |
| `232801` | {'en_US': 'BVI Antrepozite/Ilfov (ROBU1200)'} |
| `232901` | {'en_US': 'BVF Otopeni Călători (ROBU1030)'} |
| `242801` | {'en_US': 'BVI Baia Mare (ROCJ0500)'} |
| `242901` | {'en_US': 'BVF Aero Baia Mare (ROCJ0510)'} |
| `242902` | {'en_US': 'BVF Sighet (ROCJ8000)'} |
| `252901` | {'en_US': 'BVF Orşova (ROCR7280)'} |
| `252902` | {'en_US': 'BVF Porţile De Fier I (ROCR7270)'} |
| `252903` | {'en_US': 'BVF Porţile De Fier II (ROCR7200)'} |
| `252904` | {'en_US': 'BVF Drobeta Turnu Severin (ROCR9000)'} |
| `262801` | {'en_US': 'BVI Târgu Mureş (ROBV8800)'} |
| `262901` | {'en_US': 'BVF Târgu Mureş Aeroport (ROBV8820)'} |
| `272801` | {'en_US': 'BVI Piatra Neamţ (ROIS7400)'} |
| `282801` | {'en_US': 'BVI Corabia (ROCR2000)'} |
| `282802` | {'en_US': 'BVI Olt (ROCR8210)'} |
| `292801` | {'en_US': 'BVI Ploiești (ROBU7100)'} |
| `302801` | {'en_US': 'BVI Satu-Mare (ROCJ7810)'} |
| `302901` | {'en_US': 'BVF Halmeu (ROCJ4310)'} |
| `302902` | {'en_US': 'BVF Aeroport Satu Mare (ROCJ7830)'} |
| `312801` | {'en_US': 'BVI Zalău (ROCJ9700)'} |
| `322801` | {'en_US': 'BVI Sibiu (ROBV7900)'} |
| `322901` | {'en_US': 'BVF Sibiu Aeroport (ROBV7910)'} |
| `332801` | {'en_US': 'BVI Suceava (ROIS8230)'} |
| `332901` | {'en_US': 'BVF Dorneşti (ROIS2700)'} |
| `332902` | {'en_US': 'BVF Siret (ROIS8200)'} |
| `332903` | {'en_US': 'BVF Suceava Aero (ROIS8250)'} |
| `332904` | {'en_US': 'BVF Vicovu De Sus (ROIS9620)'} |
| `342801` | {'en_US': 'BVI Alexandria (ROCR0310)'} |
| `342901` | {'en_US': 'BVF Turnu Măgurele (ROCR9100)'} |
| `342902` | {'en_US': 'BVF Zimnicea (ROCR5800)'} |
| `352802` | {'en_US': 'BVI Timişoara Bază (ROTM8720)'} |
| `352901` | {'en_US': 'BVF Jimbolia (ROTM5010)'} |
| `352902` | {'en_US': 'BVF Moraviţa (ROTM5510)'} |
| `352903` | {'en_US': 'BVF Timişoara Aeroport (ROTM8730)'} |
| `362901` | {'en_US': 'BVF Sulina (ROCT8300)'} |
| `362902` | {'en_US': 'BVF Aeroport Delta Dunării Tulcea (ROGL8910)'} |
| `362903` | {'en_US': 'BVF Tulcea (ROGL8900)'} |
| `362904` | {'en_US': 'BVF Isaccea (ROGL8920)'} |
| `372801` | {'en_US': 'BVI Vaslui (ROIS9610)'} |
| `372901` | {'en_US': 'BVF Fălciu (-)'} |
| `372902` | {'en_US': 'BVF Albiţa (ROIS0100)'} |
| `382801` | {'en_US': 'BVI Râmnicu Vâlcea (ROCR7700)'} |
| `392801` | {'en_US': 'BVI Focșani (ROGL3600)'} |
| `402801` | {'en_US': 'BVI Bucureşti Poştă (ROBU1380)'} |
| `402802` | {'en_US': 'BVI Târguri și Expoziții (ROBU1400)'} |
| `402901` | {'en_US': 'BVF Băneasa (ROBU1040)'} |
| `512801` | {'en_US': 'BVI Călăraşi (ROCT1710)'} |
| `522801` | {'en_US': 'BVI Giurgiu (ROBU3910)'} |
| `522901` | {'en_US': 'BVF Zona Liberă Giurgiu (ROBU3980)'} |

**`l10n_ro_edi_stock_start_loc_type`** — {'en_US': 'Start Location Type'}

| Value | Label |
|---|---|
| `location` | {'en_US': 'Location'} |
| `bcp` | {'en_US': 'Border Crossing Point'} |
| `customs` | {'en_US': 'Customs Office'} |

**`l10n_ro_edi_stock_state`** — {'en_US': 'eTransport Status'}

| Value | Label |
|---|---|
| `stock_sent` | {'en_US': 'Sent'} |
| `stock_sending_failed` | {'en_US': 'Error'} |
| `stock_validated` | {'en_US': 'Validated'} |

**`l10n_tr_nilvera_dispatch_state`** — {'en_US': 'e-Dispatch State', 'tr_TR': 'e-İrsaliye Durumu'}

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'tr_TR': 'Gönderilecek'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi'} |

**`l10n_tr_nilvera_dispatch_type`** — {'en_US': 'Dispatch Type', 'tr_TR': 'İrsaliye Tipi'}

| Value | Label |
|---|---|
| `SEVK` | {'en_US': 'Online', 'tr_TR': 'Çevrimiçi'} |
| `MATBUDAN` | {'en_US': 'Pre-printed', 'tr_TR': 'Matbu'} |

**`move_type`** — {'en_US': 'Shipping Policy', 'tr_TR': 'Ürün Teslimat Politikası', 'ar_001': 'سياسة الشحن'}

| Value | Label |
|---|---|
| `direct` | {'en_US': 'As soon as possible', 'tr_TR': 'Mümkün olduğunca kısa sürede', 'ar_001': 'في أقرب وقت ممكن'} |
| `one` | {'en_US': 'When all products are ready', 'tr_TR': 'Tüm ürünler hazır olduğunda', 'ar_001': 'عندما تكون كافة المنتجات جاهزة'} |

**`priority`** — {'en_US': 'Priority', 'tr_TR': 'Öncelik', 'ar_001': 'الأولوية'}

| Value | Label |
|---|---|
| `0` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `1` | {'en_US': 'Urgent', 'tr_TR': 'Acil', 'ar_001': 'عاجل'} |

**`products_availability_state`** — {'en_US': 'Products Availability State', 'tr_TR': 'Ürün Kullanılabilirlik Durumu', 'ar_001': 'حالة توافر المنتجات'} (not stored)

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `expected` | {'en_US': 'Expected', 'tr_TR': 'Beklenen', 'ar_001': 'المتوقع'} |
| `late` | {'en_US': 'Late', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |

**`search_date_category`** — {'en_US': 'Date Category', 'tr_TR': 'Tarih Kategorisi', 'ar_001': 'فئة التاريخ'} (not stored)

| Value | Label |
|---|---|
| `before` | {'en_US': 'Before', 'tr_TR': 'Önce', 'ar_001': 'قبل'} |
| `yesterday` | {'en_US': 'Yesterday', 'tr_TR': 'Dün', 'ar_001': 'أمس'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `day_1` | {'en_US': 'Tomorrow', 'tr_TR': 'Yarın', 'ar_001': 'غداً'} |
| `day_2` | {'en_US': 'The day after tomorrow', 'tr_TR': 'Yarından sonraki gün', 'ar_001': 'بعد غد'} |
| `after` | {'en_US': 'After', 'tr_TR': 'Sonra', 'ar_001': 'بعد'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `waiting` | {'en_US': 'Waiting Another Operation', 'tr_TR': 'Başka Bir İşlem Bekliyor', 'ar_001': 'في انتظار عملية أخرى'} |
| `confirmed` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `assigned` | {'en_US': 'Ready', 'tr_TR': 'Hazır', 'ar_001': 'جاهز'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `stock.picking.batch`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

**`l10n_ro_edi_stock_end_bcp`** — {'en_US': 'End Border Crossing Point'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Petea (HU)'} |
| `2` | {'en_US': 'Borș(HU)'} |
| `3` | {'en_US': 'Vărșand(HU)'} |
| `4` | {'en_US': 'Nădlac(HU)'} |
| `5` | {'en_US': 'Calafat (BG)'} |
| `6` | {'en_US': 'Bechet(BG)'} |
| `7` | {'en_US': 'Turnu Măgurele(BG)'} |
| `8` | {'en_US': 'Zimnicea(BG)'} |
| `9` | {'en_US': 'Giurgiu(BG)'} |
| `10` | {'en_US': 'Ostrov(BG)'} |
| `11` | {'en_US': 'Negru Vodă(BG)'} |
| `12` | {'en_US': 'Vama Veche(BG)'} |
| `13` | {'en_US': 'Călărași(BG)'} |
| `14` | {'en_US': 'Corabia(BG)'} |
| `15` | {'en_US': 'Oltenița(BG)'} |
| `16` | {'en_US': 'Carei  (HU)'} |
| `17` | {'en_US': 'Cenad (HU)'} |
| `18` | {'en_US': 'Episcopia Bihor (HU)'} |
| `19` | {'en_US': 'Salonta (HU)'} |
| `20` | {'en_US': 'Săcuieni (HU)'} |
| `21` | {'en_US': 'Turnu (HU)'} |
| `22` | {'en_US': 'Urziceni (HU)'} |
| `23` | {'en_US': 'Valea lui Mihai (HU)'} |
| `24` | {'en_US': 'Vladimirescu (HU)'} |
| `25` | {'en_US': 'Porțile de Fier 1 (RS)'} |
| `26` | {'en_US': 'Naidăș(RS)'} |
| `27` | {'en_US': 'Stamora Moravița(RS)'} |
| `28` | {'en_US': 'Jimbolia(RS)'} |
| `29` | {'en_US': 'Halmeu (UA)'} |
| `30` | {'en_US': 'Stânca Costești (MD)'} |
| `31` | {'en_US': 'Sculeni(MD)'} |
| `32` | {'en_US': 'Albița(MD)'} |
| `33` | {'en_US': 'Oancea(MD)'} |
| `34` | {'en_US': 'Galați Giurgiulești(MD)'} |
| `35` | {'en_US': 'Constanța Sud Agigea'} |
| `36` | {'en_US': 'Siret  (UA)'} |
| `37` | {'en_US': 'Nădlac 2 - A1 (HU)'} |
| `38` | {'en_US': 'Borș 2 - A3 (HU)'} |

**`l10n_ro_edi_stock_end_customs_office`** — {'en_US': 'End Customs Office'}

| Value | Label |
|---|---|
| `12801` | {'en_US': 'BVI Alba Iulia (ROBV0300)'} |
| `22801` | {'en_US': 'BVI Arad (ROTM0200)'} |
| `22901` | {'en_US': 'BVF Arad Aeroport (ROTM0230)'} |
| `22902` | {'en_US': 'BVF Zona Liberă Curtici (ROTM2300)'} |
| `32801` | {'en_US': 'BVI Pitești (ROCR7000)'} |
| `42801` | {'en_US': 'BVI Bacău (ROIS0600)'} |
| `42901` | {'en_US': 'BVF Bacău Aeroport (ROIS0620)'} |
| `52801` | {'en_US': 'BVI Oradea (ROCJ6570)'} |
| `52901` | {'en_US': 'BVF Oradea Aeroport (ROCJ6580)'} |
| `62801` | {'en_US': 'BVI Bistriţa-Năsăud (ROCJ0400)'} |
| `72801` | {'en_US': 'BVI Botoşani (ROIS1600)'} |
| `72901` | {'en_US': 'BVF Stanca Costeşti (ROIS1610)'} |
| `72902` | {'en_US': 'BVF Rădăuţi Prut (ROIS1620)'} |
| `82801` | {'en_US': 'BVI Braşov (ROBV0900)'} |
| `92901` | {'en_US': 'BVF Zona Liberă Brăila (ROGL0710)'} |
| `92902` | {'en_US': 'BVF Brăila (ROGL0700)'} |
| `102801` | {'en_US': 'BVI Buzău (ROGL1500)'} |
| `112801` | {'en_US': 'BVI Reșița (ROTM7600)'} |
| `112901` | {'en_US': 'BVF Naidăș (ROTM6100)'} |
| `122801` | {'en_US': 'BVI Cluj Napoca (ROCJ1800)'} |
| `122901` | {'en_US': 'BVF Cluj Napoca Aero (ROCJ1810)'} |
| `132901` | {'en_US': 'BVF Constanţa Sud Agigea (ROCT1900)'} |
| `132902` | {'en_US': 'BVF Mihail Kogălniceanu (ROCT5100)'} |
| `132903` | {'en_US': 'BVF Mangalia (ROCT5400)'} |
| `132904` | {'en_US': 'BVF Constanţa Port (ROCT1970)'} |
| `142801` | {'en_US': 'BVI Sfântu Gheorghe (ROBV7820)'} |
| `152801` | {'en_US': 'BVI Târgoviște (ROBU8600)'} |
| `162801` | {'en_US': 'BVI Craiova (ROCR2100)'} |
| `162901` | {'en_US': 'BVF Craiova Aeroport (ROCR2110)'} |
| `162902` | {'en_US': 'BVF Bechet (ROCR1720)'} |
| `162903` | {'en_US': 'BVF Calafat (ROCR1700)'} |
| `172901` | {'en_US': 'BVF Zona Liberă Galaţi (ROGL3810)'} |
| `172902` | {'en_US': 'BVF Giurgiuleşti (ROGL3850)'} |
| `172903` | {'en_US': 'BVF Oancea (ROGL3610)'} |
| `172904` | {'en_US': 'BVF Galaţi (ROGL3800)'} |
| `182801` | {'en_US': 'BVI Târgu Jiu (ROCR8810)'} |
| `192801` | {'en_US': 'BVI Miercurea Ciuc (ROBV5600)'} |
| `202801` | {'en_US': 'BVI Deva (ROTM8100)'} |
| `212801` | {'en_US': 'BVI Slobozia (ROCT8220)'} |
| `222901` | {'en_US': 'BVF Iaşi Aero (ROIS4660)'} |
| `222902` | {'en_US': 'BVF Sculeni (ROIS4990)'} |
| `222903` | {'en_US': 'BVF Iaşi (ROIS4650)'} |
| `232801` | {'en_US': 'BVI Antrepozite/Ilfov (ROBU1200)'} |
| `232901` | {'en_US': 'BVF Otopeni Călători (ROBU1030)'} |
| `242801` | {'en_US': 'BVI Baia Mare (ROCJ0500)'} |
| `242901` | {'en_US': 'BVF Aero Baia Mare (ROCJ0510)'} |
| `242902` | {'en_US': 'BVF Sighet (ROCJ8000)'} |
| `252901` | {'en_US': 'BVF Orşova (ROCR7280)'} |
| `252902` | {'en_US': 'BVF Porţile De Fier I (ROCR7270)'} |
| `252903` | {'en_US': 'BVF Porţile De Fier II (ROCR7200)'} |
| `252904` | {'en_US': 'BVF Drobeta Turnu Severin (ROCR9000)'} |
| `262801` | {'en_US': 'BVI Târgu Mureş (ROBV8800)'} |
| `262901` | {'en_US': 'BVF Târgu Mureş Aeroport (ROBV8820)'} |
| `272801` | {'en_US': 'BVI Piatra Neamţ (ROIS7400)'} |
| `282801` | {'en_US': 'BVI Corabia (ROCR2000)'} |
| `282802` | {'en_US': 'BVI Olt (ROCR8210)'} |
| `292801` | {'en_US': 'BVI Ploiești (ROBU7100)'} |
| `302801` | {'en_US': 'BVI Satu-Mare (ROCJ7810)'} |
| `302901` | {'en_US': 'BVF Halmeu (ROCJ4310)'} |
| `302902` | {'en_US': 'BVF Aeroport Satu Mare (ROCJ7830)'} |
| `312801` | {'en_US': 'BVI Zalău (ROCJ9700)'} |
| `322801` | {'en_US': 'BVI Sibiu (ROBV7900)'} |
| `322901` | {'en_US': 'BVF Sibiu Aeroport (ROBV7910)'} |
| `332801` | {'en_US': 'BVI Suceava (ROIS8230)'} |
| `332901` | {'en_US': 'BVF Dorneşti (ROIS2700)'} |
| `332902` | {'en_US': 'BVF Siret (ROIS8200)'} |
| `332903` | {'en_US': 'BVF Suceava Aero (ROIS8250)'} |
| `332904` | {'en_US': 'BVF Vicovu De Sus (ROIS9620)'} |
| `342801` | {'en_US': 'BVI Alexandria (ROCR0310)'} |
| `342901` | {'en_US': 'BVF Turnu Măgurele (ROCR9100)'} |
| `342902` | {'en_US': 'BVF Zimnicea (ROCR5800)'} |
| `352802` | {'en_US': 'BVI Timişoara Bază (ROTM8720)'} |
| `352901` | {'en_US': 'BVF Jimbolia (ROTM5010)'} |
| `352902` | {'en_US': 'BVF Moraviţa (ROTM5510)'} |
| `352903` | {'en_US': 'BVF Timişoara Aeroport (ROTM8730)'} |
| `362901` | {'en_US': 'BVF Sulina (ROCT8300)'} |
| `362902` | {'en_US': 'BVF Aeroport Delta Dunării Tulcea (ROGL8910)'} |
| `362903` | {'en_US': 'BVF Tulcea (ROGL8900)'} |
| `362904` | {'en_US': 'BVF Isaccea (ROGL8920)'} |
| `372801` | {'en_US': 'BVI Vaslui (ROIS9610)'} |
| `372901` | {'en_US': 'BVF Fălciu (-)'} |
| `372902` | {'en_US': 'BVF Albiţa (ROIS0100)'} |
| `382801` | {'en_US': 'BVI Râmnicu Vâlcea (ROCR7700)'} |
| `392801` | {'en_US': 'BVI Focșani (ROGL3600)'} |
| `402801` | {'en_US': 'BVI Bucureşti Poştă (ROBU1380)'} |
| `402802` | {'en_US': 'BVI Târguri și Expoziții (ROBU1400)'} |
| `402901` | {'en_US': 'BVF Băneasa (ROBU1040)'} |
| `512801` | {'en_US': 'BVI Călăraşi (ROCT1710)'} |
| `522801` | {'en_US': 'BVI Giurgiu (ROBU3910)'} |
| `522901` | {'en_US': 'BVF Zona Liberă Giurgiu (ROBU3980)'} |

**`l10n_ro_edi_stock_end_loc_type`** — {'en_US': 'End Location Type'}

| Value | Label |
|---|---|
| `location` | {'en_US': 'Location'} |
| `bcp` | {'en_US': 'Border Crossing Point'} |
| `customs` | {'en_US': 'Customs Office'} |

**`l10n_ro_edi_stock_operation_scope`** — {'en_US': 'Operation Scope'}

| Value | Label |
|---|---|
| `101` | {'en_US': 'Marketing'} |
| `201` | {'en_US': 'Output'} |
| `301` | {'en_US': 'Gratuities'} |
| `401` | {'en_US': 'Commercial equipment'} |
| `501` | {'en_US': 'Fixed assets'} |
| `601` | {'en_US': 'Own consumption'} |
| `703` | {'en_US': 'Delivery operations with installation'} |
| `704` | {'en_US': 'Transfer between managements'} |
| `705` | {'en_US': 'Goods made available to the customer'} |
| `801` | {'en_US': 'Financial/operational leasing'} |
| `802` | {'en_US': 'Goods under warranty'} |
| `901` | {'en_US': 'Exempt operations'} |
| `1001` | {'en_US': 'Investment in progress'} |
| `1101` | {'en_US': 'Donations, help'} |
| `9901` | {'en_US': 'Other'} |
| `9999` | {'en_US': 'Same with operation'} |

**`l10n_ro_edi_stock_operation_type`** — {'en_US': 'eTransport Operation Type'}

| Value | Label |
|---|---|
| `10` | {'en_US': 'Intra-community purchase'} |
| `12` | {'en_US': 'Operations in lohn system (EU) - input'} |
| `14` | {'en_US': 'Stocks available to the customer (Call-off stock) - entry'} |
| `20` | {'en_US': 'Intra-Community delivery'} |
| `22` | {'en_US': 'Operations in lohn system (EU) - exit'} |
| `24` | {'en_US': 'Stocks available to the customer (Call-off stock) - exit'} |
| `30` | {'en_US': 'Transport on the national territory'} |
| `40` | {'en_US': 'Import'} |
| `50` | {'en_US': 'Export'} |
| `60` | {'en_US': 'Intra-community transaction - Entry for storage/formation of new transport'} |
| `70` | {'en_US': 'Intra-community transaction - Exit after storage/formation of new transport'} |

**`l10n_ro_edi_stock_start_bcp`** — {'en_US': 'Start Border Crossing Point'}

| Value | Label |
|---|---|
| `1` | {'en_US': 'Petea (HU)'} |
| `2` | {'en_US': 'Borș(HU)'} |
| `3` | {'en_US': 'Vărșand(HU)'} |
| `4` | {'en_US': 'Nădlac(HU)'} |
| `5` | {'en_US': 'Calafat (BG)'} |
| `6` | {'en_US': 'Bechet(BG)'} |
| `7` | {'en_US': 'Turnu Măgurele(BG)'} |
| `8` | {'en_US': 'Zimnicea(BG)'} |
| `9` | {'en_US': 'Giurgiu(BG)'} |
| `10` | {'en_US': 'Ostrov(BG)'} |
| `11` | {'en_US': 'Negru Vodă(BG)'} |
| `12` | {'en_US': 'Vama Veche(BG)'} |
| `13` | {'en_US': 'Călărași(BG)'} |
| `14` | {'en_US': 'Corabia(BG)'} |
| `15` | {'en_US': 'Oltenița(BG)'} |
| `16` | {'en_US': 'Carei  (HU)'} |
| `17` | {'en_US': 'Cenad (HU)'} |
| `18` | {'en_US': 'Episcopia Bihor (HU)'} |
| `19` | {'en_US': 'Salonta (HU)'} |
| `20` | {'en_US': 'Săcuieni (HU)'} |
| `21` | {'en_US': 'Turnu (HU)'} |
| `22` | {'en_US': 'Urziceni (HU)'} |
| `23` | {'en_US': 'Valea lui Mihai (HU)'} |
| `24` | {'en_US': 'Vladimirescu (HU)'} |
| `25` | {'en_US': 'Porțile de Fier 1 (RS)'} |
| `26` | {'en_US': 'Naidăș(RS)'} |
| `27` | {'en_US': 'Stamora Moravița(RS)'} |
| `28` | {'en_US': 'Jimbolia(RS)'} |
| `29` | {'en_US': 'Halmeu (UA)'} |
| `30` | {'en_US': 'Stânca Costești (MD)'} |
| `31` | {'en_US': 'Sculeni(MD)'} |
| `32` | {'en_US': 'Albița(MD)'} |
| `33` | {'en_US': 'Oancea(MD)'} |
| `34` | {'en_US': 'Galați Giurgiulești(MD)'} |
| `35` | {'en_US': 'Constanța Sud Agigea'} |
| `36` | {'en_US': 'Siret  (UA)'} |
| `37` | {'en_US': 'Nădlac 2 - A1 (HU)'} |
| `38` | {'en_US': 'Borș 2 - A3 (HU)'} |

**`l10n_ro_edi_stock_start_customs_office`** — {'en_US': 'Start Customs Office'}

| Value | Label |
|---|---|
| `12801` | {'en_US': 'BVI Alba Iulia (ROBV0300)'} |
| `22801` | {'en_US': 'BVI Arad (ROTM0200)'} |
| `22901` | {'en_US': 'BVF Arad Aeroport (ROTM0230)'} |
| `22902` | {'en_US': 'BVF Zona Liberă Curtici (ROTM2300)'} |
| `32801` | {'en_US': 'BVI Pitești (ROCR7000)'} |
| `42801` | {'en_US': 'BVI Bacău (ROIS0600)'} |
| `42901` | {'en_US': 'BVF Bacău Aeroport (ROIS0620)'} |
| `52801` | {'en_US': 'BVI Oradea (ROCJ6570)'} |
| `52901` | {'en_US': 'BVF Oradea Aeroport (ROCJ6580)'} |
| `62801` | {'en_US': 'BVI Bistriţa-Năsăud (ROCJ0400)'} |
| `72801` | {'en_US': 'BVI Botoşani (ROIS1600)'} |
| `72901` | {'en_US': 'BVF Stanca Costeşti (ROIS1610)'} |
| `72902` | {'en_US': 'BVF Rădăuţi Prut (ROIS1620)'} |
| `82801` | {'en_US': 'BVI Braşov (ROBV0900)'} |
| `92901` | {'en_US': 'BVF Zona Liberă Brăila (ROGL0710)'} |
| `92902` | {'en_US': 'BVF Brăila (ROGL0700)'} |
| `102801` | {'en_US': 'BVI Buzău (ROGL1500)'} |
| `112801` | {'en_US': 'BVI Reșița (ROTM7600)'} |
| `112901` | {'en_US': 'BVF Naidăș (ROTM6100)'} |
| `122801` | {'en_US': 'BVI Cluj Napoca (ROCJ1800)'} |
| `122901` | {'en_US': 'BVF Cluj Napoca Aero (ROCJ1810)'} |
| `132901` | {'en_US': 'BVF Constanţa Sud Agigea (ROCT1900)'} |
| `132902` | {'en_US': 'BVF Mihail Kogălniceanu (ROCT5100)'} |
| `132903` | {'en_US': 'BVF Mangalia (ROCT5400)'} |
| `132904` | {'en_US': 'BVF Constanţa Port (ROCT1970)'} |
| `142801` | {'en_US': 'BVI Sfântu Gheorghe (ROBV7820)'} |
| `152801` | {'en_US': 'BVI Târgoviște (ROBU8600)'} |
| `162801` | {'en_US': 'BVI Craiova (ROCR2100)'} |
| `162901` | {'en_US': 'BVF Craiova Aeroport (ROCR2110)'} |
| `162902` | {'en_US': 'BVF Bechet (ROCR1720)'} |
| `162903` | {'en_US': 'BVF Calafat (ROCR1700)'} |
| `172901` | {'en_US': 'BVF Zona Liberă Galaţi (ROGL3810)'} |
| `172902` | {'en_US': 'BVF Giurgiuleşti (ROGL3850)'} |
| `172903` | {'en_US': 'BVF Oancea (ROGL3610)'} |
| `172904` | {'en_US': 'BVF Galaţi (ROGL3800)'} |
| `182801` | {'en_US': 'BVI Târgu Jiu (ROCR8810)'} |
| `192801` | {'en_US': 'BVI Miercurea Ciuc (ROBV5600)'} |
| `202801` | {'en_US': 'BVI Deva (ROTM8100)'} |
| `212801` | {'en_US': 'BVI Slobozia (ROCT8220)'} |
| `222901` | {'en_US': 'BVF Iaşi Aero (ROIS4660)'} |
| `222902` | {'en_US': 'BVF Sculeni (ROIS4990)'} |
| `222903` | {'en_US': 'BVF Iaşi (ROIS4650)'} |
| `232801` | {'en_US': 'BVI Antrepozite/Ilfov (ROBU1200)'} |
| `232901` | {'en_US': 'BVF Otopeni Călători (ROBU1030)'} |
| `242801` | {'en_US': 'BVI Baia Mare (ROCJ0500)'} |
| `242901` | {'en_US': 'BVF Aero Baia Mare (ROCJ0510)'} |
| `242902` | {'en_US': 'BVF Sighet (ROCJ8000)'} |
| `252901` | {'en_US': 'BVF Orşova (ROCR7280)'} |
| `252902` | {'en_US': 'BVF Porţile De Fier I (ROCR7270)'} |
| `252903` | {'en_US': 'BVF Porţile De Fier II (ROCR7200)'} |
| `252904` | {'en_US': 'BVF Drobeta Turnu Severin (ROCR9000)'} |
| `262801` | {'en_US': 'BVI Târgu Mureş (ROBV8800)'} |
| `262901` | {'en_US': 'BVF Târgu Mureş Aeroport (ROBV8820)'} |
| `272801` | {'en_US': 'BVI Piatra Neamţ (ROIS7400)'} |
| `282801` | {'en_US': 'BVI Corabia (ROCR2000)'} |
| `282802` | {'en_US': 'BVI Olt (ROCR8210)'} |
| `292801` | {'en_US': 'BVI Ploiești (ROBU7100)'} |
| `302801` | {'en_US': 'BVI Satu-Mare (ROCJ7810)'} |
| `302901` | {'en_US': 'BVF Halmeu (ROCJ4310)'} |
| `302902` | {'en_US': 'BVF Aeroport Satu Mare (ROCJ7830)'} |
| `312801` | {'en_US': 'BVI Zalău (ROCJ9700)'} |
| `322801` | {'en_US': 'BVI Sibiu (ROBV7900)'} |
| `322901` | {'en_US': 'BVF Sibiu Aeroport (ROBV7910)'} |
| `332801` | {'en_US': 'BVI Suceava (ROIS8230)'} |
| `332901` | {'en_US': 'BVF Dorneşti (ROIS2700)'} |
| `332902` | {'en_US': 'BVF Siret (ROIS8200)'} |
| `332903` | {'en_US': 'BVF Suceava Aero (ROIS8250)'} |
| `332904` | {'en_US': 'BVF Vicovu De Sus (ROIS9620)'} |
| `342801` | {'en_US': 'BVI Alexandria (ROCR0310)'} |
| `342901` | {'en_US': 'BVF Turnu Măgurele (ROCR9100)'} |
| `342902` | {'en_US': 'BVF Zimnicea (ROCR5800)'} |
| `352802` | {'en_US': 'BVI Timişoara Bază (ROTM8720)'} |
| `352901` | {'en_US': 'BVF Jimbolia (ROTM5010)'} |
| `352902` | {'en_US': 'BVF Moraviţa (ROTM5510)'} |
| `352903` | {'en_US': 'BVF Timişoara Aeroport (ROTM8730)'} |
| `362901` | {'en_US': 'BVF Sulina (ROCT8300)'} |
| `362902` | {'en_US': 'BVF Aeroport Delta Dunării Tulcea (ROGL8910)'} |
| `362903` | {'en_US': 'BVF Tulcea (ROGL8900)'} |
| `362904` | {'en_US': 'BVF Isaccea (ROGL8920)'} |
| `372801` | {'en_US': 'BVI Vaslui (ROIS9610)'} |
| `372901` | {'en_US': 'BVF Fălciu (-)'} |
| `372902` | {'en_US': 'BVF Albiţa (ROIS0100)'} |
| `382801` | {'en_US': 'BVI Râmnicu Vâlcea (ROCR7700)'} |
| `392801` | {'en_US': 'BVI Focșani (ROGL3600)'} |
| `402801` | {'en_US': 'BVI Bucureşti Poştă (ROBU1380)'} |
| `402802` | {'en_US': 'BVI Târguri și Expoziții (ROBU1400)'} |
| `402901` | {'en_US': 'BVF Băneasa (ROBU1040)'} |
| `512801` | {'en_US': 'BVI Călăraşi (ROCT1710)'} |
| `522801` | {'en_US': 'BVI Giurgiu (ROBU3910)'} |
| `522901` | {'en_US': 'BVF Zona Liberă Giurgiu (ROBU3980)'} |

**`l10n_ro_edi_stock_start_loc_type`** — {'en_US': 'Start Location Type'}

| Value | Label |
|---|---|
| `location` | {'en_US': 'Location'} |
| `bcp` | {'en_US': 'Border Crossing Point'} |
| `customs` | {'en_US': 'Customs Office'} |

**`l10n_ro_edi_stock_state`** — {'en_US': 'eTransport Status'}

| Value | Label |
|---|---|
| `stock_sent` | {'en_US': 'Sent'} |
| `stock_sending_failed` | {'en_US': 'Error'} |
| `stock_validated` | {'en_US': 'Validated'} |

**`state`** — {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `in_progress` | {'en_US': 'In progress', 'tr_TR': 'Devam Eden', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

## `stock.picking.to.batch`

**`mode`** — {'en_US': 'Mode', 'tr_TR': 'Şekli', 'ar_001': 'الوضع'}

| Value | Label |
|---|---|
| `existing` | {'en_US': 'an existing batch transfer', 'tr_TR': 'mevcut bir batch transfer', 'ar_001': 'عملية نقل على دفعات موجودة بالفعل'} |
| `new` | {'en_US': 'a new batch transfer', 'tr_TR': 'yeni bir batch transfer', 'ar_001': 'عملية نقل على دفعات جديدة'} |

## `stock.picking.type`

**`code`** — {'en_US': 'Type of Operation', 'tr_TR': 'Operasyon Türü', 'ar_001': 'نوع العملية'}

| Value | Label |
|---|---|
| `incoming` | {'en_US': 'Receipt', 'tr_TR': 'Alım', 'ar_001': 'الإيصال'} |
| `outgoing` | {'en_US': 'Delivery', 'tr_TR': 'Teslimat', 'ar_001': 'التوصيل'} |
| `internal` | {'en_US': 'Internal Transfer', 'tr_TR': 'İç Transfer', 'ar_001': 'تحويل داخلي'} |
| `mrp_operation` | {'en_US': 'Manufacturing', 'tr_TR': 'Üretim', 'ar_001': 'التصنيع'} |
| `repair_operation` | {'en_US': 'Repair', 'tr_TR': 'Onarım', 'ar_001': 'الإصلاح'} |
| `dropship` | {'en_US': 'Dropship', 'tr_TR': 'Transit Satış', 'ar_001': 'إحالة الشحن'} |

**`create_backorder`** — {'en_US': 'Create Backorder', 'tr_TR': 'Ön Teslimat Oluştur', 'ar_001': 'إنشاء طلب متأخر'}

| Value | Label |
|---|---|
| `ask` | {'en_US': 'Ask', 'tr_TR': 'Sor', 'ar_001': 'اسأل'} |
| `always` | {'en_US': 'Always', 'tr_TR': 'Daima', 'ar_001': 'دائمًا'} |
| `never` | {'en_US': 'Never', 'tr_TR': 'Asla', 'ar_001': 'مطلقًا'} |

**`done_mrp_lot_label_to_print`** — {'en_US': 'Lot/SN Label to Print', 'tr_TR': 'Yazdırılacak Lot/SN Etiketi', 'ar_001': 'بطاقة عنوان رقم المجموعة/الرقم التسلسلي لطباعتها'}

| Value | Label |
|---|---|
| `pdf` | {'en_US': 'PDF', 'tr_TR': 'PDF', 'ar_001': 'PDF'} |
| `zpl` | {'en_US': 'ZPL', 'tr_TR': 'ZPL', 'ar_001': 'ZPL'} |

**`generated_mrp_lot_label_to_print`** — {'en_US': 'Generated Lot/SN Label to Print', 'tr_TR': 'Yazdırılacak Lot/SN Etiketi Oluşturuldu', 'ar_001': 'بطاقة عنوان رقم المجموعة/الرقم التسلسلي المُنشأ لطباعتها'}

| Value | Label |
|---|---|
| `pdf` | {'en_US': 'PDF', 'tr_TR': 'PDF', 'ar_001': 'PDF'} |
| `zpl` | {'en_US': 'ZPL', 'tr_TR': 'ZPL', 'ar_001': 'ZPL'} |

**`lot_label_format`** — {'en_US': 'Lot Label Format to auto-print', 'tr_TR': 'Otomatik yazdırılacak Lot Etiketi Formatı', 'ar_001': 'تنسيق ملصق المجموعة لطباعته تلقائياً'}

| Value | Label |
|---|---|
| `4x12_lots` | {'en_US': '4 x 12 - One per lot/SN', 'tr_TR': '4 x 12 - Parti/SN başına bir adet', 'ar_001': '4 x 12 - واحدة لكل رقم مجموعة/رقم تسلسلي'} |
| `4x12_units` | {'en_US': '4 x 12 - One per unit', 'tr_TR': '4 x 12 - Birim başına bir adet', 'ar_001': '4 x 12 - واحدة لكل وحدة'} |
| `zpl_lots` | {'en_US': 'ZPL Labels - One per lot/SN', 'tr_TR': 'ZPL Etiketleri - Her parti/seri numarası için bir adet', 'ar_001': 'علامات تصنيف ZPL - واحدة لكل رقم مجموعة/ رقم تسلسلي'} |
| `zpl_units` | {'en_US': 'ZPL Labels - One per unit', 'tr_TR': 'ZPL Etiketleri - Birim başına bir adet', 'ar_001': 'علامات تصنيف ZPL - واحدة لكل وحدة'} |

**`move_type`** — {'en_US': 'Shipping Policy', 'tr_TR': 'Ürün Teslimat Politikası', 'ar_001': 'سياسة الشحن'}

| Value | Label |
|---|---|
| `direct` | {'en_US': 'As soon as possible', 'tr_TR': 'Mümkün olduğunca kısa sürede', 'ar_001': 'في أقرب وقت ممكن'} |
| `one` | {'en_US': 'When all products are ready', 'tr_TR': 'Tüm ürünler hazır olduğunda', 'ar_001': 'عندما تكون كافة المنتجات جاهزة'} |

**`mrp_product_label_to_print`** — {'en_US': 'Product Label to Print', 'tr_TR': 'Yazdırılacak Ürün Etiketi', 'ar_001': 'بطاقة عنوان المنتج لطباعتها'}

| Value | Label |
|---|---|
| `pdf` | {'en_US': 'PDF', 'tr_TR': 'PDF', 'ar_001': 'PDF'} |
| `zpl` | {'en_US': 'ZPL', 'tr_TR': 'ZPL', 'ar_001': 'ZPL'} |

**`package_label_to_print`** — {'en_US': 'Package Label to Print', 'tr_TR': 'Yazdırılacak Paket Etiketi', 'ar_001': 'بطاقة عنوان الطرد لطباعتها'}

| Value | Label |
|---|---|
| `pdf` | {'en_US': 'PDF', 'tr_TR': 'PDF', 'ar_001': 'PDF'} |
| `zpl` | {'en_US': 'ZPL', 'tr_TR': 'ZPL', 'ar_001': 'ZPL'} |

**`product_label_format`** — {'en_US': 'Product Label Format to auto-print', 'tr_TR': 'Otomatik yazdırma için Ürün Etiketi Formatı', 'ar_001': 'تنسيق ملصق المنتج لطباعته تلقائياً'}

| Value | Label |
|---|---|
| `dymo` | {'en_US': 'Dymo', 'tr_TR': 'Dymo', 'ar_001': 'Dymo'} |
| `2x7xprice` | {'en_US': '2 x 7 with price', 'tr_TR': '2x7 fiyat ile', 'ar_001': '2 × 7 بالسعر'} |
| `4x7xprice` | {'en_US': '4 x 7 with price', 'tr_TR': '4 x 7 fiyat ile', 'ar_001': '4 × 7 بالسعر'} |
| `4x12` | {'en_US': '4 x 12', 'tr_TR': '4 x 12', 'ar_001': '4 × 12'} |
| `4x12xprice` | {'en_US': '4 x 12 with price', 'tr_TR': '4 x 12 fiyat ile', 'ar_001': '4 × 12 بالسعر'} |
| `zpl` | {'en_US': 'ZPL Labels', 'tr_TR': 'ZPL Etiketleri', 'ar_001': 'بطاقات عناوين ZPL'} |
| `zplxprice` | {'en_US': 'ZPL Labels with price', 'tr_TR': 'ZPL Etiketleri, fiyat ile', 'ar_001': 'بطاقات عناوين ZPL مع السعر'} |

**`reservation_method`** — {'en_US': 'Reservation Method', 'tr_TR': 'Rezervasyon Metodu', 'ar_001': 'طريقة الحجز'}

| Value | Label |
|---|---|
| `at_confirm` | {'en_US': 'At Confirmation', 'tr_TR': 'Onayda', 'ar_001': 'عند التأكيد'} |
| `manual` | {'en_US': 'Manually', 'tr_TR': 'Manuel olarak', 'ar_001': 'يدويًا'} |
| `by_date` | {'en_US': 'Before scheduled date', 'tr_TR': 'Planlanan Tarihten Önce', 'ar_001': 'قبل التاريخ المجدول'} |

## `stock.putaway.rule`

**`sublocation`** — {'en_US': 'Sublocation', 'tr_TR': 'Alt konum', 'ar_001': 'الموقع الفرعي'}

| Value | Label |
|---|---|
| `no` | {'en_US': 'No', 'tr_TR': 'Hayır', 'ar_001': 'لا'} |
| `last_used` | {'en_US': 'Last Used', 'tr_TR': 'Son Kullanım:', 'ar_001': 'آخر استخدام'} |
| `closest_location` | {'en_US': 'Closest Location', 'tr_TR': 'En Yakın Konum', 'ar_001': 'أقرب موقع'} |

## `stock.quant`

**`cost_method`** — {'en_US': 'Cost Method', 'tr_TR': 'Maliyet Yöntemi', 'ar_001': 'طريقة حساب التكلفة'} (not stored)

| Value | Label |
|---|---|
| `standard` | {'en_US': 'Standard Price', 'tr_TR': 'Standart Maliyet', 'ar_001': 'السعر القياسي'} |
| `fifo` | {'en_US': 'First In First Out (FIFO)', 'tr_TR': 'İlk Giren İlk Çıkar (FIFO)', 'ar_001': 'الوارد أولاً يخرج أولاً (FIFO)'} |
| `average` | {'en_US': 'Average Cost (AVCO)', 'tr_TR': 'Ortalama Maliyet (AVCO)', 'ar_001': 'متوسط التكلفة (AVCO)'} |

## `stock.replenishment.info`

**`based_on`** — {'en_US': 'Based on', 'tr_TR': 'Buna göre', 'ar_001': 'بناءً على'}

| Value | Label |
|---|---|
| `one_week` | {'en_US': 'Last 7 days', 'tr_TR': 'Son 7 gün', 'ar_001': 'آخر 7 أيام'} |
| `one_month` | {'en_US': 'Last 30 days', 'tr_TR': 'Son 30 gün', 'ar_001': 'آخر 30 يوم'} |
| `three_months` | {'en_US': 'Last 3 months', 'tr_TR': 'Son 3 Ay', 'ar_001': 'آخر 3 أشهر'} |
| `one_year` | {'en_US': 'Last 12 months', 'tr_TR': 'Son 12 ay', 'ar_001': 'آخر 12 شهراً'} |
| `last_year` | {'en_US': 'Same month last year', 'tr_TR': 'Geçen yılın aynı ayı', 'ar_001': 'نفس الشهر من العام الماضي'} |
| `last_year_2` | {'en_US': 'Next month last year', 'tr_TR': 'Geçen yıl gelecek ay', 'ar_001': 'الشهر التالي من العام الماضي'} |
| `last_year_3` | {'en_US': 'After next month last year', 'tr_TR': 'Geçen yıl gelecek aydan sonra', 'ar_001': 'بعد الشهر التالي من العام الماضي'} |
| `last_year_quarter` | {'en_US': 'Last year quarter', 'tr_TR': 'Geçen yılın çeyreği', 'ar_001': 'ربع السنة الماضي'} |

## `stock.rule`

**`action`** — {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'}

| Value | Label |
|---|---|
| `pull` | {'en_US': 'Pull From', 'tr_TR': 'Çekme', 'ar_001': 'السحب من'} |
| `push` | {'en_US': 'Push To', 'tr_TR': 'İtme', 'ar_001': 'الدفع إلى'} |
| `pull_push` | {'en_US': 'Pull & Push', 'tr_TR': 'Çekme & İtme', 'ar_001': 'السحب والدفع'} |
| `manufacture` | {'en_US': 'Manufacture', 'tr_TR': 'Üretim', 'ar_001': 'تصنيع'} |
| `buy` | {'en_US': 'Buy', 'tr_TR': 'Satınal', 'ar_001': 'شراء'} |

**`auto`** — {'en_US': 'Automatic Move', 'tr_TR': 'Otomatik Hareket', 'ar_001': 'حركة تلقائية'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual Operation', 'tr_TR': 'Manuel İşlem', 'ar_001': 'العملية اليدوية'} |
| `transparent` | {'en_US': 'Automatic No Step Added', 'tr_TR': 'Otomatik No Adımı Eklendi', 'ar_001': 'تمت إضافة رقم الخطة التلقائي'} |

**`procure_method`** — {'en_US': 'Supply Method', 'tr_TR': 'Tedarik Yöntemi', 'ar_001': 'طريقة  التزويد'}

| Value | Label |
|---|---|
| `make_to_stock` | {'en_US': 'Take From Stock', 'tr_TR': 'Stoktan Al', 'ar_001': 'الأخذ من المخزون'} |
| `make_to_order` | {'en_US': 'Trigger Another Rule', 'tr_TR': 'Başka Bir Kural Tetikle', 'ar_001': 'تشغيل قاعدة أخرى'} |
| `mts_else_mto` | {'en_US': 'Take From Stock, if unavailable, Trigger Another Rule', 'tr_TR': 'Stoktan Alın, eğer kullanılamıyorsa, Başka Bir Kuralı Tetikleyin', 'ar_001': 'الأخذ من المخزون، وفي حال عدم التوافر، قم بتشغيل قاعدة أخرى'} |

## `stock.scrap`

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

## `stock.storage.category`

**`allow_new_product`** — {'en_US': 'Allow New Product', 'tr_TR': 'Yeni Ürüne İzin Ver', 'ar_001': 'السماح بمنتج جديد'}

| Value | Label |
|---|---|
| `empty` | {'en_US': 'If the location is empty', 'tr_TR': 'Eğer konum boşsa', 'ar_001': 'إذا كان الموقع فارغاً'} |
| `same` | {'en_US': 'If all products are same', 'tr_TR': 'Tüm ürünler aynı olduğunda', 'ar_001': 'عندما تكون كافة المنتجات متشابهة'} |
| `mixed` | {'en_US': 'Allow mixed products', 'tr_TR': 'Karma ürünlere izin ver', 'ar_001': 'السماح بالمنتجات المختلطة'} |

## `stock.warehouse`

**`delivery_steps`** — {'en_US': 'Outgoing Shipments', 'tr_TR': 'Giden Sevkiyatlar', 'ar_001': 'الشحنات الصادرة'}

| Value | Label |
|---|---|
| `ship_only` | {'en_US': 'Deliver (1 step)', 'tr_TR': 'Teslimat (1 Adım)', 'ar_001': 'التوصيل (خطوة واحدة)'} |
| `pick_ship` | {'en_US': 'Pick then Deliver (2 steps)', 'tr_TR': 'Topla, sonra Teslimat (2 Adım)', 'ar_001': 'التجهيز والتوصيل (خطوتان)'} |
| `pick_pack_ship` | {'en_US': 'Pick, Pack, then Deliver (3 steps)', 'tr_TR': 'Topla, Paketler, sonra Teslimat (3 Adım)', 'ar_001': 'التجهيز ، الرزم ثم التوصيل (3 خطوات)'} |

**`manufacture_steps`** — {'en_US': 'Manufacture', 'tr_TR': 'Üretim', 'ar_001': 'تصنيع'}

| Value | Label |
|---|---|
| `mrp_one_step` | {'en_US': 'Manufacture (1 step)', 'tr_TR': 'Üretim (1.Adım)', 'ar_001': 'تصنيع (خطوة واحدة)'} |
| `pbm` | {'en_US': 'Pick components then manufacture (2 steps)', 'tr_TR': 'Bileşen malzemeleri depodan çekin, sonra üretim yapın(2 adım)', 'ar_001': 'قم بانتقاء المكونات ثم قم بالتصنيع (خطوتان)'} |
| `pbm_sam` | {'en_US': 'Pick components, manufacture, then store products (3 steps)', 'tr_TR': 'Bileşenleri seçin, ürünleri üretin, sonra stok sahasına alın (3. adım)', 'ar_001': 'انتقاء المكونات ثم التصنيع ثم تخزين المنتجات (3 خطوات)'} |

**`reception_steps`** — {'en_US': 'Incoming Shipments', 'tr_TR': 'Gelen Sevkiyatlar', 'ar_001': 'الشحنات الواردة'}

| Value | Label |
|---|---|
| `one_step` | {'en_US': 'Receive and Store (1 step)', 'tr_TR': 'Teslim alın ve Stoklayın (1 Adım)', 'ar_001': 'الاستلام والتخزين (خطوة واحدة)'} |
| `two_steps` | {'en_US': 'Receive then Store (2 steps)', 'tr_TR': 'Teslim alın ve Stoklayın (2 Adım)', 'ar_001': 'الاستلام ثم التخزين (خطوتان)'} |
| `three_steps` | {'en_US': 'Receive, Quality Control, then Store (3 steps)', 'tr_TR': 'Teslim alın, Kalite Kontrol yapın, sonra Stoklayın (3 Adım)', 'ar_001': 'الاستلام، وضبط الجودة، ثم التخزين (3 خطوات)'} |

## `stock.warehouse.orderpoint`

**`trigger`** — {'en_US': 'Trigger', 'tr_TR': 'Tetikleyici', 'ar_001': 'المشغّل'}

| Value | Label |
|---|---|
| `auto` | {'en_US': 'Auto', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |

## `survey.invite`

**`existing_mode`** — {'en_US': 'Handle existing', 'tr_TR': 'Mevcut olanları ele al', 'ar_001': 'التعامل مع الموجود'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'New invite', 'tr_TR': 'Yeni davet', 'ar_001': 'دعوة جديدة'} |
| `resend` | {'en_US': 'Resend invite', 'tr_TR': 'Davetiyeyi tekrar gönder', 'ar_001': 'إعادة ارسال الدعوة'} |

## `survey.question`

**`matrix_subtype`** — {'en_US': 'Matrix Type', 'tr_TR': 'Matris türü', 'ar_001': 'نوع المصفوفة'}

| Value | Label |
|---|---|
| `simple` | {'en_US': 'One choice per row', 'tr_TR': 'Her satır için bir seçim', 'ar_001': 'خيار واحد لكل صف'} |
| `multiple` | {'en_US': 'Multiple choices per row', 'tr_TR': 'Her satır için birden çok seçenek', 'ar_001': 'خيارات متعددة لكل صف'} |

**`question_type`** — {'en_US': 'Question Type', 'tr_TR': 'Soru Tipi', 'ar_001': 'نوع السؤال'}

| Value | Label |
|---|---|
| `simple_choice` | {'en_US': 'Multiple choice: only one answer', 'tr_TR': 'Çoklu Seçmeli: Sadece bir Cevap', 'ar_001': 'اختيار من متعدد: إجابة واحدة فقط'} |
| `multiple_choice` | {'en_US': 'Multiple choice: multiple answers allowed', 'tr_TR': 'Seçmeli: Birden Fazla Cevap', 'ar_001': 'اختيار من متعدد: يُسمح باختيار أكثر من إجابة'} |
| `text_box` | {'en_US': 'Multiple Lines Text Box', 'tr_TR': 'Çok Satırlı Metin Kutusu', 'ar_001': 'مربع نص متعدد الأسطر'} |
| `char_box` | {'en_US': 'Single Line Text Box', 'tr_TR': 'Tek Satırlı Metin Kutusu', 'ar_001': 'مربع نص سطر واحد'} |
| `numerical_box` | {'en_US': 'Numerical Value', 'tr_TR': 'Sayısal  Değer', 'ar_001': 'قيمة عددية'} |
| `scale` | {'en_US': 'Scale', 'tr_TR': 'Tartı', 'ar_001': 'الميزان'} |
| `date` | {'en_US': 'Date', 'tr_TR': 'Tarih', 'ar_001': 'التاريخ'} |
| `datetime` | {'en_US': 'Datetime', 'tr_TR': 'Tarih Saat', 'ar_001': 'التاريخ والوقت'} |
| `matrix` | {'en_US': 'Matrix', 'tr_TR': 'Matris', 'ar_001': 'المصفوفة'} |

## `survey.survey`

**`access_mode`** — {'en_US': 'Access Mode', 'tr_TR': 'Erişim Modu', 'ar_001': 'وضع الوصول'}

| Value | Label |
|---|---|
| `public` | {'en_US': 'Anyone with the link', 'tr_TR': 'Bağlantıya sahip olan herkes', 'ar_001': 'أي شخص لديه الرابط'} |
| `token` | {'en_US': 'Invited people only', 'tr_TR': 'Sadece Davet Edilen Kişiler', 'ar_001': 'الأشخاص المدعوين فقط'} |

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`certification_report_layout`** — {'en_US': 'Certification template', 'tr_TR': 'Sertifika şablonu', 'ar_001': 'قالب الشهادة'}

| Value | Label |
|---|---|
| `modern_purple` | {'en_US': 'Modern Purple', 'tr_TR': 'Modern Mor', 'ar_001': 'بنفسجي عصري'} |
| `modern_blue` | {'en_US': 'Modern Blue', 'tr_TR': 'Modern Mavi', 'ar_001': 'أزرق عصري'} |
| `modern_gold` | {'en_US': 'Modern Gold', 'tr_TR': 'Modern Altın', 'ar_001': 'ذهبي عصري'} |
| `classic_purple` | {'en_US': 'Classic Purple', 'tr_TR': 'Klasik Mor', 'ar_001': 'البنفسجي الكلاسيكي'} |
| `classic_blue` | {'en_US': 'Classic Blue', 'tr_TR': 'Klasik Mavi', 'ar_001': 'الأزرق الكلاسيكي'} |
| `classic_gold` | {'en_US': 'Classic Gold', 'tr_TR': 'Klasik Altın', 'ar_001': 'الذهبي الكلاسيكي'} |

**`progression_mode`** — {'en_US': 'Display Progress as', 'tr_TR': 'İlerlemeyi şu şekilde göster', 'ar_001': 'عرض التقدم كـ'}

| Value | Label |
|---|---|
| `percent` | {'en_US': 'Percentage left', 'tr_TR': 'Kalan yüzde', 'ar_001': 'نسبة الأسئلة المتبقية'} |
| `number` | {'en_US': 'Number', 'tr_TR': 'Numara', 'ar_001': 'عدد'} |

**`questions_layout`** — {'en_US': 'Pagination', 'tr_TR': 'Sayfalama', 'ar_001': 'ترقيم الصفحات'}

| Value | Label |
|---|---|
| `page_per_question` | {'en_US': 'One page per question', 'tr_TR': 'Soru başına bir sayfa', 'ar_001': 'صفحة واحدة لكل سؤال'} |
| `page_per_section` | {'en_US': 'One page per section', 'tr_TR': 'Bölüm başına bir sayfa', 'ar_001': 'صفحة واحدة لكل قسم'} |
| `one_page` | {'en_US': 'One page with all the questions', 'tr_TR': 'Tüm soruları içeren bir sayfa', 'ar_001': 'صفحة واحدة مع كل الأسئلة'} |

**`questions_selection`** — {'en_US': 'Question Selection', 'tr_TR': 'Soru Seçimi', 'ar_001': 'اختيار السؤال'}

| Value | Label |
|---|---|
| `all` | {'en_US': 'All questions', 'tr_TR': 'Tüm sorular', 'ar_001': 'كافة الأسئلة'} |
| `random` | {'en_US': 'Randomized per Section', 'tr_TR': 'Bölüm Başına Rastgele', 'ar_001': 'عشوائي لكل قسم'} |

**`scoring_type`** — {'en_US': 'Scoring', 'tr_TR': 'Puanlama', 'ar_001': 'حساب الدرجات'}

| Value | Label |
|---|---|
| `no_scoring` | {'en_US': 'No scoring', 'tr_TR': 'Puanlama yok', 'ar_001': 'لا توجد درجة'} |
| `scoring_with_answers_after_page` | {'en_US': 'Scoring with answers after each page', 'tr_TR': 'Sayfa başına verilen cevaplarla puanlama', 'ar_001': 'حساب الدرجات مع الإجابات بعد كل صفحة'} |
| `scoring_with_answers` | {'en_US': 'Scoring with answers at the end', 'tr_TR': 'Sonunda cevaplarla puanlama', 'ar_001': 'حساب الدرجات مع الإجابات في النهاية'} |
| `scoring_without_answers` | {'en_US': 'Scoring without answers', 'tr_TR': 'Cevaplar olmadan puanlama', 'ar_001': 'حساب الدرجات دون الإجابات'} |

**`session_state`** — {'en_US': 'Session State', 'tr_TR': 'Oturum Durumu', 'ar_001': 'حالة الجلسة'}

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready', 'tr_TR': 'Hazır', 'ar_001': 'جاهز'} |
| `in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |

**`survey_type`** — {'en_US': 'Survey Type', 'tr_TR': 'Anket Türü', 'ar_001': 'نوع الاستطلاع'}

| Value | Label |
|---|---|
| `survey` | {'en_US': 'Survey', 'tr_TR': 'Anket', 'ar_001': 'استطلاع'} |
| `live_session` | {'en_US': 'Live session', 'tr_TR': 'Canlı oturum', 'ar_001': 'جلسة مباشرة'} |
| `assessment` | {'en_US': 'Assessment', 'tr_TR': 'Değerlendirme', 'ar_001': 'التقييم'} |
| `custom` | {'en_US': 'Custom', 'tr_TR': 'Özel', 'ar_001': 'مُخصص'} |
| `recruitment` | {'en_US': 'Recruitment', 'tr_TR': 'İşe Alım', 'ar_001': 'التوظيف'} |

## `survey.user_input`

**`activity_exception_decoration`** — {'en_US': 'Activity Exception Decoration', 'tr_TR': 'Etkinlik İstisna Dekorasyonu', 'ar_001': 'زخرفة استثناء النشاط'} (not stored)

| Value | Label |
|---|---|
| `warning` | {'en_US': 'Alert', 'tr_TR': 'İkaz', 'ar_001': 'التنبيه'} |
| `danger` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

**`activity_state`** — {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'} (not stored)

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

**`state`** — {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}

| Value | Label |
|---|---|
| `new` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Completed', 'tr_TR': 'Tamamlandı', 'ar_001': 'مكتملة'} |

## `survey.user_input.line`

**`answer_type`** — {'en_US': 'Answer Type', 'tr_TR': 'Cevap Türü', 'ar_001': 'نوع الإجابة'}

| Value | Label |
|---|---|
| `text_box` | {'en_US': 'Free Text', 'tr_TR': 'Serbest metin', 'ar_001': 'نص حر'} |
| `char_box` | {'en_US': 'Text', 'tr_TR': 'Metin', 'ar_001': 'النص'} |
| `numerical_box` | {'en_US': 'Number', 'tr_TR': 'Numara', 'ar_001': 'عدد'} |
| `scale` | {'en_US': 'Number', 'tr_TR': 'Numara', 'ar_001': 'عدد'} |
| `date` | {'en_US': 'Date', 'tr_TR': 'Tarih', 'ar_001': 'التاريخ'} |
| `datetime` | {'en_US': 'Datetime', 'tr_TR': 'Tarih Saat', 'ar_001': 'التاريخ والوقت'} |
| `suggestion` | {'en_US': 'Suggestion', 'tr_TR': 'Öneri', 'ar_001': 'اقتراح'} |

## `theme.ir.asset`

**`directive`** — {'en_US': 'Directive', 'tr_TR': 'Yönerge', 'ar_001': 'التوجيه'}

| Value | Label |
|---|---|
| `append` | {'en_US': 'Append', 'tr_TR': 'Ekle', 'ar_001': 'إضافة'} |
| `prepend` | {'en_US': 'Prepend', 'tr_TR': 'Başına Ekle', 'ar_001': 'إضافة إلى البداية'} |
| `after` | {'en_US': 'After', 'tr_TR': 'Sonra', 'ar_001': 'بعد'} |
| `before` | {'en_US': 'Before', 'tr_TR': 'Önce', 'ar_001': 'قبل'} |
| `remove` | {'en_US': 'Remove', 'tr_TR': 'Kaldır', 'ar_001': 'إزالة'} |
| `replace` | {'en_US': 'Replace', 'tr_TR': 'Yerleştirme', 'ar_001': 'استبدال'} |
| `include` | {'en_US': 'Include', 'tr_TR': 'Dahil', 'ar_001': 'تضمين'} |

## `theme.ir.ui.view`

**`mode`** — {'en_US': 'Mode', 'tr_TR': 'Şekli', 'ar_001': 'الوضع'}

| Value | Label |
|---|---|
| `primary` | {'en_US': 'Base view', 'tr_TR': 'Temel görünüm', 'ar_001': 'واجهة العرض الأساسية'} |
| `extension` | {'en_US': 'Extension View', 'tr_TR': 'Uzantı Görünümü', 'ar_001': 'معاينة الامتداد'} |

## `timesheets.analysis.report`

**`timesheet_invoice_type`** — {'en_US': 'Billable Type', 'tr_TR': 'Faturalanabilir Tür', 'ar_001': 'النوع القابل للفوترة'}

| Value | Label |
|---|---|
| `billable_time` | {'en_US': 'Billed on Timesheets', 'tr_TR': 'Çalışma Çizelgesinden Faturalanan', 'ar_001': 'مفوتر في الجداول الزمنية'} |
| `billable_fixed` | {'en_US': 'Billed at a Fixed price', 'tr_TR': 'Sabit Fiyatla Faturalanan', 'ar_001': 'مفوترة بسعر ثابت'} |
| `billable_milestones` | {'en_US': 'Billed on Milestones', 'tr_TR': 'Kilometre Taşlarına Göre Faturalandırılır', 'ar_001': 'الفوترة حسب مؤشرات التقدم'} |
| `billable_manual` | {'en_US': 'Billed Manually', 'tr_TR': 'Manuel Faturalandırma', 'ar_001': 'تتم الفوترة يدوياً'} |
| `non_billable` | {'en_US': 'Non-Billable', 'tr_TR': 'Faturalandırılamaz', 'ar_001': 'غير قابلة للفوترة'} |
| `timesheet_revenues` | {'en_US': 'Timesheet Revenues', 'tr_TR': 'Çalışma Çizelge Gelirleri', 'ar_001': 'إيرادات الجداول الزمنية'} |
| `service_revenues` | {'en_US': 'Service Revenues', 'tr_TR': 'Hizmet Gelirleri', 'ar_001': 'إيرادات الخدمة'} |
| `other_revenues` | {'en_US': 'Other revenues', 'tr_TR': 'Diğer gelirler', 'ar_001': 'الإيرادات الأخرى'} |
| `other_costs` | {'en_US': 'Other costs', 'tr_TR': 'Diğer maliyetler', 'ar_001': 'التكاليف الأخرى'} |

## `uom.uom`

**`l10n_es_edi_facturae_uom_code`** — {'en_US': 'Spanish EDI Units'}

| Value | Label |
|---|---|
| `01` | {'en_US': 'Units'} |
| `02` | {'en_US': 'Hours'} |
| `03` | {'en_US': 'Kilograms'} |
| `04` | {'en_US': 'Liters'} |
| `05` | {'en_US': 'Other'} |
| `06` | {'en_US': 'Boxes'} |
| `07` | {'en_US': 'Trays, one layer no cover, plastic'} |
| `08` | {'en_US': 'Barrels'} |
| `09` | {'en_US': 'Jerricans, cylindrical'} |
| `10` | {'en_US': 'Bags'} |
| `11` | {'en_US': 'Carboys, non-protected'} |
| `12` | {'en_US': 'Bottles, non-protected, cylindrical'} |
| `13` | {'en_US': 'Canisters'} |
| `14` | {'en_US': 'Tetra Briks'} |
| `15` | {'en_US': 'Centiliters'} |
| `16` | {'en_US': 'Centimeters'} |
| `17` | {'en_US': 'Bins'} |
| `18` | {'en_US': 'Dozens'} |
| `19` | {'en_US': 'Cases'} |
| `20` | {'en_US': 'Demijohns, non-protected'} |
| `21` | {'en_US': 'Grams'} |
| `22` | {'en_US': 'Kilometers'} |
| `23` | {'en_US': 'Cans, rectangular'} |
| `24` | {'en_US': 'Bunches'} |
| `25` | {'en_US': 'Meters'} |
| `26` | {'en_US': 'Millimeters'} |
| `27` | {'en_US': '6-Packs'} |
| `28` | {'en_US': 'Packages'} |
| `29` | {'en_US': 'Portions'} |
| `30` | {'en_US': 'Rolls'} |
| `31` | {'en_US': 'Envelopes'} |
| `32` | {'en_US': 'Tubs'} |
| `33` | {'en_US': 'Cubic meter'} |
| `34` | {'en_US': 'Second'} |
| `35` | {'en_US': 'Watt'} |
| `36` | {'en_US': 'Kilowatt-hour'} |

**`l10n_hu_edi_code`** — {'en_US': 'NAV UoM code'}

| Value | Label |
|---|---|
| `PIECE` | {'en_US': 'Piece'} |
| `KILOGRAM` | {'en_US': 'Kilogram'} |
| `TON` | {'en_US': 'Ton'} |
| `KWH` | {'en_US': 'Kilowatt hour'} |
| `DAY` | {'en_US': 'Day'} |
| `HOUR` | {'en_US': 'Hour'} |
| `MINUTE` | {'en_US': 'Minute'} |
| `MONTH` | {'en_US': 'Month'} |
| `LITER` | {'en_US': 'Liter'} |
| `KILOMETER` | {'en_US': 'Kilometer'} |
| `CUBIC_METER` | {'en_US': 'Cubic meter'} |
| `METER` | {'en_US': 'Meter'} |
| `LINEAR_METER` | {'en_US': 'Linear meter'} |
| `CARTON` | {'en_US': 'Carton'} |
| `PACK` | {'en_US': 'Package'} |

## `update.product.attribute.value`

**`mode`** — {'en_US': 'Mode', 'tr_TR': 'Şekli', 'ar_001': 'الوضع'}

| Value | Label |
|---|---|
| `add` | {'en_US': 'Add to existing products', 'tr_TR': 'Mevcut ürünlere ekleme', 'ar_001': 'إضافة إلى المنتجات الموجودة'} |
| `update_extra_price` | {'en_US': 'Update the extra price on existing products', 'tr_TR': 'Mevcut ürünlerdeki ekstra fiyatı güncelleyin', 'ar_001': 'تحديث السعر الإضافي للمنتجات الموجودة'} |

## `utm.campaign`

**`ab_testing_sms_winner_selection`** — {'en_US': 'SMS Winner Selection', 'tr_TR': 'SMS Kazanan Seçimi', 'ar_001': 'اختيار الفائز عن طريق الرسائل النصية القصيرة'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `clicks_ratio` | {'en_US': 'Highest Click Rate', 'tr_TR': 'En Yüksek Tıklama Oranı', 'ar_001': 'أعلى معدل نقر'} |
| `crm_lead_count` | {'en_US': 'Leads', 'tr_TR': 'Adaylar', 'ar_001': 'العملاء المهتمين'} |
| `sale_quotation_count` | {'en_US': 'Quotations', 'tr_TR': 'Teklifler', 'ar_001': 'عروض الأسعار'} |
| `sale_invoiced_amount` | {'en_US': 'Revenues', 'tr_TR': 'Gelirler', 'ar_001': 'الإيرادات'} |

**`ab_testing_winner_selection`** — {'en_US': 'Winner Selection', 'tr_TR': 'Kazanan Seçimi', 'ar_001': 'اختيار الفائز'}

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Manual', 'tr_TR': 'Manuel', 'ar_001': 'يدوي'} |
| `opened_ratio` | {'en_US': 'Highest Open Rate', 'tr_TR': 'En Yüksek Açılma Oranı', 'ar_001': 'أعلى معدل فتح'} |
| `clicks_ratio` | {'en_US': 'Highest Click Rate', 'tr_TR': 'En Yüksek Tıklama Oranı', 'ar_001': 'أعلى معدل نقر'} |
| `replied_ratio` | {'en_US': 'Highest Reply Rate', 'tr_TR': 'En Yüksek Yanıt Oranı', 'ar_001': 'أعلى معدل رد'} |
| `crm_lead_count` | {'en_US': 'Leads', 'tr_TR': 'Adaylar', 'ar_001': 'العملاء المهتمين'} |
| `sale_quotation_count` | {'en_US': 'Quotations', 'tr_TR': 'Teklifler', 'ar_001': 'عروض الأسعار'} |
| `sale_invoiced_amount` | {'en_US': 'Revenues', 'tr_TR': 'Gelirler', 'ar_001': 'الإيرادات'} |

## `web_tour.tour.step`

**`tooltip_position`** — {'en_US': 'Tooltip Position', 'tr_TR': 'Araç İpucu Konumu', 'ar_001': 'موقع تلميح الأدوات'}

| Value | Label |
|---|---|
| `bottom` | {'en_US': 'Bottom', 'tr_TR': 'Alt', 'ar_001': 'الأسفل'} |
| `top` | {'en_US': 'Top', 'tr_TR': 'Üst', 'ar_001': 'الأعلى'} |
| `right` | {'en_US': 'Right', 'tr_TR': 'Sağ', 'ar_001': 'يمين'} |
| `left` | {'en_US': 'left', 'tr_TR': 'sol', 'ar_001': 'يسار'} |

## `website`

**`account_on_checkout`** — {'en_US': 'Customer Accounts', 'tr_TR': 'Müşteri Hesapları', 'ar_001': 'حسابات العملاء'}

| Value | Label |
|---|---|
| `optional` | {'en_US': 'Optional', 'tr_TR': 'İsteğe bağlı', 'ar_001': 'اختياري'} |
| `disabled` | {'en_US': 'Disabled (buy as guest)', 'tr_TR': 'Engelli (misafir olarak satın alın)', 'ar_001': 'معطل (الشراء كضيف)'} |
| `mandatory` | {'en_US': 'Mandatory (no guest checkout)', 'tr_TR': 'Zorunlu (misafir ödemesi yok)', 'ar_001': 'إلزامي (لا يمكن الدفع والخروج كضيف)'} |

**`add_to_cart_action`** — {'en_US': 'Add To Cart Action', 'tr_TR': 'Sepete Ekle Eylemi', 'ar_001': 'إجراء الإضافة إلى عربة التسوق'}

| Value | Label |
|---|---|
| `stay` | {'en_US': 'Stay on Product Page', 'tr_TR': 'Ürün Sayfasında Kalın', 'ar_001': 'البقاء في صفحة المنتج'} |
| `go_to_cart` | {'en_US': 'Go to cart', 'tr_TR': 'Sepete git', 'ar_001': 'الذهاب إلى عربة التسوق'} |

**`auth_signup_uninvited`** — {'en_US': 'Customer Account', 'tr_TR': 'Müşteri Hesabı', 'ar_001': 'حساب العميل'}

| Value | Label |
|---|---|
| `b2b` | {'en_US': 'On invitation', 'tr_TR': 'Davetiye ile', 'ar_001': 'بدعوة'} |
| `b2c` | {'en_US': 'Free sign up', 'tr_TR': 'Ücretsiz kaydolma', 'ar_001': 'تسجيل مجاني'} |

**`ecommerce_access`** — {'en_US': 'Ecommerce Access', 'tr_TR': 'E-ticaret Erişimi', 'ar_001': 'الوصول إلى المتجر الإلكتروني'}

| Value | Label |
|---|---|
| `everyone` | {'en_US': 'All users', 'tr_TR': 'Tüm kullanıcılar', 'ar_001': 'كافة المستخدمين'} |
| `logged_in` | {'en_US': 'Logged in users', 'tr_TR': 'Giriş yapan kullanıcılar', 'ar_001': 'المستخدمون المسجلون'} |

**`product_page_cols_order`** — {'en_US': 'Product Page main columns order', 'tr_TR': 'Ürün Sayfası ana sütun sırası'}

| Value | Label |
|---|---|
| `regular` | {'en_US': 'Regular order', 'tr_TR': 'Düzenli sipariş'} |
| `inverse` | {'en_US': 'Inverse order', 'tr_TR': 'Ters düzen'} |

**`product_page_container`** — {'en_US': 'Product Page Container', 'tr_TR': 'Ürün Sayfası Konteyner'}

| Value | Label |
|---|---|
| `unset` | {'en_US': 'Unset', 'tr_TR': 'Ayarlanmamış', 'ar_001': 'إلغاء التعيين'} |
| `regular` | {'en_US': 'Regular', 'tr_TR': 'Düzenli', 'ar_001': 'منتظم'} |
| `fluid` | {'en_US': 'Full-width', 'tr_TR': 'Tam genişlik', 'ar_001': 'العرض الكامل'} |

**`product_page_image_layout`** — {'en_US': 'Product Page Image Layout', 'tr_TR': 'Ürün Sayfası Resim Düzeni', 'ar_001': 'تخطيط الصورة في صفحة المنتج'}

| Value | Label |
|---|---|
| `carousel` | {'en_US': 'Carousel', 'tr_TR': 'Carousel', 'ar_001': 'منصّة عرض بعناصر متغيّرة'} |
| `grid` | {'en_US': 'Grid', 'tr_TR': 'Tablo', 'ar_001': 'الشبكة'} |

**`product_page_image_ratio`** — {'en_US': 'Product Page Image Ratio', 'tr_TR': 'Ürün Sayfası Görüntü Oranı'}

| Value | Label |
|---|---|
| `auto` | {'en_US': 'Auto', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |
| `21_9` | {'en_US': 'Wider (21/9)', 'tr_TR': 'Daha Geniş (21/9)', 'ar_001': 'أعرض (21/9)'} |
| `16_9` | {'en_US': 'Wide (16/9)', 'tr_TR': 'Geniş (16/9)', 'ar_001': 'عريض (16/9)'} |
| `4_3` | {'en_US': 'Landscape (4/3)', 'tr_TR': 'Manzara (4/3)', 'ar_001': 'أفقي (4/3)'} |
| `6_5` | {'en_US': 'Horizontal (6/5)', 'tr_TR': 'Yatay (6/5)'} |
| `1_1` | {'en_US': 'Default (1/1)', 'tr_TR': 'Varsayılan (1/1)', 'ar_001': 'الافتراضي (1/1)'} |
| `4_5` | {'en_US': 'Portrait (4/5)', 'tr_TR': 'Portre (4/5)', 'ar_001': 'أفقي (4/5)'} |
| `2_3` | {'en_US': 'Vertical (2/3)', 'tr_TR': 'Dikey (2/3)', 'ar_001': 'عمودي (2/3)'} |

**`product_page_image_ratio_mobile`** — {'en_US': 'Product Page Image Ratio Mobile', 'tr_TR': 'Ürün Sayfası Görsel Oranı Mobil'}

| Value | Label |
|---|---|
| `auto` | {'en_US': 'Auto', 'tr_TR': 'Otomatik', 'ar_001': 'تلقائي'} |
| `21_9` | {'en_US': 'Wider (21/9)', 'tr_TR': 'Daha Geniş (21/9)', 'ar_001': 'أعرض (21/9)'} |
| `16_9` | {'en_US': 'Wide (16/9)', 'tr_TR': 'Geniş (16/9)', 'ar_001': 'عريض (16/9)'} |
| `4_3` | {'en_US': 'Landscape (4/3)', 'tr_TR': 'Manzara (4/3)', 'ar_001': 'أفقي (4/3)'} |
| `6_5` | {'en_US': 'Horizontal (6/5)', 'tr_TR': 'Yatay (6/5)'} |
| `1_1` | {'en_US': 'Default (1/1)', 'tr_TR': 'Varsayılan (1/1)', 'ar_001': 'الافتراضي (1/1)'} |
| `4_5` | {'en_US': 'Portrait (4/5)', 'tr_TR': 'Portre (4/5)', 'ar_001': 'أفقي (4/5)'} |
| `2_3` | {'en_US': 'Vertical (2/3)', 'tr_TR': 'Dikey (2/3)', 'ar_001': 'عمودي (2/3)'} |

**`product_page_image_roundness`** — {'en_US': 'Product Page Image Roundness', 'tr_TR': 'Ürün Sayfası Görsel Yuvarlaklığı'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |
| `small` | {'en_US': 'Small', 'tr_TR': 'Küçük', 'ar_001': 'صغير'} |
| `medium` | {'en_US': 'Medium', 'tr_TR': 'Ortam', 'ar_001': 'متوسط'} |
| `big` | {'en_US': 'Big', 'tr_TR': 'Büyük', 'ar_001': 'كبير'} |

**`product_page_image_spacing`** — {'en_US': 'Product Page Image Spacing', 'tr_TR': 'Ürün Sayfası Resim Aralığı', 'ar_001': 'تباعد الصورة في صفحة المنتج'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'None', 'tr_TR': 'Hiçbiri', 'ar_001': 'لا شيء'} |
| `small` | {'en_US': 'Small', 'tr_TR': 'Küçük', 'ar_001': 'صغير'} |
| `medium` | {'en_US': 'Medium', 'tr_TR': 'Ortam', 'ar_001': 'متوسط'} |
| `big` | {'en_US': 'Big', 'tr_TR': 'Büyük', 'ar_001': 'كبير'} |

**`product_page_image_width`** — {'en_US': 'Product Page Image Width', 'tr_TR': 'Ürün Sayfası Resim Genişliği', 'ar_001': 'عرض الصورة في صفحة المنتج'}

| Value | Label |
|---|---|
| `none` | {'en_US': 'Hidden', 'tr_TR': 'Gizli', 'ar_001': 'مخفي'} |
| `33_pc` | {'en_US': '33 %'} |
| `50_pc` | {'en_US': '50 %'} |
| `66_pc` | {'en_US': '66 %'} |
| `100_pc` | {'en_US': '100 %'} |

**`shop_page_container`** — {'en_US': 'Shop Page Container', 'tr_TR': 'Mağaza Sayfası Konteyner'}

| Value | Label |
|---|---|
| `regular` | {'en_US': 'Regular', 'tr_TR': 'Düzenli', 'ar_001': 'منتظم'} |
| `fluid` | {'en_US': 'Full-width', 'tr_TR': 'Tam genişlik', 'ar_001': 'العرض الكامل'} |

**`show_line_subtotals_tax_selection`** — {'en_US': 'Line Subtotals Tax Display', 'tr_TR': 'Satır Alt Toplamları Vergi Görüntüleme', 'ar_001': 'عرض ضريبة المجاميع الفرعية للبنود'}

| Value | Label |
|---|---|
| `tax_excluded` | {'en_US': 'Tax Excluded', 'tr_TR': 'Vergi Hariç', 'ar_001': 'غير شامل الضريبة'} |
| `tax_included` | {'en_US': 'Tax Included', 'tr_TR': 'Vergi Dahil', 'ar_001': 'شامل الضريبة'} |

## `website.controller.page`

**`default_layout`** — {'en_US': 'Default Layout', 'tr_TR': 'Varsayılan Düzen', 'ar_001': 'التخطيط الافتراضي'}

| Value | Label |
|---|---|
| `grid` | {'en_US': 'Grid', 'tr_TR': 'Tablo', 'ar_001': 'الشبكة'} |
| `list` | {'en_US': 'List', 'tr_TR': 'Liste', 'ar_001': 'القائمة'} |

## `website.event.menu`

**`menu_type`** — {'en_US': 'Menu Type', 'tr_TR': 'Menü Türü', 'ar_001': 'نوع القائمة'}

| Value | Label |
|---|---|
| `community` | {'en_US': 'Community Menu', 'tr_TR': 'Topluluk Menüsü', 'ar_001': 'قائمة المجتمع'} |
| `introduction` | {'en_US': 'Home', 'tr_TR': 'Ana Sayfa', 'ar_001': 'الرئيسية'} |
| `register` | {'en_US': 'Practical', 'tr_TR': 'Uygulamalı', 'ar_001': 'عملي'} |
| `other` | {'en_US': 'Other', 'tr_TR': 'Diğer', 'ar_001': 'غير ذلك'} |
| `booth` | {'en_US': 'Event Booth Menus', 'tr_TR': 'Etkinlik Standı Menüleri', 'ar_001': 'قوائم أجنحة الفعالية'} |
| `exhibitor` | {'en_US': 'Exhibitors Menus', 'tr_TR': 'Sergileyiciler Menüleri', 'ar_001': 'قوائم العارضين'} |
| `track` | {'en_US': 'Event Tracks Menus', 'tr_TR': 'Etkinlik İzleme Menüleri', 'ar_001': 'قوائم مسارات الفعالية'} |
| `track_proposal` | {'en_US': 'Event Proposals Menus', 'tr_TR': 'Etkinlik Teklifleri Menüleri', 'ar_001': 'قوائم مقترحات الفعالية'} |

## `website.page.properties`

**`redirect_type`** — {'en_US': 'Redirect Type', 'tr_TR': 'Yönlendirme Türü', 'ar_001': 'نوع إعادة التوجيه'} (not stored)

| Value | Label |
|---|---|
| `301` | {'en_US': '301 Moved permanently', 'tr_TR': '301 Kalıcı olarak taşındı', 'ar_001': '301 تم نقله بشكل دائم'} |
| `302` | {'en_US': '302 Moved temporarily', 'tr_TR': '302 Geçici olarak taşındı', 'ar_001': '302 تم نقله بشكل مؤقت'} |

## `website.rewrite`

**`redirect_type`** — {'en_US': 'Action', 'tr_TR': 'Aksiyon', 'ar_001': 'إجراء'}

| Value | Label |
|---|---|
| `404` | {'en_US': '404 Not Found', 'tr_TR': '404 Bulunamadı', 'ar_001': '404 لم يتم العثور عليه'} |
| `301` | {'en_US': '301 Moved permanently', 'tr_TR': '301 Kalıcı olarak taşındı', 'ar_001': '301 تم نقله بشكل دائم'} |
| `302` | {'en_US': '302 Moved temporarily', 'tr_TR': '302 Geçici olarak taşındı', 'ar_001': '302 تم نقله بشكل مؤقت'} |
| `308` | {'en_US': '308 Redirect / Rewrite', 'tr_TR': '308 Yönlendirme / Yeniden Yazma', 'ar_001': '308 إعادة توجيه / إعادة كتابة'} |

