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

Entity `hr.attendance`, table `hr_attendance`. Label: Overtime Status; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_approve` | To Approve |
| `approved` | Approved |
| `refused` | Refused |

Transitions: [attendances-and-working-time state machines](../domains/attendances-and-working-time/state-machines.md).

### Attendance Overtime Line — `status`

Entity `hr.attendance.overtime.line`, table `hr_attendance_overtime_line`. Label: Status; stored; required.

| Value | Label |
|---|---|
| `to_approve` | To Approve |
| `approved` | Approved |
| `refused` | Refused |

Transitions: [attendances-and-working-time state machines](../domains/attendances-and-working-time/state-machines.md).

## automation-and-integration

### Automation Rule — `activity_state`

Entity `base.automation`, table `base_automation`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Import Module — `state`

Entity `base.import.module`, table `base_import_module`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `init` | init |
| `done` | done |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### in-app purchase Account — `state`

Entity `iap.account`, table `iap_account`. Label: State; stored; read only.

| Value | Label |
|---|---|
| `banned` | Banned |
| `registered` | Registered |
| `unregistered` | Unregistered |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding — `current_onboarding_state`

Entity `onboarding.onboarding`, table `onboarding_onboarding`. Label: Completion State; not stored; read only.

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding Step — `current_step_state`

Entity `onboarding.onboarding.step`, table `onboarding_onboarding_step`. Label: Completion State; not stored; read only.

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding Progress Tracker — `onboarding_state`

Entity `onboarding.progress`, table `onboarding_progress`. Label: Onboarding progress; stored; read only.

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

### Onboarding Progress Step Tracker — `step_state`

Entity `onboarding.progress.step`, table `onboarding_progress_step`. Label: Onboarding Step Progress; stored.

| Value | Label |
|---|---|
| `not_done` | Not done |
| `just_done` | Just done |
| `done` | Done |

Transitions: [automation-and-integration state machines](../domains/automation-and-integration/state-machines.md).

## calendar-and-scheduling

### Calendar Attendee Information — `availability`

Entity `calendar.attendee`, table `calendar_attendee`. Label: Available/Busy; stored; read only.

| Value | Label |
|---|---|
| `free` | Available |
| `busy` | Busy |

Transitions: [calendar-and-scheduling state machines](../domains/calendar-and-scheduling/state-machines.md).

### Calendar Attendee Information — `state`

Entity `calendar.attendee`, table `calendar_attendee`. Label: Status; stored.

| Value | Label |
|---|---|
| `accepted` | Yes |
| `declined` | No |
| `tentative` | Maybe |
| `needsAction` | Needs Action |

Transitions: [calendar-and-scheduling state machines](../domains/calendar-and-scheduling/state-machines.md).

## customer-relationship-management

### customer relationship management Activity Analysis — `won_status`

Entity `crm.activity.report`, table `crm_activity_report`. Label: Is Won; stored; read only.

| Value | Label |
|---|---|
| `won` | Won |
| `lost` | Lost |
| `pending` | Pending |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### customer relationship management Lead Mining Request — `state`

Entity `crm.iap.lead.mining.request`, table `crm_iap_lead_mining_request`. Label: Status; stored; required.

| Value | Label |
|---|---|
| `draft` | Draft |
| `error` | Error |
| `done` | Done |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `activity_state`

Entity `crm.lead`, table `crm_lead`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `email_state`

Entity `crm.lead`, table `crm_lead`. Label: Email Quality; stored; read only.

| Value | Label |
|---|---|
| `correct` | Correct |
| `incorrect` | Incorrect |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `phone_state`

Entity `crm.lead`, table `crm_lead`. Label: Phone Quality; stored; read only.

| Value | Label |
|---|---|
| `correct` | Correct |
| `incorrect` | Incorrect |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### Lead — `won_status`

Entity `crm.lead`, table `crm_lead`. Label: Won/Lost; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `won` | Won |
| `lost` | Lost |
| `pending` | Pending |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

### customer relationship management Reveal View — `reveal_state`

Entity `crm.reveal.view`, table `crm_reveal_view`. Label: State; stored.

| Value | Label |
|---|---|
| `to_process` | To Process |
| `not_found` | Not Found |

Transitions: [customer-relationship-management state machines](../domains/customer-relationship-management/state-machines.md).

## electronic-invoicing-and-document-exchange

### Electronic Document for an account.move — `state`

Entity `account.edi.document`, table `account_edi_document`. Label: State; stored.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `to_cancel` | To Cancel |
| `cancelled` | Cancelled |

Transitions: [electronic-invoicing-and-document-exchange state machines](../domains/electronic-invoicing-and-document-exchange/state-machines.md).

### Business Level Responses for Peppol — `pdp_ppf_state`

Entity `account.peppol.response`, table `account_peppol_response`. Label: PPF Status; stored; read only.

| Value | Label |
|---|---|
| `sent` | Sent |
| `received` | received |
| `error` | Error |

Transitions: [electronic-invoicing-and-document-exchange state machines](../domains/electronic-invoicing-and-document-exchange/state-machines.md).

### Business Level Responses for Peppol — `peppol_state`

Entity `account.peppol.response`, table `account_peppol_response`. Label: Peppol status; stored.

| Value | Label |
|---|---|
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `not_serviced` | Not Serviced |

Transitions: [electronic-invoicing-and-document-exchange state machines](../domains/electronic-invoicing-and-document-exchange/state-machines.md).

## events

### Event Booth — `activity_state`

Entity `event.booth`, table `event_booth`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Booth — `state`

Entity `event.booth`, table `event_booth`. Label: Status; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `available` | Available |
| `unavailable` | Unavailable |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event — `activity_state`

Entity `event.event`, table `event_event`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event — `kanban_state`

Entity `event.event`, table `event_event`. Label: Kanban State; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `normal` | In Progress |
| `done` | Ready for Next Stage |
| `blocked` | Blocked |
| `cancel` | Cancelled |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Automated Mailing — `mail_state`

Entity `event.mail`, table `event_mail`. Label: Global communication Status; not stored; read only.

| Value | Label |
|---|---|
| `running` | Running |
| `scheduled` | Scheduled |
| `sent` | Sent |
| `error` | Error |
| `cancelled` | Cancelled |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Registration — `activity_state`

Entity `event.registration`, table `event_registration`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Registration — `sale_status`

Entity `event.registration`, table `event_registration`. Label: Sale Status; stored; read only.

| Value | Label |
|---|---|
| `to_pay` | Not Sold |
| `sold` | Sold |
| `free` | Free |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Registration — `state`

Entity `event.registration`, table `event_registration`. Label: Status; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Unconfirmed |
| `open` | Registered |
| `done` | Attended |
| `cancel` | Cancelled |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sales Report — `event_registration_state`

Entity `event.sale.report`, table `event_sale_report`. Label: Registration Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | Unconfirmed |
| `cancel` | Cancelled |
| `open` | Confirmed |
| `done` | Attended |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sales Report — `sale_order_state`

Entity `event.sale.report`, table `event_sale_report`. Label: Sale Order Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | Quotation |
| `sent` | Quotation Sent |
| `sale` | Sales Order |
| `cancel` | Cancelled |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sales Report — `sale_status`

Entity `event.sale.report`, table `event_sale_report`. Label: Payment Status; stored.

| Value | Label |
|---|---|
| `to_pay` | Not Sold |
| `sold` | Sold |
| `free` | Free |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Sponsor — `activity_state`

Entity `event.sponsor`, table `event_sponsor`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Track — `activity_state`

Entity `event.track`, table `event_track`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [events state machines](../domains/events/state-machines.md).

### Event Track — `kanban_state`

Entity `event.track`, table `event_track`. Label: Kanban State; stored; required.

| Value | Label |
|---|---|
| `normal` | Grey |
| `done` | Green |
| `blocked` | Red |

Transitions: [events state machines](../domains/events/state-machines.md).

## expenses

### Expense — `activity_state`

Entity `hr.expense`, table `hr_expense`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

### Expense — `approval_state`

Entity `hr.expense`, table `hr_expense`. Label: Approval State; stored; read only.

| Value | Label |
|---|---|
| `submitted` | Submitted |
| `approved` | Approved |
| `refused` | Refused |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

### Expense — `state`

Entity `hr.expense`, table `hr_expense`. Label: Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `submitted` | Submitted |
| `approved` | Approved |
| `posted` | Posted |
| `in_payment` | In Payment |
| `paid` | Paid |
| `refused` | Refused |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

### Expense Split — `approval_state`

Entity `hr.expense.split`, table `hr_expense_split`. Label: Approval State; stored; read only.

| Value | Label |
|---|---|
| `submitted` | Submitted |
| `approved` | Approved |
| `refused` | Refused |

Transitions: [expenses state machines](../domains/expenses/state-machines.md).

## fiscal-localizations

### French PDP Flow — `activity_state`

Entity `l10n.fr.pdp.reports.flow`, table `l10n_fr_pdp_reports_flow`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### French PDP Flow — `period_status`

Entity `l10n.fr.pdp.reports.flow`, table `l10n_fr_pdp_reports_flow`. Label: Period Status; not stored; read only.

| Value | Label |
|---|---|
| `open` | Open |
| `grace` | Grace |
| `closed` | Closed |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### French PDP Flow — `state`

Entity `l10n.fr.pdp.reports.flow`, table `l10n_fr_pdp_reports_flow`. Label: Status; stored; required.

| Value | Label |
|---|---|
| `ready` | Ready |
| `error` | Error |
| `sent` | Sent |
| `completed` | Completed |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### e-Waybill — `activity_state`

Entity `l10n.in.ewaybill`, table `l10n_in_ewaybill`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### e-Waybill — `state`

Entity `l10n.in.ewaybill`, table `l10n_in_ewaybill`. Label: Status; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `pending` | Pending |
| `generated` | Generated |
| `cancel` | Cancelled |
| `challan` | Challan |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### TicketBAI Document — `state`

Entity `l10n_es_edi_tbai.document`, table `l10n_es_edi_tbai_document`. Label: status; stored; read only.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `accepted` | Accepted |
| `rejected` | Rejected |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Veri*Factu Document — `state`

Entity `l10n_es_edi_verifactu.document`, table `l10n_es_edi_verifactu_document`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Greece document object for tracking all sent extensible markup language to myDATA — `provider_pdf_state`

Entity `l10n_gr_edi.document`, table `l10n_gr_edi_document`. Label: Final PDF Status; stored.

| Value | Label |
|---|---|
| `pending` | Pending |
| `sent` | Sent |
| `error` | Failed |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Greece document object for tracking all sent extensible markup language to myDATA — `state`

Entity `l10n_gr_edi.document`, table `l10n_gr_edi_document`. Label: myDATA Status; stored; required.

| Value | Label |
|---|---|
| `invoice_sent` | Invoice sent |
| `invoice_error` | Invoice send failed |
| `bill_fetched` | Expense classification ready to send |
| `bill_sent` | Expense classification sent |
| `bill_error` | Expense classification send failed |
| `invoice_pending` | Invoice submission pending |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### electronic data interchange and fiscalization information for Croatian electronic invoicing — `business_document_status`

Entity `l10n_hr_edi.addendum`, table `l10n_hr_edi_addendum`. Label: Business document status; stored.

| Value | Label |
|---|---|
| `0` | APPROVED |
| `1` | REJECTED |
| `2` | PAYMENT_FULFILLED |
| `3` | PAYMENT_PARTIALLY_FULLFILLED |
| `4` | RECEIVING_CONFIRMED |
| `99` | RECEIVED |
| `None` | None |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### electronic data interchange and fiscalization information for Croatian electronic invoicing — `fiscalization_status`

Entity `l10n_hr_edi.addendum`, table `l10n_hr_edi_addendum`. Label: Fiscalization status; stored.

| Value | Label |
|---|---|
| `0` | Successful |
| `1` | Unsuccessful |
| `2` | Pending |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### electronic data interchange and fiscalization information for Croatian electronic invoicing — `mer_document_status`

Entity `l10n_hr_edi.addendum`, table `l10n_hr_edi_addendum`. Label: MojEracun document status; stored.

| Value | Label |
|---|---|
| `20` | In validation |
| `30` | Sent |
| `40` | Delivered |
| `45` | Canceled |
| `50` | Unsuccessful |
| `70` | Delivered (eReporting) |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### E-Faktur Document — `activity_state`

Entity `l10n_id_efaktur_coretax.document`, table `l10n_id_efaktur_coretax_document`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Indian permanent account number Entity — `activity_state`

Entity `l10n_in.pan.entity`, table `l10n_in_pan_entity`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Declaration of Intent — `activity_state`

Entity `l10n_it_edi_doi.declaration_of_intent`, table `l10n_it_edi_doi_declaration_of_intent`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Declaration of Intent — `state`

Entity `l10n_it_edi_doi.declaration_of_intent`, table `l10n_it_edi_doi_declaration_of_intent`. Label: State; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `active` | Active |
| `revoked` | Revoked |
| `terminated` | Terminated |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### PL Bank Account Verification — `verification_status`

Entity `l10n_pl.bank.account.verification`, table `l10n_pl_bank_account_verification`. Label: Verification Status; stored; required; read only.

| Value | Label |
|---|---|
| `valid` | Valid |
| `invalid` | Invalid |
| `incomplete_partner` | Incomplete partner |
| `not_found_partner` | Partner not found |
| `error` | An error occurred during check with Government API |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Document object for tracking CIUS-RO extensible markup language sent to E-Factura — `state`

Entity `l10n_ro_edi.document`, table `l10n_ro_edi_document`. Label: E-Factura Status; stored; required; read only.

| Value | Label |
|---|---|
| `invoice_sent` | Sent |
| `invoice_refused` | Error |
| `invoice_validated` | Validated |
| `stock_sent` | Sent |
| `stock_sending_failed` | Error |
| `stock_validated` | Validated |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### MyInvois Document — `activity_state`

Entity `myinvois.document`, table `myinvois_document`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### MyInvois Document — `myinvois_state`

Entity `myinvois.document`, table `myinvois_document`. Label: MyInvois State; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `in_progress` | Validation In Progress |
| `valid` | Valid |
| `rejected` | Rejected |
| `invalid` | Invalid |
| `cancelled` | Cancelled |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### Business Level Responses for Nemhandel — `nemhandel_state`

Entity `nemhandel.response`, table `nemhandel_response`. Label: Nemhandel status; stored.

| Value | Label |
|---|---|
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `not_serviced` | Not Serviced |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

### PDP Response wizard — `status`

Entity `pdp.response.wizard`, table `pdp_response_wizard`. Label: Status; stored.

| Value | Label |
|---|---|
| `PD` | Paid |
| `cancelled` | Cancelled |
| `suspended` | Suspended |
| `refused` | Refused |
| `AP` | Approved |
| `in_hand` | In Hand |
| `completed` | Completed |

Transitions: [fiscal-localizations state machines](../domains/fiscal-localizations/state-machines.md).

## fleet

### Vehicle — `activity_state`

Entity `fleet.vehicle`, table `fleet_vehicle`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Vehicle — `contract_state`

Entity `fleet.vehicle`, table `fleet_vehicle`. Label: Last Contract State; not stored; read only.

| Value | Label |
|---|---|
| `futur` | Incoming |
| `open` | In Progress |
| `expired` | Expired |
| `closed` | Closed |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Vehicle Contract — `activity_state`

Entity `fleet.vehicle.log.contract`, table `fleet_vehicle_log_contract`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Vehicle Contract — `state`

Entity `fleet.vehicle.log.contract`, table `fleet_vehicle_log_contract`. Label: Status; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `futur` | New |
| `open` | Running |
| `expired` | Expired |
| `closed` | Cancelled |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Services for vehicles — `activity_state`

Entity `fleet.vehicle.log.services`, table `fleet_vehicle_log_services`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Services for vehicles — `state`

Entity `fleet.vehicle.log.services`, table `fleet_vehicle_log_services`. Label: Stage; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `new` | New |
| `running` | Running |
| `done` | Done |
| `cancelled` | Cancelled |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

### Model of a vehicle — `activity_state`

Entity `fleet.vehicle.model`, table `fleet_vehicle_model`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [fleet state machines](../domains/fleet/state-machines.md).

## general-ledger

### Account — `activity_state`

Entity `account.account`, table `account_account`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Invoices Statistics — `payment_state`

Entity `account.invoice.report`, table ``. Label: Payment Status; stored; read only.

| Value | Label |
|---|---|
| `not_paid` | Not Paid |
| `in_payment` | In Payment |
| `paid` | Paid |
| `partial` | Partially Paid |
| `reversed` | Reversed |
| `blocked` | Blocked |
| `invoicing_legacy` | Invoicing App Legacy |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Invoices Statistics — `state`

Entity `account.invoice.report`, table ``. Label: Invoice Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | Draft |
| `posted` | Open |
| `cancel` | Cancelled |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal — `activity_state`

Entity `account.journal`, table `account_journal`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Account Lock Exception — `state`

Entity `account.lock_exception`, table `account_lock_exception`. Label: State; not stored; read only.

| Value | Label |
|---|---|
| `active` | Active |
| `revoked` | Revoked |
| `expired` | Expired |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `activity_state`

Entity `account.move`, table `account_move`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `edi_state`

Entity `account.move`, table `account_move`. Label: Electronic invoicing; stored; read only.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `to_cancel` | To Cancel |
| `cancelled` | Cancelled |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_es_edi_verifactu_state`

