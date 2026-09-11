# State fields

Every state field carried by every entity: 252 fields across 146 entities. This page lists the states themselves. The transitions between them, their guards and their side effects are specified in each domain's state machine document, linked in the last column.

A state field is stored as one of a fixed set of values. The value is what is persisted and what any external contract sees, so it is reproduced exactly; the label is what a person reads.

## Summary by domain

| Domain | State fields | Entities carrying one |
|---|---|---|
| attendances-and-working-time | 2 | 2 |
| automation-and-integration | 7 | 7 |
| calendar-and-scheduling | 2 | 1 |
| customer-relationship-management | 7 | 4 |
| electronic-invoicing-and-document-exchange | 3 | 2 |
| events | 14 | 7 |
| expenses | 4 | 2 |
| fiscal-localizations | 22 | 14 |
| fleet | 7 | 4 |
| general-ledger | 31 | 6 |
| human-resources-core | 9 | 6 |
| inventory-operations | 12 | 6 |
| inventory-valuation-and-costing | 2 | 1 |
| learning-surveys-and-gamification | 8 | 6 |
| lunch-ordering | 2 | 2 |
| manufacturing | 9 | 5 |
| marketing-and-mass-mailing | 6 | 5 |
| messaging-and-activities | 14 | 14 |
| multi-currency | 28 | 16 |
| payment-providers | 3 | 3 |
| payments-and-bank-reconciliation | 2 | 1 |
| point-of-sale | 9 | 4 |
| pricing-and-pricelists | 1 | 1 |
| products-and-catalog | 4 | 3 |
| projects-and-tasks | 9 | 6 |
| purchasing | 7 | 3 |
| recruitment | 3 | 1 |
| repair-and-maintenance | 6 | 3 |
| sales | 8 | 3 |
| time-off | 8 | 5 |
| website-and-storefront | 2 | 2 |
| work-entries | 1 | 1 |

## attendances-and-working-time

### Attendance — `overtime_status`

