# Calculations of the Events domain

Every formula and algorithm of the domain: its inputs, its output, its precision, the exact order of operations, the tie-breaking rules and at least one worked example with real numbers. Algorithms are written as numbered steps. Notation: `now` is the current moment in coordinated universal time; `tz(x)` is the value `x` read in the display time zone of the event; `⌊x⌋` is the whole part of `x`.

## Contents

1. [Seat counters](#1-seat-counters)
2. [Seat availability of a slot and ticket combination](#2-seat-availability-of-a-slot-and-ticket-combination)
3. [Registration open and sold out](#3-registration-open-and-sold-out)
4. [Ticket sales window and ticket availability](#4-ticket-sales-window-and-ticket-availability)
5. [Communication schedule dates](#5-communication-schedule-dates)
6. [Communication batch progress and completion](#6-communication-batch-progress-and-completion)
7. [Relative date phrase](#7-relative-date-phrase)
8. [Ticket and booth prices with taxes and discounts](#8-ticket-and-booth-prices-with-taxes-and-discounts)
9. [Per-order ticket limits](#9-per-order-ticket-limits)
10. [Sponsor opening hours](#10-sponsor-opening-hours)
11. [Talk times, the action button window and the agenda grid](#11-talk-times-the-action-button-window-and-the-agenda-grid)
12. [Participation detection](#12-participation-detection)
13. [Quiz scoring and the leaderboard](#13-quiz-scoring-and-the-leaderboard)
14. [Revenue figures and currency conversion](#14-revenue-figures-and-currency-conversion)
15. [Barcode generation and the ticket access hash](#15-barcode-generation-and-the-ticket-access-hash)
16. [Slot datetimes from date and hours](#16-slot-datetimes-from-date-and-hours)
17. [Talk suggestion ranking](#17-talk-suggestion-ranking)
18. [Display names carrying availability](#18-display-names-carrying-availability)
19. [Measures of the revenue analysis](#19-measures-of-the-revenue-analysis)

---

## 1. Seat counters

**Inputs:** the active registrations of the scope, their `state`, the seat maximum of the scope, and, for an event, the multi-slot flag and the slot count.
**Outputs:** four whole numbers, never stored, recomputed on demand.
**Precision:** whole seats.

Three scopes count seats with the same shape: the Event, the Event Slot and the Event Ticket. In every scope, only registrations that are **active** and whose state is `open` or `done` are counted.

```formula
seats_reserved = count(registrations WHERE active AND state = "open")
seats_used     = count(registrations WHERE active AND state = "done")
seats_taken    = seats_reserved + seats_used
```

The remaining seats depend on the scope:

```formula
Event:
  effective_maximum = seats_max × event_slot_count   if is_multi_slots
                    = seats_max                      otherwise
  seats_available   = effective_maximum − seats_taken    if effective_maximum > 0
                    = 0                                  otherwise

Event Slot:
  seats_available = event.seats_max − seats_taken    if event.seats_max > 0
                  = 0                                    otherwise

Event Ticket:
  seats_available = seats_max − seats_taken          if seats_max > 0
                  = 0                                    otherwise
```

A zero maximum therefore produces a zero "available" figure that must be read as "no limit", never as "sold out"; the sold-out decision is taken by the rules of sections 3 and 4, which test the limit flag first.

**Order of operations.** The three counts are read in one grouped query over the registrations of the scope, restricted to the two counted states and to active records; scopes with no counted registration keep zeros; then the available figure is derived. The counters are deliberately not stored, which means they are always exact and never need repair.

**Worked example 1 (the canonical figures).** An event caps 100 seats, has no slot and no ticket. It has 60 registrations in `open`, 10 in `done` and 5 in `cancel`.

```formula
seats_reserved  = 60
seats_used      = 10
seats_taken     = 60 + 10 = 70
effective_max   = 100
seats_available = 100 − 70 = 30
```

The 5 cancelled registrations are invisible to every counter. Archiving one of the 60 open registrations gives `seats_reserved = 59`, `seats_taken = 69`, `seats_available = 31`.

**Worked example 2 (multi-slot).** The same event is switched to multi-slot with 3 slots and keeps `seats_max = 100`, which now means 100 seats **per slot**. The 70 taken seats are spread as 40 on slot A, 20 on slot B and 10 on slot C.

```formula
Event:  effective_max = 100 × 3 = 300, seats_taken = 70, seats_available = 300 − 70 = 230
Slot A: seats_taken = 40, seats_available = 100 − 40 = 60
Slot B: seats_taken = 20, seats_available = 100 − 20 = 80
Slot C: seats_taken = 10, seats_available = 100 − 10 = 90
```

**Worked example 3 (ticket counters are independent of the event cap).** The same event offers ticket "Standard" capped at 50 and ticket "Premium" capped at 20, and keeps its own cap of 100. Of the 70 taken seats, 55 are Standard and 15 are Premium.

```formula
Standard: seats_taken = 55, seats_available = 50 − 55 = −5
Premium:  seats_taken = 15, seats_available = 20 − 15 = 5
Event:    seats_available = 100 − 70 = 30
```

A negative ticket figure is possible after the cap has been lowered below the seats already sold; the availability rule of section 2 then reports zero room for that ticket, while the event still has 30 free seats for the other ticket.

---

## 2. Seat availability of a slot and ticket combination

**Inputs:** an event, a list of `(slot, ticket)` pairs where each element may be empty.
**Output:** one availability per pair, in the same order; the special value "no limit" is a possible result.

Algorithm, for one pair:

1. Set `available = no limit`.
2. When the event limits seats **and** its maximum is non-zero:
   - if a slot is given, `available = slot.seats_available`;
   - otherwise `available = event.seats_available`.
3. When `available` is not already `0`, a ticket is given, and that ticket has a non-zero maximum:
   - if a slot is given, `ticket_available = ticket.seats_max − (number of active registrations of that slot and that ticket in state open or done)`;
   - otherwise `ticket_available = ticket.seats_available`;
   - `available = ticket_available` when `available` was still "no limit", otherwise `available = min(available, ticket_available)`.
4. Return `available`.

When at least one pair carries a slot, the per-slot-and-ticket registration counts are read once, in a single grouped query over the registrations of the event that carry a slot and are active and in state `open` or `done`.

**Verification.** The verification used by `EV-RULE-030` takes triples `(slot, ticket, count)`, computes the availabilities for the pairs, and fails a triple when `available` is not "no limit" and `available < count`. The shortfall reported is `count − available`.

**Worked example.** An event caps 100 seats, uses 3 slots and offers ticket "Standard" capped at 50 per slot. Slot A already holds 96 registrations, of which 48 are Standard.

| Pair | Event or slot part | Ticket part | Result |
|---|---|---|---|
| (slot A, Standard) | `100 − 96 = 4` | `50 − 48 = 2` | `min(4, 2) = 2` |
| (slot A, no ticket) | `4` | not applicable | `4` |
| (no slot, Standard) | `event.seats_available` | `ticket.seats_available` | the smaller of the two |
| (slot B with no registration, Standard) | `100` | `50 − 0 = 50` | `50` |

A request for 3 seats of Standard on slot A fails with *"Standard - <slot A display name>: missing 1 seat(s)"*.

---

## 3. Registration open and sold out

**Inputs:** the kanban state, the dates, the display time zone, the seat configuration, the ticket list and, for a multi-slot event, the slot list.
**Outputs:** two booleans, recomputed on demand with elevated rights.

### Registrations started

```formula
event_registrations_started =
    true                                            if start_sale_datetime is empty
    tz(now) >= tz(start_sale_datetime)              otherwise
```

where

```formula
start_sale_datetime = min( ticket.start_sale_datetime for every non-expired ticket )
                      but only when every one of those tickets has a start date;
                      empty otherwise (and empty when there is no ticket at all)
```

A single non-expired ticket without a start date therefore makes the whole event "already started selling".

### Registrations open

```formula
event_registrations_open =
      kanban_state <> "cancel"
  AND event_registrations_started
  AND (date_end is empty OR tz(date_end) >= tz(now))
  AND (NOT seats_limited OR seats_max = 0 OR seats_available > 0)
  AND ticket_condition
```

with

```formula
ticket_condition (single-slot event) =
      there is no ticket
   OR at least one ticket has sale_available = true

ticket_condition (multi-slot event) =
      event_slot_count > 0
  AND (   there is no ticket
       OR at least one ticket t satisfies:
              t.is_launched AND NOT t.is_expired
          AND at least one slot s gives availability(s, t) = no limit or > 0 )
```

### Sold out

```formula
event_registrations_sold_out =
      (seats_limited AND seats_max > 0 AND NOT seats_available > 0)
   OR (there is at least one ticket AND
         multi-slot : no pair (slot, ticket) over all slots and all tickets has
                      availability = no limit or > 0
         single-slot: every ticket has is_sold_out = true )
```

**Worked example.** Event with 100 seats, 70 taken, one ticket "Standard" capped at 50 with 55 taken and a sales window that opened yesterday and closes tomorrow, single slot.

```formula
start_sale_datetime  = yesterday → event_registrations_started = true
date_end in the future → true
seats_limited and seats_max = 100 and seats_available = 30 > 0 → true
Standard: is_launched = true, is_expired = false,
          is_sold_out = (seats_limited AND seats_available = 0) = (true AND (50 − 55) ≤ 0) = true
ticket_condition = "at least one ticket has sale_available" = false, because
          sale_available = is_launched AND NOT is_expired AND NOT is_sold_out = false
⇒ event_registrations_open = false
⇒ event_registrations_sold_out = (100-seat test false) OR (every ticket sold out = true) = true
```

The event still has 30 free seats but cannot sell them, because the only ticket is exhausted. Adding a second uncapped ticket immediately makes `ticket_condition` true and re-opens the event.

---

## 4. Ticket sales window and ticket availability

**Inputs:** the two sales datetimes of the ticket, the display time zone of its event, the ticket seat figures, the sold-out flag of the event and, with the product bridge, the archived state of the product.

```formula
is_launched = true                             if start_sale_datetime is empty
            = tz(start_sale_datetime) <= tz(now)   otherwise

is_expired  = false                            if end_sale_datetime is empty
            = tz(end_sale_datetime) < tz(now)      otherwise

is_sold_out = (seats_limited AND seats_available = 0)
              OR event.event_registrations_sold_out

sale_available = is_launched AND NOT is_expired AND NOT is_sold_out
                 AND (product is not archived, when the product bridge is installed)
```

`seats_limited` on a ticket is itself derived: it is true exactly when `seats_max` is non-zero.

**Boundary behaviour.** The start boundary is inclusive (`<=`), therefore a ticket becomes sellable at the exact second of its start. The end boundary is exclusive on the expiry side (`<`), therefore a ticket is still sellable at the exact second of its end and expires immediately after.

**Worked example.** Display time zone `Europe/Brussels`, sales window from `2026-03-01 09:00` to `2026-03-31 18:00` written in that zone (stored as `08:00` and `16:00` in coordinated universal time in winter, `07:00` and `16:00` in summer).

| Current moment (Brussels) | `is_launched` | `is_expired` | `sale_available` when seats remain |
|---|---|---|---|
| 28 February, 23:59 | false | false | false |
| 1 March, 09:00:00 | true | false | true |
| 31 March, 18:00:00 | true | false | true |
| 31 March, 18:00:01 | true | true | false |

---

## 5. Communication schedule dates

**Inputs:** the interval number, the interval unit, the interval type and the anchor date.
**Output:** one datetime in coordinated universal time, stored.

### Interval arithmetic

| Unit | Meaning of `n` units |
|---|---|
| `now` | exactly zero hours, whatever the interval number |
| `hours` | `n` hours |
| `days` | `n` days |
| `weeks` | `7 × n` days |
| `months` | `n` calendar months, keeping the day of the month and clamping to the last day when the target month is shorter |

### Anchor and sign

| Interval type | Anchor | Sign |
|---|---|---|
| `after_sub` | on the schedule: the creation moment of the **event**; on a per-attendee trace: the creation moment of the **registration** | `+` |
| `before_event` | `event.date_begin` (slot trace: `slot.start_datetime`) | `−` |
| `after_event_start` | `event.date_begin` (slot trace: `slot.start_datetime`) | `+` |
| `after_event` | `event.date_end` (slot trace: `slot.end_datetime`) | `+` |
| `before_event_end` | `event.date_end` (slot trace: `slot.end_datetime`) | `−` |

```formula
scheduled_date = anchor with sub-seconds cleared  +  sign × interval(interval_nbr, interval_unit)
```

An empty anchor gives an empty scheduled date. Recomputing any scheduled date wakes the communication job for the earliest of the new dates.

**Worked example 1 (two days before the event).** An event starts on `2026-06-15 09:00:00` in coordinated universal time. A schedule has interval number 2, unit `days`, type `before_event`.

```formula
scheduled_date = 2026-06-15 09:00:00 − 2 days = 2026-06-13 09:00:00
```

The job that runs at `2026-06-13 09:14:00` finds `scheduled_date <= now`, checks that the event has not ended, and sends the message to every attendee of the event whose state is neither `draft` nor `cancel`. A job that runs only on `2026-06-16 03:00:00`, after the event ended on `2026-06-15 18:00:00`, finds the same due date in the past but refuses to send, because the trigger is "before the event starts" and the event has ended (`EV-RULE-035`).

**Worked example 2 (per-attendee, immediate).** The shipped confirmation schedule has interval number 0, unit `now`, type `after_sub`. An attendee is created at `2026-05-02 14:22:37.812`. The trace due date is `2026-05-02 14:22:37` plus zero hours, that is `2026-05-02 14:22:37`, which is already in the past when the synchronous path runs immediately after the creation; the message is therefore sent at once.

**Worked example 3 (per-attendee, one hour later).** The same schedule with interval number 1 and unit `hours` gives `2026-05-02 15:22:37`. The trace is created immediately but is skipped by every run until the job runs at or after that moment.

**Worked example 4 (slot based).** A multi-slot event has slots on 12 June 10:00–12:00 and 12 June 14:00–16:00, both written in the display time zone `Europe/Paris` (that is `08:00`–`10:00` and `12:00`–`14:00` in coordinated universal time in summer). A schedule of 1 hour `before_event` creates two traces with the due dates `2026-06-12 07:00:00` and `2026-06-12 11:00:00` in coordinated universal time. Each trace contacts only the attendees of its own slot and finishes independently.

**Worked example 5 (months).** A schedule of 1 month `after_event` on an event ending `2026-01-31 17:00:00` gives `2026-02-28 17:00:00`, because February has no 31st.

---

## 6. Communication batch progress and completion

**Inputs:** the mode of the schedule, the resume point, the seat figures.
**Outputs:** the sent counter and the done flag.

| Mode | `mail_count_done` | `mail_done` |
|---|---|---|
| attendee based | `count(traces of this schedule WHERE mail_sent)` | left untouched: the schedule stays Running for the whole life of the event |
| slot based, per slot trace | `count(registrations WHERE event = this event AND slot = this slot AND state NOT IN ("draft","cancel") AND identifier <= slot resume point)` | `mail_count_done >= slot.seats_taken` |
| slot based, on the schedule | the sum of the counters of its slot traces | `sum >= event.seats_taken` |
| event based | `count(registrations WHERE event = this event AND state NOT IN ("draft","cancel") AND identifier <= resume point)` | `mail_count_done >= event.seats_taken` |
| nothing processed yet (no resume point) | `0` | `false` |

A global schedule that finds **no** attendee at all is marked done immediately, without sending anything.

**Worked example.** An event has 130 attendees in state `open`, none cancelled, therefore `seats_taken = 130`. The batch size is 50 and the render limit is 1000.

| Pass | Attendees read | Batches | Resume point after | `mail_count_done` | `mail_done` |
|---|---|---|---|---|---|
| 1 | 130 (fewer than the limit) | 50, 50, 30 | the 130th identifier | 130 | `130 >= 130` → true |

With a render limit of 100 instead, the first pass processes 100 attendees in two batches of 50, leaves the counter at 100, leaves `mail_done` false because `100 < 130`, and wakes the job again; the second pass reads the remaining 30, finishes them, and sets `mail_done` to true.

---

## 7. Relative date phrase

**Inputs:** a datetime (the slot start when a slot exists, otherwise the event start), the display time zone of the event, the language of the reader.
**Output:** a short phrase, used in reminder subjects and in text messages.

Algorithm:

1. Convert the current moment into the display time zone and keep its calendar date, `today_tz`.
2. Convert the input datetime into the display time zone and keep its calendar date, `event_date_tz`.
3. `diff = event_date_tz − today_tz` in whole days.
4. Return, in this order:

```formula
diff <= 0                              → "today"
diff = 1                               → "tomorrow"
diff < 7                               → "in <diff> days"
diff < 14                              → "next week"
month(event_date_tz) = month(today_tz + 1 month) → "next month"
otherwise                              → "on <date, medium format, in the reader language>"
```

**Worked examples**, with the current moment `2026-06-01 08:00` in the display time zone:

| Event start (display zone) | `diff` | Phrase |
|---|---|---|
| 2026-05-30 | −2 | `today` |
| 2026-06-01 18:00 | 0 | `today` |
| 2026-06-02 | 1 | `tomorrow` |
| 2026-06-05 | 4 | `in 4 days` |
| 2026-06-10 | 9 | `next week` |
| 2026-07-20 | 49 | `next month` |
| 2026-09-03 | 94 | `on 3 Sep 2026` |

Note the month test: it compares calendar months, not day counts, therefore an event on 20 July is described as `next month` even though it is 49 days away.

---

## 8. Ticket and booth prices with taxes and discounts

**Inputs:** the price stored on the ticket or on the booth category, the product, the taxes of the product, the company of the event, the pricelist that applies to the reader.
**Precision:** the product price precision for the stored prices; the currency precision for the amounts of a sales order.

### Ticket

```formula
price             = the stored ticket price (proposed from the product sales price when a product is chosen)
price_incl        = total including tax of ( price, quantity 1, currency of the ticket, product )
                    using only the taxes of the product
price_reduce      = (1 − contextual discount of the product for the reader) × price
price_reduce_taxinc = total including tax of ( price_reduce, quantity 1,
                      currency of the event company, product )
                      using only the taxes of the product that belong to the company of the event
```

The contextual discount is the discount that the pricelist of the reading context yields for that product; it is `0` when no pricelist applies.

### Booth category

```formula
price               = product sales price + product extra price, whenever the product has a non-zero sales price;
                      otherwise the stored value, editable
price_incl          = total including tax of ( price, quantity 1, category currency, product )
price_reduce        = (1 − contextual discount of the product) × price
price_reduce_taxinc = total including tax of ( price_reduce, quantity 1, category currency, product )
```

### Price written on a sales order line

```formula
line carries a ticket and an event →
    base = price_reduce of the ticket, read in the pricing context of the line,
           when the matching pricelist rule may NOT show a discount
         = price of the ticket,
           when the matching pricelist rule MAY show a discount
    unit price = convert(base, from the currency of the company of the ticket,
                         to the currency of the line)

line carries pending booths and an event →
    base = sum over the booths of booth.booth_category_id.price_reduce  (rule may not show a discount)
         = sum over the booths of booth.price                           (rule may show a discount)
    unit price = convert(base, from the currency of the company of the event,
                         to the currency of the line)
```text

### Public strike-through price of a ticket

A crossed-out original price is shown only when **all** of:

```
ticket.price <> 0
AND a pricelist applies to the reader
AND the pricelist rule matching the ticket product for a quantity of 1 is itself allowed to show a discount
AND (ticket.price − ticket.price_reduce) > 0
```formula

**Worked example.** A ticket has `price = 100.00` in a currency with two decimal places. The product carries one tax of 21 percent, price-excluded. The reader has a pricelist granting 10 percent on that product.

```
price                = 100.00
price_incl           = 100.00 × 1.21 = 121.00
price_reduce         = (1 − 0.10) × 100.00 = 90.00
price_reduce_taxinc  = 90.00 × 1.21 = 108.90
```formula

If the matching pricelist rule may show a discount, the sales order line takes the unit price `100.00` and the storefront displays `100.00` crossed out next to `90.00`. If the rule may not show a discount, the line takes `90.00` and no crossed-out price is shown.

---

## 9. Per-order ticket limits

**Inputs:** the tickets offered, the chosen slot when the event uses slots, the event.
**Output:** the maximum quantity the visitor may put on one order, per ticket.
**Absolute ceiling:** 30 tickets per line, which is also the upper bound accepted for `limit_max_per_order` (`EV-RULE-016`).

Algorithm:

1. When the event has **no** ticket at all, a single generic entry is returned whose limit is:
   - the available seats of the chosen slot, when a slot is given;
   - the available seats of the event, when the event limits seats;
   - the absolute ceiling of 30 otherwise.
2. When tickets exist, compute the availability of every `(chosen slot, ticket)` pair with the algorithm of section 2. Then, per ticket:

```
availability = no limit →
    limit = ticket.limit_max_per_order   when it is non-zero
          = 30                           otherwise
availability is a number →
    limit = min( ticket.limit_max_per_order or availability , availability )
```formula

   In words: an unconstrained ticket is bounded by its own per-order limit, or by the absolute ceiling; a constrained ticket is bounded by the smaller of its per-order limit and the seats that really remain.

**Worked example.** Event capped at 100 seats with 96 taken, slot A, ticket "Standard" capped at 50 with 48 taken on that slot and a per-order limit of 6, ticket "Premium" uncapped with a per-order limit of 0.

```
availability(A, Standard) = min(100 − 96, 50 − 48) = min(4, 2) = 2
  limit(Standard) = min(6, 2) = 2
availability(A, Premium)  = 100 − 96 = 4      (the event limit still applies)
  limit(Premium)  = min(0 or 4, 4) = min(4, 4) = 4
```formula

The visitor may therefore select at most 2 Standard and at most 4 Premium, and the combined check of the registration form additionally refuses a total above the 4 seats remaining on the slot.

---

## 10. Sponsor opening hours

**Inputs:** the event dates, the event display time zone, the two fractional hours of the sponsor.
**Output:** one boolean, recomputed on demand.

Algorithm:

1. When the event is not ongoing, the answer is **false**.
2. When either hour is unset, the answer is **true** (a sponsor with no hours is always open while the event runs).
3. Otherwise:

```
dt_begin  = event.date_begin read in the event time zone
dt_end    = event.date_end read in the event time zone
now_tz    = now (sub-seconds cleared) read in the event time zone
open_from = the calendar date of now_tz at hour_from, in the event time zone
open_to   = the calendar date of now_tz at hour_to, in the event time zone
            plus one day when hour_to = 0          (closing "at midnight" means the next midnight)
from      = max(dt_begin, open_from)
to        = min(dt_end,   open_to)
answer    = from <= now_tz < to
```formula

**Worked example.** An event runs from `2026-06-12 08:00` to `2026-06-14 20:00` in `Europe/Paris`. A sponsor has `hour_from = 9.5` (nine thirty) and `hour_to = 17.0`. The current moment is `2026-06-12 09:15` Paris time.

```
open_from = 2026-06-12 09:30
open_to   = 2026-06-12 17:00
from      = max(2026-06-12 08:00, 2026-06-12 09:30) = 09:30
to        = min(2026-06-14 20:00, 2026-06-12 17:00) = 17:00
09:30 <= 09:15 is false ⇒ closed
```formula

At `2026-06-12 10:00` the same computation gives open. On `2026-06-14` with `hour_to = 22.0`, `to = min(2026-06-14 20:00, 2026-06-14 22:00) = 20:00`, therefore the virtual booth closes with the event rather than at 22:00.

---

## 11. Talk times, the action button window and the agenda grid

### 11.1 Start, end and duration

The three fields form a triangle in which any two determine the third:

```
date      = date_end − duration hours     (recomputed when the end or the duration changes)
date_end  = date + duration hours         (recomputed when the start or the duration changes)
duration  = (date_end − date) in hours    (recomputed when the start or the end is written directly)
```formula

The default duration is `0.5`, that is thirty minutes.

**Worked example.** A talk starts on `2026-06-12 14:00` with a duration of `1.5`. Then `date_end = 15:30`. Writing `date_end = 16:00` recomputes `duration = 2.0`. Writing `duration = 0.75` recomputes `date_end = 14:45`.

### 11.2 Time flags

All comparisons use coordinated universal time, because only differences matter.

```
is_track_live         = date <= now < date_end
is_track_soon         = date > now AND (date − now) < 30 minutes
is_track_today        = calendar date of date = calendar date of now
is_track_upcoming     = date > now
is_track_done         = date_end <= now
track_start_relative  = ⌊date − now⌋ in seconds      when date >= now
                      = ⌊now − date⌋ in seconds      otherwise
track_start_remaining = track_start_relative         when date >= now
                      = 0                            otherwise
is_one_day            = the start and the end fall on the same calendar day in the event time zone
```formula

When the talk has neither a start nor an end, all five booleans are false and both counters are zero.

### 11.3 Action button window

```
if the button is disabled:  is_website_cta_live = false, website_cta_start_remaining = false
otherwise:
  button_start = date + website_cta_delay minutes      (a missing delay counts as zero)
  is_website_cta_live        = button_start <= now <= date_end
  website_cta_start_remaining = ⌊button_start − now⌋ in seconds  when button_start >= now
                                 = 0                                otherwise
```formula

**Worked example.** A talk runs `14:00`–`15:30` with a button delay of 20 minutes. The button appears at `14:20` and disappears at `15:30`. At `14:05` the remaining time is `900` seconds.

### 11.4 Agenda grid

**Inputs:** the talks of the event that have a start date and whose stage is visible in the agenda or which are published; the rooms; the event display time zone.
**Output:** for every calendar day, a list of fifteen-minute time slots, and for every slot and room the talks that start there with the number of slots they occupy.

Algorithm:

1. Read every candidate talk with elevated rights.
2. Collect the rooms of those talks, without duplicates, ordered by room sequence then identifier. A talk without a room is kept, and will occupy **every** room column of the rows it spans.
3. For each talk:
   - convert its start into the event time zone and round it **down** to the previous quarter hour: `rounded = start with minutes replaced by 15 × ⌊minutes ÷ 15⌋ and seconds cleared`;
   - compute the end as `rounded + duration hours` (a missing duration counts as `0.25`) and round it down the same way;
   - the number of occupied rows is `((rounded end − rounded start) in hours) × 4`;
   - split the occupation per calendar day, starting a new bucket when the next quarter hour falls on a later day.
4. Build the day list from the calendar dates of the rounded starts, sorted ascending.
5. For each day, the grid runs from the earliest rounded start of that day to the latest rounded end of that day, in quarter-hour rows; every row carries the talks that start on it, with their row span, their real start and end times formatted in the short time format of the reader language, and the list of `(row, room)` cells they occupy so that no empty cell is drawn over them.
6. Per day, the number of talks and the rooms actually used are counted, the rooms again ordered by sequence then identifier.

**Worked example.** Display zone `Europe/Brussels`. A talk starts at `10:07` and lasts `1.5` hours in room "Main Hall".

```
rounded start = 10:00
rounded end   = 10:00 + 1.5 h = 11:30 → already on a quarter
rows occupied = (11:30 − 10:00) × 4 = 6 rows
displayed times = "10:07 AM" to "11:37 AM" (the real times, not the rounded ones)
occupied cells = (10:00, Main Hall), (10:15, Main Hall), … , (11:15, Main Hall)
```formula

A second talk of 30 minutes starting at `10:20` in "Room B" is rounded to `10:15` and occupies 2 rows. The grid of that day runs from `10:00` to `11:30` in six rows, with two columns.

---

## 12. Participation detection

**Inputs:** the reader, the current site visitor when one exists, the registrations.
**Output:** the set of events in which the reader participates; used by `is_participating` and by the website visibility rule.

Algorithm:

1. Read the current site visitor from the request.
2. When the reader is the anonymous user **and** there is no visitor at all, the answer is the empty set: nothing is known about that reader.
3. Build the base condition `state IN ("open", "done")`. A `draft` or `cancel` registration never makes anyone a participant.
4. Build the identity condition:
   - with a visitor: `visitor_id = that visitor`, and the candidate contact becomes the contact of that visitor;
   - without a visitor: the candidate contact is the contact of the reader;
   - when a candidate contact exists, the identity condition becomes `visitor_id = that visitor OR partner_id = that contact`; with no visitor it is simply `partner_id = that contact`.
5. Read, with elevated rights, the distinct events of the registrations matching both conditions. Those are the events of participation.

The electronic mail address is deliberately **not** used as an identity criterion, because it is not secure enough.

**Worked example.** An anonymous visitor registers for event A; a site visitor record is created and linked to the registration. Later, the same browser opens event A: the visitor is found, a registration in state `open` links it to event A, therefore `is_participating` is true and the event is visible even when its visibility is `link` or `logged_users`. If that visitor signs in and its contact is set, a registration made earlier with the same contact on event B also makes event B a participating event.

---

## 13. Quiz scoring and the leaderboard

### Scoring

**Inputs:** the answers submitted by one visitor for one talk quiz.
**Output:** the points of that visitor for that talk.

```
points of a question = the points attached to the answer the visitor selected
points of the quiz   = Σ over the submitted answers of (answer.awarded_points)
question.awarded_points (shown to the organiser) = Σ over all answers of that question of awarded_points
```formula

Validity gates, in this order: an already completed quiz gives `track_quiz_done`; a submission whose distinct questions do not number exactly the questions of the quiz gives `quiz_incomplete`. A valid submission writes `quiz_completed = true` and `quiz_points = points of the quiz` on the visitor link.

**Worked example.** A quiz has three questions.

| Question | Answers (points) | Visitor picked | Points |
|---|---|---|---|
| Q1 | correct 10, wrong 0, wrong 0 | the correct one | 10 |
| Q2 | correct 20, wrong 5 | the wrong one | 5 |
| Q3 | correct 15, wrong 0 | the correct one | 15 |

`quiz_points = 10 + 5 + 15 = 30`. Note that an incorrect answer may still award points when the organiser configured it that way; correctness and points are two independent attributes of an answer. Submitting only two of the three questions is refused with `quiz_incomplete`.

### Leaderboard

**Inputs:** every visitor link of the talks of the event with a visitor and strictly positive points.
**Output:** an ordered list of `(visitor, points, position)`.

1. Sum `quiz_points` per visitor over the talks of the event, keeping only rows with `quiz_points > 0`.
2. Order by the summed points **descending**, then by the visitor identifier **ascending**. The identifier tie-break makes the ranking stable and favours the visitor who arrived first.
3. Walk the ordered list assigning positions `1, 2, 3, …`. The position counter advances for **every** row, including rows filtered out by a name search, therefore positions always reflect the full ranking.
4. When a name search is given, only visitors whose display name contains the searched text, compared without case, are kept in the returned list; their positions are the full-ranking positions.
5. The top three of the full ranking are returned separately.
6. Pagination shows 30 visitors per page, with at most 5 page links. When the current visitor appears in the ranking and no page was requested, the page containing that visitor is opened, computed as `⌈position ÷ 30⌉`, and the page is scrolled to that row.

**Worked example.** Five visitors score 40, 30, 30, 10 and 5 points; the two 30-point visitors have identifiers 87 and 42.

```
Ranking: 1 → 40 points
         2 → 30 points, visitor 42   (lower identifier wins the tie)
         3 → 30 points, visitor 87
         4 → 10 points
         5 → 5 points
```formula

Searching for the name of visitor 87 returns a single row displayed at position 3.

---

## 14. Revenue figures and currency conversion

### Total sales of an event

**Inputs:** the sales order lines pointing at the event, their totals including tax, their currencies, their companies, the order dates.
**Output:** one monetary amount in the currency of the company of the event.
**Visible to:** the Salesperson group.

Algorithm:

1. Read the order lines whose event is this event, whose total including tax is non-zero, and whose order state is confirmed.
2. Group them by event and by currency and sum the totals including tax.
3. Convert every currency group into the currency of the company of the event, using the rates **of today**, not the rates of the order date.
4. Sum the converted groups.

The deliberate use of today's rates keeps the figure cheap to compute; the consequence is that the figure of a past event moves with the exchange rates and is therefore an indicative figure, not an accounting one. The accounting figures live in the [General ledger](../general-ledger/README.md) domain.

**Worked example.** An event belongs to a company whose currency is `EUR` (euro). Two confirmed orders sell tickets: one for `1,200.00 EUR`, one for `3,000.00 USD` (United States dollar). Today's rate is `1 EUR = 1.10 USD`.

```
EUR group: 1,200.00 → 1,200.00 EUR
USD group: 3,000.00 ÷ 1.10 = 2,727.27 EUR   (rounded to the currency precision)
sale_price_total = 1,200.00 + 2,727.27 = 3,927.27 EUR
```text

### Number of attendees on an order

```
attendee_count = count( registrations of the order WHERE state <> "cancel" )
```formula

On a counter order, the attendee count is the number of registrations carried by the lines of the order, without a state filter.

---

## 15. Barcode generation and the ticket access hash

### Barcode

```
barcode = the decimal text of an unsigned integer built from 8 pseudo-random bytes,
          read in little-endian order
```formula

The value therefore lies between `0` and `18,446,744,073,709,551,615` and is at most 20 characters long. A decimal rendering is longer than a hexadecimal one but encodes into a denser linear barcode, because a numeric-only symbology can be used. Eight bytes are used rather than sixteen, because 16-byte barcodes are not readable by every scanner. Uniqueness is enforced by the database (`EV-RULE-025`); a collision simply makes the insert fail and the caller retries with a new value.

### Ticket access hash

```
hash = keyed digest over the pair ( event identifier , the sorted list of registration identifiers )
       using the platform secret and the purpose label "event-registration-ticket-report-access"
```formula

The comparison performed when the link is used is constant-time. Because the sorted list is part of the input, a link generated for attendees `{4, 9}` does not open the ticket of attendee `7`, and reordering the identifiers in the address does not change the result.

---

## 16. Slot datetimes from date and hours

**Inputs:** the calendar date of the slot, the two fractional hours, the display time zone of the event.
**Output:** two datetimes in coordinated universal time, stored.

```
local_start = combine( date , time_of(start_hour) )   interpreted in the event display time zone
local_end   = combine( date , time_of(end_hour)   )   interpreted in the event display time zone
start_datetime = local_start converted to coordinated universal time
end_datetime   = local_end   converted to coordinated universal time
```formula

where `time_of(h)` turns a fractional hour into hours and minutes: `time_of(9.5) = 09:30`, `time_of(13.25) = 13:15`, `time_of(23.99) = 23:59`.

**Worked example.** A slot on `2026-06-12` from `9.5` to `12.0` in `Europe/Paris` (two hours ahead of coordinated universal time in June):

```
local_start = 2026-06-12 09:30 Paris → start_datetime = 2026-06-12 07:30
local_end   = 2026-06-12 12:00 Paris → end_datetime   = 2026-06-12 10:00
```formula

The same slot in January (one hour ahead) would store `08:30` and `11:00`.

---

## 17. Talk suggestion ranking

**Inputs:** a current talk, an optional restricting condition, a wanted number of suggestions.
**Output:** the ordered list of the next talks to propose.

1. Take every talk of the same event except the current one, restricted by the optional condition, read in ascending start-date order.
2. Sort the candidates **descending** by this tuple, so that a true value or a larger number comes first:

```
( is_published,
  track_start_remaining = 0 AND track_start_relative < 600 AND NOT is_track_done,   "started less than ten minutes ago and still running"
  track_start_remaining > 0,                                                        "still to come"
  − track_start_remaining,                                                          "the sooner, the better"
  is_reminder_on,                                                                   "the reader wants it"
  NOT wishlisted_by_default,                                                        "an ordinary talk before a key talk at equal rank"
  number of tags shared with the current talk,
  the room equals the room of the current talk,
  a pseudo-random integer between 0 and 20 )
```formula

3. Return the first `limit` candidates.

The random component makes equally ranked talks rotate between readers rather than always presenting the same one. The live-video variant of this ranking additionally restricts the candidates to talks that carry a video link.

**Worked example.** Three published candidates: talk X started 4 minutes ago and is still running; talk Y starts in 20 minutes; talk Z finished yesterday. The ordering keys give X the "live" flag, Y the "upcoming" flag with `−1200` seconds, and Z neither. The order is X, then Y, then Z.

---

## 18. Display names carrying availability

Three entities change their readable name when the caller asks for availability.

```
Event:
  sold out                              → "<name> (Sold out)"
  seats_limited AND seats_max <> 0  → "<name> (<seats_available> seats remaining)"
  otherwise                             → "<name>"

Event Ticket:
  seats_max = 0 OR the event is multi-slot → "<name>"
  seats_available = 0                          → "<name> (Sold out)"
  otherwise                                    → "<name> (<seats_available> seats remaining)"

Event Slot (always):
  base = "<date, medium format>, <start hour, short time format> - <end hour, short time format>"
  and, only when availability is requested and the event limits seats and the event is NOT multi-slot:
      seats_available = 0 → "<base> (Sold out)"
      otherwise           → "<base> (<seats_available> seats remaining)"
```formula

The seat count is formatted with **zero** decimal places using the language of the reader, which means a thousands separator may appear. Availability is deliberately hidden on a ticket of a multi-slot event, and on the slot of a multi-slot event, because a single number cannot describe every slot-and-ticket combination and would mislead the reader.

**Worked example.** An event named `Summer Conference` capped at 100 with 70 taken displays `Summer Conference (30 seats remaining)`. Once 100 seats are taken it displays `Summer Conference (Sold out)`. A slot of 12 June from `9.5` to `12.0` displays `Jun 12, 2026, 9:30 AM - 12:00 PM` in an English locale.

---

## 19. Measures of the revenue analysis

The revenue analysis is a read-only join of registrations, events, slots, tickets, orders and order lines. One row exists per registration. Two measures are computed per row:

```
sale_price =
    0                                                         if order_line.quantity = 0
    order_line.total_including_tax
      ÷ (order.currency_rate, replaced by 1.0 when it is zero or missing)
      ÷ order_line.quantity                                   otherwise

sale_price_untaxed =
    0                                                         if order_line.quantity = 0
    order_line.subtotal_excluding_tax
      ÷ (order.currency_rate, replaced by 1.0 when it is zero or missing)
      ÷ order_line.quantity                                   otherwise
```formula

Both measures are therefore **per seat** and expressed in the company currency, since dividing by the order rate removes the order currency. A registration with no order line shows zero on both measures.

**Worked example.** An order line sells 4 seats for a total including tax of `484.00` and a subtotal of `400.00`, on an order whose currency rate is `1.10`.

```
sale_price         = 484.00 ÷ 1.10 ÷ 4 = 110.00 per seat
sale_price_untaxed = 400.00 ÷ 1.10 ÷ 4 =  90.909… per seat
```formula

Summing `sale_price` over the four rows of that line gives `440.00`, the order total expressed in the company currency.

---

## Reconciliation notes

1. **Provenance.** Every formula of this file comes from version M, which was the only version that
   carried calculations; version P announced the same list in its reading order (seat availability,
   communication scheduling arithmetic, price derivations, time-zone arithmetic, quiz point awards)
   and each of those five subjects is present here.
2. **Identifiers inside the formulas.** The quantities are named by the storage names the database
   carries — `seats_max` rather than the readable substitute version M used, `limit_max_per_order`,
   `date_tz`, `color` — so that a formula can be checked against a field table without a translation
   step.
3. **Effective maximum of a multi-slot event.** The event-level available figure uses `seats_max`
   multiplied by the slot count, while each slot uses `seats_max` alone. That asymmetry is deliberate
   and is confirmed by the source; the worked example in section 1 carries it through.

```