Entity `account.move`, table `account_move`. Label: Veri*Factu Status; stored; read only.

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |
| `cancelled` | Cancelled |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_es_tbai_state`

Entity `account.move`, table `account_move`. Label: TicketBAI status; not stored; read only.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `cancelled` | Cancelled |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_fr_pdp_status`

Entity `account.move`, table `account_move`. Label: E-Reporting Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `out_of_scope` | Out of scope |
| `pending` | Pending |
| `ready` | Ready |
| `error` | Error |
| `sent` | Sent |
| `completed` | Completed |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_gr_edi_state`

Entity `account.move`, table `account_move`. Label: myDATA Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `invoice_sent` | Invoice sent |
| `bill_fetched` | Expense classification ready to send |
| `bill_sent` | Expense classification sent |
| `invoice_pending` | Invoice submission pending |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_hu_edi_state`

Entity `account.move`, table `account_move`. Label: NAV 3.0 status; stored.

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

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_in_edi_status`

Entity `account.move`, table `account_move`. Label: India E-Invoice Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `cancelled` | Cancelled |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_it_edi_state`

Entity `account.move`, table `account_move`. Label: SDI State; stored; changes are tracked in the message thread.

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

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_jo_edi_state`

Entity `account.move`, table `account_move`. Label: JoFotara State; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `demo` | Sent (Demo) |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_my_edi_state`