Entity `hr.attendance`, table `hr_attendance`. Label: {'en_US': 'Overtime Status', 'tr_TR': 'Fazla Mesai Durumu', 'ar_001': 'حالة ساعات العمل الإصافية'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [attendances-and-working-time state machines](../domains/attendances-and-working-time/state-machines.md).

### Attendance Overtime Line — `status`

Entity `hr.attendance.overtime.line`, table `hr_attendance_overtime_line`. Label: {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}; stored; required.

| Value | Label |
|---|---|
| `to_approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [attendances-and-working-time state machines](../domains/attendances-and-working-time/state-machines.md).

## automation-and-integration

### Automation Rule — `activity_state`

Entity `base.automation`, table `base_automation`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Import Module — `state`

Entity `base.import.module`, table `base_import_module`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `init` | {'en_US': 'init', 'tr_TR': 'başlatma', 'ar_001': 'إطلاق'} |
| `done` | {'en_US': 'done', 'tr_TR': 'biten', 'ar_001': 'تم'} |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### in-app purchase Account — `state`

Entity `iap.account`, table `iap_account`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `banned` | {'en_US': 'Banned', 'tr_TR': 'Yasaklı', 'ar_001': 'محظور'} |
| `registered` | {'en_US': 'Registered', 'tr_TR': 'Kayıtlı', 'ar_001': 'مُسجل'} |
| `unregistered` | {'en_US': 'Unregistered', 'tr_TR': 'Kaydedilmemiş', 'ar_001': 'غير مسجل'} |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding — `current_onboarding_state`

Entity `onboarding.onboarding`, table `onboarding_onboarding`. Label: {'en_US': 'Completion State', 'tr_TR': 'Tamamlanma Durumu', 'ar_001': 'حالة الإنجاز'}; not stored; read only.

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding Step — `current_step_state`

Entity `onboarding.onboarding.step`, table `onboarding_onboarding_step`. Label: {'en_US': 'Completion State', 'tr_TR': 'Tamamlanma Durumu', 'ar_001': 'حالة الإنجاز'}; not stored; read only.

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding Progress Tracker — `onboarding_state`

Entity `onboarding.progress`, table `onboarding_progress`. Label: {'en_US': 'Onboarding progress', 'tr_TR': 'Katılım ilerlemesi', 'ar_001': 'تقدم التأهيل'}; stored; read only.

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding Progress Step Tracker — `step_state`

Entity `onboarding.progress.step`, table `onboarding_progress_step`. Label: {'en_US': 'Onboarding Step Progress', 'tr_TR': 'Katılım Adım İlerlemesi', 'ar_001': 'تقدم خطوة التأهيل'}; stored.

| Value | Label |
|---|---|
| `not_done` | {'en_US': 'Not done', 'tr_TR': 'Yapılmadı', 'ar_001': 'غير منتهي'} |
| `just_done` | {'en_US': 'Just done', 'tr_TR': 'Yeni bitti', 'ar_001': 'انتهت للتو'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

## calendar-and-scheduling

### Calendar Attendee Information — `availability`

Entity `calendar.attendee`, table `calendar_attendee`. Label: {'en_US': 'Available/Busy', 'tr_TR': 'Müsait/Meşgul', 'ar_001': 'متاح/مشغول'}; stored; read only.

| Value | Label |
|---|---|
| `free` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `busy` | {'en_US': 'Busy', 'tr_TR': 'Meşgul', 'ar_001': 'مشغول'} |

Transitions: [calendar-and-scheduling state machines](../domains/calendar-and-scheduling/state-machines.md).

### Calendar Attendee Information — `state`

Entity `calendar.attendee`, table `calendar_attendee`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored.

| Value | Label |
|---|---|
| `accepted` | {'en_US': 'Yes', 'tr_TR': 'Evet', 'ar_001': 'نعم'} |
| `declined` | {'en_US': 'No', 'tr_TR': 'Hayır', 'ar_001': 'لا'} |
| `tentative` | {'en_US': 'Maybe', 'tr_TR': 'Belki', 'ar_001': 'ربما'} |
| `needsAction` | {'en_US': 'Needs Action', 'tr_TR': 'Eylem Gerektiriyor', 'ar_001': 'يتطلب اتخاذ إجراء'} |

Transitions: [calendar-and-scheduling state machines](../domains/calendar-and-scheduling/state-machines.md).

## customer-relationship-management

### customer relationship management Activity Analysis — `won_status`

Entity `crm.activity.report`, table `crm_activity_report`. Label: {'en_US': 'Is Won', 'tr_TR': 'Kazanıldı', 'ar_001': 'رابحة'}; stored; read only.

| Value | Label |
|---|---|
| `won` | {'en_US': 'Won', 'tr_TR': 'Kazanıldı', 'ar_001': 'تم الفوز بها'} |
| `lost` | {'en_US': 'Lost', 'tr_TR': 'Kayıp', 'ar_001': 'ضائع'} |
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### customer relationship management Lead Mining Request — `state`

Entity `crm.iap.lead.mining.request`, table `crm_iap_lead_mining_request`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `activity_state`

Entity `crm.lead`, table `crm_lead`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `email_state`

Entity `crm.lead`, table `crm_lead`. Label: {'en_US': 'Email Quality', 'tr_TR': 'E-Posta Kalitesi', 'ar_001': 'جودة البريد الإلكتروني'}; stored; read only.

| Value | Label |
|---|---|
| `correct` | {'en_US': 'Correct', 'tr_TR': 'Doğru', 'ar_001': 'صحيحة'} |
| `incorrect` | {'en_US': 'Incorrect', 'tr_TR': 'Yanlış', 'ar_001': 'غير صحيح'} |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `phone_state`

Entity `crm.lead`, table `crm_lead`. Label: {'en_US': 'Phone Quality', 'tr_TR': 'Telefon Kalitesi', 'ar_001': 'جودة الهاتف'}; stored; read only.

| Value | Label |
|---|---|
| `correct` | {'en_US': 'Correct', 'tr_TR': 'Doğru', 'ar_001': 'صحيحة'} |
| `incorrect` | {'en_US': 'Incorrect', 'tr_TR': 'Yanlış', 'ar_001': 'غير صحيح'} |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `won_status`

Entity `crm.lead`, table `crm_lead`. Label: {'en_US': 'Won/Lost', 'tr_TR': 'Kazanılan / Kaybedilen', 'ar_001': 'رابحة/ضائعة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `won` | {'en_US': 'Won', 'tr_TR': 'Kazanıldı', 'ar_001': 'تم الفوز بها'} |
| `lost` | {'en_US': 'Lost', 'tr_TR': 'Kayıp', 'ar_001': 'ضائع'} |
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### customer relationship management Reveal View — `reveal_state`

Entity `crm.reveal.view`, table `crm_reveal_view`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}; stored.

| Value | Label |
|---|---|
| `to_process` | {'en_US': 'To Process', 'tr_TR': 'İşlenecek', 'ar_001': 'للمعالجة'} |
| `not_found` | {'en_US': 'Not Found', 'tr_TR': 'Bulunamadı', 'ar_001': 'لم يتم العثور عليه'} |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

## electronic-invoicing-and-document-exchange

### Electronic Document for an account.move — `state`

Entity `account.edi.document`, table `account_edi_document`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}; stored.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'tr_TR': 'Giden', 'ar_001': 'بانتظار الإرسال'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `to_cancel` | {'en_US': 'To Cancel', 'tr_TR': 'İptal etmek için', 'ar_001': 'للإلغاء'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [electronic-invoicing-and-document-exchange state machines](../domains/electronic-invoicing-and-document-exchange/state-machines.md).

### Business Level Responses for Peppol — `pdp_ppf_state`

Entity `account.peppol.response`, table `account_peppol_response`. Label: {'en_US': 'PPF Status'}; stored; read only.

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent'} |
| `received` | {'en_US': 'received'} |
| `error` | {'en_US': 'Error'} |

Transitions: [electronic-invoicing-and-document-exchange state machines](../domains/electronic-invoicing-and-document-exchange/state-machines.md).

### Business Level Responses for Peppol — `peppol_state`

Entity `account.peppol.response`, table `account_peppol_response`. Label: {'en_US': 'Peppol status', 'ar_001': 'Peppol status'}; stored.

| Value | Label |
|---|---|
| `processing` | {'en_US': 'Pending Reception', 'ar_001': 'بانتظار الاستقبال'} |
| `done` | {'en_US': 'Done', 'ar_001': 'تم الانتهاء '} |
| `error` | {'en_US': 'Error', 'ar_001': 'خطأ'} |
| `not_serviced` | {'en_US': 'Not Serviced', 'ar_001': 'غير متوفر'} |

Transitions: [electronic-invoicing-and-document-exchange state machines](../domains/electronic-invoicing-and-document-exchange/state-machines.md).

## events

### Event Booth — `activity_state`

Entity `event.booth`, table `event_booth`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Booth — `state`

Entity `event.booth`, table `event_booth`. Label: {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `unavailable` | {'en_US': 'Unavailable', 'tr_TR': 'Kullanım dışı', 'ar_001': 'غير متاح'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event — `activity_state`

Entity `event.event`, table `event_event`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event — `kanban_state`

Entity `event.event`, table `event_event`. Label: {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `normal` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Ready for Next Stage', 'tr_TR': 'Bir Sonraki Aşama için Hazır', 'ar_001': 'جاهز للمرحلة التالية'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Automated Mailing — `mail_state`

Entity `event.mail`, table `event_mail`. Label: {'en_US': 'Global communication Status', 'tr_TR': 'Küresel iletişim Durumu', 'ar_001': 'حالة التواصل العالمي'}; not stored; read only.

| Value | Label |
|---|---|
| `running` | {'en_US': 'Running', 'tr_TR': 'Devam Eden', 'ar_001': 'جاري'} |
| `scheduled` | {'en_US': 'Scheduled', 'tr_TR': 'Planlandı', 'ar_001': 'تمت الجدولة'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Registration — `activity_state`

Entity `event.registration`, table `event_registration`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Registration — `sale_status`

Entity `event.registration`, table `event_registration`. Label: {'en_US': 'Sale Status', 'tr_TR': 'Satış Durumu', 'ar_001': 'حالة البيع'}; stored; read only.

| Value | Label |
|---|---|
| `to_pay` | {'en_US': 'Not Sold', 'tr_TR': 'Satılmadı', 'ar_001': 'لم تباع'} |
| `sold` | {'en_US': 'Sold', 'tr_TR': 'Satılan', 'ar_001': 'المبيعات'} |
| `free` | {'en_US': 'Free', 'tr_TR': 'Ücretsiz', 'ar_001': 'مجاني'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Registration — `state`

Entity `event.registration`, table `event_registration`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Unconfirmed', 'tr_TR': 'Onaysız', 'ar_001': 'غير مؤكد'} |
| `open` | {'en_US': 'Registered', 'tr_TR': 'Kayıtlı', 'ar_001': 'مُسجل'} |
| `done` | {'en_US': 'Attended', 'tr_TR': 'Katıldı', 'ar_001': 'حضر'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sales Report — `event_registration_state`

Entity `event.sale.report`, table `event_sale_report`. Label: {'en_US': 'Registration Status', 'tr_TR': 'Kayıt durumu', 'ar_001': 'حالة التسجيل'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Unconfirmed', 'tr_TR': 'Onaysız', 'ar_001': 'غير مؤكد'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `open` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `done` | {'en_US': 'Attended', 'tr_TR': 'Katıldı', 'ar_001': 'حضر'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sales Report — `sale_order_state`

Entity `event.sale.report`, table `event_sale_report`. Label: {'en_US': 'Sale Order Status', 'tr_TR': 'Satış Siparişi Durumu', 'ar_001': 'حالة أمر البيع'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Quotation', 'tr_TR': 'Teklif', 'ar_001': 'عرض سعر'} |
| `sent` | {'en_US': 'Quotation Sent', 'tr_TR': 'Teklif Gönderildi', 'ar_001': 'تم إرسال عرض السعر'} |
| `sale` | {'en_US': 'Sales Order', 'tr_TR': 'Satış Siparişi', 'ar_001': 'أمر البيع'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sales Report — `sale_status`

Entity `event.sale.report`, table `event_sale_report`. Label: {'en_US': 'Payment Status', 'tr_TR': 'Ödeme Durumu', 'ar_001': 'حالة الدفع'}; stored.

| Value | Label |
|---|---|
| `to_pay` | {'en_US': 'Not Sold', 'tr_TR': 'Satılmadı', 'ar_001': 'لم تباع'} |
| `sold` | {'en_US': 'Sold', 'tr_TR': 'Satılan', 'ar_001': 'المبيعات'} |
| `free` | {'en_US': 'Free', 'tr_TR': 'Ücretsiz', 'ar_001': 'مجاني'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sponsor — `activity_state`

Entity `event.sponsor`, table `event_sponsor`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Track — `activity_state`

Entity `event.track`, table `event_track`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Track — `kanban_state`

Entity `event.track`, table `event_track`. Label: {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}; stored; required.

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Grey', 'tr_TR': 'Gri', 'ar_001': 'رمادي'} |
| `done` | {'en_US': 'Green', 'tr_TR': 'Yeşil', 'ar_001': 'أخضر'} |
| `blocked` | {'en_US': 'Red', 'tr_TR': 'Kırmızı', 'ar_001': 'أحمر'} |

Transitions: [events state machines](../domains/events/state-machines.md).

## expenses

### Expense — `activity_state`

Entity `hr.expense`, table `hr_expense`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

### Expense — `approval_state`

Entity `hr.expense`, table `hr_expense`. Label: {'en_US': 'Approval State', 'tr_TR': 'Onay Durumu', 'ar_001': 'حالة الموافقة'}; stored; read only.

| Value | Label |
|---|---|
| `submitted` | {'en_US': 'Submitted', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

### Expense — `state`

Entity `hr.expense`, table `hr_expense`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `submitted` | {'en_US': 'Submitted', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `posted` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |
| `in_payment` | {'en_US': 'In Payment', 'tr_TR': 'Ödeme', 'ar_001': 'بانتظار التسوية'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

### Expense Split — `approval_state`

Entity `hr.expense.split`, table `hr_expense_split`. Label: {'en_US': 'Approval State', 'tr_TR': 'Onay Durumu', 'ar_001': 'حالة الموافقة'}; stored; read only.

| Value | Label |
|---|---|
| `submitted` | {'en_US': 'Submitted', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

## fiscal-localizations

### French PDP Flow — `activity_state`

Entity `l10n.fr.pdp.reports.flow`, table `l10n_fr_pdp_reports_flow`. Label: {'en_US': 'Activity State'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### French PDP Flow — `period_status`

Entity `l10n.fr.pdp.reports.flow`, table `l10n_fr_pdp_reports_flow`. Label: {'en_US': 'Period Status'}; not stored; read only.

| Value | Label |
|---|---|
| `open` | {'en_US': 'Open'} |
| `grace` | {'en_US': 'Grace'} |
| `closed` | {'en_US': 'Closed'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### French PDP Flow — `state`

Entity `l10n.fr.pdp.reports.flow`, table `l10n_fr_pdp_reports_flow`. Label: {'en_US': 'Status'}; stored; required.

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready'} |
| `error` | {'en_US': 'Error'} |
| `sent` | {'en_US': 'Sent'} |
| `completed` | {'en_US': 'Completed'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### e-Waybill — `activity_state`

Entity `l10n.in.ewaybill`, table `l10n_in_ewaybill`. Label: {'en_US': 'Activity State'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### e-Waybill — `state`

Entity `l10n.in.ewaybill`, table `l10n_in_ewaybill`. Label: {'en_US': 'Status'}; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Pending'} |
| `generated` | {'en_US': 'Generated'} |
| `cancel` | {'en_US': 'Cancelled'} |
| `challan` | {'en_US': 'Challan'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### TicketBAI Document — `state`

Entity `l10n_es_edi_tbai.document`, table `l10n_es_edi_tbai_document`. Label: {'en_US': 'status'}; stored; read only.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `accepted` | {'en_US': 'Accepted'} |
| `rejected` | {'en_US': 'Rejected'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Veri*Factu Document — `state`

Entity `l10n_es_edi_verifactu.document`, table `l10n_es_edi_verifactu_document`. Label: {'en_US': 'Status'}; stored; read only.

| Value | Label |
|---|---|
| `rejected` | {'en_US': 'Rejected'} |
| `registered_with_errors` | {'en_US': 'Registered with Errors'} |
| `accepted` | {'en_US': 'Accepted'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Greece document object for tracking all sent extensible markup language to myDATA — `provider_pdf_state`

Entity `l10n_gr_edi.document`, table `l10n_gr_edi_document`. Label: {'en_US': 'Final PDF Status'}; stored.

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Pending'} |
| `sent` | {'en_US': 'Sent'} |
| `error` | {'en_US': 'Failed'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Greece document object for tracking all sent extensible markup language to myDATA — `state`

Entity `l10n_gr_edi.document`, table `l10n_gr_edi_document`. Label: {'en_US': 'myDATA Status'}; stored; required.

| Value | Label |
|---|---|
| `invoice_sent` | {'en_US': 'Invoice sent'} |
| `invoice_error` | {'en_US': 'Invoice send failed'} |
| `bill_fetched` | {'en_US': 'Expense classification ready to send'} |
| `bill_sent` | {'en_US': 'Expense classification sent'} |
| `bill_error` | {'en_US': 'Expense classification send failed'} |
| `invoice_pending` | {'en_US': 'Invoice submission pending'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### electronic data interchange and fiscalization information for Croatian electronic invoicing — `business_document_status`

Entity `l10n_hr_edi.addendum`, table `l10n_hr_edi_addendum`. Label: {'en_US': 'Business document status'}; stored.

| Value | Label |
|---|---|
| `0` | {'en_US': 'APPROVED'} |
| `1` | {'en_US': 'REJECTED'} |
| `2` | {'en_US': 'PAYMENT_FULFILLED'} |
| `3` | {'en_US': 'PAYMENT_PARTIALLY_FULLFILLED'} |
| `4` | {'en_US': 'RECEIVING_CONFIRMED'} |
| `99` | {'en_US': 'RECEIVED'} |
| `None` | {'en_US': 'None'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### electronic data interchange and fiscalization information for Croatian electronic invoicing — `fiscalization_status`

Entity `l10n_hr_edi.addendum`, table `l10n_hr_edi_addendum`. Label: {'en_US': 'Fiscalization status'}; stored.

| Value | Label |
|---|---|
| `0` | {'en_US': 'Successful'} |
| `1` | {'en_US': 'Unsuccessful'} |
| `2` | {'en_US': 'Pending'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### electronic data interchange and fiscalization information for Croatian electronic invoicing — `mer_document_status`

Entity `l10n_hr_edi.addendum`, table `l10n_hr_edi_addendum`. Label: {'en_US': 'MojEracun document status'}; stored.

| Value | Label |
|---|---|
| `20` | {'en_US': 'In validation'} |
| `30` | {'en_US': 'Sent'} |
| `40` | {'en_US': 'Delivered'} |
| `45` | {'en_US': 'Canceled'} |
| `50` | {'en_US': 'Unsuccessful'} |
| `70` | {'en_US': 'Delivered (eReporting)'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### E-Faktur Document — `activity_state`

Entity `l10n_id_efaktur_coretax.document`, table `l10n_id_efaktur_coretax_document`. Label: {'en_US': 'Activity State'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Indian permanent account number Entity — `activity_state`

Entity `l10n_in.pan.entity`, table `l10n_in_pan_entity`. Label: {'en_US': 'Activity State'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Declaration of Intent — `activity_state`

Entity `l10n_it_edi_doi.declaration_of_intent`, table `l10n_it_edi_doi_declaration_of_intent`. Label: {'en_US': 'Activity State'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Declaration of Intent — `state`

Entity `l10n_it_edi_doi.declaration_of_intent`, table `l10n_it_edi_doi_declaration_of_intent`. Label: {'en_US': 'State'}; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft'} |
| `active` | {'en_US': 'Active'} |
| `revoked` | {'en_US': 'Revoked'} |
| `terminated` | {'en_US': 'Terminated'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### PL Bank Account Verification — `verification_status`

Entity `l10n_pl.bank.account.verification`, table `l10n_pl_bank_account_verification`. Label: {'en_US': 'Verification Status'}; stored; required; read only.

| Value | Label |
|---|---|
| `valid` | {'en_US': 'Valid'} |
| `invalid` | {'en_US': 'Invalid'} |
| `incomplete_partner` | {'en_US': 'Incomplete partner'} |
| `not_found_partner` | {'en_US': 'Partner not found'} |
| `error` | {'en_US': 'An error occurred during check with Government API'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Document object for tracking CIUS-RO extensible markup language sent to E-Factura — `state`

Entity `l10n_ro_edi.document`, table `l10n_ro_edi_document`. Label: {'en_US': 'E-Factura Status'}; stored; required; read only.

| Value | Label |
|---|---|
| `invoice_sent` | {'en_US': 'Sent'} |
| `invoice_refused` | {'en_US': 'Error'} |
| `invoice_validated` | {'en_US': 'Validated'} |
| `stock_sent` | {'en_US': 'Sent'} |
| `stock_sending_failed` | {'en_US': 'Error'} |
| `stock_validated` | {'en_US': 'Validated'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### MyInvois Document — `activity_state`

Entity `myinvois.document`, table `myinvois_document`. Label: {'en_US': 'Activity State'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### MyInvois Document — `myinvois_state`

Entity `myinvois.document`, table `myinvois_document`. Label: {'en_US': 'MyInvois State'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'Validation In Progress'} |
| `valid` | {'en_US': 'Valid'} |
| `rejected` | {'en_US': 'Rejected'} |
| `invalid` | {'en_US': 'Invalid'} |
| `cancelled` | {'en_US': 'Cancelled'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Business Level Responses for Nemhandel — `nemhandel_state`

Entity `nemhandel.response`, table `nemhandel_response`. Label: {'en_US': 'Nemhandel status'}; stored.

| Value | Label |
|---|---|
| `processing` | {'en_US': 'Pending Reception'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |
| `not_serviced` | {'en_US': 'Not Serviced'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### PDP Response wizard — `status`

Entity `pdp.response.wizard`, table `pdp_response_wizard`. Label: {'en_US': 'Status'}; stored.

| Value | Label |
|---|---|
| `PD` | {'en_US': 'Paid'} |
| `cancelled` | {'en_US': 'Cancelled'} |
| `suspended` | {'en_US': 'Suspended'} |
| `refused` | {'en_US': 'Refused'} |
| `AP` | {'en_US': 'Approved'} |
| `in_hand` | {'en_US': 'In Hand'} |
| `completed` | {'en_US': 'Completed'} |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

## fleet

### Vehicle — `activity_state`

Entity `fleet.vehicle`, table `fleet_vehicle`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Vehicle — `contract_state`

Entity `fleet.vehicle`, table `fleet_vehicle`. Label: {'en_US': 'Last Contract State', 'tr_TR': 'Son Sözleşme Durumu', 'ar_001': 'حالة آخر عقد'}; not stored; read only.

| Value | Label |
|---|---|
| `futur` | {'en_US': 'Incoming', 'tr_TR': 'Gelen', 'ar_001': 'واردة'} |
| `open` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `closed` | {'en_US': 'Closed', 'tr_TR': 'Kapanmış', 'ar_001': 'مغلق'} |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Vehicle Contract — `activity_state`

Entity `fleet.vehicle.log.contract`, table `fleet_vehicle_log_contract`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Vehicle Contract — `state`

Entity `fleet.vehicle.log.contract`, table `fleet_vehicle_log_contract`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `futur` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `open` | {'en_US': 'Running', 'tr_TR': 'Devam Eden', 'ar_001': 'جاري'} |
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `closed` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Services for vehicles — `activity_state`

Entity `fleet.vehicle.log.services`, table `fleet_vehicle_log_services`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Services for vehicles — `state`

Entity `fleet.vehicle.log.services`, table `fleet_vehicle_log_services`. Label: {'en_US': 'Stage', 'tr_TR': 'Aşama', 'ar_001': 'المرحلة'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `new` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `running` | {'en_US': 'Running', 'tr_TR': 'Devam Eden', 'ar_001': 'جاري'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Model of a vehicle — `activity_state`

Entity `fleet.vehicle.model`, table `fleet_vehicle_model`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

## general-ledger

### Account — `activity_state`

Entity `account.account`, table `account_account`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Invoices Statistics — `payment_state`

Entity `account.invoice.report`, table ``. Label: {'en_US': 'Payment Status', 'tr_TR': 'Ödeme Durumu', 'ar_001': 'حالة الدفع'}; stored; read only.

| Value | Label |
|---|---|
| `not_paid` | {'en_US': 'Not Paid', 'tr_TR': 'Ödenmemiş', 'ar_001': 'غير مدفوع'} |
| `in_payment` | {'en_US': 'In Payment', 'tr_TR': 'Ödeme', 'ar_001': 'بانتظار التسوية'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `partial` | {'en_US': 'Partially Paid', 'tr_TR': 'Kısmen ödenmiş', 'ar_001': 'مسدد جزئياً'} |
| `reversed` | {'en_US': 'Reversed', 'tr_TR': 'Tersine çevrildi', 'ar_001': 'معكوس'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `invoicing_legacy` | {'en_US': 'Invoicing App Legacy', 'tr_TR': 'Eski Faturalandırma Uygulaması', 'ar_001': 'تراث تطبيق الفوترة'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Invoices Statistics — `state`

Entity `account.invoice.report`, table ``. Label: {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `posted` | {'en_US': 'Open', 'tr_TR': 'Açık', 'ar_001': 'فتح'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal — `activity_state`

Entity `account.journal`, table `account_journal`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Account Lock Exception — `state`

Entity `account.lock_exception`, table `account_lock_exception`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'المحافظة'}; not stored; read only.

| Value | Label |
|---|---|
| `active` | {'en_US': 'Active', 'tr_TR': 'Etkin', 'ar_001': 'نشط'} |
| `revoked` | {'en_US': 'Revoked', 'tr_TR': 'Geriye alındı', 'ar_001': 'ملغي'} |
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `activity_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Electronic invoicing', 'tr_TR': 'Elektronik faturalama', 'ar_001': 'الفوترة الإلكترونية'}; stored; read only.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'tr_TR': 'Giden', 'ar_001': 'بانتظار الإرسال'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `to_cancel` | {'en_US': 'To Cancel', 'tr_TR': 'İptal etmek için', 'ar_001': 'للإلغاء'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_es_edi_verifactu_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Veri*Factu Status'}; stored; read only.

