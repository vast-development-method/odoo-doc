# Accounting effects of Calendar and Scheduling

## 1. The domain posts nothing to the ledger

Calendar and Scheduling creates **no journal entry and no journal item**, in any circumstance, on any
of the records it owns.

This is not an omission. The reasoning is worth stating, because a rebuild that adds a ledger effect
here would be wrong.

1. **Nothing the domain owns carries a monetary amount.** Not one field of Calendar Event, Calendar
   Attendee Information, Event Recurrence Rule, Event Alarm, Event Meeting Type, Calendar Filters, the
   two panels or the four synchronisation entities holds a value, a currency, a tax, an account or an
   analytic distribution. The complete field tables are in [entities.md](entities.md); the exhaustive
   list of what the domain computes is in [calculations.md](calculations.md), and every quantity in it
   is an instant, a duration, a count or an interval.
2. **A meeting is a plan, not a transaction.** It records an intention to spend time. Nothing is
   consumed, delivered, owed or received when a meeting is created, moved, answered or deleted.
3. **The domain has no company field.** A ledger effect needs a company, a journal and a currency; a
   Calendar Event has none of the three. The only company-aware computation in the domain is the
   working-hours intersection, and it reads working schedules, not accounts.
4. **The domain's only outbound artefacts are messages.** Invitations, rescheduling notices,
   reminders, cancellations and text messages cost nothing that the platform records. Where a text
   message does cost money, the cost is recorded by the messaging domain against its own credit
   mechanism, not here; see [../messaging-and-activities/](../messaging-and-activities/).

---

## 2. Ledger effects this domain triggers indirectly

A meeting is frequently the visible half of something that does have a financial consequence
elsewhere. In every case below the consequence belongs to the other domain, and this domain
contributes only the time record.

| Trigger in this domain | Consequence elsewhere | Where it is specified |
|---|---|---|
| A meeting is created against an opportunity, through the opportunity reference on Calendar Event | None directly. The meeting advances the sales conversation; the ledger effect arrives only if the opportunity becomes a quotation and then an order. | [../customer-relationship-management/](../customer-relationship-management/), then [../sales/](../sales/) |
| A meeting is created against an applicant, through the applicant reference on Calendar Event | None directly. The interview may lead to a hire, whose payroll consequences are recorded elsewhere. | [../recruitment/](../recruitment/), then [../human-resources-core/](../human-resources-core/) |
| An absence is approved and materialises as a meeting in the attendees' calendars | The absence itself may generate work entries and payroll consequences. The meeting is a display artefact. | [../time-off/](../time-off/), then [../work-entries/](../work-entries/) |
| A meeting is used as the anchor for a timesheet entry by a person recording their time against it | The timesheet entry may be billable and may reach a customer invoice. The meeting itself is never billed. | [../timesheets/](../timesheets/) |
| A text-message reminder is sent | The message consumes the platform's messaging credit. That consumption is accounted for by the messaging domain, not here. | [../messaging-and-activities/](../messaging-and-activities/) |
| An external calendar service is called | Nothing. The two services are used under the platform operator's own application registration and no charge is recorded by the platform. | [external-calendar-synchronisation.md](external-calendar-synchronisation.md) |

---

## 3. What a rebuild must therefore reproduce

1. No journal, no account, no tax and no analytic distribution appears anywhere in this domain's data
   model or its user interface.
2. No operation of this domain opens, posts, reverses or reconciles anything.
3. Removing every accounting capability package from the platform leaves this domain fully
   functional. Conversely, installing every accounting package changes nothing about it.
4. The cross-domain links in chapter 2 are one-directional: the other domain reads the meeting, the
   meeting never reads a ledger.

For the platform-wide picture of which domain posts what, see
[../../overview/architecture.md](../../overview/architecture.md) and
[../general-ledger/](../general-ledger/).