Entity `account.move`, table `account_move`. Label: MyInvois State; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `in_progress` | Validation In Progress |
| `valid` | Valid |
| `rejected` | Rejected |
| `invalid` | Invalid |
| `cancelled` | Cancelled |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_pl_edi_status`

Entity `account.move`, table `account_move`. Label: KSeF Status; stored; read only.

| Value | Label |
|---|---|
| `sent` | Sent (In Progress) |
| `accepted` | Accepted |
| `rejected` | Rejected |
| `fetch_ready` | Fetch Ready |
| `fetched` | Fetched |
| `fetch_failed` | Fetch Failed |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_ro_edi_state`

Entity `account.move`, table `account_move`. Label: E-Factura Status; stored; read only.

| Value | Label |
|---|---|
| `invoice_not_indexed` | Not indexed |
| `invoice_sent` | Sent |
| `invoice_refused` | Refused |
| `invoice_validated` | Validated |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_rs_edi_state`

Entity `account.move`, table `account_move`. Label: Serbia E-Invoice state; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `sent` | Sent |
| `sending_failed` | Error |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_tr_nilvera_send_status`

Entity `account.move`, table `account_move`. Label: Nilvera Status; stored; read only.

| Value | Label |
|---|---|
| `error` | Error |
| `not_sent` | Not sent |
| `sent` | Sent and waiting response |
| `succeed` | Successful |
| `waiting` | Waiting |
| `unknown` | Unknown |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_tw_edi_refund_state`

Entity `account.move`, table `account_move`. Label: Refund State; stored; read only.

| Value | Label |
|---|---|
| `to_be_agreed` | To be agreed |
| `agreed` | Agreed |
| `disagreed` | Disagreed |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_tw_edi_state`

Entity `account.move`, table `account_move`. Label: Invoice Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `invoiced` | Invoiced |
| `valid` | Valid |
| `invalid` | Invalid |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `l10n_vn_edi_invoice_state`