| Value | Label |
|---|---|
| `rejected` | {'en_US': 'Rejected'} |
| `registered_with_errors` | {'en_US': 'Registered with Errors'} |
| `accepted` | {'en_US': 'Accepted'} |
| `cancelled` | {'en_US': 'Cancelled'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_es_tbai_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'TicketBAI status'}; not stored; read only.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |
| `cancelled` | {'en_US': 'Cancelled'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_fr_pdp_status`

Entity `account.move`, table `account_move`. Label: {'en_US': 'E-Reporting Status'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `out_of_scope` | {'en_US': 'Out of scope'} |
| `pending` | {'en_US': 'Pending'} |
| `ready` | {'en_US': 'Ready'} |
| `error` | {'en_US': 'Error'} |
| `sent` | {'en_US': 'Sent'} |
| `completed` | {'en_US': 'Completed'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_gr_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'myDATA Status'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `invoice_sent` | {'en_US': 'Invoice sent'} |
| `bill_fetched` | {'en_US': 'Expense classification ready to send'} |
| `bill_sent` | {'en_US': 'Expense classification sent'} |
| `invoice_pending` | {'en_US': 'Invoice submission pending'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_hu_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'NAV 3.0 status'}; stored.

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

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_in_edi_status`

Entity `account.move`, table `account_move`. Label: {'en_US': 'India E-Invoice Status'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |
| `cancelled` | {'en_US': 'Cancelled'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_it_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'SDI State'}; stored; changes are tracked in the message thread.

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

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_jo_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'JoFotara State'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'ar_001': 'بانتظار الإرسال'} |
| `sent` | {'en_US': 'Sent', 'ar_001': 'تم الإرسال'} |
| `demo` | {'en_US': 'Sent (Demo)'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_my_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'MyInvois State'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'Validation In Progress'} |
| `valid` | {'en_US': 'Valid'} |
| `rejected` | {'en_US': 'Rejected'} |
| `invalid` | {'en_US': 'Invalid'} |
| `cancelled` | {'en_US': 'Cancelled'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_pl_edi_status`

Entity `account.move`, table `account_move`. Label: {'en_US': 'KSeF Status'}; stored; read only.

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent (In Progress)'} |
| `accepted` | {'en_US': 'Accepted'} |
| `rejected` | {'en_US': 'Rejected'} |
| `fetch_ready` | {'en_US': 'Fetch Ready'} |
| `fetched` | {'en_US': 'Fetched'} |
| `fetch_failed` | {'en_US': 'Fetch Failed'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_ro_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'E-Factura Status'}; stored; read only.

| Value | Label |
|---|---|
| `invoice_not_indexed` | {'en_US': 'Not indexed'} |
| `invoice_sent` | {'en_US': 'Sent'} |
| `invoice_refused` | {'en_US': 'Refused'} |
| `invoice_validated` | {'en_US': 'Validated'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_rs_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Serbia E-Invoice state'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `sent` | {'en_US': 'Sent'} |
| `sending_failed` | {'en_US': 'Error'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_tr_nilvera_send_status`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Nilvera Status', 'tr_TR': 'Nilvera Durumu'}; stored; read only.

| Value | Label |
|---|---|
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata'} |
| `not_sent` | {'en_US': 'Not sent', 'tr_TR': 'Gönderilmedi'} |
| `sent` | {'en_US': 'Sent and waiting response', 'tr_TR': 'Gönderildi ve yanıt bekleniyor'} |
| `succeed` | {'en_US': 'Successful', 'tr_TR': 'Başarılı'} |
| `waiting` | {'en_US': 'Waiting', 'tr_TR': 'Bekliyorum'} |
| `unknown` | {'en_US': 'Unknown', 'tr_TR': 'Bilinmiyor'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_tw_edi_refund_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Refund State'}; stored; read only.

| Value | Label |
|---|---|
| `to_be_agreed` | {'en_US': 'To be agreed'} |
| `agreed` | {'en_US': 'Agreed'} |
| `disagreed` | {'en_US': 'Disagreed'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_tw_edi_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Invoice Status'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `invoiced` | {'en_US': 'Invoiced'} |
| `valid` | {'en_US': 'Valid'} |
| `invalid` | {'en_US': 'Invalid'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_vn_edi_invoice_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Sinvoice Status'}; stored.

| Value | Label |
|---|---|
| `ready_to_send` | {'en_US': 'Ready to send'} |
| `sent` | {'en_US': 'Sent'} |
| `payment_state_to_update` | {'en_US': 'Payment status to update'} |
| `canceled` | {'en_US': 'Canceled'} |
| `adjusted` | {'en_US': 'Adjusted'} |
| `replaced` | {'en_US': 'Replaced'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `nemhandel_move_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Nemhandel status'}; stored; read only.

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready to send'} |
| `to_send` | {'en_US': 'Queued'} |
| `processing` | {'en_US': 'Pending Reception'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |
| `BusinessAccept` | {'en_US': 'Approved'} |
| `BusinessReject` | {'en_US': 'Rejected'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `payment_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Payment Status', 'tr_TR': 'Ödeme Durumu', 'ar_001': 'حالة الدفع'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `not_paid` | {'en_US': 'Not Paid', 'tr_TR': 'Ödenmemiş', 'ar_001': 'غير مدفوع'} |
| `in_payment` | {'en_US': 'In Payment', 'tr_TR': 'Ödeme', 'ar_001': 'بانتظار التسوية'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `partial` | {'en_US': 'Partially Paid', 'tr_TR': 'Kısmen ödenmiş', 'ar_001': 'مسدد جزئياً'} |
| `reversed` | {'en_US': 'Reversed', 'tr_TR': 'Tersine çevrildi', 'ar_001': 'معكوس'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `invoicing_legacy` | {'en_US': 'Invoicing App Legacy', 'tr_TR': 'Eski Faturalandırma Uygulaması', 'ar_001': 'تراث تطبيق الفوترة'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `pdp_ppf_lifecycle_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'PPF Lifeycle Status'}; stored; read only.

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'In Progress'} |
| `sent` | {'en_US': 'Sent'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `pdp_ppf_move_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'PPF Invoice Status'}; stored; read only.

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'In Progress'} |
| `sent` | {'en_US': 'Sent'} |
| `done` | {'en_US': 'Done'} |
| `error` | {'en_US': 'Error'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `peppol_move_state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'E-Invoicing Status'}; stored; read only.

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

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `state`

Entity `account.move`, table `account_move`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `posted` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Payments — `activity_state`

Entity `account.payment`, table `account_payment`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Payments — `state`

Entity `account.payment`, table `account_payment`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'المحافظة'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `in_process` | {'en_US': 'In Process', 'tr_TR': 'İşleniyor', 'ar_001': 'قيد التنفيذ'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `canceled` | {'en_US': 'Canceled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

## human-resources-core

### Department — `activity_state`

Entity `hr.department`, table `hr_department`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `activity_state`

Entity `hr.employee`, table `hr_employee`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `attendance_state`

Entity `hr.employee`, table `hr_employee`. Label: {'en_US': 'Attendance Status', 'tr_TR': 'Katılım Durumu', 'ar_001': 'حالة الحضور'}; not stored; read only.

| Value | Label |
|---|---|
| `checked_out` | {'en_US': 'Checked out', 'tr_TR': 'Çıkış Yapıldı', 'ar_001': 'تم تسجيل الخروج'} |
| `checked_in` | {'en_US': 'Checked in', 'tr_TR': 'Giriş Yapıldı', 'ar_001': 'تم تسجيل الحضور'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `current_leave_state`

Entity `hr.employee`, table `hr_employee`. Label: {'en_US': 'Current Time Off Status', 'tr_TR': 'Mevcut İzin Durumu', 'ar_001': 'حالة الإجازة الحالية'}; not stored; read only.

| Value | Label |
|---|---|
| `confirm` | {'en_US': 'Waiting Approval', 'tr_TR': 'Onay Bekliyor', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Waiting Second Approval', 'tr_TR': 'İkinci Onayı Bekliyor', 'ar_001': 'بانتظار الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `hr_presence_state`

Entity `hr.employee`, table `hr_employee`. Label: {'en_US': 'Hr Presence State', 'tr_TR': 'İk Mevcudiyet Durumu', 'ar_001': 'حالة حضور الموارد البشرية'}; not stored; read only.

| Value | Label |
|---|---|
| `present` | {'en_US': 'Present', 'tr_TR': 'Mevcut', 'ar_001': 'حاضر'} |
| `absent` | {'en_US': 'Absent', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غائب'} |
| `archive` | {'en_US': 'Archived', 'tr_TR': 'Arşivlendi', 'ar_001': 'مؤرشف'} |
| `out_of_working_hour` | {'en_US': 'Off-Hours', 'tr_TR': 'Mesai Dışı', 'ar_001': 'خارج ساعات العمل'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Public Employee — `hr_presence_state`

Entity `hr.employee.public`, table `hr_employee_public`. Label: {'en_US': 'Hr Presence State', 'tr_TR': 'İk Mevcudiyet Durumu', 'ar_001': 'حالة حضور الموارد البشرية'}; not stored; read only.

| Value | Label |
|---|---|
| `present` | {'en_US': 'Present', 'tr_TR': 'Mevcut', 'ar_001': 'حاضر'} |
| `absent` | {'en_US': 'Absent', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غائب'} |
| `archive` | {'en_US': 'Archived', 'tr_TR': 'Arşivlendi', 'ar_001': 'مؤرشف'} |
| `out_of_working_hour` | {'en_US': 'Off-Hours', 'tr_TR': 'Mesai Dışı', 'ar_001': 'خارج ساعات العمل'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Job Position — `activity_state`

Entity `hr.job`, table `hr_job`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Resume line of an employee — `expiration_status`

Entity `hr.resume.line`, table `hr_resume_line`. Label: {'en_US': 'Expiration Status', 'tr_TR': 'Son geçerlilik tarihi durumu', 'ar_001': 'حالة انتهاء الصلاحية'}; stored; read only.

| Value | Label |
|---|---|
| `expired` | {'en_US': 'Expired', 'tr_TR': 'Süresi Doldu', 'ar_001': 'منتهي الصلاحية'} |
| `expiring` | {'en_US': 'Expiring', 'tr_TR': 'Süresi dolmak üzere', 'ar_001': 'انتهاء الصلاحية'} |
| `valid` | {'en_US': 'Valid', 'tr_TR': 'Geçerli', 'ar_001': 'صالح'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Version — `activity_state`

Entity `hr.version`, table `hr_version`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

## inventory-operations

### Stock Quantity Report — `state`

Entity `report.stock.quantity`, table `report_stock_quantity`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `forecast` | {'en_US': 'Forecasted Stock', 'tr_TR': 'Öngörülen Stok', 'ar_001': 'المخزون المتوقع'} |
| `in` | {'en_US': 'Forecasted Receipts', 'tr_TR': 'Öngörülen Alım', 'ar_001': 'الإيصالات المتوقعة'} |
| `out` | {'en_US': 'Forecasted Deliveries', 'tr_TR': 'Öngörülen Teslimatlar', 'ar_001': 'التوصيلات المتوقعة'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Lot/Serial — `activity_state`

Entity `stock.lot`, table `stock_lot`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Stock Move — `state`

Entity `stock.move`, table `stock_move`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `waiting` | {'en_US': 'Waiting Another Move', 'tr_TR': 'Başka Bir Hareketi Bekliyor', 'ar_001': 'في انتظار حركة أخرى'} |
| `confirmed` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `partially_available` | {'en_US': 'Partially Available', 'tr_TR': 'Kısmi Uygunluk', 'ar_001': 'متوفر جزئياً'} |
| `assigned` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `activity_state`

Entity `stock.picking`, table `stock_picking`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `l10n_ro_edi_stock_state`

Entity `stock.picking`, table `stock_picking`. Label: {'en_US': 'eTransport Status'}; stored; read only.

| Value | Label |
|---|---|
| `stock_sent` | {'en_US': 'Sent'} |
| `stock_sending_failed` | {'en_US': 'Error'} |
| `stock_validated` | {'en_US': 'Validated'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `l10n_tr_nilvera_dispatch_state`

Entity `stock.picking`, table `stock_picking`. Label: {'en_US': 'e-Dispatch State', 'tr_TR': 'e-İrsaliye Durumu'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send', 'tr_TR': 'Gönderilecek'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `products_availability_state`

Entity `stock.picking`, table `stock_picking`. Label: {'en_US': 'Products Availability State', 'tr_TR': 'Ürün Kullanılabilirlik Durumu', 'ar_001': 'حالة توافر المنتجات'}; not stored; read only.

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `expected` | {'en_US': 'Expected', 'tr_TR': 'Beklenen', 'ar_001': 'المتوقع'} |
| `late` | {'en_US': 'Late', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `state`

Entity `stock.picking`, table `stock_picking`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `waiting` | {'en_US': 'Waiting Another Operation', 'tr_TR': 'Başka Bir İşlem Bekliyor', 'ar_001': 'في انتظار عملية أخرى'} |
| `confirmed` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `assigned` | {'en_US': 'Ready', 'tr_TR': 'Hazır', 'ar_001': 'جاهز'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Batch Transfer — `activity_state`

Entity `stock.picking.batch`, table `stock_picking_batch`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Batch Transfer — `l10n_ro_edi_stock_state`

Entity `stock.picking.batch`, table `stock_picking_batch`. Label: {'en_US': 'eTransport Status'}; stored; read only.

| Value | Label |
|---|---|
| `stock_sent` | {'en_US': 'Sent'} |
| `stock_sending_failed` | {'en_US': 'Error'} |
| `stock_validated` | {'en_US': 'Validated'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Batch Transfer — `state`

Entity `stock.picking.batch`, table `stock_picking_batch`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `in_progress` | {'en_US': 'In progress', 'tr_TR': 'Devam Eden', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Scrap — `state`

Entity `stock.scrap`, table `stock_scrap`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

## inventory-valuation-and-costing

### Stock Landed Cost — `activity_state`

Entity `stock.landed.cost`, table `stock_landed_cost`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [inventory-valuation-and-costing state machines](../domains/inventory-valuation-and-costing/state-machines.md).

### Stock Landed Cost — `state`

Entity `stock.landed.cost`, table `stock_landed_cost`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'İşlenmiş', 'ar_001': 'مُرحّل'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [inventory-valuation-and-costing state machines](../domains/inventory-valuation-and-costing/state-machines.md).

## learning-surveys-and-gamification

### Gamification Challenge — `state`

Entity `gamification.challenge`, table `gamification_challenge`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `inprogress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Gamification Goal — `state`

Entity `gamification.goal`, table `gamification_goal`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}; stored; required.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `inprogress` | {'en_US': 'In progress', 'tr_TR': 'Devam Eden', 'ar_001': 'قيد التنفيذ'} |
| `reached` | {'en_US': 'Reached', 'tr_TR': 'Ulaşıldı', 'ar_001': 'تم الوصول'} |
| `failed` | {'en_US': 'Failed', 'tr_TR': 'Başarısız', 'ar_001': 'فشل'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Course — `activity_state`

Entity `slide.channel`, table `slide_channel`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Channel / Partners (Members) — `member_status`

Entity `slide.channel.partner`, table `slide_channel_partner`. Label: {'en_US': 'Attendee Status', 'tr_TR': 'Katılımcı Durumu', 'ar_001': 'حالة الحاضر'}; stored; required; read only.

| Value | Label |
|---|---|
| `invited` | {'en_US': 'Invite Sent', 'tr_TR': 'Davet Gönderildi', 'ar_001': 'تم إرسال الدعوة'} |
| `joined` | {'en_US': 'Joined', 'tr_TR': 'Katıldı', 'ar_001': 'تم الانضمام'} |
| `ongoing` | {'en_US': 'Ongoing', 'tr_TR': 'Devam eden', 'ar_001': 'جاري'} |
| `completed` | {'en_US': 'Finished', 'tr_TR': 'Bitmiş', 'ar_001': 'مُنتهي'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey — `activity_state`

Entity `survey.survey`, table `survey_survey`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey — `session_state`

Entity `survey.survey`, table `survey_survey`. Label: {'en_US': 'Session State', 'tr_TR': 'Oturum Durumu', 'ar_001': 'حالة الجلسة'}; stored.

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready', 'tr_TR': 'Hazır', 'ar_001': 'جاهز'} |
| `in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey User Input — `activity_state`

Entity `survey.user_input`, table `survey_user_input`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey User Input — `state`

Entity `survey.user_input`, table `survey_user_input`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `new` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Completed', 'tr_TR': 'Tamamlandı', 'ar_001': 'مكتملة'} |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

## lunch-ordering

### Lunch Order — `state`

Entity `lunch.order`, table `lunch_order`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `new` | {'en_US': 'To Order', 'tr_TR': 'Sipariş Ver', 'ar_001': 'بحاجة إلى الطلب'} |
| `ordered` | {'en_US': 'Ordered', 'tr_TR': 'Sipariş edildi', 'ar_001': 'تم طلبه'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `confirmed` | {'en_US': 'Received', 'tr_TR': 'Alınan', 'ar_001': 'تم الاستلام'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [lunch-ordering state machines](../domains/lunch-ordering/state-machines.md).

### Lunch Supplier — `activity_state`

Entity `lunch.supplier`, table `lunch_supplier`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [lunch-ordering state machines](../domains/lunch-ordering/state-machines.md).

## manufacturing

### Manufacturing Order — `activity_state`

Entity `mrp.production`, table `mrp_production`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Manufacturing Order — `components_availability_state`

Entity `mrp.production`, table `mrp_production`. Label: {'en_US': 'Components Availability State', 'tr_TR': 'Bileşenlerin Uygunluk Durumu', 'ar_001': 'حالة توافر المكونات'}; not stored; read only.

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `expected` | {'en_US': 'Expected', 'tr_TR': 'Beklenen', 'ar_001': 'المتوقع'} |
| `late` | {'en_US': 'Late', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `unavailable` | {'en_US': 'Not Available', 'tr_TR': 'Mevcut Değil', 'ar_001': 'غير متاح'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Manufacturing Order — `reservation_state`

Entity `mrp.production`, table `mrp_production`. Label: {'en_US': 'MO Readiness', 'tr_TR': 'MO Hazırlık', 'ar_001': 'جاهزية أمر التصنيع'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `confirmed` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `assigned` | {'en_US': 'Ready', 'tr_TR': 'Hazır', 'ar_001': 'جاهز'} |
| `waiting` | {'en_US': 'Waiting Another Operation', 'tr_TR': 'Başka Bir İşlem Bekliyor', 'ar_001': 'في انتظار عملية أخرى'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Manufacturing Order — `state`

Entity `mrp.production`, table `mrp_production`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `confirmed` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `to_close` | {'en_US': 'To Close', 'tr_TR': 'Kapatılacak', 'ar_001': 'للإقفال'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Work Center Usage — `activity_state`

Entity `mrp.routing.workcenter`, table `mrp_routing_workcenter`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Unbuild Order — `activity_state`

Entity `mrp.unbuild`, table `mrp_unbuild`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Unbuild Order — `state`

Entity `mrp.unbuild`, table `mrp_unbuild`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Work Center — `working_state`

Entity `mrp.workcenter`, table `mrp_workcenter`. Label: {'en_US': 'Workcenter Status', 'tr_TR': 'İş Merkezi Durumu', 'ar_001': 'حالة مركز العمل'}; stored; read only.

| Value | Label |
|---|---|
| `normal` | {'en_US': 'Normal', 'tr_TR': 'Normal', 'ar_001': 'عادي'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `done` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Work Order — `state`

Entity `mrp.workorder`, table `mrp_workorder`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `ready` | {'en_US': 'To Do', 'tr_TR': 'Yapılacak', 'ar_001': 'المهام المراد تنفيذها'} |
| `progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Finished', 'tr_TR': 'Bitmiş', 'ar_001': 'مُنتهي'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

## marketing-and-mass-mailing

### Marketing Card Campaign — `activity_state`

Entity `card.campaign`, table `card_campaign`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Marketing Card — `share_status`

Entity `card.card`, table `card_card`. Label: {'en_US': 'Share Status', 'tr_TR': 'Durumu Paylaş', 'ar_001': 'مشاركة الحالة'}; stored.

| Value | Label |
|---|---|
| `shared` | {'en_US': 'Shared', 'tr_TR': 'Paylaşılan', 'ar_001': 'مشارك'} |
| `visited` | {'en_US': 'Visited', 'tr_TR': 'Ziyaret edildi', 'ar_001': 'قام بالزيارة'} |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mass Mailing — `activity_state`

Entity `mailing.mailing`, table `mailing_mailing`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mass Mailing — `state`

Entity `mailing.mailing`, table `mailing_mailing`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `in_queue` | {'en_US': 'In Queue', 'tr_TR': 'Sırada', 'ar_001': 'في قائمة الانتظار'} |
| `sending` | {'en_US': 'Sending', 'tr_TR': 'Gönderiyor', 'ar_001': 'جاري الإرسال'} |
| `done` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mailing Statistics — `trace_status`

Entity `mailing.trace`, table `mailing_trace`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored.

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

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mass Mailing Statistics — `state`

Entity `mailing.trace.report`, table `mailing_trace_report`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `test` | {'en_US': 'Tested', 'tr_TR': 'Test Edilmiş', 'ar_001': 'تم الاختبار'} |
| `done` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

## messaging-and-activities

### Digest — `state`

Entity `digest.digest`, table `digest_digest`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `activated` | {'en_US': 'Activated', 'tr_TR': 'Etkinleştirildi', 'ar_001': 'مفعل'} |
| `deactivated` | {'en_US': 'Deactivated', 'tr_TR': 'Devre dışı', 'ar_001': 'معطل'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Discussion Channel — `livechat_status`

Entity `discuss.channel`, table `discuss_channel`. Label: {'en_US': 'Livechat Status', 'tr_TR': 'Canlı Sohbet  Durumu', 'ar_001': 'حالة الدردشة المباشرة'}; stored.

| Value | Label |
|---|---|
| `in_progress` | {'en_US': 'In progress', 'tr_TR': 'Devam Eden', 'ar_001': 'قيد التنفيذ'} |
| `waiting` | {'en_US': 'Waiting for customer', 'tr_TR': 'Müşteri bekleniyor', 'ar_001': 'بانتظار العميل'} |
| `need_help` | {'en_US': 'Looking for help', 'tr_TR': 'Yardım arıyorum', 'ar_001': 'أبحث عن المساعدة'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Incoming Mail Server — `state`

Entity `fetchmail.server`, table `fetchmail_server`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Not Confirmed', 'tr_TR': 'Onaylanmadı', 'ar_001': 'غير مؤكد'} |
| `done` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Keep the channel member history — `help_status`

Entity `im_livechat.channel.member.history`, table `im_livechat_channel_member_history`. Label: {'en_US': 'Help Status', 'tr_TR': 'Yardım Durumu', 'ar_001': 'حالة المساعدة'}; stored; read only.

| Value | Label |
|---|---|
| `requested` | {'en_US': 'Help Requested', 'tr_TR': 'Yardım İstendi', 'ar_001': 'تم طلب المساعدة'} |
| `provided` | {'en_US': 'Help Provided', 'tr_TR': 'Sağlanan Yardım', 'ar_001': 'تم تقديم المساعدة'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Activity — `state`

Entity `mail.activity`, table `mail_activity`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Activity Mixin — `activity_state`

Entity `mail.activity.mixin`, table ``. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Email Aliases — `alias_status`

Entity `mail.alias`, table `mail_alias`. Label: {'en_US': 'Alias Status', 'tr_TR': 'Takma Ad Durumu', 'ar_001': 'حالة اللقب'}; stored; read only.

| Value | Label |
|---|---|
| `not_tested` | {'en_US': 'Not Tested', 'tr_TR': 'Test Edilmedi', 'ar_001': 'لم يتم اختباره'} |
| `valid` | {'en_US': 'Valid', 'tr_TR': 'Geçerli', 'ar_001': 'صالح'} |
| `invalid` | {'en_US': 'Invalid', 'tr_TR': 'Geçersiz', 'ar_001': 'غير صالح'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Mailing List Message — `moderation_status`

Entity `mail.group.message`, table `mail_group_message`. Label: {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}; stored; required.

| Value | Label |
|---|---|
| `pending_moderation` | {'en_US': 'Pending Moderation', 'tr_TR': 'Denetleme Bekliyor', 'ar_001': 'بانتظار الإشراف'} |
| `accepted` | {'en_US': 'Accepted', 'tr_TR': 'Kabul Edildi', 'ar_001': 'تم القبول'} |
| `rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Mailing List black/white list — `status`

Entity `mail.group.moderation`, table `mail_group_moderation`. Label: {'en_US': 'Status', 'tr_TR': 'Durum', 'ar_001': 'الحالة'}; stored; required.

| Value | Label |
|---|---|
| `allow` | {'en_US': 'Always Allow', 'tr_TR': 'Daima İzin Ver', 'ar_001': 'السماح دائمًا'} |
| `ban` | {'en_US': 'Permanent Ban', 'tr_TR': 'Kalıcı Engel', 'ar_001': 'حظر دائم'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Outgoing Mails — `state`

Entity `mail.mail`, table `mail_mail`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `outgoing` | {'en_US': 'Outgoing', 'tr_TR': 'Giden', 'ar_001': 'الصادرة'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `received` | {'en_US': 'Received', 'tr_TR': 'Alınan', 'ar_001': 'تم الاستلام'} |
| `exception` | {'en_US': 'Delivery Failed', 'tr_TR': 'Gönderim Başarısız', 'ar_001': 'فشل التسليم'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Message Notifications — `notification_status`

Entity `mail.notification`, table `mail_notification`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored.

| Value | Label |
|---|---|
| `ready` | {'en_US': 'Ready to Send', 'tr_TR': 'Gönderime Hazır', 'ar_001': 'جاهز للإرسال'} |
| `process` | {'en_US': 'Processing', 'tr_TR': 'İşleniyor', 'ar_001': 'معالجة'} |
| `pending` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `sent` | {'en_US': 'Delivered', 'tr_TR': 'Teslim Edilen', 'ar_001': 'تم التوصيل'} |
| `bounce` | {'en_US': 'Bounced', 'tr_TR': 'İletilmeyen', 'ar_001': 'الرسائل المرتدة'} |
| `exception` | {'en_US': 'Exception', 'tr_TR': 'İstisna', 'ar_001': 'استثناء'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### User/Guest Presence — `status`

Entity `mail.presence`, table `mail_presence`. Label: {'en_US': 'IM Status', 'tr_TR': 'Anlık İleti Durumu', 'ar_001': 'حالة المحادثات الفورية'}; stored.

| Value | Label |
|---|---|
| `online` | {'en_US': 'Online', 'tr_TR': 'Çevrim içi', 'ar_001': 'عبر الإنترنت'} |
| `away` | {'en_US': 'Away', 'tr_TR': 'Dışarıda', 'ar_001': 'بعيد'} |
| `offline` | {'en_US': 'Offline', 'tr_TR': 'Çevrim dışı', 'ar_001': 'غير متصل بالإنترنت'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Outgoing text message — `state`

Entity `sms.sms`, table `sms_sms`. Label: {'en_US': 'SMS Status', 'tr_TR': 'SMS Durumu', 'ar_001': 'حالة الرسائل النصية القصيرة'}; stored; required; read only.

| Value | Label |
|---|---|
| `outgoing` | {'en_US': 'In Queue', 'tr_TR': 'Sırada', 'ar_001': 'في قائمة الانتظار'} |
| `process` | {'en_US': 'Processing', 'tr_TR': 'İşleniyor', 'ar_001': 'معالجة'} |
| `pending` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `sent` | {'en_US': 'Delivered', 'tr_TR': 'Teslim Edilen', 'ar_001': 'تم التوصيل'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Snailmail Letter — `state`

Entity `snailmail.letter`, table `snailmail_letter`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required; read only.

| Value | Label |
|---|---|
| `pending` | {'en_US': 'In Queue', 'tr_TR': 'Sırada', 'ar_001': 'في قائمة الانتظار'} |
| `sent` | {'en_US': 'Sent', 'tr_TR': 'Gönderildi', 'ar_001': 'تم الإرسال'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |
| `canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

## multi-currency

### Language Export — `state`

Entity `base.language.export`, table `base_language_export`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}; stored.

| Value | Label |
|---|---|
| `choose` | {'en_US': 'choose', 'tr_TR': 'seç', 'ar_001': 'اختر'} |
| `get` | {'en_US': 'get', 'tr_TR': 'alma', 'ar_001': 'احصل على'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Update Module — `state`

Entity `base.module.update`, table `base_module_update`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `init` | {'en_US': 'init', 'tr_TR': 'başlatma', 'ar_001': 'إطلاق'} |
| `done` | {'en_US': 'done', 'tr_TR': 'biten', 'ar_001': 'تم'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Merge Partner Wizard — `state`

Entity `base.partner.merge.automatic.wizard`, table `base_partner_merge_automatic_wizard`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}; stored; required; read only.

| Value | Label |
|---|---|
| `option` | {'en_US': 'Option', 'tr_TR': 'Seçenek', 'ar_001': 'الخيار'} |
| `selection` | {'en_US': 'Selection', 'tr_TR': 'Seçim', 'ar_001': 'قائمة خيارات'} |
| `finished` | {'en_US': 'Finished', 'tr_TR': 'Bitmiş', 'ar_001': 'مُنتهي'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Server Actions — `activity_state`

Entity `ir.actions.server`, table ``. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Server Actions — `state`

Entity `ir.actions.server`, table ``. Label: {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}; stored; required; changes are tracked in the message thread.

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

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Configuration Wizards — `state`

Entity `ir.actions.todo`, table `ir_actions_todo`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required.

| Value | Label |
|---|---|
| `open` | {'en_US': 'To Do', 'tr_TR': 'Yapılacak', 'ar_001': 'المهام المراد تنفيذها'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Scheduled Actions — `activity_state`

Entity `ir.cron`, table `ir_cron`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Models — `state`

Entity `ir.model`, table `ir_model`. Label: {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}; stored; read only.

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Custom Object', 'tr_TR': 'Özel Nesne', 'ar_001': 'كائن مخصص'} |
| `base` | {'en_US': 'Base Object', 'tr_TR': 'Temel Nesne', 'ar_001': 'الكائن الأساسي'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Fields — `state`

Entity `ir.model.fields`, table `ir_model_fields`. Label: {'en_US': 'Type', 'tr_TR': 'Tip', 'ar_001': 'النوع'}; stored; required; read only.

| Value | Label |
|---|---|
| `manual` | {'en_US': 'Custom Field', 'tr_TR': 'Özel Alan', 'ar_001': 'حقل مخصص'} |
| `base` | {'en_US': 'Base Field', 'tr_TR': 'Temel Alan', 'ar_001': 'الحقل الأساسي'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Module — `state`

Entity `ir.module.module`, table `ir_module_module`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `uninstallable` | {'en_US': 'Uninstallable', 'tr_TR': 'Kaldırılamaz', 'ar_001': 'يمكن إلغاء تثبيته'} |
| `uninstalled` | {'en_US': 'Not Installed', 'tr_TR': 'Yüklü Değil', 'ar_001': 'غير مثبت'} |
| `installed` | {'en_US': 'Installed', 'tr_TR': 'Yüklü', 'ar_001': 'تم التثبيت'} |
| `to upgrade` | {'en_US': 'To be upgraded', 'tr_TR': 'Yükseltilecek', 'ar_001': 'بانتظار الترقية'} |
| `to remove` | {'en_US': 'To be removed', 'tr_TR': 'Kaldırılacak', 'ar_001': 'بانتظار الإزالة'} |
| `to install` | {'en_US': 'To be installed', 'tr_TR': 'Yüklenecek', 'ar_001': 'بانتظار التثبيت'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Module dependency — `state`

Entity `ir.module.module.dependency`, table `ir_module_module_dependency`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; not stored; read only.

| Value | Label |
|---|---|
| `uninstallable` | {'en_US': 'Uninstallable', 'tr_TR': 'Kaldırılamaz', 'ar_001': 'يمكن إلغاء تثبيته'} |
| `uninstalled` | {'en_US': 'Not Installed', 'tr_TR': 'Yüklü Değil', 'ar_001': 'غير مثبت'} |
| `installed` | {'en_US': 'Installed', 'tr_TR': 'Yüklü', 'ar_001': 'تم التثبيت'} |
| `to upgrade` | {'en_US': 'To be upgraded', 'tr_TR': 'Yükseltilecek', 'ar_001': 'بانتظار الترقية'} |
| `to remove` | {'en_US': 'To be removed', 'tr_TR': 'Kaldırılacak', 'ar_001': 'بانتظار الإزالة'} |
| `to install` | {'en_US': 'To be installed', 'tr_TR': 'Yüklenecek', 'ar_001': 'بانتظار التثبيت'} |
| `unknown` | {'en_US': 'Unknown', 'tr_TR': 'Bilinmeyen', 'ar_001': 'غير معروف'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Module exclusion — `state`

Entity `ir.module.module.exclusion`, table `ir_module_module_exclusion`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; not stored; read only.

| Value | Label |
|---|---|
| `uninstallable` | {'en_US': 'Uninstallable', 'tr_TR': 'Kaldırılamaz', 'ar_001': 'يمكن إلغاء تثبيته'} |
| `uninstalled` | {'en_US': 'Not Installed', 'tr_TR': 'Yüklü Değil', 'ar_001': 'غير مثبت'} |
| `installed` | {'en_US': 'Installed', 'tr_TR': 'Yüklü', 'ar_001': 'تم التثبيت'} |
| `to upgrade` | {'en_US': 'To be upgraded', 'tr_TR': 'Yükseltilecek', 'ar_001': 'بانتظار الترقية'} |
| `to remove` | {'en_US': 'To be removed', 'tr_TR': 'Kaldırılacak', 'ar_001': 'بانتظار الإزالة'} |
| `to install` | {'en_US': 'To be installed', 'tr_TR': 'Yüklenecek', 'ar_001': 'بانتظار التثبيت'} |
| `unknown` | {'en_US': 'Unknown', 'tr_TR': 'Bilinmeyen', 'ar_001': 'غير معروف'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `account_peppol_proxy_state`

Entity `res.company`, table `res_company`. Label: {'en_US': 'PEPPOL status', 'tr_TR': 'PEPPOL status', 'ar_001': 'PEPPOL status'}; stored; required.

| Value | Label |
|---|---|
| `not_registered` | {'en_US': 'Not registered', 'tr_TR': 'Kayıtlı değil', 'ar_001': 'غير مسجل'} |
| `sender` | {'en_US': 'Can send but not receive', 'tr_TR': 'Gönderebilir ama alamaz', 'ar_001': 'Can send but not receive'} |
| `smp_registration` | {'en_US': 'Can send, pending registration to receive', 'tr_TR': 'Gönderebilir, almak için kayıt bekleniyor', 'ar_001': 'Can send, pending registration to receive'} |
| `receiver` | {'en_US': 'Can send and receive', 'tr_TR': 'Gönderip alabilir', 'ar_001': 'Can send and receive'} |
| `rejected` | {'en_US': 'Rejected', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `l10n_dk_nemhandel_proxy_state`

Entity `res.company`, table `res_company`. Label: {'en_US': 'Nemhandel status'}; stored; required.

| Value | Label |
|---|---|
| `not_registered` | {'en_US': 'Not registered'} |
| `in_verification` | {'en_US': 'In verification'} |
| `receiver` | {'en_US': 'Can send and receive'} |
| `rejected` | {'en_US': 'Rejected'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `l10n_hr_mer_connection_state`

Entity `res.company`, table `res_company`. Label: {'en_US': 'MojEracun connection status'}; stored; required; read only.

| Value | Label |
|---|---|
| `inactive` | {'en_US': 'Inactive'} |
| `active` | {'en_US': 'Active'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `l10n_it_eco_index_liquidation_state`

Entity `res.company`, table `res_company`. Label: {'en_US': 'Liquidation state'}; stored.

| Value | Label |
|---|---|
| `LS` | {'en_US': 'The company is in a state of liquidation'} |
| `LN` | {'en_US': 'The company is not in a state of liquidation'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `pdp_kyc_status`

Entity `res.company`, table `res_company`. Label: {'en_US': 'Pdp Kyc Status'}; stored.

| Value | Label |
|---|---|
| `processing` | {'en_US': 'Processing'} |
| `success` | {'en_US': 'Success'} |
| `fail` | {'en_US': 'Fail'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `activity_state`

Entity `res.partner`, table `res_partner`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `l10n_my_tin_validation_state`

Entity `res.partner`, table `res_partner`. Label: {'en_US': 'Tin Validation State'}; stored.

| Value | Label |
|---|---|
| `valid` | {'en_US': 'Valid'} |
| `invalid` | {'en_US': 'Invalid'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `l10n_tr_nilvera_customer_status`

Entity `res.partner`, table `res_partner`. Label: {'en_US': 'Nilvera Status'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `not_checked` | {'en_US': 'Not Verified'} |
| `earchive` | {'en_US': 'E-Archive'} |
| `einvoice` | {'en_US': 'E-Invoice'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `nemhandel_verification_state`

Entity `res.partner`, table `res_partner`. Label: {'en_US': 'Nemhandel endpoint verification'}; stored.

| Value | Label |
|---|---|
| `not_verified` | {'en_US': 'Not verified yet'} |
| `not_valid` | {'en_US': 'Not on Nemhandel'} |
| `valid` | {'en_US': 'Valid'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `pdp_verification_display_state`

Entity `res.partner`, table `res_partner`. Label: {'en_US': 'E-Invoicing State'}; not stored; read only.

| Value | Label |
|---|---|
| `not_verified` | {'en_US': 'Not verified yet'} |
| `pdp_not_valid` | {'en_US': 'Partner is not in the annuaire'} |
| `pdp_not_valid_format` | {'en_US': 'Partner cannot receive format'} |
| `pdp_valid` | {'en_US': 'Partner is in the annuaire'} |
| `peppol_not_valid` | {'en_US': 'Partner is not on Peppol'} |
| `peppol_not_valid_format` | {'en_US': 'Partner cannot receive format'} |
| `peppol_valid` | {'en_US': 'Partner is on Peppol'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `peppol_verification_state`

Entity `res.partner`, table `res_partner`. Label: {'en_US': 'Peppol status', 'tr_TR': 'Peppol status', 'ar_001': 'Peppol status'}; stored.

| Value | Label |
|---|---|
| `not_verified` | {'en_US': 'Unchecked', 'ar_001': 'غير محدد'} |
| `not_valid` | {'en_US': 'Partner is not on Peppol', 'ar_001': 'الشريك غير مسجل في Peppol'} |
| `not_valid_format` | {'en_US': 'Partner cannot receive format', 'ar_001': 'لا يمكن للشريك تلقي التنسيق'} |
| `valid` | {'en_US': 'Partner is on Peppol', 'ar_001': 'الشريك موجود على Peppol'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Bank Accounts — `activity_state`

Entity `res.partner.bank`, table `res_partner_bank`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### User — `manual_im_status`

Entity `res.users`, table `res_users`. Label: {'en_US': 'IM status manually set by the user', 'tr_TR': 'Kullanıcı tarafından manuel olarak ayarlanan IM durumu', 'ar_001': 'تم ضبط حالة المراسلة الفورية يدويًا من قِبَل المستخدم'}; stored.

| Value | Label |
|---|---|
| `away` | {'en_US': 'Away', 'tr_TR': 'Dışarıda', 'ar_001': 'بعيد'} |
| `busy` | {'en_US': 'Do Not Disturb', 'tr_TR': 'Rahatsız Etmeyin', 'ar_001': 'عدم الإزعاج'} |
| `offline` | {'en_US': 'Offline', 'tr_TR': 'Çevrim dışı', 'ar_001': 'غير متصل بالإنترنت'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### User — `odoobot_state`

Entity `res.users`, table `res_users`. Label: {'en_US': 'OdooBot Status', 'tr_TR': 'OdooBot Durumu', 'ar_001': 'حالة OdooBot'}; stored; read only.

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

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### User — `state`

Entity `res.users`, table `res_users`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; not stored; read only.

| Value | Label |
|---|---|
| `new` | {'en_US': 'Invited', 'tr_TR': 'Davetli', 'ar_001': 'تمت دعوته'} |
| `active` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Users Deletion Request — `state`

Entity `res.users.deletion`, table `res_users_deletion`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}; stored; required.

| Value | Label |
|---|---|
| `todo` | {'en_US': 'To Do', 'tr_TR': 'Yapılacak', 'ar_001': 'المهام المراد تنفيذها'} |
| `done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `fail` | {'en_US': 'Failed', 'tr_TR': 'Başarısız', 'ar_001': 'فشل'} |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

## payment-providers

### Payment Provider — `state`

Entity `payment.provider`, table `payment_provider`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}; stored; required.

| Value | Label |
|---|---|
| `disabled` | {'en_US': 'Disabled', 'tr_TR': 'Devre Dışı', 'ar_001': 'معطل'} |
| `enabled` | {'en_US': 'Enabled', 'tr_TR': 'Etkin', 'ar_001': 'ممكن'} |
| `test` | {'en_US': 'Test Mode', 'tr_TR': 'Test Modu', 'ar_001': 'وضع الاختبار'} |

Transitions: [payment-providers state machines](../domains/payment-providers/state-machines.md).

### Payment Token — `demo_simulated_state`

Entity `payment.token`, table `payment_token`. Label: {'en_US': 'Simulated State', 'tr_TR': 'Simüle Edilmiş Durum', 'ar_001': 'الحالة التي تم إنشاؤها بالمحاكاة'}; stored.

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `done` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `cancel` | {'en_US': 'Canceled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

Transitions: [payment-providers state machines](../domains/payment-providers/state-machines.md).

### Payment Transaction — `state`

Entity `payment.transaction`, table `payment_transaction`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `pending` | {'en_US': 'Pending', 'tr_TR': 'Beklemede', 'ar_001': 'قيد الانتظار'} |
| `authorized` | {'en_US': 'Authorized', 'tr_TR': 'Yetkili', 'ar_001': 'مصرح به'} |
| `done` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `cancel` | {'en_US': 'Canceled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `error` | {'en_US': 'Error', 'tr_TR': 'Hata', 'ar_001': 'خطأ'} |

Transitions: [payment-providers state machines](../domains/payment-providers/state-machines.md).

## payments-and-bank-reconciliation

### Account payment check — `activity_state`

Entity `l10n_latam.check`, table `l10n_latam_check`. Label: {'en_US': 'Activity State'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [payments-and-bank-reconciliation state machines](../domains/payments-and-bank-reconciliation/state-machines.md).

### Account payment check — `issue_state`

Entity `l10n_latam.check`, table `l10n_latam_check`. Label: {'en_US': 'Issue State'}; stored; read only.

| Value | Label |
|---|---|
| `handed` | {'en_US': 'Handed'} |
| `debited` | {'en_US': 'Debited'} |
| `voided` | {'en_US': 'Voided'} |

Transitions: [payments-and-bank-reconciliation state machines](../domains/payments-and-bank-reconciliation/state-machines.md).

## point-of-sale

### Point of Sale Configuration — `status`

Entity `pos.config`, table `pos_config`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; not stored; read only.

| Value | Label |
|---|---|
| `inactive` | {'en_US': 'Inactive', 'tr_TR': 'Pasif', 'ar_001': 'غير نشط'} |
| `active` | {'en_US': 'Active', 'tr_TR': 'Etkin', 'ar_001': 'نشط'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `invoice_status`

Entity `pos.order`, table `pos_order`. Label: {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}; not stored; read only.

| Value | Label |
|---|---|
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to_invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `l10n_es_edi_verifactu_state`

Entity `pos.order`, table `pos_order`. Label: {'en_US': 'Veri*Factu Status'}; stored; read only.

| Value | Label |
|---|---|
| `rejected` | {'en_US': 'Rejected'} |
| `registered_with_errors` | {'en_US': 'Registered with Errors'} |
| `accepted` | {'en_US': 'Accepted'} |
| `cancelled` | {'en_US': 'Cancelled'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `l10n_es_tbai_state`

Entity `pos.order`, table `pos_order`. Label: {'en_US': 'TicketBAI status'}; not stored; read only.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `l10n_jo_edi_pos_state`

Entity `pos.order`, table `pos_order`. Label: {'en_US': 'JoFotara State'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | {'en_US': 'To Send'} |
| `sent` | {'en_US': 'Sent'} |
| `demo` | {'en_US': 'Sent (Demo)'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `state`

Entity `pos.order`, table `pos_order`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Session — `activity_state`

Entity `pos.session`, table `pos_session`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Session — `state`

Entity `pos.session`, table `pos_session`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required; read only.

| Value | Label |
|---|---|
| `opening_control` | {'en_US': 'Opening Control', 'tr_TR': 'Açılış Kontrolü', 'ar_001': 'التحكم في الفتح'} |
| `opened` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `closing_control` | {'en_US': 'Closing Control', 'tr_TR': 'Kapanış Kotrolü', 'ar_001': 'التحكم في الإغلاق'} |
| `closed` | {'en_US': 'Closed & Posted', 'tr_TR': 'Kapalı & Onaylı', 'ar_001': 'تم الإغلاق والترحيل'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders Report — `state`

Entity `report.pos.order`, table `report_pos_order`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

## pricing-and-pricelists

### Product Margin — `invoice_state`

Entity `product.margin`, table `product_margin`. Label: {'en_US': 'Invoice State', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}; stored; required.

| Value | Label |
|---|---|
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `open_paid` | {'en_US': 'Open and Paid', 'tr_TR': 'Açık ve Ödendi', 'ar_001': 'مفتوحة ومدفوعة'} |
| `draft_open_paid` | {'en_US': 'Draft, Open and Paid', 'tr_TR': 'Taslak, Açık ve Ödenmiş', 'ar_001': 'مسودة، مفتوحة ومدفوعة'} |

Transitions: [pricing-and-pricelists state machines](../domains/pricing-and-pricelists/state-machines.md).

## products-and-catalog

### Pricelist — `activity_state`

Entity `product.pricelist`, table `product_pricelist`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

### Product Variant — `activity_state`

Entity `product.product`, table `product_product`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

### Product Variant — `invoice_state`

Entity `product.product`, table `product_product`. Label: {'en_US': 'Invoice State', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}; not stored; read only.

| Value | Label |
|---|---|
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `open_paid` | {'en_US': 'Open and Paid', 'tr_TR': 'Açık ve Ödendi', 'ar_001': 'مفتوحة ومدفوعة'} |
| `draft_open_paid` | {'en_US': 'Draft, Open and Paid', 'tr_TR': 'Taslak, Açık ve Ödenmiş', 'ar_001': 'مسودة، مفتوحة ومدفوعة'} |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

### Product — `activity_state`

Entity `product.template`, table `product_template`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

## projects-and-tasks

### Project — `activity_state`

Entity `project.project`, table `project_project`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Project — `last_update_status`

Entity `project.project`, table `project_project`. Label: {'en_US': 'Last Update Status'}; stored; required.

| Value | Label |
|---|---|
| `on_track` | {'en_US': 'On Track', 'tr_TR': 'İzleme Açık', 'ar_001': 'في المسار'} |
| `at_risk` | {'en_US': 'At Risk', 'tr_TR': 'Riskli', 'ar_001': 'في خطر'} |
| `off_track` | {'en_US': 'Off Track', 'tr_TR': 'İzleme Kapalı', 'ar_001': 'خارج المسار'} |
| `on_hold` | {'en_US': 'On Hold', 'tr_TR': 'Beklemede', 'ar_001': 'قيد الانتظار'} |
| `to_define` | {'en_US': 'Set Status', 'tr_TR': 'Durumu Ayarla', 'ar_001': 'قم بتحديد الحالة'} |
| `done` | {'en_US': 'Complete', 'tr_TR': 'Tamamlandı', 'ar_001': 'مكتمل'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Task — `activity_state`

Entity `project.task`, table `project_task`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Task — `state`

Entity `project.task`, table `project_task`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `01_in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `02_changes_requested` | {'en_US': 'Changes Requested', 'tr_TR': 'İstenen Değişiklikler', 'ar_001': 'التغييرات المطلوبة'} |
| `03_approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `1_done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `1_canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `04_waiting_normal` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Burndown Chart — `state`

Entity `project.task.burndown.chart.report`, table ``. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `01_in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `1_done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `04_waiting_normal` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `03_approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `1_canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `02_changes_requested` | {'en_US': 'Changes Requested', 'tr_TR': 'İstenen Değişiklikler', 'ar_001': 'التغييرات المطلوبة'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Task Stage — `rating_status`

Entity `project.task.type`, table `project_task_type`. Label: {'en_US': 'Customer Ratings Status', 'tr_TR': 'Müşteri Puanları Durumu', 'ar_001': 'حالة تقييمات العملاء'}; stored; required.

| Value | Label |
|---|---|
| `stage` | {'en_US': 'when reaching this stage', 'tr_TR': 'bu aşamaya ulaşıldığında'} |
| `periodic` | {'en_US': 'on a periodic basis', 'tr_TR': 'belirli aralıklarla', 'ar_001': 'على نحو دوري'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Project Update — `activity_state`

Entity `project.update`, table `project_update`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Project Update — `status`

Entity `project.update`, table `project_update`. Label: {'en_US': 'Status'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `on_track` | {'en_US': 'On Track', 'tr_TR': 'İzleme Açık', 'ar_001': 'في المسار'} |
| `at_risk` | {'en_US': 'At Risk', 'tr_TR': 'Riskli', 'ar_001': 'في خطر'} |
| `off_track` | {'en_US': 'Off Track', 'tr_TR': 'İzleme Kapalı', 'ar_001': 'خارج المسار'} |
| `on_hold` | {'en_US': 'On Hold', 'tr_TR': 'Beklemede', 'ar_001': 'قيد الانتظار'} |
| `done` | {'en_US': 'Complete', 'tr_TR': 'Tamamlandı', 'ar_001': 'مكتمل'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Tasks Analysis — `state`

Entity `report.project.task.user`, table `report_project_task_user`. Label: {'en_US': 'State', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `01_in_progress` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `1_done` | {'en_US': 'Done', 'tr_TR': 'Yapıldı', 'ar_001': 'منتهي'} |
| `04_waiting_normal` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `03_approved` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `1_canceled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `02_changes_requested` | {'en_US': 'Changes Requested', 'tr_TR': 'İstenen Değişiklikler', 'ar_001': 'التغييرات المطلوبة'} |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

## purchasing

### Purchase Order — `activity_state`

Entity `purchase.order`, table `purchase_order`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan'} |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Order — `invoice_status`

Entity `purchase.order`, table `purchase_order`. Label: {'en_US': 'Billing Status', 'tr_TR': 'Faturalama Durumu', 'ar_001': 'حالة الفاتورة'}; stored; read only.

| Value | Label |
|---|---|
| `no` | {'en_US': 'Nothing to Bill', 'tr_TR': 'Faturalanacak Bir Şey Yok', 'ar_001': 'لا يوجد شي لفوترته'} |
| `to invoice` | {'en_US': 'Waiting Bills', 'tr_TR': 'Bekleyen Faturalar', 'ar_001': 'الفواتير قيد الانتظار'} |
| `invoiced` | {'en_US': 'Fully Billed', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Order — `receipt_status`

Entity `purchase.order`, table `purchase_order`. Label: {'en_US': 'Receipt Status', 'tr_TR': 'Alım Durumu', 'ar_001': 'حالة الإيصال'}; stored; read only.

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Not Received', 'tr_TR': 'Alınmadı', 'ar_001': 'لم يتم الاستلام'} |
| `partial` | {'en_US': 'Partially Received', 'tr_TR': 'Kısmen Alındı', 'ar_001': 'تم الاستلام جزئياً'} |
| `full` | {'en_US': 'Fully Received', 'tr_TR': 'Tamamen Alındı', 'ar_001': 'تم الاستلام كلياً'} |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Order — `state`

Entity `purchase.order`, table `purchase_order`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'RFQ', 'tr_TR': 'Teklif Talebi', 'ar_001': 'طلب عرض سعر'} |
| `sent` | {'en_US': 'RFQ Sent', 'tr_TR': 'Teklif Talebi Gönderildi', 'ar_001': 'تم إرسال طلب عرض السعر'} |
| `to approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `purchase` | {'en_US': 'Purchase Order', 'tr_TR': 'Satınalma Siparişi', 'ar_001': 'أمر شراء'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Report — `state`

Entity `purchase.report`, table ``. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft RFQ', 'tr_TR': 'Taslak Teklif Talebi', 'ar_001': 'مسودة طلب عرض سعر'} |
| `sent` | {'en_US': 'RFQ Sent', 'tr_TR': 'Teklif Talebi Gönderildi', 'ar_001': 'تم إرسال طلب عرض السعر'} |
| `to approve` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `purchase` | {'en_US': 'Purchase Order', 'tr_TR': 'Satınalma Siparişi', 'ar_001': 'أمر شراء'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Requisition — `activity_state`

Entity `purchase.requisition`, table `purchase_requisition`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Requisition — `state`

Entity `purchase.requisition`, table `purchase_requisition`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Draft', 'tr_TR': 'Taslak', 'ar_001': 'مسودة'} |
| `confirmed` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `done` | {'en_US': 'Closed', 'tr_TR': 'Kapanmış', 'ar_001': 'مغلق'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

## recruitment

### Applicant — `activity_state`

Entity `hr.applicant`, table `hr_applicant`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [recruitment state machines](../domains/recruitment/state-machines.md).

### Applicant — `application_status`

Entity `hr.applicant`, table `hr_applicant`. Label: {'en_US': 'Application Status', 'tr_TR': 'Başvuru Durumu', 'ar_001': 'حالة طلب التقديم'}; not stored; read only.

| Value | Label |
|---|---|
| `ongoing` | {'en_US': 'Ongoing', 'tr_TR': 'Devam eden', 'ar_001': 'جاري'} |
| `hired` | {'en_US': 'Hired', 'tr_TR': 'İşe Alınan', 'ar_001': 'تم التعيين'} |
| `refused` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `archived` | {'en_US': 'Archived', 'tr_TR': 'Arşivlendi', 'ar_001': 'مؤرشف'} |

Transitions: [recruitment state machines](../domains/recruitment/state-machines.md).

### Applicant — `kanban_state`

Entity `hr.applicant`, table `hr_applicant`. Label: {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}; stored; required.

| Value | Label |
|---|---|
| `normal` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `done` | {'en_US': 'Ready for Next Stage', 'tr_TR': 'Bir Sonraki Aşama için Hazır', 'ar_001': 'جاهز للمرحلة التالية'} |
| `waiting` | {'en_US': 'Waiting', 'tr_TR': 'Bekleyen', 'ar_001': 'قيد الانتظار'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |

Transitions: [recruitment state machines](../domains/recruitment/state-machines.md).

## repair-and-maintenance

### Maintenance Equipment — `activity_state`

Entity `maintenance.equipment`, table `maintenance_equipment`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Maintenance Request — `activity_state`

Entity `maintenance.request`, table `maintenance_request`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Maintenance Request — `kanban_state`

Entity `maintenance.request`, table `maintenance_request`. Label: {'en_US': 'Kanban State', 'tr_TR': 'Kanban Durumu', 'ar_001': 'حالة كانبان'}; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `normal` | {'en_US': 'In Progress', 'tr_TR': 'İşlemde', 'ar_001': 'قيد التنفيذ'} |
| `blocked` | {'en_US': 'Blocked', 'tr_TR': 'Engellendi', 'ar_001': 'محجوب'} |
| `done` | {'en_US': 'Ready for next stage', 'tr_TR': 'Sonraki aşaması için hazır', 'ar_001': 'جاهز للمرحلة التالية'} |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Repair Order — `activity_state`

Entity `repair.order`, table `repair_order`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Repair Order — `parts_availability_state`

Entity `repair.order`, table `repair_order`. Label: {'en_US': 'Parts Availability State', 'tr_TR': 'Parça Uygunluk Durumu', 'ar_001': 'حالة توافر الأجزاء'}; not stored; read only.

| Value | Label |
|---|---|
| `available` | {'en_US': 'Available', 'tr_TR': 'Uygun', 'ar_001': 'متاح'} |
| `expected` | {'en_US': 'Expected', 'tr_TR': 'Beklenen', 'ar_001': 'المتوقع'} |
| `late` | {'en_US': 'Late', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Repair Order — `state`

Entity `repair.order`, table `repair_order`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `confirmed` | {'en_US': 'Confirmed', 'tr_TR': 'Doğrulanmış', 'ar_001': 'تم التأكيد'} |
| `under_repair` | {'en_US': 'Under Repair', 'tr_TR': 'Onarımda', 'ar_001': 'قيد الإصلاح'} |
| `done` | {'en_US': 'Repaired', 'tr_TR': 'Onarıldı', 'ar_001': 'تم الإصلاح'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

## sales

### Sales Order — `activity_state`

Entity `sale.order`, table `sale_order`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue'} |
| `today` | {'en_US': 'Today'} |
| `planned` | {'en_US': 'Planned'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order — `delivery_status`

Entity `sale.order`, table `sale_order`. Label: {'en_US': 'Delivery Status', 'tr_TR': 'Teslim Durumu', 'ar_001': 'حالة التوصيل'}; stored; read only.

| Value | Label |
|---|---|
| `pending` | {'en_US': 'Not Delivered', 'tr_TR': 'Teslim Edilmedi', 'ar_001': 'لم يتم توصيلها'} |
| `started` | {'en_US': 'Started', 'tr_TR': 'Başladı', 'ar_001': 'بدأ'} |
| `partial` | {'en_US': 'Partially Delivered', 'tr_TR': 'Kısmen Teslim Edildi', 'ar_001': 'تم التوصيل جزئياً'} |
| `full` | {'en_US': 'Fully Delivered', 'tr_TR': 'Tam Teslim', 'ar_001': 'تم التوصيل بالكامل'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order — `invoice_status`

Entity `sale.order`, table `sale_order`. Label: {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}; stored; read only.

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order — `state`

Entity `sale.order`, table `sale_order`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Quotation', 'tr_TR': 'Teklif', 'ar_001': 'عرض سعر'} |
| `sent` | {'en_US': 'Quotation Sent', 'tr_TR': 'Gönderilen', 'ar_001': 'تم إرسال عرض السعر'} |
| `sale` | {'en_US': 'Sales Order', 'tr_TR': 'Satış Siparişi', 'ar_001': 'أمر البيع'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order Line — `invoice_status`

Entity `sale.order.line`, table `sale_order_line`. Label: {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}; stored; read only.

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Analysis Report — `invoice_status`

Entity `sale.report`, table ``. Label: {'en_US': 'Order Invoice Status', 'tr_TR': 'Sipariş Fatura Durumu', 'ar_001': 'حالة فاتورة الطلب'}; stored; read only.

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Analysis Report — `line_invoice_status`

Entity `sale.report`, table ``. Label: {'en_US': 'Invoice Status', 'tr_TR': 'Fatura Durumu', 'ar_001': 'حالة الفاتورة'}; stored; read only.

| Value | Label |
|---|---|
| `upselling` | {'en_US': 'Upselling Opportunity', 'tr_TR': 'Arttırılmış Satış Fırsatı', 'ar_001': 'فرصة الارتقاء بالصفقة'} |
| `invoiced` | {'en_US': 'Fully Invoiced', 'tr_TR': 'Tamamı Faturalanan', 'ar_001': 'مفوتر بالكامل'} |
| `to invoice` | {'en_US': 'To Invoice', 'tr_TR': 'Faturalanacak', 'ar_001': 'بانتظار الفوترة'} |
| `no` | {'en_US': 'Nothing to Invoice', 'tr_TR': 'Faturalandıracak bir şey yok', 'ar_001': 'لا توجد مبالغ لفوترتها'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Analysis Report — `state`

Entity `sale.report`, table ``. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'Quotation', 'tr_TR': 'Teklif', 'ar_001': 'عرض سعر'} |
| `sent` | {'en_US': 'Quotation Sent', 'tr_TR': 'Gönderilen', 'ar_001': 'تم إرسال عرض السعر'} |
| `sale` | {'en_US': 'Sales Order', 'tr_TR': 'Satış Siparişi', 'ar_001': 'أمر البيع'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `paid` | {'en_US': 'Paid', 'tr_TR': 'Ödendi', 'ar_001': 'مدفوع'} |
| `invoiced` | {'en_US': 'Invoiced', 'tr_TR': 'Faturalanan', 'ar_001': 'مفوتر'} |
| `done` | {'en_US': 'Posted', 'tr_TR': 'Onaylanmış', 'ar_001': 'مُرحّل'} |

Transitions: [sales state machines](../domains/sales/state-machines.md).

## time-off

### Time Off — `activity_state`

Entity `hr.leave`, table `hr_leave`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off — `state`

Entity `hr.leave`, table `hr_leave`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Allocation — `activity_state`

Entity `hr.leave.allocation`, table `hr_leave_allocation`. Label: {'en_US': 'Activity State', 'tr_TR': 'Aktivite Durumu', 'ar_001': 'حالة النشاط'}; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | {'en_US': 'Overdue', 'tr_TR': 'Geciken', 'ar_001': 'متأخر'} |
| `today` | {'en_US': 'Today', 'tr_TR': 'Bugün', 'ar_001': 'اليوم'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Allocation — `state`

Entity `hr.leave.allocation`, table `hr_leave_allocation`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Summary / Report — `holiday_status`

Entity `hr.leave.employee.type.report`, table `hr_leave_employee_type_report`. Label: {'en_US': 'Holiday Status', 'tr_TR': 'Tatil Durumu', 'ar_001': 'حالة الإجازة'}; stored.

| Value | Label |
|---|---|
| `taken` | {'en_US': 'Taken', 'tr_TR': 'Alınmış', 'ar_001': 'تم أخذها'} |
| `left` | {'en_US': 'Left', 'tr_TR': 'Sol', 'ar_001': 'يسار'} |
| `planned` | {'en_US': 'Planned', 'tr_TR': 'Planlanan', 'ar_001': 'المخطط له'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Summary / Report — `state`

Entity `hr.leave.employee.type.report`, table `hr_leave_employee_type_report`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Summary / Report — `state`

Entity `hr.leave.report`, table `hr_leave_report`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Calendar — `state`

Entity `hr.leave.report.calendar`, table `hr_leave_report_calendar`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الحالة'}; stored; read only.

| Value | Label |
|---|---|
| `cancel` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |
| `confirm` | {'en_US': 'To Approve', 'tr_TR': 'Onaylanacak', 'ar_001': 'بانتظار الموافقة'} |
| `refuse` | {'en_US': 'Refused', 'tr_TR': 'Reddedildi', 'ar_001': 'تم الرفض'} |
| `validate1` | {'en_US': 'Second Approval', 'tr_TR': 'İkinci Onay', 'ar_001': 'الموافقة الثانية'} |
| `validate` | {'en_US': 'Approved', 'tr_TR': 'Onaylanmış', 'ar_001': 'تمت الموافقة'} |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

## website-and-storefront

### Forum Post — `state`

Entity `forum.post`, table `forum_post`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; stored.

| Value | Label |
|---|---|
| `active` | {'en_US': 'Active', 'tr_TR': 'Etkin', 'ar_001': 'نشط'} |
| `pending` | {'en_US': 'Waiting Validation', 'tr_TR': 'Bekleyen Doğrulama', 'ar_001': 'بانتظار التصديق'} |
| `close` | {'en_US': 'Closed', 'tr_TR': 'Kapanmış', 'ar_001': 'مغلق'} |
| `offensive` | {'en_US': 'Offensive', 'tr_TR': 'Saldırgan', 'ar_001': 'مسيء'} |
| `flagged` | {'en_US': 'Flagged', 'tr_TR': 'İşaretli', 'ar_001': 'مُعلّم بعلامة'} |

Transitions: [website-and-storefront state machines](../domains/website-and-storefront/state-machines.md).

### Portal User Config — `email_state`

Entity `portal.wizard.user`, table `portal_wizard_user`. Label: {'en_US': 'Status', 'tr_TR': 'Durumu', 'ar_001': 'الحالة'}; not stored; read only.

| Value | Label |
|---|---|
| `ok` | {'en_US': 'Valid', 'tr_TR': 'Geçerli', 'ar_001': 'صالح'} |
| `ko` | {'en_US': 'Invalid', 'tr_TR': 'Geçersiz', 'ar_001': 'غير صالح'} |
| `exist` | {'en_US': 'Already Registered', 'tr_TR': 'Zaten Kayıtlı', 'ar_001': 'مسجل بالفعل'} |

Transitions: [website-and-storefront state machines](../domains/website-and-storefront/state-machines.md).

## work-entries

### human resources Work Entry — `state`

Entity `hr.work.entry`, table `hr_work_entry`. Label: {'en_US': 'State', 'tr_TR': 'İl/Eyalet', 'ar_001': 'الولاية'}; stored.

| Value | Label |
|---|---|
| `draft` | {'en_US': 'New', 'tr_TR': 'Yeni', 'ar_001': 'جديد'} |
| `conflict` | {'en_US': 'In Conflict', 'tr_TR': 'Çatışma Halinde'} |
| `validated` | {'en_US': 'In Payslip', 'tr_TR': 'Maaş Bordrosunda', 'ar_001': 'في كشوف الرواتب'} |
| `cancelled` | {'en_US': 'Cancelled', 'tr_TR': 'İptal Edildi', 'ar_001': 'تم الإلغاء'} |

Transitions: [work-entries state machines](../domains/work-entries/state-machines.md).

