# Work Entries — Configuration

Every setting, shipped record, access group, access right, record rule, scheduled job and operational
parameter of the domain, with its default and its effect, together with the master data another domain
must provide before this one can run.

---

## 1. Capability packages

The domain is delivered as one core package and two companion packages. A rebuild may implement the
core alone; each companion adds behaviour only when the domain it bridges to is also present.

| Capability package | Depends on | What it adds |
|---|---|---|
| Work Entries, the core | Human Resources Core (and through it Attendances and Working Time) | The Work Entry, the Work Entry Type, the calendar filter, the regeneration wizard, the whole generation engine on the Employee Version, the work entry kind on a working schedule line and on a working time exclusion, the conflict engine, the calendar view and the daily job |
| Time Off in Payslips | Time Off, Work Entries | The work entry kind on a Time Off Type, the absence link and the absence-state mirror on a Work Entry, the generation of absence rows when a request is validated, the archiving of the rows an absence swallows, the regeneration when a request is refused or cancelled, the conflict re-check window around every change to a request, the guard that stops a request with a validated row being cancelled, the precedence ladder that puts a company closure above an individual absence, and the per-kind absence-hours total |
| Work Entries for France | Time Off for France, Time Off in Payslips | The gap filling of [calculations.md, chapter 13](calculations.md#13-the-gap-filling-rule-for-french-part-time-absences) and the exemption of French part-time employees from the outside-schedule conflict test |

The second package installs itself automatically whenever both the domains it bridges are present; the
third installs itself automatically whenever the second is present and the French absence package is
too. On installation the second package re-runs the four conflict conditions over **every** existing
work entry, so that entries created before the bridge existed are re-examined against the absence
rules.

---

## 2. Master data prerequisites

| Prerequisite | Owning domain | Why it is needed |
|---|---|---|
| At least one Company carrying a country | [Contacts and organizations](../contacts-and-organizations/README.md) | The country filters the catalogue of kinds through rules [`WKE-007`](business-rules.md#2-the-work-entry-type-catalogue), [`WKE-016`](business-rules.md#4-the-required-fields-and-the-defaults-of-a-work-entry) and [`WKE-056`](business-rules.md#11-permission-checks) |
| A company working schedule carrying a time zone | [Attendances and working time](../attendances-and-working-time/README.md) | The last fallback of the zone resolution, and the source of the calendar's unusual days |
| At least one Working Schedule with lines and a time zone | Attendances and working time | Without it nothing can be expanded and nothing is generated |
| Employees with a resource and at least one Employee Version | [Human resources core](../human-resources-core/README.md) | A work entry requires both an employee and a version |
| A contract start date on every version that should generate | Human resources core | A version without one is skipped silently |
| Work entry kinds on the working schedule lines | This domain | Defaulted to the shipped ordinary-attendance kind |
| Work entry kinds on the working time exclusions that represent public holidays | This domain, populated by an administrator | Without one the closure produces a generic-absence row |
| Work entry kinds on the Time Off Types | [Time off](../time-off/README.md), populated by an administrator | Without one the absence produces a generic-absence row |
| Login users carrying the human resources groups | [Identity and access](../identity-and-access/README.md) | Determines who may see, write and regenerate |

---

## 3. The shipped catalogue of work entry kinds

One hundred and eighteen work entry kinds are shipped. All of them are defined by the core package,
deliberately: a test refuses an installation in which a work entry kind is defined anywhere else, so
that the catalogue has exactly one home. All of them are loaded with the no-update marker, so an
administrator's later edits survive an upgrade of the package.

### 3.1 The universal kinds

These nine are present in every installation and carry no country, so every company may use them.

| Name | Payroll code | Display code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|---|
| Attendance | `WORK100` | `A` | 0 | 1.0 | no | no | 25 |
| Overtime Hours | `OVERTIME` | `OT` | 4 | 1.0 | no | yes | 25 |
| Out of Contract | `OUT` | `OoC` | 0 | 1.0 | no | no | 25 |
| Generic Time Off | `LEAVE100` | `GTO` | 3 | 1.0 | yes | no | 25 |
| Compensatory Time Off | `LEAVE105` | `CTO` | 3 | 1.0 | yes | no | 25 |
| Home Working | `WORK110` | `HW` | 2 | 1.0 | yes | no | 25 |
| Unpaid | `LEAVE90` | `UN` | 5 | 1.0 | yes | no | 25 |
| Sick Time Off | `LEAVE110` | `STO` | 5 | 1.0 | yes | no | 25 |
| Paid Time Off | `LEAVE120` | `PTO` | 5 | 1.0 | yes | no | 25 |

Four of the nine carry meaning for the engine itself:

- **Attendance**, payroll code `WORK100`, is the default of every working schedule line and the
  fallback kind of every attendance interval whose line names none. Its country may never be changed
  (rule [`WKE-004`](business-rules.md#2-the-work-entry-type-catalogue)).
- **Generic Time Off**, payroll code `LEAVE100`, is the fallback kind of every absence interval for
  which the precedence ladder finds nothing.
- **Sick Time Off**, payroll code `LEAVE110`, is one of the three codes that are not displaced by a
  public holiday (rule [`WKE-046`](business-rules.md#9-interaction-with-absence-requests)).
- **Out of Contract**, payroll code `OUT`, labels time inside a payroll period during which the
  employee had no contract. This domain ships the kind; only a payroll capability produces entries of
  it.

Note that **Home Working**, payroll code `WORK110`, carries the absence flag even though its name
describes worked time. That is the shipped value, and it is deliberate: home working is measured
against the theoretical schedule rather than by its own clock span, and a home-working schedule line
is excluded from the schedule's weekly hours.

### 3.2 The country kinds

One hundred and nine further kinds are shipped, each carrying a country, and each therefore visible
only to a company of that country (rule [`WKE-056`](business-rules.md#11-permission-checks)). None of
them carries a display code. They are listed here in full, because a rebuild that serves any of these
countries must reproduce their payroll codes character for character: statutory exports key on them.

#### Australia

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Paid Time Off | `AU.PT` | 5 | 1.0 | no | no | 25 |
| Long Service Leave | `AU.LS` | 5 | 1.0 | no | no | 25 |
| Personal Leave | `AU.PL` | 5 | 1.0 | no | no | 25 |
| Other Paid Leave | `AU.O` | 5 | 1.0 | no | no | 25 |
| Paid Parental Leave | `AU.P` | 5 | 1.0 | no | no | 25 |
| Workers Compensation | `AU.W` | 5 | 1.0 | no | no | 25 |
| Ancillary and Defence Leave | `AU.A` | 5 | 1.0 | no | no | 25 |
| Cash Out of Leave in Service | `AU.C` | 5 | 1.0 | no | no | 25 |
| Unused Leave on Termination | `AU.U` | 5 | 1.0 | no | no | 25 |
| Overtime: Regular | `l10n_au_overtime_regular` | 4 | 1.0 | no | no | 25 |
| Overtime: Saturday | `l10n_au_overtime_saturday` | 4 | 1.0 | no | no | 25 |
| Overtime: Sunday | `l10n_au_overtime_sunday` | 4 | 1.0 | no | no | 25 |
| Overtime: Public Time Off | `l10n_au_overtime_pto` | 4 | 1.0 | no | no | 25 |
| Overtime: Saturday & Public Time Off | `l10n_au_overtime_saturday_pto` | 4 | 1.0 | no | no | 25 |
| Overtime: Sunday & Public Time Off | `l10n_au_overtime_sunday_pto` | 4 | 1.0 | no | no | 25 |

#### Belgium

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Public Holiday | `LEAVE500` | 3 | 1.0 | no | no | 25 |
| Solicitation Time Off | `LEAVE600` | 0 | 1.0 | no | no | 25 |
| Unjustified Reason | `LEAVE700` | 0 | 1.0 | no | no | 25 |
| Small Unemployment (Brief Holiday) | `LEAVE205` | 5 | 1.0 | no | no | 25 |
| Economic Unemployment | `LEAVE6665` | 5 | 1.0 | no | no | 25 |
| Corona Unemployment | `LEAVE6666` | 5 | 1.0 | no | no | 25 |
| Maternity Time Off | `LEAVE210` | 5 | 1.0 | no | no | 25 |
| Paternity Time Off (Paid by Company) | `LEAVE220` | 5 | 1.0 | no | no | 25 |
| Paternity Time Off (Legal) | `LEAVE230` | 5 | 1.0 | no | no | 25 |
| Unpredictable Reason | `LEAVE250` | 5 | 1.0 | no | no | 25 |
| Training | `LEAVE265` | 5 | 1.0 | no | no | 25 |
| Educational Time Off | `LEAVE260` | 5 | 1.0 | no | no | 25 |
| Flemish Educational Time Off | `LEAVE261` | 5 | 1.0 | no | no | 25 |
| Long Term Sick | `LEAVE280` | 5 | 1.0 | no | no | 25 |
| Breastfeeding Break | `LEAVE290` | 5 | 1.0 | no | no | 25 |
| Medical Assistance | `MEDIC01` | 5 | 1.0 | no | no | 25 |
| Youth Time Off | `YOUNG01` | 5 | 1.0 | no | no | 25 |
| Recovery Additional Time | `LEAVE295` | 5 | 1.0 | no | no | 25 |
| Additional Time (Paid) | `LEAVE297` | 5 | 1.0 | no | no | 25 |
| Notice (Unprovided) | `LEAVE211` | 5 | 1.0 | no | no | 25 |
| Public Holiday Compensation | `PHC1` | 3 | 1.0 | no | no | 25 |
| Extra Legal Time Off | `LEAVE213` | 5 | 1.0 | no | no | 25 |
| Sick Time Off (Without Guaranteed Salary) | `LEAVE214` | 5 | 1.0 | no | no | 25 |
| Recovery Bank Holiday | `LEAVE215` | 5 | 1.0 | no | no | 25 |
| European Time Off | `LEAVE216` | 5 | 1.0 | no | no | 25 |
| Credit Time | `LEAVE300` | 8 | 1.0 | no | no | 25 |
| Parental Time Off | `LEAVE301` | 8 | 1.0 | no | no | 25 |
| Simple Holiday Pay - Variable Salary | `LEAVE1731` | 3 | 1.0 | no | no | 25 |
| Work Accident | `LEAVE115` | 5 | 1.0 | no | no | 25 |
| Partial Incapacity (due to illness) | `LEAVE281` | 4 | 1.0 | no | no | 25 |
| Strike | `LEAVE251` | 5 | 1.0 | no | no | 25 |
| Brief Holiday (Birth) | `LEAVE206` | 5 | 1.0 | no | no | 25 |

#### Egypt

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Paid Sick Time off | `EGSICKLEAVE100` | 0 | 1.0 | no | no | 25 |
| Sick Time off (75% Paid) | `EGSICKLEAVE75` | 0 | 1.0 | no | no | 25 |
| Sick Time off (Unpaid) | `EGSICKLEAVE0` | 0 | 1.0 | no | no | 25 |

#### Hong Kong

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Sick Leave 80% | `HKLEAVE111` | 6 | 0.8 | no | no | 25 |
| Compassionate Leave | `HKLEAVE130` | 5 | 1.0 | no | no | 25 |
| Marriage Leave | `HKLEAVE140` | 5 | 1.0 | no | no | 25 |
| Examination Leave | `HKLEAVE150` | 5 | 1.0 | no | no | 25 |
| Maternity Leave | `HKLEAVE210` | 5 | 1.0 | no | no | 25 |
| Maternity Leave 80% | `HKLEAVE211` | 6 | 0.8 | no | no | 25 |
| Paternity Leave | `HKLEAVE220` | 5 | 1.0 | no | no | 25 |
| Paternity Leave 80% | `HKLEAVE221` | 6 | 0.8 | no | no | 25 |
| Statutory Holiday | `HKLEAVE500` | 5 | 1.0 | no | no | 25 |
| Public Holiday | `HKLEAVE510` | 5 | 1.0 | no | no | 25 |
| Weekend | `HKLEAVE600` | 10 | 1.0 | no | no | 25 |

#### Indonesia

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Bereavement Leave | `IDLEAVE100` | 5 | 1.0 | no | no | 25 |
| Marriage Leave | `IDLEAVE110` | 5 | 1.0 | no | no | 25 |
| Maternity Leave | `IDLEAVE120` | 5 | 1.0 | no | no | 25 |
| Paternity Leave | `IDLEAVE130` | 5 | 1.0 | no | no | 25 |
| Public Holiday | `IDLEAVE140` | 5 | 1.0 | no | no | 25 |

#### Jordan

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Sick Leave (Unpaid) | `SICKLEAVE0` | 7 | 1 | no | no | 25 |

#### Luxembourg

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Situational Unemployment | `LEAVE400` | 5 | 1.0 | no | no | 25 |

#### Mexico

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Work risk (IMSS) | `LEAVE1000` | 5 | 1.0 | no | no | 25 |
| Maternity (IMSS) | `LEAVE1100` | 5 | 1.0 | no | no | 25 |
| Disability due to illness (IMSS) | `LEAVE1200` | 5 | 1.0 | no | no | 25 |
| Medical Leave for Care of Children Diagnosed with Cancer (IMSS) | `LEAVE1300` | 5 | 1.0 | no | no | 25 |

#### Poland

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Sick Time Off (Paid at 80%) | `PLSICK3166` | 4 | 0.8 | no | no | 25 |

#### Saudi Arabia

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Sick Leave (100% Paid) | `SASICKLEAVE100` | 0 | 1.0 | no | no | 25 |
| Sick Leave (75% Paid) | `SASICKLEAVE75` | 0 | 0.75 | no | no | 25 |
| Sick Leave (Unpaid) | `SASICKLEAVE0` | 0 | 0 | no | no | 25 |
| Maternity Leave | `SAMATERNITY` | 0 | 1.0 | no | no | 25 |
| Emergency Leave | `SAEMERGENCY` | 0 | 1.0 | no | no | 25 |

#### Slovakia

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Sick Time Off Day 1-3 (Paid at 25%) | `SICK25` | 5 | 0.25 | no | no | 25 |
| Sick Time Off Day 4-10 (Paid at 55%) | `SICK55` | 5 | 0.55 | no | no | 25 |
| Sick Time Off Day up to 52 weeks (Paid at 55% by social security) | `SICK0` | 5 | 0.0 | no | no | 25 |
| Maternity Time Off (Paid at 75%) | `MATERNITY` | 5 | 0.75 | no | no | 25 |
| Parental Time Off | `PARENTAL` | 8 | 0.0 | no | no | 25 |

#### Switzerland

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Public Holiday | `CHPUBHOL` | 3 | 1.0 | no | no | 25 |
| Monthly Salary | `CH_1000` | 5 | 1.0 | no | no | 0 |
| Hourly Salary | `CH_1005` | 5 | 1.0 | no | no | 5 |
| Lesson Salary | `CH_1006` | 5 | 1.0 | no | no | 10 |
| Overtime 100% | `CH_1065` | 5 | 1.0 | no | no | 6 |
| Overtime 125% | `CH_1061` | 5 | 1.0 | no | no | 7 |
| Unpaid Salary | `CH_UNPAID` | 5 | 1.0 | no | no | 15 |
| Salary in case of Illness | `CH_ILLNESS` | 5 | 1.0 | no | no | 20 |
| Salary in case of Accident | `CH_ACCIDENT` | 5 | 1.0 | no | no | 25 |
| Salary in case of Maternity / Paternity Leave | `CH_MATERNITY` | 5 | 1.0 | no | no | 30 |
| Salary in case of Military Leave | `CH_MILITARY` | 5 | 1.0 | no | no | 35 |
| Hourly Salary in case of Illness | `CH_ILLNESS_HOURLY` | 5 | 1.0 | no | no | 20 |
| Hourly Salary in case of Accident | `CH_ACCIDENT_HOURLY` | 5 | 1.0 | no | no | 25 |
| Hourly Salary in case of Maternity / Paternity Leave | `CH_MATERNITY_HOURLY` | 5 | 1.0 | no | no | 30 |
| Hourly Salary in case of Military Leave | `CH_MILITARY_HOURLY` | 5 | 1.0 | no | no | 35 |
| Interruption of Work | `CH_Interruption` | 5 | 1.0 | no | no | 40 |

#### United Arab Emirates

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Sick Leave 50 | `AESICKLEAVE50` | 0 | 1.0 | no | no | 25 |
| Sick Leave 1 | `AESICKLEAVE0` | 0 | 1.0 | no | no | 25 |
| Public Holiday | `AEPUBLICH` | 0 | 1.0 | no | no | 25 |
| Overtime Weekdays Daytime | `OVTWD` | 0 | 1.25 | no | yes | 25 |
| Overtime Weekdays Nighttime | `OVTWDN` | 0 | 1.5 | no | yes | 25 |
| Overtime Off-days | `OVTOD` | 0 | 1.5 | no | yes | 25 |

#### United States

| Name | Payroll code | Colour | Pay rate | Absence | Added to monthly pay | Order |
|---|---|---|---|---|---|---|
| Overtime Hours (Paid at 150%) | `USOVERTIME150` | 4 | 1.5 | no | no | 25 |
| Double Time Hours | `USDOUBLE` | 4 | 2.0 | no | no | 25 |
| Retro Overtime Hours | `USRETROOVERTIME` | 4 | 1.5 | no | no | 25 |
| Retro Regular Pay Hours | `USRETROREGULAR` | 4 | 1.0 | no | no | 25 |

### 3.3 The demonstration kinds

Two further kinds are loaded only with demonstration data and must not be relied on by a rebuild:
"Extra Hours", payroll code `WORK300`, colour 8; and "Long Term Time Off", payroll code `LEAVE200`,
colour 4. Neither carries a display code, a country or the absence flag.

### 3.4 What a rebuild must reproduce

| Obligation | Why |
|---|---|
| The payroll codes, character for character | Statutory exports, country payroll rules and the three codes of rule [`WKE-046`](business-rules.md#9-interaction-with-absence-requests) key on them |
| The absence flag of each kind | It changes how the kind's hours are measured and whether its schedule lines count towards the weekly hours |
| The pay rate of each kind | It is copied onto every entry created with the kind and is read by payroll |
| The country of each kind | It decides who can see and choose the kind |
| The stable reference to the ordinary-attendance kind and to the generic-absence kind | The generation engine resolves both by reference, and produces unlabelled or generic rows if either is missing |
| The display codes | Only cosmetic; a rebuild may choose its own |
| The colours | Only cosmetic |
| The order values | Only cosmetic; all but the Swiss kinds use the default of twenty-five |

---

## 4. Shipped links from an absence kind to a work entry kind

The bridging package ships the links below, so that an installation that has both domains produces
correctly labelled absence rows without any configuration. Each row means: an absence of this kind
produces work entries of that kind. The absence kinds themselves are owned by
[Time Off](../time-off/configuration.md).

**Universal**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Compensatory Days | Compensatory Time Off | `LEAVE105` |
| Unpaid | Unpaid | `LEAVE90` |
| Sick Time Off | Sick Time Off | `LEAVE110` |
| Paid Time Off | Paid Time Off | `LEAVE120` |

**United Arab Emirates**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Sick Leave 50% | Sick Leave 50 | `AESICKLEAVE50` |
| Sick Leave 0% | Sick Leave 1 | `AESICKLEAVE0` |

**Belgium**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Small Unemployment | Small Unemployment (Brief Holiday) | `LEAVE205` |
| Brief Holiday (Birth) | Brief Holiday (Birth) | `LEAVE206` |
| Maternity Time Off | Maternity Time Off | `LEAVE210` |
| Unpredictable Reason | Unpredictable Reason | `LEAVE250` |
| Training Time Off | Educational Time Off | `LEAVE260` |
| Extra Legal Time Off | Extra Legal Time Off | `LEAVE213` |
| Recovery Bank Holiday | Recovery Bank Holiday | `LEAVE215` |
| European Time Off | European Time Off | `LEAVE216` |
| Credit Time | Credit Time | `LEAVE300` |
| Work Accident Time Off | Work Accident | `LEAVE115` |
| Strike | Strike | `LEAVE251` |
| Sick Leave Without Certificate | Sick Time Off | `LEAVE110` |

**Switzerland**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Unpaid leave | Unpaid Salary | `CH_UNPAID` |
| Illness leave | Salary in case of Illness | `CH_ILLNESS` |
| Accident leave | Salary in case of Accident | `CH_ACCIDENT` |
| Maternity / Paternity leave | Salary in case of Maternity / Paternity Leave | `CH_MATERNITY` |
| Military leave | Salary in case of Military Leave | `CH_MILITARY` |
| Interruption of Work | Interruption of Work | `CH_Interruption` |

**Egypt**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Paid Sick time off | Paid Sick Time off | `EGSICKLEAVE100` |
| Sick Leave (75% Paid) | Sick Time off (75% Paid) | `EGSICKLEAVE75` |
| Sick Leave (UnPaid) | Sick Time off (Unpaid) | `EGSICKLEAVE0` |

**Hong Kong**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Annual Leaves | Paid Time Off | `LEAVE120` |
| Compensation Leaves | Compensatory Time Off | `LEAVE105` |
| Sick Leaves | Sick Time Off | `LEAVE110` |
| Sick Leaves 80% | Sick Leave 80% | `HKLEAVE111` |
| Unpaid Leaves | Unpaid | `LEAVE90` |
| Marriage Leaves | Marriage Leave | `HKLEAVE140` |
| Maternity Leaves | Maternity Leave | `HKLEAVE210` |
| Maternity Leaves 80% | Maternity Leave 80% | `HKLEAVE211` |
| Paternity Leaves | Paternity Leave | `HKLEAVE220` |
| Paternity Leaves 80% | Paternity Leave 80% | `HKLEAVE221` |
| Compassionate Leaves | Compassionate Leave | `HKLEAVE130` |
| Examination Leaves | Examination Leave | `HKLEAVE150` |

**Indonesia**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Annual Leaves | Paid Time Off | `LEAVE120` |
| Sick Leaves | Sick Time Off | `LEAVE110` |
| Unpaid Leaves | Unpaid | `LEAVE90` |
| Marriage Leaves | Marriage Leave | `IDLEAVE110` |
| Maternity Leaves | Maternity Leave | `IDLEAVE120` |
| Paternity Leaves | Paternity Leave | `IDLEAVE130` |
| Bereavement Leaves | Bereavement Leave | `IDLEAVE100` |

**Jordan**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Sick Leave (Unpaid) | Sick Leave (Unpaid) | `SICKLEAVE0` |

**Luxembourg**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Unemployment (Weather / Situational) | Situational Unemployment | `LEAVE400` |

**Mexico**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Work risk (IMSS) | Work risk (IMSS) | `LEAVE1000` |
| Maternity (IMSS) | Maternity (IMSS) | `LEAVE1100` |
| Disability due to illness (IMSS) | Disability due to illness (IMSS) | `LEAVE1200` |

**Poland**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Sick Leaves 80% | Sick Time Off (Paid at 80%) | `PLSICK3166` |

**Saudi Arabia**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Sick Leave (100% Paid) | Sick Leave (100% Paid) | `SASICKLEAVE100` |
| Sick Leave (75% Paid) | Sick Leave (75% Paid) | `SASICKLEAVE75` |
| Sick Leave (UnPaid) | Sick Leave (Unpaid) | `SASICKLEAVE0` |
| Maternity Leave | Maternity Leave | `SAMATERNITY` |
| Emergency Leave | Emergency Leave | `SAEMERGENCY` |

**Slovakia**

| Absence kind | Work entry kind produced | Payroll code |
|---|---|---|
| Maternity Time Off | Maternity Time Off (Paid at 75%) | `MATERNITY` |

Four of these links are loaded with the no-update marker and are not created if the absence kind is
absent — the universal four. The rest are written unconditionally when the corresponding country's
absence kinds exist.

An absence kind with **no** work entry kind still produces work entries; the precedence ladder simply
falls through to the shipped generic-absence kind, payroll code `LEAVE100`.

---

## 5. Settings and parameters

The domain introduces **no** company setting, **no** system parameter, **no** numbering sequence, **no**
message template and **no** activity type. Everything configurable lives on records:

| Where | Field | Default | Effect |
|---|---|---|---|
| Employee Version | Generation source | `calendar` "Working Schedule" | Chooses where the day book comes from; see rule [`WKE-060`](business-rules.md#12-the-generation-source-extension-point) |
| Employee Version | Generated From, Generated To | today at midnight, both equal | The coverage markers; their equality is the sentinel meaning nothing has been generated |
| Employee Version | Last Generation Date | empty | Read by the daily job so that one version is not revisited twice in a day |
| Employee (mirror) | Generation source | the current version's value | Writing it writes the current version's |
| Working Schedule Line | Work entry kind | the shipped ordinary-attendance kind | The kind of the rows the line produces; a kind carrying the absence flag also removes the line from the schedule's weekly hours |
| Working Time Exclusion | Work entry kind | empty | The kind of the rows the closure produces; empty falls through to the shipped generic-absence kind |
| Time Off Type | Work entry kind | empty | The kind of the rows an absence of that kind produces; empty falls through to the shipped generic-absence kind |
| Work Entry Type | Pay rate | 1.0 | Copied onto every entry created with the kind |
| Work Entry Type | Order | 25 | Presentation order only; editable only by the technical group |
| Work Entry | Duration | 8 | The default of a hand-created entry |
| Work Entry | Kind | the first kind in identifier order | A weak default so that a manually opened form is not empty |
| Calendar filter | Ticked | true | Whether a pinned employee's entries are shown |

Two behavioural markers may be placed on the operating context by a caller and are part of the
contract:

| Marker | Effect |
|---|---|
| Suppress the conflict check | The four conditions do not run for the duration of the call; set by the reset pass itself and available to bulk loaders. Rule [`WKE-027`](business-rules.md#5-the-four-conflict-conditions) |
| Skip the regeneration guards | The three regeneration guards do not run; set by the regeneration triggered by a version change. Rule [`WKE-051`](business-rules.md#10-regeneration-guards) |
| Salary simulation | Suppresses both the out-of-period removal and the regeneration after a version write, so a simulation never touches the real day book |

---

## 6. Security

### 6.1 Access groups used

The domain defines no group of its own. It uses four groups owned by other domains:

| Group | Owned by | Role here |
|---|---|---|
| Human Resources Officer | [Human resources core](../human-resources-core/configuration.md) | The everyday user of the day book |
| Human Resources Manager | Human resources core | Configures kinds, sets the generation source, regenerates |
| Settings | [Identity and access](../identity-and-access/configuration.md) | The only group that may delete a work entry |
| Internal User | Identity and access | Scope of the calendar-filter record rule |

### 6.2 The access matrix

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Work Entry | Human Resources Officer | yes | yes | yes | no |
| Work Entry | Settings | yes | yes | yes | yes |
| Work Entry Type | Human Resources Officer | yes | no | no | no |
| Work Entry Type | Human Resources Manager | yes | yes | yes | yes |
| Work Entry Employee Filter | Human Resources Officer | yes | yes | yes | yes |
| Work Entry Regeneration Wizard | Human Resources Manager | yes | yes | yes | yes |

No group other than those listed has any access at all. In particular an ordinary internal user
cannot read a work entry, and an employee cannot see their own.

### 6.3 Record rules

| Rule name, reproduced | Entity | Applies to | Operations | Condition |
|---|---|---|---|---|
| "HR Work Entry Contract: Multi Company" | Work Entry | every group | read, write, create, delete | the entry's company is one of the acting companies |
| "HR Work Entry: Multi Company" | Work Entry Type | every group | read, write, create, delete | the kind's country is one of the countries of the acting companies, or is empty |
| "Work entries/Employee calendar filter: only self" | Work Entry Employee Filter | Internal User | write, create, delete — **not** read | the row's user is the acting user |

The abbreviations inside the three quoted rule names are part of the stored strings and are
deliberately not expanded.

The read operation is deliberately left out of the third rule because the calendar's grouping query
reads the whole table; leaving read restricted would make the calendar unable to count how many users
have pinned an employee.

### 6.4 Field-level restrictions

| Field | Entity | Visible to |
|---|---|---|
| Generated From, Generated To, Last Generation Date | Employee Version | Human Resources Officer and above |
| Generation source and the invalid-source indicator | Employee Version | Human Resources Manager only |
| Generation source and the invalid-source indicator | Employee | Human Resources Manager only |
| Has work entries | Employee | Settings group and Human Resources Officer |
| Work entry kind | Working Schedule Line | Human Resources Officer and above |
| Work entry kind | Working Time Exclusion | Human Resources Officer and above |
| Order, on a work entry kind | Work Entry Type | the technical group only |

### 6.5 Elevated execution

Six operations run with elevated rights, deliberately:

1. Generation, per group of versions, also switching the acting company to the group's company.
2. The existence query of rule [`WKE-005`](business-rules.md#2-the-work-entry-type-catalogue), so that
   an invisible work entry still blocks a country change.
3. The reset pass of rule [`WKE-025`](business-rules.md#5-the-four-conflict-conditions).
4. The deletion of non-validated entries when a version is removed.
5. The archiving and regeneration that follow the refusal or cancellation of an absence request.
6. The refuse shortcut offered on a work entry linked to an absence. The approve shortcut does **not**
   run elevated: approving must remain subject to the approver's own rights.

---

## 7. Scheduled jobs

One scheduled job, and it is the backbone of the domain.

| Property | Value |
|---|---|
| Name, reproduced | "Generate Missing Work Entries" |
| Subject | the Employee Version |
| Runs as | the platform's root user |
| Interval | every one day |
| Active on installation | yes |
| Loaded with the no-update marker | yes, so an administrator's changes to the schedule survive |
| Batch size | one hundred versions |
| Re-triggers itself | yes, when more than one batch is outstanding |

Its procedure is in [workflows.md, chapter 4](workflows.md#4-generating-a-month-automatically) and its
period arithmetic in [calculations.md, chapter 16](calculations.md#16-the-window-arithmetic-of-the-daily-job).
Three properties of it are worth restating here because they are configuration decisions a rebuild
must make consciously:

- **It looks two months wide**, from the first day of the current month to the last day of the next.
  A rebuild that looks only at the current month leaves the first days of a new month ungenerated
  until the month turns.
- **It processes one company per run.** Versions of other companies are left for a later run. This is
  not an optimisation; it is what keeps one company's closures out of another company's day book.
- **It orders statically generated versions first**, because several versions sharing one schedule can
  be expanded together, which a variable source cannot.

The job is also the only place in the domain where the language of the run is set explicitly, to the
language of its own user, so that the descriptions it writes onto generated entries are in one
predictable language rather than in whatever language the last interactive request used.

---

## 8. What a rebuild must seed

| Seed | Content |
|---|---|
| The catalogue of kinds | At minimum the nine universal kinds of [3.1](#31-the-universal-kinds), with their payroll codes and absence flags exactly as shipped |
| Two stable references | One to the ordinary-attendance kind and one to the generic-absence kind, resolvable by the engine |
| The daily job | One job as described in [chapter 7](#7-scheduled-jobs) |
| The three record rules | As described in [6.3](#63-record-rules) |
| The six access-matrix lines | As described in [6.2](#62-the-access-matrix) |
| The composite partial index | On the pair of the version and the date, restricted to rows whose state is `draft` or `validated`; see [entities.md, section 4.4](entities.md#44-indexes) |
| The uniqueness constraint | On the pair of the user and the employee of a calendar filter row, with the message of rule [`WKE-062`](business-rules.md#13-everything-else) |
| The links from absence kinds | The four universal ones of [chapter 4](#4-shipped-links-from-an-absence-kind-to-a-work-entry-kind), plus those of every country the rebuild serves |

Nothing else is seeded. There is no default work entry, no default calendar filter and no default
regeneration wizard.