Entity `account.move`, table `account_move`. Label: Sinvoice Status; stored.

| Value | Label |
|---|---|
| `ready_to_send` | Ready to send |
| `sent` | Sent |
| `payment_state_to_update` | Payment status to update |
| `canceled` | Canceled |
| `adjusted` | Adjusted |
| `replaced` | Replaced |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `nemhandel_move_state`

Entity `account.move`, table `account_move`. Label: Nemhandel status; stored; read only.

| Value | Label |
|---|---|
| `ready` | Ready to send |
| `to_send` | Queued |
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `BusinessAccept` | Approved |
| `BusinessReject` | Rejected |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `payment_state`

Entity `account.move`, table `account_move`. Label: Payment Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `not_paid` | Not Paid |
| `in_payment` | In Payment |
| `paid` | Paid |
| `partial` | Partially Paid |
| `reversed` | Reversed |
| `blocked` | Blocked |
| `invoicing_legacy` | Invoicing App Legacy |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `pdp_ppf_lifecycle_state`

Entity `account.move`, table `account_move`. Label: PPF Lifeycle Status; stored; read only.

| Value | Label |
|---|---|
| `in_progress` | In Progress |
| `sent` | Sent |
| `done` | Done |
| `error` | Error |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `pdp_ppf_move_state`

Entity `account.move`, table `account_move`. Label: PPF Invoice Status; stored; read only.

| Value | Label |
|---|---|
| `in_progress` | In Progress |
| `sent` | Sent |
| `done` | Done |
| `error` | Error |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `peppol_move_state`

Entity `account.move`, table `account_move`. Label: E-Invoicing Status; stored; read only.

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

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Journal Entry — `state`

Entity `account.move`, table `account_move`. Label: Status; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `posted` | Posted |
| `cancel` | Cancelled |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Payments — `activity_state`

Entity `account.payment`, table `account_payment`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

### Payments — `state`

Entity `account.payment`, table `account_payment`. Label: State; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_process` | In Process |
| `paid` | Paid |
| `canceled` | Canceled |
| `rejected` | Rejected |

Transitions: [general-ledger state machines](../domains/general-ledger/state-machines.md).

## human-resources-core

### Department — `activity_state`

Entity `hr.department`, table `hr_department`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `activity_state`

Entity `hr.employee`, table `hr_employee`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `attendance_state`

Entity `hr.employee`, table `hr_employee`. Label: Attendance Status; not stored; read only.

| Value | Label |
|---|---|
| `checked_out` | Checked out |
| `checked_in` | Checked in |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `current_leave_state`

Entity `hr.employee`, table `hr_employee`. Label: Current Time Off Status; not stored; read only.

| Value | Label |
|---|---|
| `confirm` | Waiting Approval |
| `refuse` | Refused |
| `validate1` | Waiting Second Approval |
| `validate` | Approved |
| `cancel` | Cancelled |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Employee — `hr_presence_state`

Entity `hr.employee`, table `hr_employee`. Label: Hr Presence State; not stored; read only.

| Value | Label |
|---|---|
| `present` | Present |
| `absent` | Absent |
| `archive` | Archived |
| `out_of_working_hour` | Off-Hours |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Public Employee — `hr_presence_state`

Entity `hr.employee.public`, table `hr_employee_public`. Label: Hr Presence State; not stored; read only.

| Value | Label |
|---|---|
| `present` | Present |
| `absent` | Absent |
| `archive` | Archived |
| `out_of_working_hour` | Off-Hours |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Job Position — `activity_state`

Entity `hr.job`, table `hr_job`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Resume line of an employee — `expiration_status`

Entity `hr.resume.line`, table `hr_resume_line`. Label: Expiration Status; stored; read only.

| Value | Label |
|---|---|
| `expired` | Expired |
| `expiring` | Expiring |
| `valid` | Valid |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

### Version — `activity_state`

Entity `hr.version`, table `hr_version`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [human-resources-core state machines](../domains/human-resources-core/state-machines.md).

## inventory-operations

### Stock Quantity Report — `state`

Entity `report.stock.quantity`, table `report_stock_quantity`. Label: State; stored; read only.

| Value | Label |
|---|---|
| `forecast` | Forecasted Stock |
| `in` | Forecasted Receipts |
| `out` | Forecasted Deliveries |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Lot/Serial — `activity_state`

Entity `stock.lot`, table `stock_lot`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Stock Move — `state`

Entity `stock.move`, table `stock_move`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | New |
| `waiting` | Waiting Another Move |
| `confirmed` | Waiting |
| `partially_available` | Partially Available |
| `assigned` | Available |
| `done` | Done |
| `cancel` | Cancelled |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `activity_state`

Entity `stock.picking`, table `stock_picking`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `l10n_ro_edi_stock_state`

Entity `stock.picking`, table `stock_picking`. Label: eTransport Status; stored; read only.

| Value | Label |
|---|---|
| `stock_sent` | Sent |
| `stock_sending_failed` | Error |
| `stock_validated` | Validated |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `l10n_tr_nilvera_dispatch_state`

Entity `stock.picking`, table `stock_picking`. Label: e-Dispatch State; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `products_availability_state`

Entity `stock.picking`, table `stock_picking`. Label: Products Availability State; not stored; read only.

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Transfer — `state`

Entity `stock.picking`, table `stock_picking`. Label: Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `waiting` | Waiting Another Operation |
| `confirmed` | Waiting |
| `assigned` | Ready |
| `done` | Done |
| `cancel` | Cancelled |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Batch Transfer — `activity_state`

Entity `stock.picking.batch`, table `stock_picking_batch`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Batch Transfer — `l10n_ro_edi_stock_state`

Entity `stock.picking.batch`, table `stock_picking_batch`. Label: eTransport Status; stored; read only.

| Value | Label |
|---|---|
| `stock_sent` | Sent |
| `stock_sending_failed` | Error |
| `stock_validated` | Validated |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Batch Transfer — `state`

Entity `stock.picking.batch`, table `stock_picking_batch`. Label: State; stored; required; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_progress` | In progress |
| `done` | Done |
| `cancel` | Cancelled |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

### Scrap — `state`

Entity `stock.scrap`, table `stock_scrap`. Label: Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Done |

Transitions: [inventory-operations state machines](../domains/inventory-operations/state-machines.md).

## inventory-valuation-and-costing

### Stock Landed Cost — `activity_state`

Entity `stock.landed.cost`, table `stock_landed_cost`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [inventory-valuation-and-costing state machines](../domains/inventory-valuation-and-costing/state-machines.md).

### Stock Landed Cost — `state`

Entity `stock.landed.cost`, table `stock_landed_cost`. Label: State; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Posted |
| `cancel` | Cancelled |

Transitions: [inventory-valuation-and-costing state machines](../domains/inventory-valuation-and-costing/state-machines.md).

## learning-surveys-and-gamification

### Gamification Challenge — `state`

Entity `gamification.challenge`, table `gamification_challenge`. Label: State; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `inprogress` | In Progress |
| `done` | Done |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Gamification Goal — `state`

