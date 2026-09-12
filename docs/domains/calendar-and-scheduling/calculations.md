# Calculations of Calendar and Scheduling

## 1. Overview

This domain computes no money. What it computes is time: instants, durations, repetitions,
horizons, lead times and intersections of working intervals. Those computations are where two
independent implementations diverge, so each one is given here with its inputs named in words, its
evaluation order, its rounding rule and at least one worked example carried to the last digit the
rule produces.

Three conventions hold throughout.

**Instants are held in coordinated universal time.** Every stored start and stop of a timed meeting
is an instant in coordinated universal time. Conversion to a reader's clock happens on the way out.
The one exception is an all-day meeting, whose start and stop deliberately carry a conventional
eight and eighteen o'clock so that the meeting occupies the same calendar days everywhere; see
[chapter 4](#4-all-day-days-and-instants).

**Durations are decimal hours.** The duration field holds hours as a decimal number, so half an hour
is 0.5 and one hour and forty-five minutes is 1.75.

**A whole number of minutes is the unit of reminder arithmetic.** Every reminder comparison happens
in whole minutes, never in the reminder's own unit.

Contents:

1. [Overview](#1-overview)
2. [The default start and the default duration](#2-the-default-start-and-the-default-duration)
3. [Duration and stop](#3-duration-and-stop)
4. [All-day days and instants](#4-all-day-days-and-instants)
5. [The display time](#5-the-display-time)
6. [The reminder lead time in minutes](#6-the-reminder-lead-time-in-minutes)
7. [The reminder firing instant and the wake-up](#7-the-reminder-firing-instant-and-the-wake-up)
8. [The next reminder instant in a series](#8-the-next-reminder-instant-in-a-series)
9. [Pattern serialisation and parsing](#9-pattern-serialisation-and-parsing)
10. [Occurrence generation and time zones](#10-occurrence-generation-and-time-zones)
11. [The occurrence horizon and its caps](#11-the-occurrence-horizon-and-its-caps)
12. [Range calculation and count inflation](#12-range-calculation-and-count-inflation)
13. [The end day of a trimmed series](#13-the-end-day-of-a-trimmed-series)
14. [The time shift of a recurrence update](#14-the-time-shift-of-a-recurrence-update)
15. [Attendee counters](#15-attendee-counters)
16. [Unavailability from overlapping meetings](#16-unavailability-from-overlapping-meetings)
17. [Unavailability from working schedules](#17-unavailability-from-working-schedules)
18. [The pattern sentence](#18-the-pattern-sentence)
19. [The identifier of an occurrence in the first external calendar service](#19-the-identifier-of-an-occurrence-in-the-first-external-calendar-service)
20. [Mapping reminders to and from the external services](#20-mapping-reminders-to-and-from-the-external-services)
21. [Rounding summary](#21-rounding-summary)

---

## 2. The default start and the default duration

### 2.1 The default start

A meeting created without a start begins at the next half-hour boundary counted from midnight.

```formula
minutes past the hour  =  the minute part of the current instant
seconds into the minute = the second and sub-second part of the current instant
complement in minutes  =  (30 − (minutes past the hour modulo 30)) modulo 30
default start          =  current instant
                        + complement in minutes
                        − seconds into the minute
```

The second subtraction is what makes the result land exactly on a boundary rather than merely
thirty minutes later. An instant that already sits on a boundary is left untouched, because the
complement is then zero and there are no seconds to remove.

**Worked example.** The current instant is 16 April 2024 at 10:07:23. Minutes past the hour are 7,
so the complement is (30 − (7 modulo 30)) modulo 30 = 23 minutes. Seconds into the minute are 23.
The default start is 10:07:23 + 23 minutes − 23 seconds = **16 April 2024 at 10:30:00**.

**Second worked example.** The current instant is 16 April 2024 at 10:30:00. Minutes past the hour
are 30, so the complement is (30 − 0) modulo 30 = 0 minutes, and there are no seconds. The default
start is **16 April 2024 at 10:30:00**, unchanged.

### 2.2 The default duration

```formula
default duration in hours =
    the first value found among:
      1. the reader's own stored default for this company
      2. the reader's own stored default for any company
      3. this company's stored default
      4. the platform-wide stored default
    otherwise 1
```

The four lookups are made in that order and the first that yields a value wins. All four read the
default-values register of the platform, keyed by the entity Calendar Event and the field `duration`.

**Worked example.** No default is stored anywhere. The default duration is **1 hour**. A reader who
then stores a personal default of 2 for no particular company gets **2 hours**, and a second company
that stores 8 gives readers acting in that company **8 hours**, because the third lookup succeeds.

### 2.3 The default stop

```formula
default stop = default start + default duration in hours
```

**Worked example.** Continuing the first example of 2.1 with a default duration of 1 hour, the
default stop is **16 April 2024 at 11:30:00**.

---

## 3. Duration and stop

The two fields derive from each other; which one wins depends on which one the caller wrote.

### 3.1 Stop from start and duration

```formula
minutes of the meeting = round to the nearest whole minute of ( duration in hours × 60 )
stop                   = start + minutes of the meeting
                         − 1 second, when the meeting is all-day
```

The rounding is to the nearest whole minute; a value that falls exactly halfway is rounded to the
nearer even minute. An empty duration is read as 1 hour before the multiplication.

**Worked example.** Start 16 April 2024 at 09:00:00 with a duration of 1.75 hours.
1.75 × 60 = 105.00 minutes, which needs no rounding. The stop is **16 April 2024 at 10:45:00**.

**Worked example of the halfway rule, rounding up.** Duration 0.125 hours. 0.125 × 60 = 7.5 minutes
exactly. The two candidates are 7 and 8; 8 is even, so the stop is start + **8 minutes**.

**Worked example of the halfway rule, rounding down.** Duration 0.375 hours. 0.375 × 60 = 22.5
minutes exactly. The two candidates are 22 and 23; 22 is even, so the stop is start + **22 minutes**.

**Worked example, all-day.** An all-day meeting starting on 12 July 2024 at the conventional
08:00:00 with a duration of 10 hours gets a stop of 08:00:00 + 600 minutes − 1 second =
**12 July 2024 at 17:59:59**.

### 3.2 Duration from start and stop

```formula
duration in hours = round to two decimals of ( ( stop − start ) in seconds ÷ 3600 )
```

A missing start or a missing stop yields zero. The rounding is to two decimals, half away from zero.

**Worked example.** Start 16 April 2024 at 09:00:00, stop 16 April 2024 at 12:19:59. The difference
is 11 999 seconds. 11 999 ÷ 3600 = 3.333055…, which rounds to **3.33 hours**.

**Worked example.** Start 21 October 2019 at 08:00:00, stop 23 October 2019 at 18:00:00. The
difference is 208 800 seconds. 208 800 ÷ 3600 = 58.00 exactly, so the duration is **58.0 hours**.
An all-day meeting therefore carries a large duration rather than zero, whatever the length of the
day.

---

## 4. All-day days and instants

### 4.1 From instants to days

```formula
when the meeting is all-day and both instants are set:
    start day = the calendar-date part of the start
    stop day  = the calendar-date part of the stop
otherwise:
    start day = empty
    stop day  = empty
```

### 4.2 From days back to instants

```formula
when the meeting is all-day:
    candidate start = the start day, or the start when there is no start day, at 08:00:00
    candidate stop  = the stop day, or the stop when there is no stop day, at 18:00:00
    when both days are set:
        start = candidate start
        stop  = candidate stop
    otherwise:
        start day = candidate start
        stop day  = candidate stop
```

**Worked example.** A meeting is written with the all-day marker, a start of 16 October 2018 at
00:00:00, a start day of 16 October 2018, a stop of 18 October 2018 at 00:00:00 and a stop day of
18 October 2018. Both days are set, so the instants are rewritten: the start becomes
**16 October 2018 at 08:00:00** and the stop **18 October 2018 at 18:00:00**. The duration derived
from those two is 58.0 hours.

### 4.3 The day of a start, in the series' time zone

When a series computes which weekday, which day of the month and which position within the month a
reference occurrence falls on, it must read the occurrence's day in the series' own time zone, not in
coordinated universal time.

```formula
when the occurrence has no start:               reference day = today
when the series has a time zone and repeats:
    instant to convert = the start, or the start with its clock set to 12:00 when the meeting is all-day
    reference day      = the calendar-date part of ( instant to convert, read in the series' time zone )
otherwise:                                      reference day = the calendar-date part of the start
```

Setting the clock to noon for an all-day meeting is what stops a shift of a few hours from moving the
meeting to the previous or the next day.

**Worked example.** An all-day occurrence stored with a start of 22 October 2019 at 08:00:00 in a
series whose time zone is twelve hours ahead of coordinated universal time. Without the noon rule
the converted instant would be 22 October at 20:00 and the day would still be right; with a start
stored at 00:00 it would have been 22 October at 12:00. Applying the rule, the instant to convert is
22 October at 12:00 and the reference day is **22 October 2019** in every zone from twelve hours
behind to eleven hours ahead.

### 4.4 The position of a day inside its month

```formula
occurrence within the month = the smallest whole number not less than ( day of month ÷ 7 )
position                    = −1 when that number is 4 or 5, otherwise that number
```

The mapping of the fourth and the fifth occurrence to "Last" is deliberate: a month has either four
or five of any weekday, so "the fourth Tuesday" and "the last Tuesday" are treated as the same
request.

**Worked examples.** 17 December 2019 is a Tuesday. 17 ÷ 7 = 2.43, whose ceiling is 3, so the
position is **3**, the third Tuesday. 25 December 2019 is a Wednesday. 25 ÷ 7 = 3.57, whose ceiling
is 4, so the position is **−1**, the last Wednesday. 1 April 2026 gives 1 ÷ 7 = 0.14, whose ceiling
is 1, so the position is **1**.

---

## 5. The display time

```formula
zone   = the zone carried in the reading context,
         else the reader's own contact time zone,
         else coordinated universal time
day format  = the reader's language date format
time format = the reader's language time format
local start = the start read in that zone
local stop  = the stop read in that zone

when the meeting is all-day:
    display time = "All Day, " + local start formatted with the day format

when the duration in hours is strictly less than 24:
    end of the sentence = ( local start + round to the nearest whole minute of ( duration × 60 ) )
                          formatted with the time format
    display time = local start with the day format
                 + " at ("
                 + local start with the time format
                 + " To "
                 + end of the sentence
                 + ") ("
                 + the name of the zone
                 + ")"

otherwise:
    display time = local start with the day format
                 + " at "
                 + local start with the time format
                 + " To" + a line break + " "
                 + local stop with the day format
                 + " at "
                 + local stop with the time format
                 + " ("
                 + the name of the zone
                 + ")"
```

**Worked example, short meeting.** Start 16 April 2024 at 08:00:00 in coordinated universal time,
duration 2 hours, zone two hours ahead, day format month, day and year separated by solidi, time
format hours, minutes and seconds separated by colons. The local start is 10:00:00 on 16 April 2024.
The end of the sentence is 10:00:00 + 120 minutes = 12:00:00. The display time is
**"04/16/2024 at (10:00:00 To 12:00:00) (Europe/Brussels)"**.

**Worked example, long meeting.** Start 21 October 2019 at 08:00:00, stop 23 October 2019 at
18:00:00, duration 58.0 hours, zone coordinated universal time. Since 58 is not less than 24 the
third branch applies and the display time is **"10/21/2019 at 08:00:00 To⏎ 10/23/2019 at 18:00:00
(UTC)"**, where ⏎ marks the line break that is part of the text.

**Worked example, all-day.** An all-day meeting starting 31 July 2013 yields **"All Day,
07/31/2013"**.

The same computation is available with a forced zone, which is what the message templates use so
that each recipient reads the times in their own zone.

---

## 6. The reminder lead time in minutes

```formula
when the unit is minutes: lead time in minutes = duration
when the unit is hours:   lead time in minutes = duration × 60
when the unit is days:    lead time in minutes = duration × 60 × 24
otherwise:                lead time in minutes = 0
```

No rounding is involved: the duration is a whole number and the multipliers are whole numbers.

**Worked examples.** 15 with the unit minutes gives **15**. 3 with the unit hours gives 3 × 60 =
**180**. 1 with the unit days gives 1 × 60 × 24 = **1440**. 6 with the unit hours gives **360**.

### 6.1 Searching on the lead time

A filter on the lead time is rewritten into a filter on the two stored fields, so that a reminder of
one hour is found by a search for ninety minutes or less:

```formula
filter on lead time with operator ⊙ and value v
  ≡ ( unit is minutes and duration ⊙ v )
    or ( unit is hours   and duration ⊙ ( v ÷ 60 ) )
    or ( unit is days    and duration ⊙ ( v ÷ 60 ÷ 24 ) )
```

**Worked example.** A search for a lead time of at most 90 minutes becomes: the unit is minutes and
the duration is at most 90; or the unit is hours and the duration is at most 90 ÷ 60 = 1.5; or the
unit is days and the duration is at most 90 ÷ 1440 = 0.0625. A one-hour reminder satisfies the
second alternative, because 1 ≤ 1.5, and is therefore returned.

A filter listing several values becomes the alternation of the equality filters; a filter for an
empty value becomes a filter on an empty duration; operators other than equality, membership and the
four inequalities are not supported.

---

## 7. The reminder firing instant and the wake-up

### 7.1 The firing instant

```formula
firing instant = the occurrence's start − the reminder's lead time in minutes
```

**Worked example.** An occurrence starting 13 April 2022 at 10:15:00 with a reminder of 5 minutes
fires at **13 April 2022 at 10:10:00**. The same occurrence with a reminder of 1 hour fires at
**13 April 2022 at 09:15:00**.

### 7.2 Whether a wake-up is planted

For each reminder of a triggering channel attached to an occurrence:

```formula
plant a wake-up at the firing instant when
      ( the occurrence's series holds no wake-up
        or the wake-up it holds is at a different instant )
  and ( the reminder job has never run
        or the firing instant is strictly later than its last run )
```

The triggering channels are electronic mail and, when the text-message package is installed, text
message. In-application reminders plant no wake-up; they are polled instead.

**Worked example.** The reminder job last ran on 13 April 2022 at 09:00:00. An occurrence starting
that day at 10:15:00 carries a five-minute reminder, so the firing instant is 10:10:00, which is
later than the last run and the series holds nothing: **one wake-up is planted at 10:10:00**. A
second occurrence of the same series starting 14 April at 10:15:00 would plant a wake-up at 14 April
10:10:00, but the series already holds one at a different instant, so it is planted and replaces the
first only if the two occurrences are processed in that order; in practice the series is asked for
its next occurrence only, so exactly **one wake-up exists per series**.

### 7.3 The selection window of the reminder job

```formula
an occurrence and one of its reminders are selected when
      the occurrence is active
  and the reminder's channel is the channel being processed
  and ( the occurrence's start − the reminder's duration in its own unit ) ≥ the previous run
  and ( the occurrence's start − the reminder's duration in its own unit ) <  the current instant
```

The previous run is the last-call instant of the reminder job, or, when it has never run, today less
one week.

**Worked example.** The job runs on 13 April 2022 at 10:11:00 and last ran at 09:46:00. An
occurrence starts at 10:15:00 and carries a five-minute reminder, so its firing instant is 10:10:00.
Since 09:46:00 ≤ 10:10:00 < 10:11:00, the pair is **selected**. A reminder of thirty minutes on the
same occurrence fires at 09:45:00, which is earlier than 09:46:00, so it is **not selected**.

---

## 8. The next reminder instant in a series

After a run has sent the reminders of an occurrence that belongs to a series, the series plants its
next wake-up. Let the occurrence be the one just processed.

```formula
sorted reminders  = the occurrence's reminders ordered by lead time in minutes, ascending
triggered reminder = the first of those that the run selected
there are earlier reminders left = ( the first sorted reminder is not the triggered reminder )

when the series holds a wake-up whose instant is at or before now:
    next instant = the occurrence's start − the first sorted reminder's lead time
                       when there are earlier reminders left
                   otherwise the occurrence's start

when the series holds no wake-up at all and the occurrence has at least one reminder:
    later occurrences = the occurrences of the series whose start is strictly later than this one
    when there are later occurrences:
        next occurrence = the earliest of them
        next instant    = the next occurrence's start − the first sorted reminder's lead time

otherwise: no wake-up is planted
```

**Worked example, daily series.** A daily series of three occurrences starts on 13 April 2022 at
10:15:00, with a five-minute reminder. At creation one wake-up is planted at 10:10:00 on 13 April.
The job runs at 10:11:00 on 13 April and sends that reminder; the series' wake-up instant, 10:10:00,
is at or before now, and the occurrence has no earlier reminder left, so the next instant is the
occurrence's own start, 10:15:00 — a wake-up that fires almost at once and then finds nothing to
send. The job runs again on 14 April at 10:11:00, sends the reminder of the second occurrence, and
the series now holds no wake-up, so the next instant is the third occurrence's start,
15 April 10:15:00, less five minutes: **15 April 2022 at 10:10:00**.

**Worked example, monthly series with an hour of lead.** A monthly series of two occurrences is
created on 16 April 2024 at 10:00:00; the occurrences start at 12:00:00 on 16 April and on 16 May,
and carry a one-hour reminder. At creation one wake-up is planted at **16 April 2024 at 11:00:00**.
The wake-ups are then cleared and the job is run on 22 April at 10:00:00: the series holds no
wake-up, the later occurrence starts on 16 May at 12:00:00, and the next instant is
**16 May 2024 at 11:00:00**.

---

## 9. Pattern serialisation and parsing

### 9.1 Serialisation

The pattern text is the standard calendar-interchange recurrence rule. It is rebuilt whenever any
pattern field changes, and only written back when the new text differs from the stored one.

The text is produced from the pattern fields in this order:

1. The frequency, taken from the four values `daily`, `weekly`, `monthly` and `yearly`.
2. The interval.
3. For a monthly series counted by number, the day of the month.
4. For a monthly series counted by position, the weekday together with its position, the position
   being 1, 2, 3, 4 or −1.
5. For a weekly series, the list of selected weekdays and the first day of the week taken from the
   reader's language.
6. The end: a count when the termination mode is by count; nothing when it is endless, other than
   the horizon count of [chapter 11](#11-the-occurrence-horizon-and-its-caps); an end instant equal
   to the last microsecond of the end day when the mode is by end day.
7. A start line naming the first instant of the series.

Two guards run before anything is produced: the interval must be strictly positive and, when the
mode is by count, the count must be strictly positive. Their messages are in
[business-rules.md](business-rules.md#3-recurrence).

### 9.2 Parsing

Writing the pattern text directly parses it back into the pattern fields.

1. Extension parameters, that is any parameter whose name begins with `X-`, are stripped, wherever
   they appear, together with the separator that precedes them; a leading colon or semicolon left
   behind is removed.
2. When the remaining text carries a trailing `Z` on its end instant and the series' first instant
   carries no zone, that first instant is read as coordinated universal time before parsing.
3. The frequency, the count, the interval and the end instant are read from the text.
4. When the text names weekdays, all seven weekday markers are cleared, the named ones are set and
   the frequency is forced to weekly.
5. When the text names a weekday with a position, the weekday and the position are stored, the
   monthly mode becomes by position and the frequency is forced to monthly.
6. When the text names a day of the month and the frequency is monthly, the day is stored and the
   monthly mode becomes by number.
7. The termination mode becomes by end day when an end instant was read, otherwise by count when a
   count was read, otherwise endless.
8. An end instant that carries a zone is converted into the series' own time zone before its day is
   stored, so that the stored end day is the right local boundary.

**Worked example of step 8.** The text ends with an end instant of 26 October 2023 at 02:59:59 in
coordinated universal time. In a series whose time zone is three hours behind, that instant is
25 October 2023 at 23:59:59 locally, so the stored end day is **25 October 2023**. In a series whose
time zone is coordinated universal time the stored end day is **26 October 2023**. In a series whose
time zone is five and a half hours ahead the instant is 26 October at 08:29:59 locally, so the
stored end day is **26 October 2023**. With an end instant of 25 October 2023 at 18:59:59 in
coordinated universal time and the same five-and-a-half-hour zone, the local instant is 26 October
at 00:29:59, so the stored end day is **26 October 2023** even though the instant names the
twenty-fifth.

**Worked example of step 1.** The text
`RRULE;X-EVOLUTION-ENDDATE=20191112;X-OTHER-PARAM=0:X-AMAZING=1;FREQ=WEEKLY;COUNT=3;X-MAIL-special=1;BYDAY=WE`
is reduced to `RRULE:FREQ=WEEKLY;COUNT=3;BYDAY=WE`, which yields a weekly series of three occurrences
on Wednesdays. The text `X-EVOLUTION-ENDDATE=20371102T114500Z:FREQ=WEEKLY;COUNT=720;BYDAY=MO` is
reduced to `FREQ=WEEKLY;COUNT=720;BYDAY=MO`, which yields a weekly series of seven hundred and twenty
occurrences on Mondays.

---

## 10. Occurrence generation and time zones

Generating the occurrences of a series is a five-step procedure.

1. **Find the period start.** Take the reference occurrence's start and move it back to the start of
   the period the pattern counts in: for a weekly series, the first day of that week in the reader's
   language; for a monthly series, the first day of that month; otherwise the day itself. When the
   move crosses a change of daylight saving time in the series' time zone, the move is abandoned and
   the reference start is used unchanged; this is what stops a series created just after a clock
   change from generating a duplicate occurrence.
2. **Decide whether the series is all-day.** Add one for every all-day occurrence and subtract one
   for every timed occurrence; the series is treated as all-day when the total is zero or more. A
   series is therefore all-day when at least half of its occurrences are.
3. **Generate.** For an all-day series, generate straight from the period start. For a timed series,
   first read the period start in the series' time zone, drop the zone marker, and generate from
   that; the generator then works on wall-clock values.
4. **Re-attach the zone.** For a timed series, attach the series' time zone to each generated
   wall-clock value, choosing standard time whenever the value is ambiguous, and convert back to
   coordinated universal time.
5. **Pair with a stop.** Each generated start is paired with a stop obtained by adding the reference
   occurrence's own length, that is its stop minus its start.

**Worked example, a clock change in the middle of a monthly series.** A series repeats on the first
of every month at six in the morning in a zone five hours behind coordinated universal time in
winter and four hours behind in summer, from February to May 2019. Working in wall-clock values the
generator produces 1 February 06:00, 1 March 06:00, 1 April 06:00 and 1 May 06:00. The zone changes
on 10 March. Re-attaching the zone gives instants of **1 February 11:00**, **1 March 11:00**,
**1 April 10:00** and **1 May 10:00** in coordinated universal time. The wall-clock time stays at six
in the morning, which is the point of the exercise.

**Worked example, an ambiguous wall-clock time.** A weekly series repeats on Sundays at 01:30 in a
zone that leaves daylight saving time on 27 October 2002, so that 01:30 happens twice that day. The
first occurrence is 20 October 2002 at 01:30 local, which is **20 October 05:30** in coordinated
universal time. The second is 27 October at 01:30 local; standard time is chosen, five hours behind,
so the instant is **27 October 06:30**. Both occurrences keep a duration of 1 hour.

**Worked example, a wall-clock time that does not exist.** A weekly series repeats on Sundays at
02:30 in a zone that enters daylight saving time on 7 April 2002 by skipping the hour from 02:00 to
03:00. The first occurrence is 31 March 2002 at 02:30 local, which is **31 March 07:30** in
coordinated universal time. The second falls on a wall-clock value that does not exist; it is
resolved as the same number of minutes after midnight, which lands at 03:30 in the new offset, that
is **7 April 07:30** in coordinated universal time. Both occurrences keep a duration of 1 hour.

**Worked example, an all-day series across a clock change.** An all-day series repeats on Mondays
from 23 March 2020, in a zone that changes on 29 March. The occurrences are generated without any
zone conversion, so they are **23 March 00:00 to 23 March 23:59** and **30 March 00:00 to 30 March
23:59**, keeping the same calendar days for every reader.

**Worked example, a backward move across a clock change.** A monthly series counted by number is
created on 27 March 2023 at 09:00 in a zone one hour ahead in winter and two hours ahead in summer,
which changed on 26 March. The stored start is therefore 27 March 07:00 in coordinated universal
time. The period start would be 1 March, which is on the other side of the clock change, so the move
is abandoned and 27 March is used. The two occurrences are **27 March 07:00** and **27 April 07:00**,
each lasting one hour.

---

## 11. The occurrence horizon and its caps

Two ceilings bound every series: a horizon expressed in years, read from the system parameter
`calendar.max_recurrence_years` and defaulting to fifteen, and a hard cap of **720** occurrences.

```formula
when the termination mode is by count:
    generated count = the smaller of ( the count , 720 )

when the termination mode is by end day:
    no count is imposed; the generator stops at the last microsecond of the end day

when the termination mode is endless and the frequency is yearly:
    generated count = the smaller of ( horizon in years , 720 )

when the termination mode is endless and the frequency is monthly:
    generated count = the smaller of ( horizon in years × 12 , 720 )

when the termination mode is endless and the frequency is weekly:
    selected weekdays = how many of the seven weekday markers are set
    weeks in the horizon = whole part of ( horizon in years × 365 ÷ 7 )
    generated count = the smaller of
        ( whole part of ( weeks in the horizon × selected weekdays ÷ the larger of ( interval , 1 ) ) , 720 )

when the termination mode is endless and the frequency is daily:
    generated count = the smaller of
        ( whole part of ( horizon in years × 365 ÷ the larger of ( interval , 1 ) ) , 720 )
```

Every division above discards the fractional part rather than rounding it.

**Worked examples with the shipped horizon of fifteen years.**

| Frequency | Interval | Weekdays | Arithmetic | Generated count |
|---|---|---|---|---|
| daily | 1 | — | whole part of (15 × 365 ÷ 1) = 5475; smaller of (5475, 720) | **720** |
| monthly | 1 | — | 15 × 12 = 180; smaller of (180, 720) | **180** |
| yearly | 1 | — | smaller of (15, 720) | **15** |

**Worked examples with a horizon of five years.**

| Frequency | Interval | Weekdays | Arithmetic | Generated count |
|---|---|---|---|---|
| daily | 1 | — | whole part of (5 × 365 ÷ 1) = 1825; smaller of (1825, 720) | **720** |
| monthly | 1 | — | 5 × 12 = 60 | **60** |
| yearly | 1 | — | 5 | **5** |
| weekly | 1 | all seven | weeks = whole part of (5 × 365 ÷ 7) = whole part of (1825 ÷ 7) = 260; 260 × 7 ÷ 1 = 1820; smaller of (1820, 720) | **720** |

**Worked examples with a horizon of two years, a series starting on Wednesday 1 April 2026.**

| Frequency | Interval | Weekdays | Arithmetic | Generated count | Occurrences kept |
|---|---|---|---|---|---|
| weekly | 1 | Wednesday | weeks = whole part of (2 × 365 ÷ 7) = whole part of (730 ÷ 7) = 104; 104 × 1 ÷ 1 = 104 | **104** | 104, because the first generated Wednesday is the start day itself |
| weekly | 1 | Monday, Wednesday, Friday | 104 × 3 ÷ 1 = 312 | **312** | 311, because the Monday of the first week precedes the start day and is dropped by [chapter 12](#12-range-calculation-and-count-inflation) |
| weekly | 2 | Wednesday | whole part of (104 × 1 ÷ 2) = 52 | **52** | 52 |

**Worked example of the hard cap with days lost at the front.** With a horizon of five years, all
seven weekdays selected, an interval of one and a start on Wednesday 1 April 2026 in a language whose
week begins on Sunday, the generated count is 720. The period start is Sunday 29 March 2026, so the
first three generated occurrences, on Sunday 29, Monday 30 and Tuesday 31 March, precede the start
day and are dropped. **717 occurrences remain.** No inflation happens because inflation only applies
when the termination mode is by count.

---

## 12. Range calculation and count inflation

Because the generator starts at the beginning of the period rather than at the reference occurrence,
a weekly or monthly series can produce occurrences that lie before the reference occurrence. Those
are dropped. When the termination mode is by count, dropping them would leave too few, so the count
is temporarily raised.

1. Remember the original count when the termination mode is by count.
2. Generate the candidate ranges from the reference occurrence's start and its length.
3. Keep the candidates whose start day and whose stop day are both at or after the reference
   occurrence's start day.
4. When an original count was remembered and fewer candidates survived than that count, set the
   count to ( 2 × the original count ) − the number of survivors, generate again, and restore the
   original count.
5. Drop the past candidates again from the new set.

```formula
inflated count = ( 2 × original count ) − number of surviving candidates
```

**Worked example.** A weekly series repeats on Tuesdays with a count of 3, and its reference
occurrence starts on Wednesday 23 October 2019. The week begins on Monday, so the period start is
Monday 21 October. The first generation yields Tuesday 22 October, Tuesday 29 October and Tuesday
5 November. The first of those precedes the reference day, so two survive, which is fewer than three.
The count becomes ( 2 × 3 ) − 2 = **4**. The second generation yields 22 October, 29 October,
5 November and 12 November; dropping 22 October leaves **29 October, 5 November and 12 November**,
exactly three occurrences. The stored count is restored to 3.

**Worked example with no inflation.** A weekly series repeats on Tuesdays with a count of 3 and its
reference occurrence starts on Monday 21 October 2019 at 08:00, lasting until 23 October at 18:00.
The period start is Monday 21 October, and the generated Tuesdays are 22 October, 29 October and
5 November, all at or after the reference day. Three survive, which equals the count, so no
inflation happens. Each start is paired with a stop two days and ten hours later, giving
**22 October 08:00 to 24 October 18:00**, **29 October 08:00 to 31 October 18:00** and
**5 November 08:00 to 7 November 18:00**.

### 12.1 Reconciling generated ranges with existing occurrences

An occurrence is considered already in step with the pattern when the pair of its start and its stop
is exactly one of the generated ranges. Those occurrences are kept and their ranges are not created
again; every other occurrence of the series is detached; every range not matched by an occurrence is
created by copying the reference occurrence's values and overriding the start, the stop, the series
reference and the follow marker.

Because the comparison is on the exact pair of instants, applying a series twice with no change in
between creates nothing and detaches nothing.

---

## 13. The end day of a trimmed series

When a series is stopped at an occurrence and at least one earlier occurrence remains, the series
becomes an end-day series.

```formula
when the occurrence is all-day:
    boundary = the start of the period containing the occurrence's start day
otherwise:
    boundary instant   = the start of the period containing the occurrence's start
    boundary localised = boundary instant, read as coordinated universal time,
                         converted into the series' time zone
    boundary           = the calendar-date part of boundary localised
end day = boundary − 1 day
```

The start of the period is the first day of the week in the reader's language for a weekly series,
the first day of the month for a monthly series, and the day itself otherwise.

**Worked example.** A weekly series on Tuesdays holds occurrences starting 22 October, 29 October
and 5 November 2019 at 01:00 in coordinated universal time; its time zone is four hours ahead; the
week begins on Monday. The series is stopped at the second occurrence. The period start is Monday
28 October at 00:00. Read four hours ahead, that is 28 October at 04:00, whose day is 28 October.
The end day is 28 October − 1 day = **27 October 2019**, and the termination mode becomes by end day.
The series then holds a single occurrence, the one starting 22 October.

**Worked example, all-day.** The same series with all-day occurrences starting 22 October at 08:00.
The period start is computed on the start day, giving Monday 28 October, and the end day is again
**27 October 2019**.

---

## 14. The time shift of a recurrence update

When a time field is written with the scope "This and following events" or "All events", the change
is not applied to the addressed occurrence alone: it is translated into a shift and applied to the
occurrence that will act as the pattern for the rebuilt series.

Let *this* be the addressed occurrence and *base* the occurrence the series will be rebuilt from.

```formula
when a new start is supplied:
    shift of the start = new start − this start
    base start   = base start   + shift of the start
    base stop    = base stop    + shift of the start
    base start day = base start day + ( the day of the new start − the day of this start )
    base stop day  = base stop day  + ( the day of the new start − the day of this start )

when a new stop is supplied and no new start is supplied:
    shift of the stop = new stop − this stop
    base start   = base start   + shift of the stop
    base start day = base start day + ( the day of the new stop − the day of this stop )

when a new stop is supplied:
    shift of the stop = new stop − this stop
    base stop    = base stop    + shift of the stop
    base stop day = base stop day + ( the day of the new stop − the day of this stop )
```

The two shifts are computed independently, so supplying both a new start and a new stop can change
the length of the meeting.

**Worked example, both instants supplied.** A weekly series on Tuesdays holds three occurrences:
22 October 01:00 to 24 October 18:00, 29 October 01:00 to 31 October 18:00 and 5 November 01:00 to
7 November 18:00, all in 2019. The reader opens the second occurrence and writes, with the scope
"All events", a start four days later and a stop five days later, together with the weekday markers
changed from Tuesday to Saturday.

- Shift of the start = ( 2 November 01:00 ) − ( 29 October 01:00 ) = **+4 days**.
- Base start = 22 October 01:00 + 4 days = **26 October 01:00**.
- Base stop, first assignment = 24 October 18:00 + 4 days = 28 October 18:00.
- Shift of the stop = ( 5 November 18:00 ) − ( 31 October 18:00 ) = **+5 days**.
- Base stop, final assignment = 24 October 18:00 + 5 days = **29 October 18:00**.

The base occurrence therefore becomes 26 October 01:00 to 29 October 18:00, a length of 3 days and
17 hours. Rebuilding the weekly series on Saturdays with a count of 3 gives
**26 October 01:00 to 29 October 18:00**, **2 November 01:00 to 5 November 18:00** and
**9 November 01:00 to 12 November 18:00**.

**Worked example, only the stop supplied.** The same series; the reader opens the first occurrence
and writes, with the scope "All events", a stop one hour later. There is no new start, so the start
is shifted by the same amount: shift of the stop = +1 hour, base start = 22 October 01:00 + 1 hour =
**02:00**, base stop = 24 October 18:00 + 1 hour = **19:00**. The rebuilt series is
**22 October 02:00 to 24 October 19:00**, **29 October 02:00 to 31 October 19:00** and
**5 November 02:00 to 7 November 19:00**.

### 14.1 The count of the new series

```formula
count of the new series = the supplied count,
                          or, when none is supplied, the number of detached occurrences,
                          never less than 1
```

**Worked example.** A weekly series of three occurrences is trimmed at its second occurrence with
the scope "This and following events" and no new count. Two occurrences are detached, so the new
series has a count of **2**. Trimming at the first occurrence detaches three, so the new series has a
count of **3**.

---

## 15. Attendee counters

```formula
accepted count   = how many attendee records answer "Yes"
declined count   = how many attendee records answer "No"
tentative count  = how many attendee records answer "Maybe"
attendees count  = how many attendee contacts the event has
awaiting count   = attendees count − accepted count − declined count − tentative count
```

**Worked example.** An event has five attendee contacts and five attendee records: two accepted, one
declined, one tentative and one with no answer. The counters are accepted 2, declined 1, tentative 1,
attendees 5, and awaiting = 5 − 2 − 1 − 1 = **1**.

**Compatibility finding.** The waiting counter is derived from the number of attendee *contacts*
while the three answered counters are derived from the attendee *records*. The two are kept in step
by the rules of [business-rules.md](business-rules.md#5-attendees-and-invitations), so in ordinary
use they agree; a caller that writes attendee records directly without writing the attendee contacts
can nevertheless make the waiting counter negative. A corrected behaviour would count the attendee
records that answer "Needs Action" instead of subtracting.

---

## 16. Unavailability from overlapping meetings

The set of unavailable attendees is computed for a group of events at once, so that the search over
time is made once per contiguous stretch of time rather than once per event.

1. Group the events being computed into intervals: two events belong to the same interval when their
   time spans touch or overlap. Each interval carries its earliest start, its latest stop and the
   events inside it.
2. For each interval, fetch every event that has at least one of the interval's attendee contacts,
   whose availability marker is "Busy", whose stop is at or after the interval start and whose start
   is at or before the interval stop. Group those by attendee contact.
3. For each event in the interval and each of its attendee contacts, the contact is unavailable when
   at least one of that contact's fetched events, other than the event itself, overlaps it.

```formula
two spans overlap when  start of the first < stop of the second
                   and  start of the second < stop of the first
```

The comparison is strict at both ends, so a meeting that ends exactly when another begins does not
make its attendees unavailable.

**Worked example.** Contact A and contact B attend a meeting from 13 December 2020 at 17:00 to
22:00. No other meeting exists, so neither is unavailable. A second meeting is then created from
17:00 to 22:00 with contact A alone. For the first meeting, contact A now has another busy meeting
whose span 17:00–22:00 overlaps 17:00–22:00, so **contact A is unavailable** on both meetings;
contact B has no other meeting, so B stays available.

**Worked example of the strict comparison.** Contact A attends a meeting from 09:00 to 10:00 and
another from 10:00 to 11:00. The first span's stop equals the second span's start, so
10:00 < 10:00 is false and the two do **not** overlap: contact A is available on both.

---

## 17. Unavailability from working schedules

When the working-hours package is installed, the computation of chapter 16 is extended: an attendee
is also unavailable when the meeting falls outside their working schedule.

### 17.1 The interval of an event

```formula
company interval = the working intervals of the current company's working schedule,
                   over the whole span from the earliest start at 00:00:00
                   to the latest stop at 23:59:59, read as coordinated universal time

when the event is all-day:
    day interval = from the start day at 00:00:00 to the stop day at 23:59:59
    when at least one of the days from the start day to the stop day inclusive
         has no overlap with the company interval:
        event interval = empty
    otherwise:
        event interval = day interval intersected with the company interval

otherwise:
    event interval = from the start to the stop, read as coordinated universal time
```

An event whose interval is empty is skipped: nobody is declared unavailable for it.

**Worked example.** The company works Monday to Friday from 08:00 to 12:00 and from 13:00 to 16:00,
in a zone two hours ahead in July. An all-day event on Friday 12 July 2024 has an interval of
**08:00 to 12:00 and 13:00 to 16:00 local time**, that is seven hours. An all-day event on Saturday
13 July has no overlap with the company interval, so its interval is **empty** and the availability
of its attendees is not computed. An all-day event running from Friday 12 July to Monday 15 July
likewise has an empty interval, because Saturday 13 July and Sunday 14 July have no working hours.
A timed event from 08:30 to 09:30 in coordinated universal time keeps exactly that span.

### 17.2 The schedule of an attendee

1. Find the employees whose work contact is the attendee contact, within the companies the reader has
   selected.
2. For each employee, obtain the periods over which one working schedule applies; an employee with
   no working schedule of their own falls back to the current company's schedule.
3. For each schedule, compute the working intervals over the whole span, in the schedule's own time
   zone, for the employees' resources.
4. The employee's schedule is the union, over their periods, of the intersection of the period with
   the intervals of the schedule that applies to it.
5. The attendee contact's schedule is the union of the schedules of their employees.

### 17.3 The verdict

```formula
common interval = the attendee's schedule intersected with the event interval
the attendee is unavailable when
    the total length of the common interval ≠ the total length of the event interval
```

**Worked example.** The event interval is the seven hours of the Friday example above. An attendee
whose schedule is Tuesday to Friday, 08:00 to 12:00 and 13:00 to 16:00, has a common interval of
seven hours, which equals the event interval, so the attendee is **available**. An attendee whose
schedule is Monday to Friday from 15:00 to 22:00 has a common interval of 15:00 to 16:00, that is one
hour, which is not seven, so the attendee is **unavailable**.

### 17.4 The working hours shown behind the calendar

The screen asks for the working hours common to every selected attendee over a window.

1. Read the window from the first day at 00:00:00 to the last day at 23:59:59, in coordinated
   universal time.
2. Compute each attendee's schedule as in 17.2.
3. Intersect all of them.
4. Turn each remaining interval into an entry holding the weekday, the start time and the end time.

```formula
weekday number = ( the interval's start weekday counted from Monday as 0 , plus 1 ) modulo 7
start time     = the interval's start, read in the reader's time zone, as hours and minutes
end time       = the interval's stop,  read in the reader's time zone, as hours and minutes
```

When the intersection is empty, a single entry is returned holding the weekday number 7, a start
time of 00:00 and an end time of 00:00, which shades the whole week.

**Worked example.** Two employees share a schedule of Monday to Friday, 08:00 to 12:00 and 13:00 to
16:00, in a zone two hours ahead. The intersection over a week gives ten intervals. The Monday
morning interval starts on a Monday, whose number counted from Monday is 0, so its weekday number is
( 0 + 1 ) modulo 7 = **1**, with a start time of **08:00** and an end time of **12:00**. The Sunday
of a schedule that included Sundays would give ( 6 + 1 ) modulo 7 = **0**. Two employees on
disjoint schedules, one working 08:00–16:00 and the other 15:00–22:00, intersect to 15:00–16:00; two
employees on wholly disjoint schedules intersect to nothing, and the single shading entry is
returned.

---

## 18. The pattern sentence

The sentence stored on the series is rebuilt whenever the pattern text changes.

| Frequency | Termination | Sentence |
|---|---|---|
| daily | by count | "Every %(interval)s Days for %(count)s events" |
| daily | by end day | "Every %(interval)s Days until %(until)s" |
| daily | endless | "Every %(interval)s Days" |
| weekly | by count | "Every %(interval)s Weeks on %(days)s for %(count)s events" |
| weekly | by end day | "Every %(interval)s Weeks on %(days)s until %(until)s" |
| weekly | endless | "Every %(interval)s Weeks on %(days)s" |
| monthly by position | by count | "Every %(interval)s Months on the %(position)s %(weekday)s for %(count)s events" |
| monthly by position | by end day | "Every %(interval)s Months on the %(position)s %(weekday)s until %(until)s" |
| monthly by position | endless | "Every %(interval)s Months on the %(position)s %(weekday)s" |
| monthly by number | by count | "Every %(interval)s Months day %(day)s for %(count)s events" |
| monthly by number | by end day | "Every %(interval)s Months day %(day)s until %(until)s" |
| monthly by number | endless | "Every %(interval)s Months day %(day)s" |
| yearly | by count | "Every %(interval)s Years for %(count)s events" |
| yearly | by end day | "Every %(interval)s Years until %(until)s" |
| yearly | endless | "Every %(interval)s Years" |

The placeholders are the interval, the count, the end day, the day number, the position label and the
weekday label. The list of days for a weekly sentence is built from the weekday markers, in the order
Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday, joined with a comma and a space and
written out in full.

**Worked examples.** A daily series with an interval of 2 and a count of 3 reads
**"Every 2 Days for 3 events"**. The same series ending on 15 November 2024 reads
**"Every 2 Days until 2024-11-15"**. A weekly series with an interval of 2 on Tuesdays and
Wednesdays with a count of 3 reads **"Every 2 Weeks on Tuesday, Wednesday for 3 events"**. A monthly
series with an interval of 2 on the first Monday with a count of 3 reads
**"Every 2 Months on the First Monday for 3 events"**. A monthly series with an interval of 2 on the
twenty-seventh with a count of 3 reads **"Every 2 Months day 27 for 3 events"**. A yearly series with
an interval of 2 and no end reads **"Every 2 Years"**.

---

## 19. The identifier of an occurrence in the first external calendar service

The first external calendar service identifies an occurrence of a series by concatenating the
series' own identifier with the occurrence's original start.

```formula
when the series has no identifier: the occurrence has none

when the occurrence is all-day:
    time part = the start day written as four digits of year, two of month and two of day,
                with no separators
otherwise:
    time part = the start instant written as four digits of year, two of month, two of day,
                the letter T, two digits of hour, two of minute and two of second,
                with no separators, followed by the letter Z

occurrence identifier = the series identifier + "_" + time part
```

The derivation deliberately does not depend on the occurrence's current start: an occurrence that is
moved keeps the identifier it was created with, because the remote service also keys the occurrence
on its original start.

**Worked examples.** A series whose identifier is `abc123def456` with a timed occurrence starting
16 April 2024 at 09:30:00 in coordinated universal time yields **`abc123def456_20240416T093000Z`**.
The same series with an all-day occurrence on 16 April 2024 yields **`abc123def456_20240416`**.

---

## 20. Mapping reminders to and from the external services

### 20.1 Outward, first service

```formula
for each reminder of the event:
    method  = "email" when the channel is electronic mail, otherwise "popup"
    minutes = the reminder's lead time in minutes
```

The set is sent as explicit overrides, with the service's own defaults switched off.

**Worked example.** An event carries a three-hour electronic-mail reminder and a fifteen-minute
in-application reminder. The outward set is **method "email" with 180 minutes** and **method "popup"
with 15 minutes**, and the default-reminder flag is false.

### 20.2 Inward, first service

For each remote reminder:

1. The channel is electronic mail when the remote method is `email`, and in-application otherwise.
2. Look for an existing reminder with that channel and exactly that number of minutes; when one
   exists, attach it.
3. Otherwise create one. The unit is days when the minutes divide by 1440 exactly, hours when they
   divide by 60 exactly, and minutes otherwise. The duration is the minutes divided by the
   corresponding multiplier, and the name is the channel label, then " - ", then that duration, then
   a space and the unit word.

```formula
when minutes modulo 1440 = 0:  unit = days,    duration = minutes ÷ 1440
when minutes modulo 60   = 0:  unit = hours,   duration = minutes ÷ 60
otherwise:                     unit = minutes, duration = minutes
```

**Worked examples.** 120 minutes with the method `popup` gives the unit hours and a duration of
120 ÷ 60 = 2, named **"Notification - 2.0 Hours"**; the decimal point appears because the division
produces a decimal value, while the stored duration is the whole number 2. 2880 minutes with the
method `email` gives the unit days and a duration of 2880 ÷ 1440 = 2, named **"Email - 2.0 Days"**.
45 minutes gives the unit minutes and a duration of 45, named **"Notification - 45 Minutes"**, with
no decimal point because no division took place.

When the remote event carries no explicit reminders at all, the service's own default reminders are
used instead, but only when the remote event says it wants them.

### 20.3 Outward, second service

Only in-application reminders cross to the second service, and only one of them.

```formula
the first in-application reminder of the event, taken in the event's own reminder order
reminder switch  = true when such a reminder exists, false otherwise
minutes before   = that reminder's lead time in minutes, or 0 when there is none
```

**Worked example.** An event carries a six-hour electronic-mail reminder and a thirty-minute
in-application reminder. Outward, the reminder switch is **true** and the minutes before the start
are **30**. The electronic-mail reminder is not sent at all.

### 20.4 Inward, second service

```formula
when the remote reminder switch is on:
    minutes = the remote minutes before the start, or 0
    look for an existing in-application reminder whose lead time equals those minutes
    when one exists and the event does not already carry it: attach it
    when none exists: create one, named as below, and attach it
    detach every other in-application reminder of the event

when the remote reminder switch is off:
    detach every in-application reminder of the event
```

The created reminder's name is "Notification - At time of event" when the minutes are zero;
otherwise it follows the same unit rule as 20.2, giving "Notification - 2.0 Days",
"Notification - 2.0 Hours" or "Notification - 45 Minutes".

**Worked example.** A remote event switches its reminder on with 15 minutes before the start. The
shipped fifteen-minute in-application reminder exists, so it is attached and every other
in-application reminder of the event is detached. A remote event that switches its reminder off
causes every in-application reminder to be detached, leaving the electronic-mail reminders in place.

---

## 21. Rounding summary

| Quantity | Precision | Method | Where |
|---|---|---|---|
| Length of a meeting when the stop is derived | whole minutes | nearest, halves to the nearer even minute | [3.1](#31-stop-from-start-and-duration) |
| Duration in hours when it is derived | two decimals | nearest, halves away from zero | [3.2](#32-duration-from-start-and-stop) |
| End of the sentence in the display time | whole minutes | nearest, halves to the nearer even minute | [5](#5-the-display-time) |
| Lead time in minutes | whole minutes | exact, no rounding | [6](#6-the-reminder-lead-time-in-minutes) |
| Position of a weekday inside a month | whole number | rounded up, then 4 and 5 mapped to −1 | [4.4](#44-the-position-of-a-day-inside-its-month) |
| Weeks in an endless weekly horizon | whole number | fractional part discarded | [11](#11-the-occurrence-horizon-and-its-caps) |
| Occurrences in an endless horizon | whole number | fractional part discarded, then capped at 720 | [11](#11-the-occurrence-horizon-and-its-caps) |
| Duration of a reminder created from a remote one | whole number in the stored field, decimal in the name | division, no rounding | [20.2](#202-inward-first-service) |
| Working-hours entries | whole minutes | the clock reading, no rounding | [17.4](#174-the-working-hours-shown-behind-the-calendar) |