Entity `gamification.goal`, table `gamification_goal`. Label: State; stored; required.

| Value | Label |
|---|---|
| `draft` | Draft |
| `inprogress` | In progress |
| `reached` | Reached |
| `failed` | Failed |
| `canceled` | Cancelled |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Course — `activity_state`

Entity `slide.channel`, table `slide_channel`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Channel / Partners (Members) — `member_status`

Entity `slide.channel.partner`, table `slide_channel_partner`. Label: Attendee Status; stored; required; read only.

| Value | Label |
|---|---|
| `invited` | Invite Sent |
| `joined` | Joined |
| `ongoing` | Ongoing |
| `completed` | Finished |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey — `activity_state`

Entity `survey.survey`, table `survey_survey`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey — `session_state`

Entity `survey.survey`, table `survey_survey`. Label: Session State; stored.

| Value | Label |
|---|---|
| `ready` | Ready |
| `in_progress` | In Progress |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey User Input — `activity_state`

Entity `survey.user_input`, table `survey_user_input`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

### Survey User Input — `state`

Entity `survey.user_input`, table `survey_user_input`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `new` | New |
| `in_progress` | In Progress |
| `done` | Completed |

Transitions: [learning-surveys-and-gamification state machines](../domains/learning-surveys-and-gamification/state-machines.md).

## lunch-ordering

### Lunch Order — `state`

Entity `lunch.order`, table `lunch_order`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `new` | To Order |
| `ordered` | Ordered |
| `sent` | Sent |
| `confirmed` | Received |
| `cancelled` | Cancelled |

Transitions: [lunch-ordering state machines](../domains/lunch-ordering/state-machines.md).

### Lunch Supplier — `activity_state`

Entity `lunch.supplier`, table `lunch_supplier`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [lunch-ordering state machines](../domains/lunch-ordering/state-machines.md).

## manufacturing

### Manufacturing Order — `activity_state`

Entity `mrp.production`, table `mrp_production`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Manufacturing Order — `components_availability_state`

Entity `mrp.production`, table `mrp_production`. Label: Components Availability State; not stored; read only.

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |
| `unavailable` | Not Available |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Manufacturing Order — `reservation_state`

Entity `mrp.production`, table `mrp_production`. Label: MO Readiness; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `confirmed` | Waiting |
| `assigned` | Ready |
| `waiting` | Waiting Another Operation |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Manufacturing Order — `state`

Entity `mrp.production`, table `mrp_production`. Label: State; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `confirmed` | Confirmed |
| `progress` | In Progress |
| `to_close` | To Close |
| `done` | Done |
| `cancel` | Cancelled |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Work Center Usage — `activity_state`

Entity `mrp.routing.workcenter`, table `mrp_routing_workcenter`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Unbuild Order — `activity_state`

Entity `mrp.unbuild`, table `mrp_unbuild`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Unbuild Order — `state`

Entity `mrp.unbuild`, table `mrp_unbuild`. Label: Status; stored.

| Value | Label |
|---|---|
| `draft` | Draft |
| `done` | Done |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Work Center — `working_state`

Entity `mrp.workcenter`, table `mrp_workcenter`. Label: Workcenter Status; stored; read only.

| Value | Label |
|---|---|
| `normal` | Normal |
| `blocked` | Blocked |
| `done` | In Progress |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

### Work Order — `state`

Entity `mrp.workorder`, table `mrp_workorder`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `blocked` | Blocked |
| `ready` | To Do |
| `progress` | In Progress |
| `done` | Finished |
| `cancel` | Cancelled |

Transitions: [manufacturing state machines](../domains/manufacturing/state-machines.md).

## marketing-and-mass-mailing

### Marketing Card Campaign — `activity_state`

Entity `card.campaign`, table `card_campaign`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Marketing Card — `share_status`

Entity `card.card`, table `card_card`. Label: Share Status; stored.

| Value | Label |
|---|---|
| `shared` | Shared |
| `visited` | Visited |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mass Mailing — `activity_state`

Entity `mailing.mailing`, table `mailing_mailing`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mass Mailing — `state`

Entity `mailing.mailing`, table `mailing_mailing`. Label: Status; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_queue` | In Queue |
| `sending` | Sending |
| `done` | Sent |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mailing Statistics — `trace_status`

Entity `mailing.trace`, table `mailing_trace`. Label: Status; stored.

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

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

### Mass Mailing Statistics — `state`

Entity `mailing.trace.report`, table `mailing_trace_report`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | Draft |
| `test` | Tested |
| `done` | Sent |

Transitions: [marketing-and-mass-mailing state machines](../domains/marketing-and-mass-mailing/state-machines.md).

## messaging-and-activities

### Digest — `state`

Entity `digest.digest`, table `digest_digest`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `activated` | Activated |
| `deactivated` | Deactivated |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Discussion Channel — `livechat_status`

Entity `discuss.channel`, table `discuss_channel`. Label: Livechat Status; stored.

| Value | Label |
|---|---|
| `in_progress` | In progress |
| `waiting` | Waiting for customer |
| `need_help` | Looking for help |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Incoming Mail Server — `state`

Entity `fetchmail.server`, table `fetchmail_server`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | Not Confirmed |
| `done` | Confirmed |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Keep the channel member history — `help_status`

Entity `im_livechat.channel.member.history`, table `im_livechat_channel_member_history`. Label: Help Status; stored; read only.

| Value | Label |
|---|---|
| `requested` | Help Requested |
| `provided` | Help Provided |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Activity — `state`

Entity `mail.activity`, table `mail_activity`. Label: State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |
| `done` | Done |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Activity Mixin — `activity_state`

Entity `mail.activity.mixin`, table ``. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Email Aliases — `alias_status`

Entity `mail.alias`, table `mail_alias`. Label: Alias Status; stored; read only.

| Value | Label |
|---|---|
| `not_tested` | Not Tested |
| `valid` | Valid |
| `invalid` | Invalid |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Mailing List Message — `moderation_status`

Entity `mail.group.message`, table `mail_group_message`. Label: Status; stored; required.

| Value | Label |
|---|---|
| `pending_moderation` | Pending Moderation |
| `accepted` | Accepted |
| `rejected` | Rejected |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Mailing List black/white list — `status`

Entity `mail.group.moderation`, table `mail_group_moderation`. Label: Status; stored; required.

| Value | Label |
|---|---|
| `allow` | Always Allow |
| `ban` | Permanent Ban |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Outgoing Mails — `state`

Entity `mail.mail`, table `mail_mail`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `outgoing` | Outgoing |
| `sent` | Sent |
| `received` | Received |
| `exception` | Delivery Failed |
| `cancel` | Cancelled |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Message Notifications — `notification_status`

Entity `mail.notification`, table `mail_notification`. Label: Status; stored.

| Value | Label |
|---|---|
| `ready` | Ready to Send |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `bounce` | Bounced |
| `exception` | Exception |
| `canceled` | Cancelled |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### User/Guest Presence — `status`

Entity `mail.presence`, table `mail_presence`. Label: IM Status; stored.

| Value | Label |
|---|---|
| `online` | Online |
| `away` | Away |
| `offline` | Offline |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Outgoing text message — `state`

Entity `sms.sms`, table `sms_sms`. Label: SMS Status; stored; required; read only.

| Value | Label |
|---|---|
| `outgoing` | In Queue |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `error` | Error |
| `canceled` | Cancelled |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

### Snailmail Letter — `state`

Entity `snailmail.letter`, table `snailmail_letter`. Label: Status; stored; required; read only.

| Value | Label |
|---|---|
| `pending` | In Queue |
| `sent` | Sent |
| `error` | Error |
| `canceled` | Cancelled |

Transitions: [messaging-and-activities state machines](../domains/messaging-and-activities/state-machines.md).

## multi-currency

### Language Export — `state`

Entity `base.language.export`, table `base_language_export`. Label: State; stored.

| Value | Label |
|---|---|
| `choose` | choose |
| `get` | get |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Update Module — `state`

Entity `base.module.update`, table `base_module_update`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `init` | init |
| `done` | done |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Merge Partner Wizard — `state`

Entity `base.partner.merge.automatic.wizard`, table `base_partner_merge_automatic_wizard`. Label: State; stored; required; read only.

| Value | Label |
|---|---|
| `option` | Option |
| `selection` | Selection |
| `finished` | Finished |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Server Actions — `activity_state`

Entity `ir.actions.server`, table ``. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Server Actions — `state`

Entity `ir.actions.server`, table ``. Label: Type; stored; required; changes are tracked in the message thread.

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

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Configuration Wizards — `state`

Entity `ir.actions.todo`, table `ir_actions_todo`. Label: Status; stored; required.

| Value | Label |
|---|---|
| `open` | To Do |
| `done` | Done |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Scheduled Actions — `activity_state`

Entity `ir.cron`, table `ir_cron`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Models — `state`

Entity `ir.model`, table `ir_model`. Label: Type; stored; read only.

| Value | Label |
|---|---|
| `manual` | Custom Object |
| `base` | Base Object |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Fields — `state`

Entity `ir.model.fields`, table `ir_model_fields`. Label: Type; stored; required; read only.

| Value | Label |
|---|---|
| `manual` | Custom Field |
| `base` | Base Field |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Module — `state`

Entity `ir.module.module`, table `ir_module_module`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `uninstallable` | Uninstallable |
| `uninstalled` | Not Installed |
| `installed` | Installed |
| `to upgrade` | To be upgraded |
| `to remove` | To be removed |
| `to install` | To be installed |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Module dependency — `state`

Entity `ir.module.module.dependency`, table `ir_module_module_dependency`. Label: Status; not stored; read only.

| Value | Label |
|---|---|
| `uninstallable` | Uninstallable |
| `uninstalled` | Not Installed |
| `installed` | Installed |
| `to upgrade` | To be upgraded |
| `to remove` | To be removed |
| `to install` | To be installed |
| `unknown` | Unknown |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Module exclusion — `state`

Entity `ir.module.module.exclusion`, table `ir_module_module_exclusion`. Label: Status; not stored; read only.

| Value | Label |
|---|---|
| `uninstallable` | Uninstallable |
| `uninstalled` | Not Installed |
| `installed` | Installed |
| `to upgrade` | To be upgraded |
| `to remove` | To be removed |
| `to install` | To be installed |
| `unknown` | Unknown |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `account_peppol_proxy_state`

Entity `res.company`, table `res_company`. Label: PEPPOL status; stored; required.

| Value | Label |
|---|---|
| `not_registered` | Not registered |
| `sender` | Can send but not receive |
| `smp_registration` | Can send, pending registration to receive |
| `receiver` | Can send and receive |
| `rejected` | Rejected |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `l10n_dk_nemhandel_proxy_state`

Entity `res.company`, table `res_company`. Label: Nemhandel status; stored; required.

| Value | Label |
|---|---|
| `not_registered` | Not registered |
| `in_verification` | In verification |
| `receiver` | Can send and receive |
| `rejected` | Rejected |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `l10n_hr_mer_connection_state`

Entity `res.company`, table `res_company`. Label: MojEracun connection status; stored; required; read only.

| Value | Label |
|---|---|
| `inactive` | Inactive |
| `active` | Active |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `l10n_it_eco_index_liquidation_state`

Entity `res.company`, table `res_company`. Label: Liquidation state; stored.

| Value | Label |
|---|---|
| `LS` | The company is in a state of liquidation |
| `LN` | The company is not in a state of liquidation |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Companies — `pdp_kyc_status`

Entity `res.company`, table `res_company`. Label: Pdp Kyc Status; stored.

| Value | Label |
|---|---|
| `processing` | Processing |
| `success` | Success |
| `fail` | Fail |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `activity_state`

Entity `res.partner`, table `res_partner`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `l10n_my_tin_validation_state`

Entity `res.partner`, table `res_partner`. Label: Tin Validation State; stored.

| Value | Label |
|---|---|
| `valid` | Valid |
| `invalid` | Invalid |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `l10n_tr_nilvera_customer_status`

Entity `res.partner`, table `res_partner`. Label: Nilvera Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `not_checked` | Not Verified |
| `earchive` | E-Archive |
| `einvoice` | E-Invoice |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `nemhandel_verification_state`

Entity `res.partner`, table `res_partner`. Label: Nemhandel endpoint verification; stored.

| Value | Label |
|---|---|
| `not_verified` | Not verified yet |
| `not_valid` | Not on Nemhandel |
| `valid` | Valid |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `pdp_verification_display_state`

Entity `res.partner`, table `res_partner`. Label: E-Invoicing State; not stored; read only.

| Value | Label |
|---|---|
| `not_verified` | Not verified yet |
| `pdp_not_valid` | Partner is not in the annuaire |
| `pdp_not_valid_format` | Partner cannot receive format |
| `pdp_valid` | Partner is in the annuaire |
| `peppol_not_valid` | Partner is not on Peppol |
| `peppol_not_valid_format` | Partner cannot receive format |
| `peppol_valid` | Partner is on Peppol |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Contact — `peppol_verification_state`

Entity `res.partner`, table `res_partner`. Label: Peppol status; stored.

| Value | Label |
|---|---|
| `not_verified` | Unchecked |
| `not_valid` | Partner is not on Peppol |
| `not_valid_format` | Partner cannot receive format |
| `valid` | Partner is on Peppol |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Bank Accounts — `activity_state`

Entity `res.partner.bank`, table `res_partner_bank`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### User — `manual_im_status`

Entity `res.users`, table `res_users`. Label: IM status manually set by the user; stored.

| Value | Label |
|---|---|
| `away` | Away |
| `busy` | Do Not Disturb |
| `offline` | Offline |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### User — `system_robot_state`

Entity `res.users`, table `res_users`. Label: System Robot Status; stored; read only.

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

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### User — `state`

Entity `res.users`, table `res_users`. Label: Status; not stored; read only.

| Value | Label |
|---|---|
| `new` | Invited |
| `active` | Confirmed |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

### Users Deletion Request — `state`

Entity `res.users.deletion`, table `res_users_deletion`. Label: State; stored; required.

| Value | Label |
|---|---|
| `todo` | To Do |
| `done` | Done |
| `fail` | Failed |

Transitions: [multi-currency state machines](../domains/multi-currency/state-machines.md).

## payment-providers

### Payment Provider — `state`

Entity `payment.provider`, table `payment_provider`. Label: State; stored; required.

| Value | Label |
|---|---|
| `disabled` | Disabled |
| `enabled` | Enabled |
| `test` | Test Mode |

Transitions: [payment-providers state machines](../domains/payment-providers/state-machines.md).

### Payment Token — `demo_simulated_state`

Entity `payment.token`, table `payment_token`. Label: Simulated State; stored.

| Value | Label |
|---|---|
| `pending` | Pending |
| `done` | Confirmed |
| `cancel` | Canceled |
| `error` | Error |

Transitions: [payment-providers state machines](../domains/payment-providers/state-machines.md).

### Payment Transaction — `state`

Entity `payment.transaction`, table `payment_transaction`. Label: Status; stored; required; read only.

| Value | Label |
|---|---|
| `draft` | Draft |
| `pending` | Pending |
| `authorized` | Authorized |
| `done` | Confirmed |
| `cancel` | Canceled |
| `error` | Error |

Transitions: [payment-providers state machines](../domains/payment-providers/state-machines.md).

## payments-and-bank-reconciliation

### Account payment check — `activity_state`

Entity `l10n_latam.check`, table `l10n_latam_check`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [payments-and-bank-reconciliation state machines](../domains/payments-and-bank-reconciliation/state-machines.md).

### Account payment check — `issue_state`

Entity `l10n_latam.check`, table `l10n_latam_check`. Label: Issue State; stored; read only.

| Value | Label |
|---|---|
| `handed` | Handed |
| `debited` | Debited |
| `voided` | Voided |

Transitions: [payments-and-bank-reconciliation state machines](../domains/payments-and-bank-reconciliation/state-machines.md).

## point-of-sale

### Point of Sale Configuration — `status`

Entity `pos.config`, table `pos_config`. Label: Status; not stored; read only.

| Value | Label |
|---|---|
| `inactive` | Inactive |
| `active` | Active |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `invoice_status`

Entity `pos.order`, table `pos_order`. Label: Invoice Status; not stored; read only.

| Value | Label |
|---|---|
| `invoiced` | Fully Invoiced |
| `to_invoice` | To Invoice |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `l10n_es_edi_verifactu_state`

Entity `pos.order`, table `pos_order`. Label: Veri*Factu Status; stored; read only.

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |
| `cancelled` | Cancelled |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `l10n_es_tbai_state`

Entity `pos.order`, table `pos_order`. Label: TicketBAI status; not stored; read only.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `l10n_jo_edi_pos_state`

Entity `pos.order`, table `pos_order`. Label: JoFotara State; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `demo` | Sent (Demo) |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders — `state`

Entity `pos.order`, table `pos_order`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | New |
| `cancel` | Cancelled |
| `paid` | Paid |
| `done` | Posted |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Session — `activity_state`

Entity `pos.session`, table `pos_session`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Session — `state`

Entity `pos.session`, table `pos_session`. Label: Status; stored; required; read only.

| Value | Label |
|---|---|
| `opening_control` | Opening Control |
| `opened` | In Progress |
| `closing_control` | Closing Control |
| `closed` | Closed & Posted |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

### Point of Sale Orders Report — `state`

Entity `report.pos.order`, table `report_pos_order`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | New |
| `paid` | Paid |
| `done` | Posted |
| `cancel` | Cancelled |

Transitions: [point-of-sale state machines](../domains/point-of-sale/state-machines.md).

## pricing-and-pricelists

### Product Margin — `invoice_state`

Entity `product.margin`, table `product_margin`. Label: Invoice State; stored; required.

| Value | Label |
|---|---|
| `paid` | Paid |
| `open_paid` | Open and Paid |
| `draft_open_paid` | Draft, Open and Paid |

Transitions: [pricing-and-pricelists state machines](../domains/pricing-and-pricelists/state-machines.md).

## products-and-catalog

### Pricelist — `activity_state`

Entity `product.pricelist`, table `product_pricelist`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

### Product Variant — `activity_state`

Entity `product.product`, table `product_product`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

### Product Variant — `invoice_state`

Entity `product.product`, table `product_product`. Label: Invoice State; not stored; read only.

| Value | Label |
|---|---|
| `paid` | Paid |
| `open_paid` | Open and Paid |
| `draft_open_paid` | Draft, Open and Paid |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

### Product — `activity_state`

Entity `product.template`, table `product_template`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [products-and-catalog state machines](../domains/products-and-catalog/state-machines.md).

## projects-and-tasks

### Project — `activity_state`

Entity `project.project`, table `project_project`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Project — `last_update_status`

Entity `project.project`, table `project_project`. Label: Last Update Status; stored; required.

| Value | Label |
|---|---|
| `on_track` | On Track |
| `at_risk` | At Risk |
| `off_track` | Off Track |
| `on_hold` | On Hold |
| `to_define` | Set Status |
| `done` | Complete |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Task — `activity_state`

Entity `project.task`, table `project_task`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Task — `state`

Entity `project.task`, table `project_task`. Label: State; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `02_changes_requested` | Changes Requested |
| `03_approved` | Approved |
| `1_done` | Done |
| `1_canceled` | Cancelled |
| `04_waiting_normal` | Waiting |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Burndown Chart — `state`

Entity `project.task.burndown.chart.report`, table ``. Label: State; stored; read only.

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `1_done` | Done |
| `04_waiting_normal` | Waiting |
| `03_approved` | Approved |
| `1_canceled` | Cancelled |
| `02_changes_requested` | Changes Requested |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Task Stage — `rating_status`

Entity `project.task.type`, table `project_task_type`. Label: Customer Ratings Status; stored; required.

| Value | Label |
|---|---|
| `stage` | when reaching this stage |
| `periodic` | on a periodic basis |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Project Update — `activity_state`

Entity `project.update`, table `project_update`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Project Update — `status`

Entity `project.update`, table `project_update`. Label: Status; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `on_track` | On Track |
| `at_risk` | At Risk |
| `off_track` | Off Track |
| `on_hold` | On Hold |
| `done` | Complete |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

### Tasks Analysis — `state`

Entity `report.project.task.user`, table `report_project_task_user`. Label: State; stored; read only.

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `1_done` | Done |
| `04_waiting_normal` | Waiting |
| `03_approved` | Approved |
| `1_canceled` | Cancelled |
| `02_changes_requested` | Changes Requested |

Transitions: [projects-and-tasks state machines](../domains/projects-and-tasks/state-machines.md).

## purchasing

### Purchase Order — `activity_state`

Entity `purchase.order`, table `purchase_order`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Order — `invoice_status`

Entity `purchase.order`, table `purchase_order`. Label: Billing Status; stored; read only.

| Value | Label |
|---|---|
| `no` | Nothing to Bill |
| `to invoice` | Waiting Bills |
| `invoiced` | Fully Billed |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Order — `receipt_status`

Entity `purchase.order`, table `purchase_order`. Label: Receipt Status; stored; read only.

| Value | Label |
|---|---|
| `pending` | Not Received |
| `partial` | Partially Received |
| `full` | Fully Received |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Order — `state`

Entity `purchase.order`, table `purchase_order`. Label: Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | RFQ |
| `sent` | RFQ Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Report — `state`

Entity `purchase.report`, table ``. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | Draft RFQ |
| `sent` | RFQ Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Requisition — `activity_state`

Entity `purchase.requisition`, table `purchase_requisition`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

### Purchase Requisition — `state`

Entity `purchase.requisition`, table `purchase_requisition`. Label: Status; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Draft |
| `confirmed` | Confirmed |
| `done` | Closed |
| `cancel` | Cancelled |

Transitions: [purchasing state machines](../domains/purchasing/state-machines.md).

## recruitment

### Applicant — `activity_state`

Entity `hr.applicant`, table `hr_applicant`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [recruitment state machines](../domains/recruitment/state-machines.md).

### Applicant — `application_status`

Entity `hr.applicant`, table `hr_applicant`. Label: Application Status; not stored; read only.

| Value | Label |
|---|---|
| `ongoing` | Ongoing |
| `hired` | Hired |
| `refused` | Refused |
| `archived` | Archived |

Transitions: [recruitment state machines](../domains/recruitment/state-machines.md).

### Applicant — `kanban_state`

Entity `hr.applicant`, table `hr_applicant`. Label: Kanban State; stored; required.

| Value | Label |
|---|---|
| `normal` | In Progress |
| `done` | Ready for Next Stage |
| `waiting` | Waiting |
| `blocked` | Blocked |

Transitions: [recruitment state machines](../domains/recruitment/state-machines.md).

## repair-and-maintenance

### Maintenance Equipment — `activity_state`

Entity `maintenance.equipment`, table `maintenance_equipment`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Maintenance Request — `activity_state`

Entity `maintenance.request`, table `maintenance_request`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Maintenance Request — `kanban_state`

Entity `maintenance.request`, table `maintenance_request`. Label: Kanban State; stored; required; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `normal` | In Progress |
| `blocked` | Blocked |
| `done` | Ready for next stage |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Repair Order — `activity_state`

Entity `repair.order`, table `repair_order`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Repair Order — `parts_availability_state`

Entity `repair.order`, table `repair_order`. Label: Parts Availability State; not stored; read only.

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

### Repair Order — `state`

Entity `repair.order`, table `repair_order`. Label: Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | New |
| `confirmed` | Confirmed |
| `under_repair` | Under Repair |
| `done` | Repaired |
| `cancel` | Cancelled |

Transitions: [repair-and-maintenance state machines](../domains/repair-and-maintenance/state-machines.md).

## sales

### Sales Order — `activity_state`

Entity `sale.order`, table `sale_order`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order — `delivery_status`

Entity `sale.order`, table `sale_order`. Label: Delivery Status; stored; read only.

| Value | Label |
|---|---|
| `pending` | Not Delivered |
| `started` | Started |
| `partial` | Partially Delivered |
| `full` | Fully Delivered |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order — `invoice_status`

Entity `sale.order`, table `sale_order`. Label: Invoice Status; stored; read only.

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order — `state`

Entity `sale.order`, table `sale_order`. Label: Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `draft` | Quotation |
| `sent` | Quotation Sent |
| `sale` | Sales Order |
| `cancel` | Cancelled |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Order Line — `invoice_status`

Entity `sale.order.line`, table `sale_order_line`. Label: Invoice Status; stored; read only.

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Analysis Report — `invoice_status`

Entity `sale.report`, table ``. Label: Order Invoice Status; stored; read only.

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Analysis Report — `line_invoice_status`

Entity `sale.report`, table ``. Label: Invoice Status; stored; read only.

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

Transitions: [sales state machines](../domains/sales/state-machines.md).

### Sales Analysis Report — `state`

Entity `sale.report`, table ``. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `draft` | Quotation |
| `sent` | Quotation Sent |
| `sale` | Sales Order |
| `cancel` | Cancelled |
| `paid` | Paid |
| `invoiced` | Invoiced |
| `done` | Posted |

Transitions: [sales state machines](../domains/sales/state-machines.md).

## time-off

### Time Off — `activity_state`

Entity `hr.leave`, table `hr_leave`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off — `state`

Entity `hr.leave`, table `hr_leave`. Label: Status; stored; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |
| `cancel` | Cancelled |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Allocation — `activity_state`

Entity `hr.leave.allocation`, table `hr_leave_allocation`. Label: Activity State; not stored; read only.

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Allocation — `state`

Entity `hr.leave.allocation`, table `hr_leave_allocation`. Label: Status; stored; read only; changes are tracked in the message thread.

| Value | Label |
|---|---|
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Summary / Report — `holiday_status`

Entity `hr.leave.employee.type.report`, table `hr_leave_employee_type_report`. Label: Holiday Status; stored.

| Value | Label |
|---|---|
| `taken` | Taken |
| `left` | Left |
| `planned` | Planned |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Summary / Report — `state`

Entity `hr.leave.employee.type.report`, table `hr_leave_employee_type_report`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Summary / Report — `state`

Entity `hr.leave.report`, table `hr_leave_report`. Label: Status; stored; read only.

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

### Time Off Calendar — `state`

Entity `hr.leave.report.calendar`, table `hr_leave_report_calendar`. Label: State; stored; read only.

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

Transitions: [time-off state machines](../domains/time-off/state-machines.md).

## website-and-storefront

### Forum Post — `state`

Entity `forum.post`, table `forum_post`. Label: Status; stored.

| Value | Label |
|---|---|
| `active` | Active |
| `pending` | Waiting Validation |
| `close` | Closed |
| `offensive` | Offensive |
| `flagged` | Flagged |

Transitions: [website-and-storefront state machines](../domains/website-and-storefront/state-machines.md).

### Portal User Config — `email_state`

Entity `portal.wizard.user`, table `portal_wizard_user`. Label: Status; not stored; read only.

| Value | Label |
|---|---|
| `ok` | Valid |
| `ko` | Invalid |
| `exist` | Already Registered |

Transitions: [website-and-storefront state machines](../domains/website-and-storefront/state-machines.md).

## work-entries

### human resources Work Entry — `state`

Entity `hr.work.entry`, table `hr_work_entry`. Label: State; stored.

| Value | Label |
|---|---|
| `draft` | New |
| `conflict` | In Conflict |
| `validated` | In Payslip |
| `cancelled` | Cancelled |

Transitions: [work-entries state machines](../domains/work-entries/state-machines.md).

