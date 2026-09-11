# Entities

Every entity this folder owns is specified below: its purpose, its lifecycle, its complete field
table, its relations, its uniqueness constraints, its ordering, its display-name rule, its archival
behaviour, its company behaviour and the extension points other packages contribute to it.

Reproduced identifiers are written in code font. Each field table carries the identifier, the full
name in words, the type, the target where the field points at another record, whether the field is
required, its default, its computed rule with the values that rule depends on, whether the computed
value is stored, and its meaning.

A generated reference page exists for each entity; it is linked from the entity's heading.

Contents:

1. [Automation Rule](#1-automation-rule-baseautomation)
2. [Server Action as used by automations](#2-server-action-as-used-by-automations-iractionsserver)
3. [In-Application Purchase Service](#3-in-application-purchase-service-iapservice)
4. [In-Application Purchase Account](#4-in-application-purchase-account-iapaccount)
5. [Lead Enrichment Interface](#5-lead-enrichment-interface-iapenrichapi)
6. [Partner Autocomplete Interface](#6-partner-autocomplete-interface-iapautocompleteapi)
7. [Data Import Session](#7-data-import-session-base_importimport)
8. [Data Import Column Mapping](#8-data-import-column-mapping-base_importmapping)
9. [Module Import Wizard](#9-module-import-wizard-baseimportmodule)
10. [Module Activation Request](#10-module-activation-request-basemoduleinstallrequest)
11. [Module Activation Review](#11-module-activation-review-basemoduleinstallreview)
12. [Recycling Model](#12-recycling-model-data_recyclemodel)
13. [Recycling Record](#13-recycling-record-data_recyclerecord)
14. [Privacy Log](#14-privacy-log-privacylog)
15. [Privacy Lookup Wizard](#15-privacy-lookup-wizard-privacylookupwizard)
16. [Privacy Lookup Wizard Line](#16-privacy-lookup-wizard-line-privacylookupwizardline)
17. [Geocoding Provider](#17-geocoding-provider-basegeo_provider)
18. [Geocoder](#18-geocoder-basegeocoder)
19. [Google Service](#19-google-service-googleservice)
20. [Microsoft Service](#20-microsoft-service-microsoftservice)
21. [Google Gmail Mixin](#21-google-gmail-mixin-googlegmailmixin)
22. [Microsoft Outlook Mixin](#22-microsoft-outlook-mixin-microsoftoutlookmixin)
23. [Transifex Translation](#23-transifex-translation-transifextranslation)
24. [Code Translation](#24-code-translation-transifexcodetranslation)
25. [Onboarding](#25-onboarding-onboardingonboarding)
26. [Onboarding Step](#26-onboarding-step-onboardingonboardingstep)
27. [Onboarding Progress Tracker](#27-onboarding-progress-tracker-onboardingprogress)
28. [Onboarding Progress Step Tracker](#28-onboarding-progress-step-tracker-onboardingprogressstep)
29. [Tour](#29-tour-web_tourtour)
30. [Tour Step](#30-tour-step-web_tourtourstep)
31. [Sparse Fields Test](#31-sparse-fields-test-sparse_fieldstest)
32. [Attachment: the cloud-storage kind](#32-attachment-the-cloud-storage-kind-irattachment)
33. [Extension points contributed to foreign entities](#33-extension-points-contributed-to-foreign-entities)

---

# 1. Automation Rule (`base.automation`)

Reference page: [`../../references/entities/base.automation.md`](../../references/entities/base.automation.md).

**Transport name** `base.automation`. **Storage name** `base_automation`. **Kind** persistent.
**Contributed by** the automation-rules package.

## 1.1 Purpose

One Automation Rule binds four things together:

1. **A record type** — the thing being watched.
2. **A trigger** — the moment at which the rule considers itself relevant.
3. **A condition pair** — a filter evaluated before the change and a filter evaluated after it.
4. **A list of Server Actions** — what to do to the records that pass.

The rule has no other purpose. It stores no business data, it produces no document and it appears
in no report. It exists so that behaviour can be added to any record type without writing anything.

## 1.2 Behaviours the entity carries

The Automation Rule is itself a discussion thread: it carries a message history, followers and
scheduled activities, so that a change to a rule is traceable and a colleague can be asked to
review it. The thread behaviour is specified in
[`../messaging-and-activities/entities.md`](../messaging-and-activities/entities.md). The fields
that are tracked in that thread are listed in the field table below.

## 1.3 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed rule and dependencies | Stored | Meaning |
|---|---|---|---|---|---|---|---|---|
| `name` | Automation Rule Name | single line text | — | yes | none | — | yes | The label of the rule. Translatable. Changes are recorded in the thread. |
| `description` | Description | rich text | — | no | none | — | yes | Free notes about what the rule does and why it exists. The form places it on a tab labelled "Notes" with the placeholder "Keep track of what this automation does and why it exists...". |
| `model_id` | Model | many to one | `ir.model` (record type) | yes | none | — | yes | The watched record type. Restricted to non-abstract record types. Deleting the record type deletes the rule. Changes are recorded in the thread. |
| `model_name` | Model Name | single line text | — | no | none | related through `model_id.model`; writing it sets `model_id` to the record type with that transport name | no | A convenience mirror of the watched record type's transport name, used by the filter editors. |
| `model_is_mail_thread` | Model Is Mail Thread | boolean | — | no | none | related through `model_id.is_mail_thread` | no | Whether the watched record type carries a discussion thread. Governs which triggers are legal. |
| `action_server_ids` | Actions | one to many | `ir.actions.server` (Server Action) through `base_automation_id` | no | none | computed by the rule in §1.6.1, depends on `model_id`; writable | yes | The ordered list of actions the rule runs. New lines created from the rule carry the usage value `base_automation` and the rule's record type as their default. |
| `url` | Web address | single line text | — | no | none | computed by the formula in [`calculations.md`](calculations.md) §2.1, depends on `trigger` and `webhook_uuid` | no | The secret address an outside system calls for a webhook rule; empty for every other trigger. Its help text reads "Use this URL in the third-party app to call this webhook." |
| `webhook_uuid` | Webhook universally unique identifier | single line text | — | no | a freshly generated universally unique identifier | — | yes | The secret path segment of the web address. Read-only in the interface, never copied when the rule is duplicated, and replaceable on demand. |
| `record_getter` | Record getter | single line text | — | no | `model.env[payload.get('_model')].browse(int(payload.get('_id')))` | — | yes | The expression evaluated to decide which record an incoming webhook call applies to. Its help text reads "This code will be run to find on which record the automation rule should be run." |
| `log_webhook_calls` | Log Calls | boolean | — | no | false | — | yes | When set, every incoming webhook call — successful or not — writes a logging entry. |
| `active` | Active | boolean | — | no | true | — | yes | When cleared the rule is hidden and never executes. Its help text reads "When unchecked, the rule is hidden and will not be executed." |
| `trigger` | Trigger | selection | — | yes | none | computed to empty when `model_id` changes; writable | yes | The moment the rule reacts to. The nineteen values are listed in §1.4. Changes are recorded in the thread. |
| `trg_selection_field_id` | Trigger Field | many to one | `ir.model.fields.selection` (field choice) | no | none | computed to empty whenever `trigger` changes; writable | yes | The chosen value of a selection field, for the two triggers that watch a selection. Restricted to choices of a field in `trigger_field_ids`. Its help text reads "Some triggers need a reference to a selection field. This field is used to store it." |
| `trg_field_ref_model_name` | Trigger Field Model | single line text | — | no | none | computed by the rule in §1.6.6, depends on `trigger` and `trg_field_ref` | no | The transport name of the record type the trigger reference points at. |
| `trg_field_ref` | Trigger Reference | many to one by reference, target named by `trg_field_ref_model_name` | any | no | none | computed to empty whenever `trigger` changes; writable | yes | The chosen stage or tag, for the two triggers that watch a relation. Its help text reads "Some triggers need a reference to another field. This field is used to store it." |
| `trg_date_id` | Trigger Date | many to one | `ir.model.fields` (field definition) | no | none | computed by the rule in §1.6.2, depends on `trigger`; writable | yes | The date or date-and-time field the delay is measured from. Restricted to date and date-and-time fields of the watched record type. Changes are recorded in the thread. Its help text reads "When should the condition be triggered. If present, will be checked by the scheduler. If empty, will be checked at creation and update." |
| `trg_date_range` | Delay | integer | — | no | none | computed by the rule in §1.6.3, depends on `trigger`; writable | yes | How many delay units elapse before the rule fires. Changes are recorded in the thread. |
| `trg_date_range_mode` | Delay mode | selection | — | no | none | computed by the rule in §1.6.3, depends on `trigger`; writable | yes | `after` "After" or `before` "Before". Changes are recorded in the thread. |
| `trg_date_range_type` | Delay unit | selection | — | no | none | computed by the rule in §1.6.3, depends on `trigger`; writable | yes | `minutes` "Minutes", `hour` "Hours", `day` "Days", `month` "Months". Changes are recorded in the thread. |
| `trg_date_calendar_id` | Use Calendar | many to one | `resource.calendar` (working schedule) | no | none | computed by the rule in §1.6.4, depends on `trigger`, `trg_date_id` and `trg_date_range_type` | yes | When the delay unit is days, the delay may be counted in working days of this schedule instead of calendar days. Its help text reads "When calculating a day-based timed condition, it is possible to use a calendar to compute the date based on working days." |
| `filter_pre_domain` | Before Update Domain | single line text | — | no | none | computed by the rule in §1.6.7, depends on `trigger` and `trg_field_ref`; writable | yes | The filter the record must satisfy **before** the change. Never evaluated on creation. Its help text reads "If present, this condition must be satisfied before the update of the record. Not checked on record creation." |
| `previous_domain` | Previous Domain | single line text | — | no | the current value of `filter_domain` | — | no | A scratch copy of the after-filter, held only while a form is open, so that the interface can compute which field names entered or left the filter. |
| `filter_domain` | Apply on | single line text | — | no | none | computed by the rule in §1.6.8, depends on `trigger`, `trg_selection_field_id` and `trg_field_ref`; writable | yes | The filter the record must satisfy **after** the change, and the only filter for creation, deletion and time triggers. Its help text reads "If present, this condition must be satisfied before executing the automation rule." |
| `last_run` | Last Run | date and time | — | no | none | — | yes | The moment the time-based scheduler last processed this rule. Read-only, never copied on duplication. |
| `on_change_field_ids` | On Change Fields Trigger | many to many | `ir.model.fields` (field definition) through the link table `base_automation_onchange_fields_rel` | no | none | computed by the rule in §1.6.9, depends on `model_id`, `trigger` and `filter_domain`; writable | yes | For the live-update trigger only: the fields whose editing in an open form runs the rule. Its help text reads "Fields that trigger the onchange." |
| `trigger_field_ids` | Trigger Fields | many to many | `ir.model.fields` (field definition) | no | none | computed by the rule in §1.6.10, depends on `model_id`, `trigger` and `filter_domain`; writable | yes | The watched-field set. The rule runs only when at least one of them changed. Its help text reads "The automation rule will be triggered if and only if one of these fields is updated.If empty, all fields are watched." |

In addition the entity carries the standard audit fields `id`, `create_date`, `create_uid`,
`write_date` and `write_uid`, and every field the discussion-thread and activity behaviours add.

## 1.4 The nineteen triggers

| Stored value | Label | Family | Meaning |
|---|---|---|---|
| `on_stage_set` | Stage is set to | creation and update | The record's stage field reaches the chosen stage. |
| `on_user_set` | User is set | creation and update | The record's responsible-user field becomes non-empty. |
| `on_tag_set` | Tag is added | creation and update | The chosen tag is added to the record's tag field. |
| `on_state_set` | State is set to | creation and update | The record's state field reaches the chosen value. |
| `on_priority_set` | Priority is set to | creation and update | The record's priority field reaches the chosen value. |
| `on_archive` | On archived | update | The record's active flag becomes false. |
| `on_unarchive` | On unarchived | update | The record's active flag becomes true. |
| `on_create` | On create | creation | A record is created. |
| `on_create_or_write` | On create and edit | creation and update | A record is created, or one of the watched fields is written. |
| `on_write` | On update | update | One of the watched fields is written. Superseded in the interface by `on_create_or_write`, but still selectable and still honoured. |
| `on_unlink` | On deletion | deletion | A record is about to be deleted. |
| `on_change` | On UI change | live form | A watched field is edited in an open form, whether or not the form is saved. |
| `on_time` | Based on date field | time | A delay before or after the chosen date field elapses. |
| `on_time_created` | After creation | time | A delay after the creation timestamp elapses. |
| `on_time_updated` | After last update | time | A delay after the last-write timestamp elapses. |
| `on_message_received` | On incoming message | message | A message is posted whose author is outside the company, or has no author. |
| `on_message_sent` | On outgoing message | message | A message is posted whose author is an internal party. |
| `on_webhook` | On webhook | call | An outside system calls the rule's secret web address. |

The families matter because they decide which operation the rule intercepts; see
[`workflows.md`](workflows.md) §1.

Three constant sets are used throughout the specification:

- **Creation triggers**: `on_create`, `on_create_or_write`, `on_priority_set`, `on_stage_set`,
  `on_state_set`, `on_tag_set`, `on_user_set`.
- **Update triggers**: `on_write`, `on_archive`, `on_unarchive`, `on_create_or_write`,
  `on_priority_set`, `on_stage_set`, `on_state_set`, `on_tag_set`, `on_user_set`.
- **Time triggers**: `on_time`, `on_time_created`, `on_time_updated`.
- **Message triggers**: `on_message_received`, `on_message_sent`.

## 1.5 The trigger-specific field

Several computed rules need "the field this trigger is about". It is resolved as follows, always
restricted to the watched record type, and always taking the first match in identifier order:

| Trigger | Search |
|---|---|
| `on_create_or_write` | Every field named inside `filter_domain` (see [`calculations.md`](calculations.md) §1.1) |
| `on_stage_set` | A many-to-one field named `stage_id` or `x_studio_stage_id` |
| `on_tag_set` | A many-to-many field named `tag_ids` or `x_studio_tag_ids` |
| `on_priority_set` | A selection field named `priority` or `x_studio_priority` |
| `on_state_set` | A selection field named `state` or `x_studio_state` |
| `on_user_set` | A many-to-one or many-to-many field pointing at `res.users` and named `user_id`, `user_ids`, `x_studio_user_id` or `x_studio_user_ids` |
| `on_archive`, `on_unarchive` | A boolean field named `active` or `x_active` |
| `on_time_created` | The date-and-time field named `create_date` |
| `on_time_updated` | The date-and-time field named `write_date` |
| every other trigger | Nothing |

## 1.6 Computed rules in detail

### 1.6.1 The action list

When the watched record type changes, every action in the list whose own record type is no longer
the rule's record type is unlinked from the rule. Nothing else is removed.

### 1.6.2 The trigger date field

Rules whose trigger is not a time trigger have the field cleared. Rules whose trigger is a time
trigger take the trigger-specific field of §1.5 — which for `on_time` is nothing, so the user
chooses it, and for `on_time_created` and `on_time_updated` is the creation or last-write stamp.

### 1.6.3 The delay triple

If the trigger is not a time trigger, the delay, the mode and the unit are all cleared.
Otherwise: a missing unit becomes `hour`; a missing mode, or any mode on a trigger other than
`on_time`, becomes `after`.

A separate interface rule fires when the delay itself is edited: a negative delay is replaced by
its absolute value, and, for the `on_time` trigger only, the mode is flipped from `after` to
`before` or from `before` to `after`. Editing the delay to a different positive number leaves the
mode alone. A worked sequence is in [`acceptance-criteria.md`](acceptance-criteria.md) §1.12.

### 1.6.4 The working schedule

The schedule is cleared for any rule that is not a time rule, that has no trigger date field, or
whose delay unit is not days. It is never set automatically; the user chooses it.

### 1.6.5 The selection choice and the reference

Both are cleared whenever the trigger changes. Both are then set by the user.

### 1.6.6 The reference target

Rules whose trigger is neither `on_stage_set` nor `on_tag_set`, and rules whose reference has never
been touched, have the target cleared. For the others, the target is the record type the
trigger-specific field points at; when that field points at nothing, the target is cleared.

### 1.6.7 The before-filter

Cleared for every trigger except `on_tag_set`. For `on_tag_set` with a chosen tag, it becomes the
filter *the tag field does not contain the chosen tag*; with no chosen tag it is cleared. This is
what makes "tag is added" mean *added now*, not *present*.

### 1.6.8 The after-filter

Let the trigger-specific field of §1.5 be resolved first. For `on_create_or_write` and for the
three time triggers the filter is cleared and left to the user. When the field resolves to nothing
the filter is cleared. Otherwise:

| Trigger | Computed filter |
|---|---|
| `on_state_set`, `on_priority_set` | *the field equals the chosen selection value*; cleared when no value is chosen |
| `on_stage_set` | *the field equals the chosen reference*; cleared when no reference is chosen |
| `on_tag_set` | *the field contains the chosen reference*; cleared when no reference is chosen |
| `on_user_set` | *the field is not empty* |
| `on_archive` | *the field equals false* |
| `on_unarchive` | *the field equals true* |

### 1.6.9 The live-update field set

Cleared for every trigger other than `on_change`. For `on_change` the set is kept in step with the
after-filter: field names that left the filter are dropped from the set, field names that entered
it are added. Fields the user added by hand and that never appeared in a filter are kept.

### 1.6.10 The watched-field set

For `on_create_or_write` the set is kept in step with the after-filter exactly as in §1.6.9. For
every other trigger the set becomes exactly the trigger-specific field of §1.5 — a single field, or
empty when the trigger has none.

## 1.7 Relations

| Relation | Cardinality | Target | On delete |
|---|---|---|---|
| Watched record type | many to one | `ir.model` | deleting the record type deletes the rule |
| Actions | one to many | `ir.actions.server` | deleting the rule deletes its actions |
| Selection choice | many to one | `ir.model.fields.selection` | platform default |
| Trigger date field | many to one | `ir.model.fields` | platform default |
| Working schedule | many to one | `resource.calendar` | platform default |
| Live-update fields | many to many | `ir.model.fields` | link removed |
| Watched fields | many to many | `ir.model.fields` | link removed |

## 1.8 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none. Two rules may watch the same record type with the same trigger; both run.
- **Ordering**: the platform default, by ascending identifier.
- **Display name**: the rule's name.
- **Archival**: supported through `active`. An archived rule is not searched by the trigger lookup,
  is not patched into the registry and is skipped by the time-based scheduler. The list action
  opens with the *Include Archived* filter switched on, so archived rules are visible by default.
- **Company**: none. A rule is global; it has no company field and no company record rule.
- **Duplication**: duplicating a rule first duplicates its actions and then attaches the copies to
  the copy, so the two rules do not share action records. The webhook identifier is not copied; the
  copy gets a fresh one.

## 1.9 Fields whose change rewrites the registry

Writing any of `model_id`, `active`, `trigger` or `on_change_field_ids` causes the scheduler
interval to be recomputed **and** the registry patches to be reinstalled. Writing
`trg_date_range` or `trg_date_range_type` alone causes only the scheduler interval to be
recomputed. Creating or deleting a rule always does both. When the set of live-update rules changes,
the client template cache is emptied as well, so that the forms carry the new live-update fields.

---

# 2. Server Action as used by automations (`ir.actions.server`)

Reference page: [`../../references/entities/ir.actions.server.md`](../../references/entities/ir.actions.server.md).

The Server Action belongs to [`../platform-foundation/`](../platform-foundation/). This folder
specifies only what the automation package adds to it and what an automation rule may put in it.

## 2.1 Fields added

| Identifier | Full name | Type | Target | Required | Default | Meaning |
|---|---|---|---|---|---|---|
| `usage` | Action Usage | selection | — | yes | `ir_actions_server` | Gains the extra value `base_automation` "Automation Rule". Deleting the automation package cascades: actions carrying this usage are deleted with it. |
| `base_automation_id` | Automation Rule | many to one | `base.automation` | no | none | The owning rule. Indexed, skipping empty values. Deleting the rule deletes the action. |

## 2.2 Behaviour added

| Addition | Effect |
|---|---|
| Warning dependency | The action's warning text is recomputed when its record type or its owning rule changes. |
| Warning text | When the action has an owning rule and the action's record type differs from the rule's record type, the warning reads "Model of action <the action name> should match the one from automated rule <the rule name>." |
| Child restriction | An action of the *Multi Actions* kind may not take an automation rule's action as a child; the candidate list excludes every action that has an owning rule. |
| Record-type restriction | For an action whose usage is `base_automation`, the list of record types it may target is narrowed to the single record type of its owning rule — and to nothing at all when that record type is not in the list the platform would have allowed. |
| Evaluation names | When the action is of the *Execute Code* kind, two extra names are available to the code: a structured-text helper, and, when the action is running inside an incoming webhook call, the payload of that call. |
| Jump to the rule | A button on the action form, visible only when the action has an owning rule, opens that rule. The same jump is available from a Scheduled Action, which forwards it to its own server action. |

## 2.3 The action kinds an automation rule may hold

| Stored value | Label | Contributed by | Usable with trigger `on_change`? | Usable with trigger `on_unlink`? |
|---|---|---|---|---|
| `object_write` | Update Record | platform | no | yes |
| `object_create` | Create Record | platform | no | yes |
| `object_copy` | Duplicate Record | platform | no | yes |
| `code` | Execute Code | platform | **yes** | yes |
| `webhook` | Send Webhook Notification | platform | no | yes |
| `multi` | Multi Actions | platform | no | yes |
| `next_activity` | Create Activity | messaging | no | **no** |
| `mail_post` | Send Email | messaging | no | **no** |
| `followers` | Add Followers | messaging | no | **no** |
| `remove_followers` | Remove Followers | messaging | no | yes |
| `sms` | Send SMS | messaging | no | yes |

The two "no" columns are enforced by the constraints in [`business-rules.md`](business-rules.md)
§1.4 and §1.5.

## 2.4 The outgoing webhook action

An action of the `webhook` kind carries two more fields of its own:

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `webhook_url` | Webhook web address | single line text | Where the request is sent. Its help text reads "URL to send the POST request to." |
| `webhook_field_ids` | Webhook Fields | many to many to `ir.model.fields` through the link table `ir_act_server_webhook_field_rel` | Which of the record's fields travel in the body. Its help text reads "Fields to send in the POST request. The id and model of the record are always sent as '_id' and '_model'. The name of the action that triggered the webhook is always sent as '_name'." |
| `webhook_sample_payload` | Sample Payload | long text | A preview of the body, recomputed from the kind, the record type, the chosen fields and the action name. |

The payload contract, the delivery and the timeout are specified in
[`calculations.md`](calculations.md) §2.3 and [`workflows.md`](workflows.md) §3.

---

# 3. In-Application Purchase Service (`iap.service`)

Reference page: [`../../references/entities/iap.service.md`](../../references/entities/iap.service.md).

**Transport name** `iap.service`. **Storage name** `iap_service`. **Kind** persistent.

## 3.1 Purpose

One record describes one purchasable outside service: what it is called, what its billing unit is
called, and whether its balance is a whole number of units or a fractional amount. A service record
is shipped by whichever package needs the service; the record is the anchor that accounts point at.

## 3.2 Field table

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Name | single line text | yes | none | The label shown to the user, for example `Lead Generation`. |
| `technical_name` | Technical Name | single line text | yes | none | The token the calling code uses to find the account. Read-only in the interface. Unique. |
| `description` | Description | single line text | yes | none | One sentence explaining what the service does. Translatable. |
| `unit_name` | Unit Name | single line text | yes | none | The plural noun for the billing unit, for example `Credits` or `Enrichments`. Translatable. |
| `integer_balance` | Integer Balance | boolean | yes | none | When true the balance is displayed as a whole number; when false it is rounded to four decimal places. |

## 3.3 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: a database constraint named `_unique_technical_name` enforces one record per
  technical name. Its message is "Only one service can exist with a specific technical_name".
- **Ordering**: platform default, by ascending identifier.
- **Display name**: the name.
- **Archival**: not supported; the entity has no active flag.
- **Company**: none.

## 3.4 Shipped services

Two service records ship with the packages in this folder:

| External identifier | Name | Technical name | Unit name | Whole balance | Contributed by |
|---|---|---|---|---|---|
| `iap.iap_service_reveal` | Lead Generation | `reveal` | Credits | yes | the metered-services package |
| `partner_autocomplete.iap_service_partner_autocomplete` | Partner Autocomplete | `partner_autocomplete` | Enrichments | yes | the company-autocomplete package |

Their descriptions are reproduced in [`configuration.md`](configuration.md) §7.1.

---

# 4. In-Application Purchase Account (`iap.account`)

Reference page: [`../../references/entities/iap.account.md`](../../references/entities/iap.account.md).

**Transport name** `iap.account`. **Storage name** `iap_account`. **Kind** persistent.

## 4.1 Purpose

One record holds the credential for one service. The credential is a token; the outside service
recognises the database by that token and bills the units it consumes against it. The record also
mirrors, for display only, the balance the outside service reports, and carries the settings for
the low-balance alert.

An account is never created deliberately by a user in the ordinary case: the calling code asks for
"the account for service X", and one is created on the spot if none exists.

## 4.2 Behaviours the entity carries

When the electronic-mail bridge package is installed, the account becomes a discussion thread, and
three of its fields are tracked in that thread: the company list, the alert threshold and the alert
recipients.

## 4.3 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed or related | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | single line text | — | no | none | — | The label. When left empty at creation it is filled with the service's name. |
| `service_id` | Service | many to one | `iap.service` | yes | none | — | Which service this credential is for. |
| `service_name` | Service Name | single line text | — | no | none | related through `service_id.technical_name` | The technical token, used for lookups. |
| `service_locked` | Service Locked | boolean | — | no | false | — | Set to true once the outside service confirms the account exists. While true the service may no longer be changed on the form. |
| `description` | Description | single line text | — | no | none | related through `service_id.description` | The service's one-sentence description. |
| `account_token` | Account Token | single line text, at most 43 characters | — | no | a freshly generated hexadecimal universally unique identifier | — | The credential. Readable only by the settings-administration group. Never copied when the record is duplicated. Its help text reads "Account token is your authentication key for this service. Do not share it." |
| `company_ids` | Companies | many to many | `res.company` | no | none | — | The companies allowed to use this credential. Empty means every company. Changes are tracked in the thread when the electronic-mail bridge is installed. |
| `balance` | Balance | single line text | — | no | none | — | The balance last reported by the outside service, already formatted with its unit name. Read-only; see [`calculations.md`](calculations.md) §4.1. |
| `warning_threshold` | Email Alert Threshold | decimal number | — | no | 0 | — | When the balance falls to or below this number the outside service sends an alert to the recipients. Tracked in the thread. |
| `warning_user_ids` | Email Alert Recipients | many to many | `res.users` | no | none | — | Who receives the low-balance alert. Tracked in the thread. |
| `state` | State | selection | — | no | none | — | The registration state the outside service reports: `banned` "Banned", `registered` "Registered", `unregistered` "Unregistered". Read-only. |
| `sender_name` | Sender Name | single line text | — | no | none | — | Added by the text-message package: the name displayed as the sender of a text message. Read-only. Its help text reads "This is the name that will be displayed as the sender of the Text Message." |

## 4.4 Relations

| Relation | Cardinality | Target | Notes |
|---|---|---|---|
| Service | many to one | `iap.service` | Required |
| Companies | many to many | `res.company` | Empty means unrestricted |
| Alert recipients | many to many | `res.users` | Each must have an electronic mail address |

## 4.5 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none in the database. The lookup described in [`workflows.md`](workflows.md) §6.1
  imposes a practical uniqueness by always reusing the newest matching account.
- **Ordering**: platform default. The lookup explicitly sorts by descending identifier.
- **Display name**: the name.
- **Archival**: not supported.
- **Company**: multi-company by list rather than by single owner. A record rule restricts ordinary
  users to accounts whose company list is empty or intersects their allowed companies; see
  [`configuration.md`](configuration.md) §5.2.
- **Duplication**: allowed by the record but disabled on both the form and the list view.

## 4.6 Behaviour at creation and on read

- **At creation**, an account with no name takes its service's name. When the database is marked as
  a neutralised copy, the freshly generated token has the suffix `+disabled` appended after
  stripping anything already following a plus sign, so that a copy of a production database cannot
  spend the original's units.
- **On read through the web client**, the account list first calls the outside service to refresh
  the balance, the alert threshold, the registration state and the lock flag of every account being
  read. The refresh is skipped while tests are running. A failure to reach the service is logged as
  a warning and leaves the stored values untouched.
- **On write**, when the alert threshold or the alert recipients changed and the reading context
  does not carry the suppression flag, the new alert configuration is pushed to the outside service.
  A failure is logged as a warning and does not prevent the write.

---

# 5. Lead Enrichment Interface (`iap.enrich.api`)

Reference page: [`../../references/entities/iap.enrich.api.md`](../../references/entities/iap.enrich.api.md).

**Transport name** `iap.enrich.api`. **Kind** abstract behaviour; no table, no fields.

## 5.1 Purpose

The behaviour turns a set of electronic mail addresses into company data. It is called by the
customer-relationship domain when a lead is enriched.

## 5.2 Operations

| Operation | Behaviour |
|---|---|
| Contact the service | Fetches the account for the service whose technical name is `reveal`, adds the account token and the database identifier to the parameters, reads the base address from the system parameter `enrich.endpoint` defaulting to `https://iap-services.odoo.com`, and posts the call with a three-hundred-second timeout. |
| Request enrichment | Sends a mapping of lead identifier to electronic mail address under the key `domains` to the path `/iap/clearbit/1/lead_enrichment_email` and returns a mapping of lead identifier to company data or false. |

Insufficient balance surfaces as a distinct failure carrying the remaining balance, the service
name, the purchase address and the message "You don't have enough credits on your account to use
this service."

---

# 6. Partner Autocomplete Interface (`iap.autocomplete.api`)

Reference page: [`../../references/entities/iap.autocomplete.api.md`](../../references/entities/iap.autocomplete.api.md).

**Transport name** `iap.autocomplete.api`. **Kind** abstract behaviour; no table, no fields.

## 6.1 Purpose

The behaviour searches a commercial company directory by name, by tax registration number, by
company registration number or by internet domain, and returns structured company data that the
client offers to copy onto a Contact.

## 6.2 Operations

| Operation | Behaviour |
|---|---|
| Contact the service | Refuses outright while tests are running with the message "Test mode". Fetches the account for the service whose technical name is `partner_autocomplete`; when that account has no token the call fails with "No account token". Adds the database identifier, the release string, the current language, the account token, the company's country code and the company's postal code to the parameters. Reads the base address from the system parameter `iap.partner_autocomplete.endpoint` defaulting to `https://partner-autocomplete.odoo.com` and posts to that address followed by `/api/dnb/1` and the action name. Default timeout fifteen seconds. |
| Request autocompletion | Wraps the call and converts every failure into a second return value: an exhausted balance and the test-mode refusal both become `Insufficient Credit`; a missing token becomes `No account token`; a connection, protocol, access or user failure becomes the failure's own text. On success the second value is false. |

The five actions the caller may name are `search_by_name`, `search_by_vat`, `enrich_by_duns`,
`enrich_by_gst` and `enrich_by_domain`.

---

# 7. Data Import Session (`base_import.import`)

Reference page: [`../../references/entities/base_import.import.md`](../../references/entities/base_import.import.md).

**Transport name** `base_import.import`. **Storage name** `base_import_import`. **Kind** transient.

## 7.1 Purpose

One record represents one uploaded file being mapped onto one record type. It holds the file bytes
and the metadata needed to read them; every decision about how to read the file is passed in with
each call as an options mapping, not stored on the record.

## 7.2 Lifecycle

Transient records are garbage-collected by the platform. This entity raises the survival time to
**twelve hours**, so that a user who leaves a large mapping half-finished can come back to it.

## 7.3 Field table

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `res_model` | Model | single line text | no | none | The transport name of the record type being imported into. |
| `file` | File | binary, stored in the row rather than as an attachment | no | none | The raw bytes of the uploaded file. Its help text reads "File to check and/or import, raw binary (not base64)". |
| `file_name` | File Name | single line text | no | none | The name the browser reported. Used as a last-resort hint at the file format. |
| `file_type` | File Type | single line text | no | none | The media type the browser reported. Used as a second-choice hint at the file format. |

## 7.4 The options mapping

The options mapping is not stored; it travels with every call. Its keys are:

| Key | Meaning |
|---|---|
| `has_headers` | Whether the first row carries column headings. |
| `encoding` | The character encoding of a delimited file; detected when absent. |
| `separator` | The column separator of a delimited file; inferred when absent. |
| `quoting` | The single character that quotes a value in a delimited file. |
| `sheet` | Which sheet of a spreadsheet workbook to read. |
| `sheets` | Filled by the reader with the list of sheet names found. |
| `date_format` | The pattern used to parse dates; inferred when absent. |
| `datetime_format` | The pattern used to parse dates with times; inferred when absent. |
| `float_thousand_separator` | The grouping character of decimal numbers. |
| `float_decimal_separator` | The decimal character of decimal numbers. |
| `skip` | How many data rows to skip before importing, used to resume a partial import. |
| `limit` | How many rows one batch imports. |
| `keep_matches` | Keep the user's own column mapping instead of proposing one. |
| `fields` | The user's own column mapping, used when the previous key is set. |
| `advanced` | Whether the advanced mapping editor was open. |
| `fallback_values` | Per field, the value to substitute when the file value is not acceptable. |
| `name_create_enabled_fields` | Per field, permission to create a missing linked record from its name. |
| `import_set_empty_fields` | The fields whose unrecognised values become empty instead of failing. |
| `import_skip_records` | The fields whose unrecognised values cause the whole row to be skipped. |

## 7.5 Uniqueness, ordering, display name, archival, company

- **Uniqueness, ordering, archival, company**: none; a transient working record.
- **Display name**: the platform default.
- **Access**: every internal user may read, write and create; nobody may delete. A record rule
  limits every user to the sessions they created themselves.

## 7.6 Constants

| Constant | Value | Where used |
|---|---|---|
| Field recursion limit | 3 | How deep the importable-field tree descends through one-to-many fields |
| Error preview length | 200 bytes | How much of a delimited file is echoed back when reading it failed |
| Download chunk size | 32768 bytes | How much of a remote file is read at a time |
| Fuzzy match distance | 0.2 | The largest distance at which a column heading is still matched to a field |

---

# 8. Data Import Column Mapping (`base_import.mapping`)

Reference page: [`../../references/entities/base_import.mapping.md`](../../references/entities/base_import.mapping.md).

**Transport name** `base_import.mapping`. **Storage name** `base_import_mapping`. **Kind** persistent.

## 8.1 Purpose

When a user imports repeatedly from the same outside system, the column headings that system
produces never match the field names of this system. One record remembers one association: for this
record type, a column with this heading means this field. The next import proposes the remembered
association before doing anything else.

## 8.2 Field table

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `res_model` | Model | single line text | no | none | The transport name of the record type the mapping applies to. Indexed. |
| `column_name` | Column Name | single line text | no | none | The heading exactly as it appeared in the file. |
| `field_name` | Field Name | single line text | no | none | The field, or the slash-joined path of fields, the heading maps to. |

## 8.3 Lifecycle

A mapping is created or updated only when an import actually succeeded and the file had headings.
For each column with a heading, the existing mapping for that record type and heading is looked up;
if it exists and points at a different field it is repointed, and if it does not exist it is
created. Mappings are never deleted by the system.

## 8.4 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none in the database; the update rule above keeps one row per record type and
  heading in practice.
- **Ordering**: platform default.
- **Display name**: platform default.
- **Archival, company**: none.
- **Access**: every internal user has full rights, including delete.

---

# 9. Module Import Wizard (`base.import.module`)

Reference page: [`../../references/entities/base.import.module.md`](../../references/entities/base.import.module.md).

**Transport name** `base.import.module`. **Storage name** `base_import_module`. **Kind** transient.

## 9.1 Purpose

One record carries one uploaded packaged-module archive through its installation. The archive may
hold several packages at once; they are installed in dependency order.

## 9.2 Field table

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `module_file` | Module archive file | binary, stored in the row | yes | none | The uploaded archive. The form restricts the file chooser to the `.zip` extension and labels the field "Module file (.zip)". |
| `state` | Status | selection | no | `init` | `init` while the archive is being described, `done` once the installation finished. Read-only. |
| `import_message` | Import Message | long text | no | none | The message shown after installation. Read-only on the form. |
| `force` | Force init | boolean | no | false | Install in initialisation mode even for an already installed package, which rewrites records marked as never-updated. Visible only to the technical-features group. Its help text reads "Force init mode even if installed. (will update `noupdate='1'` records)". |
| `with_demo` | Import demo data of module | boolean | no | false | Also load the package's demonstration data. Labelled "Load demo data" on the form. |
| `modules_dependencies` | Modules Dependencies | long text | no | none | The description of what else will be installed, or the warning that something is missing. Read-only, hidden once the state is `done`. |

## 9.3 Operations

| Operation | Behaviour |
|---|---|
| Install | Decodes the archive and runs the archive installer of [`workflows.md`](workflows.md) §8, then navigates the browser to the application root. |
| List the dependencies to install | Returns the names of the packages the archive depends on that are present in the database but not installed. |
| Open the modules | Opens a list and form of the packages whose names are carried in the reading context under the key `module_name`. |

## 9.4 Uniqueness, ordering, display name, archival, company

None; a transient working record. Access is limited to the settings-administration group, which may
read, write and create but not delete.

---

# 10. Module Activation Request (`base.module.install.request`)

Reference page: [`../../references/entities/base.module.install.request.md`](../../references/entities/base.module.install.request.md).

**Transport name** `base.module.install.request`. **Storage name** `base_module_install_request`.
**Kind** transient.

## 10.1 Purpose

An ordinary user cannot install a package. The request record lets them ask: it captures which
package, who is asking, who should be asked, and why, and then sends one electronic mail to each
recipient.

## 10.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Meaning |
|---|---|---|---|---|---|---|---|
| `module_id` | Module | many to one | `ir.module.module` | yes | none | — | The package being requested. Restricted to packages whose state is `uninstalled`. Read-only. Deleting the package deletes the request. |
| `user_id` | User | many to one | `res.users` | yes | the current user | — | Who is asking. |
| `user_ids` | Send to | many to many | `res.users` | no | none | computed, depends on `module_id` | Every user who holds the settings-administration group, directly or by implication. |
| `body_html` | Body | rich text | — | no | none | — | The justification. The form labels the group "Why do you need this module?" and offers the placeholder "e.g. I'd like to use the SMS Marketing module to organize the promotion of our internal events, and exhibitions. I need access for 3 people of my team." |

## 10.3 Display name and access

- **Display name**: the package.
- **Access**: every internal user may read, write and create; nobody may delete. The package
  catalogue itself is opened to every internal user in read-only form by the same package, so that
  the request can be raised from the application list.

---

# 11. Module Activation Review (`base.module.install.review`)

Reference page: [`../../references/entities/base.module.install.review.md`](../../references/entities/base.module.install.review.md).

**Transport name** `base.module.install.review`. **Storage name** `base_module_install_review`.
**Kind** transient.

## 11.1 Purpose

The administrator's side of the request: a confirmation screen that names every application that
will be installed as a consequence, followed by one button that installs them.

## 11.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Meaning |
|---|---|---|---|---|---|---|---|
| `module_id` | Module | many to one | `ir.module.module` | yes | none | — | The package under review. Restricted to packages whose state is `uninstalled`. Read-only. Deleting the package deletes the review. |
| `module_ids` | Depending Apps | many to many | `ir.module.module` | no | none | computed, depends on `module_id` | The package plus every application it pulls in, computed by §11.3. |
| `modules_description` | Modules Description | rich text | — | no | none | computed, depends on `module_id` | The rendered list of those applications. |

## 11.3 The dependency expansion

1. Refuse with "No module selected." when no package is given.
2. Refuse with "The module is already installed." when the package's state is `installed`.
3. Take every package the chosen package depends on, directly or indirectly.
4. Keep the chosen package, plus those dependencies that are flagged as applications.
5. Add, for each dependency, that dependency's own upstream dependencies.
6. Return the accumulated set.

## 11.4 Operation

Installing runs the platform's immediate-install operation on the chosen package and then sends the
client to the application home.

## 11.5 Access

The settings-administration group may read, write and create; nobody may delete.

---

# 12. Recycling Model (`data_recycle.model`)

Reference page: [`../../references/entities/data_recycle.model.md`](../../references/entities/data_recycle.model.md).

**Transport name** `data_recycle.model`. **Storage name** `data_recycle_model`. **Kind** persistent.

## 12.1 Purpose

One record is one recycling rule. It names a record type, an optional filter, an optional age
threshold measured from a chosen date field, and what to do with the records that match: archive
them or delete them. It also decides whether that happens automatically or after a human validates
each candidate, and who is reminded that candidates are waiting.

## 12.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Meaning |
|---|---|---|---|---|---|---|---|
| `active` | Active | boolean | — | no | true | — | Archiving a rule deletes every candidate it had found; see §12.6. |
| `name` | Name | single line text | — | yes | none | computed from `res_model_id`; writable; copied on duplication | The label. Filled with the record type's name when it is empty. |
| `res_model_id` | Model | many to one | `ir.model` | yes | none | — | The watched record type. Deleting it deletes the rule. |
| `res_model_name` | Model Name | single line text | — | no | none | related through `res_model_id.model`, stored | The record type's transport name. |
| `recycle_record_ids` | Recycle Records | one to many | `data_recycle.record` through `recycle_model_id` | no | none | — | The candidates found so far. |
| `recycle_mode` | Recycle Mode | selection | — | yes | `manual` | — | `manual` "Manual" — candidates wait for a human; `automatic` "Automatic" — candidates are acted on immediately. |
| `recycle_action` | Recycle Action | selection | — | yes | `unlink` | — | `archive` "Archive" or `unlink` "Delete". |
| `domain` | Filter | single line text | — | no | none | computed to `[]` from `res_model_id`; writable | An extra condition candidates must satisfy. |
| `time_field_id` | Time Field | many to one | `ir.model.fields` | no | none | — | The date or date-and-time field the age is measured on. Restricted to stored date and date-and-time fields of the watched record type. Deleting the field deletes the rule. |
| `time_field_delta` | Delta | integer | — | no | 1 | — | How many units old a record must be. |
| `time_field_delta_unit` | Delta Unit | selection | — | no | `months` | — | `days` "Days", `weeks` "Weeks", `months` "Months", `years` "Years". |
| `include_archived` | Include Archived | boolean | — | no | false | — | Whether already archived records are also candidates. Shown on the form only when the action is delete. |
| `records_to_recycle_count` | Records To Recycle | integer | — | no | none | computed, not stored | How many candidates are waiting. |
| `notify_user_ids` | Notify Users | many to many | `res.users` | no | the current user | — | Who is reminded. Restricted to users who hold the settings-administration group. Its help text reads "List of users to notify when there are new records to recycle". |
| `notify_frequency` | Notify | integer | — | no | 1 | — | How many periods between two reminders. |
| `notify_frequency_period` | Notify Frequency Period | selection | — | no | `weeks` | — | `days` "Days", `weeks` "Weeks", `months` "Months". |
| `last_notification` | Last Notification | date and time | — | no | none | — | When the last reminder went out. Read-only. |

## 12.3 Database constraint

| Name | Condition | Message |
|---|---|---|
| `_check_notif_freq` | The notification frequency is greater than zero | "The notification frequency should be greater than 0" |

## 12.4 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none.
- **Ordering**: by name, ascending.
- **Display name**: the name.
- **Archival**: supported. The configuration list deliberately shows both archived and active rules
  and renders the archived ones muted.
- **Company**: none on the rule itself; the candidates it produces carry one.

## 12.5 Batch sizes

| Mode | Candidates created per batch | Why |
|---|---|---|
| `automatic` | 5 000 | Each batch is immediately acted on, which is slow |
| `manual` | 50 000 | Each batch is only written |

Outside a test run, each batch is committed before the next one starts, so that a timeout does not
discard the work already done.

## 12.6 Behaviour on write

Writing `active` to false deletes every Recycling Record that points at the rule. This is done
before the write itself is applied.

---

# 13. Recycling Record (`data_recycle.record`)

Reference page: [`../../references/entities/data_recycle.record.md`](../../references/entities/data_recycle.record.md).

**Transport name** `data_recycle.record`. **Storage name** `data_recycle_record`. **Kind** persistent.

## 13.1 Purpose

One record is one candidate: the rule that found it, and the identity of the record it refers to.
It carries no copy of that record's data; it reads the original every time it needs to show
anything.

## 13.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed or related | Meaning |
|---|---|---|---|---|---|---|---|
| `active` | Active | boolean | — | no | true | — | Cleared when the user discards the candidate. A discarded candidate is never proposed again unless the rule is archived and rebuilt. |
| `name` | Record Name | single line text | — | no | none | computed with elevated rights from `res_id`, not stored | The display name of the original record; `Undefined Name` when the original has no display name, and `**Record Deleted**` when the original no longer exists. |
| `recycle_model_id` | Recycle Model | many to one | `data_recycle.model` | no | none | — | The rule that found the candidate. Indexed, skipping empty values. Deleting the rule deletes the candidate. |
| `res_id` | Record identifier | integer | — | no | none | — | The identifier of the original record. Indexed. Never aggregated in reports. |
| `res_model_id` | Model | many to one | `ir.model` | no | none | related through `recycle_model_id.res_model_id`, stored, read-only | The original record's type. |
| `res_model_name` | Model Name | single line text | — | no | none | related through `recycle_model_id.res_model_name`, stored, read-only | The original record type's transport name. |
| `company_id` | Company | many to one | `res.company` | no | none | computed from `res_id`, stored | The original record's company when its type has one, otherwise empty. |

## 13.3 Reading the originals

Candidates are grouped by record type and read in one batch per type, with elevated rights and with
archived records included, then filtered to those that still exist. A candidate whose original has
disappeared shows the deleted marker and contributes no company.

## 13.4 Operations

| Operation | Behaviour |
|---|---|
| Validate | Reads the originals; groups them by record type into those to archive and those to delete according to each candidate's rule; archives the first group with elevated rights; deletes the second group with elevated rights; then deletes every candidate that was processed, including those whose original had already disappeared. |
| Discard | Clears the active flag on the candidates. |

## 13.5 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none. The candidate search avoids duplicates by excluding originals that already
  have a candidate for the same rule, archived candidates included.
- **Ordering**: platform default.
- **Display name**: the computed record name.
- **Archival**: supported, and used as the "discarded" marker.
- **Company**: stored, computed from the original.

---

# 14. Privacy Log (`privacy.log`)

Reference page: [`../../references/entities/privacy.log.md`](../../references/entities/privacy.log.md).

**Transport name** `privacy.log`. **Storage name** `privacy_log`. **Kind** persistent.

## 14.1 Purpose

Proof that a request to erase a person's data was handled, kept in a form that does not itself store
the person's data. The name and the electronic mail address are masked before they are written; what
is kept in full is the list of record types and identifiers that were found, and the list of actions
that were taken.

## 14.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Meaning |
|---|---|---|---|---|---|---|
| `date` | Date | date and time | — | yes | the current moment | When the session happened. |
| `anonymized_name` | Anonymized Name | single line text | — | yes | none | The masked name; see [`calculations.md`](calculations.md) §12.1. |
| `anonymized_email` | Anonymized Email | single line text | — | yes | none | The masked electronic mail address; see [`calculations.md`](calculations.md) §12.2. |
| `user_id` | Handled By | many to one | `res.users` | yes | the current user | Who ran the session. |
| `execution_details` | Execution Details | long text | — | no | none | One line per action taken, in the order the actions were taken. |
| `records_description` | Found Records | long text | — | no | none | One line per record type found, with the count and the identifiers. |
| `additional_note` | Additional Note | long text | — | no | none | Free text the handler adds afterwards. |

## 14.3 Behaviour at creation

Both the name and the electronic mail address are masked at creation; the unmasked values are never
written. The masking rules are deterministic, so two sessions for the same person produce the same
masked values and can be found together.

## 14.4 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none in the database. In practice one log exists per wizard session, because the
  wizard creates it once and updates it thereafter.
- **Ordering**: platform default.
- **Display name**: the handling user.
- **Archival, company**: none.
- **Creation from the interface**: both actions that open the log list set the create flag to false,
  so a log can never be typed by hand.

---

# 15. Privacy Lookup Wizard (`privacy.lookup.wizard`)

Reference page: [`../../references/entities/privacy.lookup.wizard.md`](../../references/entities/privacy.lookup.wizard.md).

**Transport name** `privacy.lookup.wizard`. **Storage name** `privacy_lookup_wizard`. **Kind** transient.

## 15.1 Purpose

One search of the entire database for one person, identified by a name and an electronic mail
address, followed by a per-record decision to keep, archive or delete.

## 15.2 Lifecycle

The wizard's transient records are kept for **twenty-four hours** and are never discarded because
there are too many of them: the maximum-count limit is set to zero, which switches count-based
collection off.

## 15.3 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | single line text | — | yes | none | — | The person's name, matched case-insensitively and as a substring. |
| `email` | Email | single line text | — | yes | none | — | The person's electronic mail address. |
| `line_ids` | Lines | one to many | `privacy.lookup.wizard.line` through `wizard_id` | no | none | — | One line per record found. |
| `execution_details` | Execution Details | long text | — | no | none | computed and stored, depends on each line's own execution details | Every action line, joined by newlines. Recomputing it also writes the Privacy Log. |
| `log_id` | Log | many to one | `privacy.log` | no | none | — | The log for this session. |
| `records_description` | Records Description | long text | — | no | none | computed, not stored, depends on `line_ids` | One line per record type, with the count and the identifiers. |
| `line_count` | Line Count | integer | — | no | none | computed, not stored, depends on `line_ids` | How many records were found. Shown as a button labelled "References". |

## 15.4 Display name

Fixed: the wizard's display name is always "Privacy Lookup", whatever the search terms are.

## 15.5 The record types excluded from the search

| Excluded record type | Why |
|---|---|
| `res.partner` | Already searched explicitly by the first part of the query |
| `res.users` | Already searched explicitly by the first part of the query |
| `mail.notification` | Deleted automatically with its message |
| `mail.followers` | Deleted automatically with its record |
| `discuss.channel.member` | Deleted automatically with its channel |
| `mail.message` | Searched explicitly by the second part of the query, on authorship |

Beyond that list, transient record types and record types with no table of their own are skipped.

## 15.6 Operations

| Operation | Behaviour |
|---|---|
| Look up | Builds and runs the query of [`calculations.md`](calculations.md) §11, replaces the whole line list with the result, and opens the line list. |
| Post the log | Creates the Privacy Log the first time there is anything to record, and updates it thereafter. |
| Open the lines | Opens the line list filtered to this wizard, grouped by record type. |

## 15.7 Access

Only the settings-administration group may use the wizard, and only that group may read, write,
create or delete it.

---

# 16. Privacy Lookup Wizard Line (`privacy.lookup.wizard.line`)

Reference page: [`../../references/entities/privacy.lookup.wizard.line.md`](../../references/entities/privacy.lookup.wizard.line.md).

**Transport name** `privacy.lookup.wizard.line`. **Storage name** `privacy_lookup_wizard_line`.
**Kind** transient. Same twenty-four-hour survival and same switched-off count limit as the wizard.

## 16.1 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Meaning |
|---|---|---|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `privacy.lookup.wizard` | no | none | — | The session this line belongs to. |
| `res_id` | Resource identifier | integer | — | yes | none | — | The identifier of the found record. |
| `res_name` | Resource name | single line text | — | no | none | computed and stored, depends on `res_model` and `res_id` | The found record's display name, read with elevated rights; when the record has no display name, the record type's name followed by a slash and the identifier. Left untouched when the record no longer exists. |
| `res_model_id` | Related Document Model | many to one | `ir.model` | no | none | — | The found record's type. Deleting the record type deletes the line. |
| `res_model` | Document Model | single line text | — | no | none | related through `res_model_id.model`, stored, read-only | The transport name. |
| `resource_ref` | Record | reference to any record type | any | no | none | computed, not stored, depends on `res_model`, `res_id` and `is_unlinked`; writing it sets `res_id` | A clickable link to the record, offered only when the reader may read it and it has not been deleted. |
| `has_active` | Has Active | boolean | — | no | none | computed and stored, depends on `res_model_id` | Whether the found record's type supports archiving. |
| `is_active` | Is Active | boolean | — | no | none | — | The archive switch. Toggling it archives or restores the found record immediately. |
| `is_unlinked` | Is Unlinked | boolean | — | no | none | — | Set once the line's record has been deleted. |
| `execution_details` | Execution Details | single line text | — | no | the empty string | — | What was done to this record, as one line of text. |

## 16.2 The reference selection

The reference field may point at any record type; its choice list is every record type in the
database, read with elevated rights, presented as the transport name and the record type's name.

## 16.3 Operations

| Operation | Behaviour |
|---|---|
| Toggle the archive switch | Writes the switch's new value onto the found record with elevated rights, and sets the execution details to `Archived <record type name> #<identifier>` or `Unarchived <record type name> #<identifier>`. Does nothing when the line has no record type or no identifier. |
| Delete | Refuses when the line is already deleted. Otherwise deletes the found record with elevated rights, sets the execution details to `Deleted <record type name> #<identifier>`, and marks the line as deleted. |
| Archive the selection | For each selected line that supports archiving and is currently active, clears the switch and applies it. |
| Delete the selection | For each selected line that is not already deleted, deletes it. |
| Open the record | Opens the found record's form. |

## 16.4 Access

The settings-administration group may read, write and create; nobody may delete the line records
themselves — a line is removed only when the whole wizard is rebuilt.

---

# 17. Geocoding Provider (`base.geo_provider`)

Reference page: [`../../references/entities/base.geo_provider.md`](../../references/entities/base.geo_provider.md).

**Transport name** `base.geo_provider`. **Storage name** `base_geo_provider`. **Kind** persistent.

## 17.1 Purpose

The registry of address-resolution providers. Each record names a provider and carries the token
that selects its code path.

The same entity is described from the Contact's point of view in
[`../contacts-and-organizations/entities.md`](../contacts-and-organizations/entities.md); this
folder specifies the provider registry and the calls, that folder specifies the coordinates stored
on the Contact.

## 17.2 Field table

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `tech_name` | Technical Name | single line text | no | none | The token that selects the code path. |
| `name` | Name | single line text | no | none | The label shown in the settings. |

## 17.3 Shipped providers

| External identifier | Technical name | Name |
|---|---|---|
| `base_geolocalize.geoprovider_open_street` | `openstreetmap` | `Open Street Map` |
| `base_geolocalize.geoprovider_google_map` | `googlemap` | `Google Place Map` |

## 17.4 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none.
- **Ordering**: platform default; the fallback provider is therefore the lowest identifier, which is
  the first shipped provider.
- **Display name**: the name.
- **Archival, company**: none.
- **Access**: every internal user may read; nobody may create, write or delete through the access
  matrix. The form disables creation and deletion as well.

---

# 18. Geocoder (`base.geocoder`)

Reference page: [`../../references/entities/base.geocoder.md`](../../references/entities/base.geocoder.md).

**Transport name** `base.geocoder`. **Kind** abstract behaviour; no table, no fields.

## 18.1 Purpose

The dispatcher. It resolves which provider is configured, builds the query string in the shape that
provider expects, calls it, and converts the answer to a latitude and longitude pair.

## 18.2 Operations

| Operation | Behaviour |
|---|---|
| Resolve the provider | Reads the system parameter `base_geolocalize.geo_provider` with elevated rights. When it is empty, or points at a record that no longer exists, the first provider in the table is used instead. |
| Build the query string | Dispatches to the per-provider builder whose name ends with the provider's technical name when one exists, otherwise to the default builder. |
| Find coordinates | Dispatches to the per-provider caller whose name ends with the provider's technical name. When no such caller exists the operation fails with "Provider %s is not implemented for geolocation service.", the placeholder being the technical name. A failure the caller raised deliberately is passed on; any other failure is logged at debug level and the answer becomes "not found". |
| Reverse lookup | Calls the first provider's reverse service with a latitude and a longitude. Refuses while tests are running with "OpenStreetMap calls disabled in testing environment." |
| Describe a position | Takes the city and the country code from the request's geographic lookup; when either is missing, falls back to the reverse lookup. Assembles postal code, city and country into one line; when nothing is known the answer is "Unknown". |

The two callers, the two builders and the assembly rule are specified in
[`calculations.md`](calculations.md) §13.

---

# 19. Google Service (`google.service`)

Reference page: [`../../references/entities/google.service.md`](../../references/entities/google.service.md).

**Transport name** `google.service`. **Kind** abstract behaviour; no table, no fields.

## 19.1 Purpose

The shared delegated-authorisation helper for one family of outside services. It builds the consent
address, exchanges an authorisation code for tokens, refreshes an expired token and performs the
request itself.

## 19.2 The three fixed addresses

| Purpose | Address |
|---|---|
| Consent | `https://accounts.google.com/o/oauth2/auth` |
| Token exchange and refresh | `https://accounts.google.com/o/oauth2/token` |
| Service calls | `https://www.googleapis.com` |

The request helper asserts that the host it is about to call is the host of one of the last two
addresses; any other host is refused before the request is made.

## 19.3 Credentials

| System parameter | Meaning |
|---|---|
| `google_<service>_client_id` | The public application identifier for the named service. Never secret; it appears in the consent address in clear. |
| `google_<service>_client_secret` | The secret. Read only inside a request; never returned to a caller. |

## 19.4 Operations

| Operation | Behaviour |
|---|---|
| Build the consent address | Assembles a query from the response kind `code`, the application identifier, the requested scope and the return address, plus, when supplied, the state, the approval prompt and the access kind. |
| Exchange the code | Posts the code, the application identifier, the secret, the grant kind `authorization_code` and the return address to the token address, and returns the access token, the refresh token and the lifetime in seconds. On a protocol failure it raises the configuration warning "Something went wrong during your token generation. Maybe your Authorization Code is invalid or already expired". |
| Refresh the token | Posts the refresh token, the application identifier, the secret and the grant kind `refresh_token`, and returns the new access token and its lifetime. |
| Perform a request | Accepts the methods `GET`, `DELETE`, `POST`, `PATCH` and `PUT`; anything else fails with "Method not supported [%s] not in [GET, POST, PUT, PATCH or DELETE]!". Twenty-second timeout. A no-content answer is turned into false; a not-found answer is turned into the empty string. The secret is masked in the debug log to its first four characters followed by twelve asterisks. The answer is a triple of status, body and the moment the service says it answered. |

## 19.5 The callback route

The consent flow returns to `/google_account/authentication`. That route is specified in
[`interfaces.md`](interfaces.md) §5.2.

---

# 20. Microsoft Service (`microsoft.service`)

Reference page: [`../../references/entities/microsoft.service.md`](../../references/entities/microsoft.service.md).

**Transport name** `microsoft.service`. **Kind** abstract behaviour; no table, no fields.

## 20.1 Purpose

The same helper for the other service family, with two differences: the consent and token addresses
are themselves configurable, and the state travelling through the consent round-trip carries four
values instead of two.

## 20.2 The three addresses

| Purpose | Default address | System parameter that overrides it |
|---|---|---|
| Consent | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` | `microsoft_account.auth_endpoint` |
| Token exchange and refresh | `https://login.microsoftonline.com/common/oauth2/v2.0/token` | `microsoft_account.token_endpoint` |
| Service calls | `https://graph.microsoft.com` | none |

The request helper asserts the host is the host of the default token address or of the service
address.

## 20.3 Credentials

| System parameter | Meaning |
|---|---|
| `microsoft_<service>_client_id` | The public application identifier. |
| `microsoft_<service>_client_secret` | The secret. |
| `microsoft_redirect_uri` | The return address registered with the provider. Shipped with the value `urn:ietf:wg:oauth:2.0:oob`. |

## 20.4 The state carried through consent

| Key | Value |
|---|---|
| `d` | The database name |
| `s` | The service name |
| `f` | The address to return the browser to |
| `u` | The database's universally unique identifier |

## 20.5 Operations

| Operation | Behaviour |
|---|---|
| Calendar scope | Returns the fixed scope `offline_access openid Calendars.ReadWrite`. |
| Build the consent address | Assembles the response kind `code`, the application identifier, the state above as structured text, the scope, the return address and the access kind `offline`. |
| Exchange the code | Posts the code, the identifier, the secret, the grant kind `authorization_code`, the calendar scope and the return address; returns the access token, the refresh token and the lifetime. On a protocol failure it raises the configuration warning "Something went wrong during your token generation. Maybe your Authorization Code is invalid". |
| Refresh the token | Posts the refresh token and the grant kind `refresh_token`; returns the new access token, its lifetime and, in the extended form, the new refresh token. |
| Perform a request | The same five methods, the same refusal text, a twenty-second timeout, and a not-found or no-content answer turned into an empty mapping. |

## 20.6 Tokens stored on the user

The package adds three fields to the User; they are listed in §33.11.

---

# 21. Google Gmail Mixin (`google.gmail.mixin`)

Reference page: [`../../references/entities/google.gmail.mixin.md`](../../references/entities/google.gmail.mixin.md).

**Transport name** `google.gmail.mixin`. **Kind** abstract behaviour. It is mixed into the outgoing
Mail Server and the Incoming Mail Server, both owned by
[`../messaging-and-activities/`](../messaging-and-activities/).

## 21.1 Purpose

To let a mail server authenticate against one electronic mail provider with a delegated
authorisation instead of a password. The behaviour stores the long-lived refresh token, caches the
short-lived access token, builds the consent address, and produces the authentication string the
mail protocols expect.

## 21.2 Field table

| Identifier | Full name | Type | Required | Default | Computed | Visible to | Meaning |
|---|---|---|---|---|---|---|---|
| `active` | Active | boolean | no | true | — | everyone | Inherited by the host record. |
| `google_gmail_refresh_token` | Refresh Token | single line text | no | none | — | settings-administration group only | The long-lived token. Never copied on duplication. |
| `google_gmail_access_token` | Access Token | single line text | no | none | — | settings-administration group only | The short-lived token. Never copied. |
| `google_gmail_access_token_expiration` | Access Token Expiration Timestamp | integer | no | none | — | settings-administration group only | The second-precision instant the access token expires. Never copied. |
| `google_gmail_uri` | Resource identifier | single line text | no | none | computed, not stored | settings-administration group only | The consent address, or empty when the credentials are not configured. Its help text reads "The URL to generate the authorization code from Google". |

## 21.3 Constants

| Constant | Value |
|---|---|
| Requested scope | `https://mail.google.com/ https://www.googleapis.com/auth/userinfo.email` |
| Token request timeout | 5 seconds |
| Token validity threshold | 10 seconds — the request timeout plus five |
| Token exchange address | `https://oauth2.googleapis.com/token` |
| Return address | the database's base address followed by `/google_gmail/confirm` |
| Relay address default | `https://gmail.api.odoo.com`, overridden by the system parameter `mail.server.gmail.iap.endpoint` |

## 21.4 The consent address

Built only when both the application identifier and the secret are configured. Its query carries the
application identifier, the return address, the response kind `code`, the scope above, the access
kind `offline`, the prompt `consent` — the last two are what make the provider return a refresh
token — and a state holding the record's transport name, its identifier and a cross-site protection
token.

## 21.5 The cross-site protection token

A keyed digest computed over the purpose text `google_gmail_oauth` and the pair of the record's
transport name and identifier, produced with elevated rights. The callback route refuses any call
whose token does not match, which prevents a third party from making an administrator disconnect a
mail server by following a crafted link.

## 21.6 Operations

| Operation | Behaviour |
|---|---|
| Open the consent address | Specified in [`workflows.md`](workflows.md) §14.1. |
| Fetch the refresh token | Exchanges the authorisation code; returns the refresh token, the access token and the expiry instant computed as the current second plus the returned lifetime. |
| Fetch an access token | When the credentials are configured, refreshes directly; otherwise goes through the relay. |
| Fetch a token | Posts the application identifier, the secret, the grant kind, the return address and the extra values to the token exchange address with the five-second timeout. Any non-successful answer fails with "An error occurred when fetching the access token." |
| Fetch an access token through the relay | Calls the relay path `/api/mail_oauth/1/gmail_access_token` with the refresh token and the database identifier. A transport failure fails with "Oops, we could not authenticate you. Please try again later."; an answer carrying an error is translated by the table in §21.7. |
| Build the authentication string | Specified in [`calculations.md`](calculations.md) §15.1. |

## 21.7 Relay error translation

| Error token | Message shown |
|---|---|
| `not_configured` | "Something went wrong. Try again later" |
| `no_subscription` | "You don't have an active subscription" |
| anything else | the token itself, shown unchanged |

The same table is used by the other electronic mail provider behaviour.

---

# 22. Microsoft Outlook Mixin (`microsoft.outlook.mixin`)

Reference page: [`../../references/entities/microsoft.outlook.mixin.md`](../../references/entities/microsoft.outlook.mixin.md).

**Transport name** `microsoft.outlook.mixin`. **Kind** abstract behaviour, mixed into the same two
mail-server entities.

## 22.1 Field table

| Identifier | Full name | Type | Required | Default | Computed | Visible to | Meaning |
|---|---|---|---|---|---|---|---|
| `active` | Active | boolean | no | true | — | everyone | Inherited by the host record. |
| `microsoft_outlook_refresh_token` | Outlook Refresh Token | single line text | no | none | — | settings-administration group only | The long-lived token. Never copied. |
| `microsoft_outlook_access_token` | Outlook Access Token | single line text | no | none | — | settings-administration group only | The short-lived token. Never copied. |
| `microsoft_outlook_access_token_expiration` | Outlook Access Token Expiration Timestamp | integer | no | none | — | settings-administration group only | The second-precision expiry instant. Never copied. |
| `microsoft_outlook_uri` | Authentication Resource identifier | single line text | no | none | computed, not stored | settings-administration group only | The consent address. Its help text reads "The URL to generate the authorization code from Outlook". |

## 22.2 Constants

| Constant | Value |
|---|---|
| Base scope | `openid email offline_access https://outlook.office.com/User.read` followed by the host record's own scope |
| Outgoing server scope | `https://outlook.office.com/SMTP.Send` |
| Incoming server scope | `https://outlook.office.com/IMAP.AccessAsUser.All` |
| Token request timeout | 5 seconds |
| Token validity threshold | 10 seconds |
| Consent and token base address | `https://login.microsoftonline.com/common/oauth2/v2.0/`, overridden by the system parameter `microsoft_outlook.endpoint` |
| Return address | the database's base address followed by `/microsoft_outlook/confirm` |
| Relay address default | `https://outlook.api.odoo.com`, overridden by the system parameter `mail.server.outlook.iap.endpoint` |

## 22.3 Differences from the other provider behaviour

| Point | This behaviour | The other behaviour |
|---|---|---|
| Consent address | Built as the base address followed by `authorize` and the query | A fixed address |
| Response mode | The query carries `query` explicitly | Not sent |
| Refresh answer | Returns a **new refresh token** as well, which is written back onto the record | Returns only the access token |
| Failure text on token fetch | "An error occurred when fetching the access token. %s", the placeholder being the provider's own description, or "Unknown error." when none is given | "An error occurred when fetching the access token." with no placeholder |
| Missing refresh token | Fails with "Please connect with your Outlook account before using it." | Attempts the fetch and fails inside it |
| Cross-site protection purpose | `microsoft_outlook_oauth` | `google_gmail_oauth` |

---

# 23. Transifex Translation (`transifex.translation`)

Reference page: [`../../references/entities/transifex.translation.md`](../../references/entities/transifex.translation.md).

**Transport name** `transifex.translation`. **Kind** abstract behaviour; no table, no fields.

## 23.1 Purpose

To turn a translated term into a deep link into the outside translation platform, so that a user who
spots a bad translation can propose a correction where the translation actually lives.

## 23.2 Operations

| Operation | Behaviour |
|---|---|
| Read the project map | Reads the translation-platform configuration files found beside the package directories and returns, for each package, the project it belongs to. The result is cached for the life of the process. Two section spellings are accepted; see [`calculations.md`](calculations.md) §16.1. |
| Update the address | Fills the platform address on each translation handed to it. Specified in [`calculations.md`](calculations.md) §16.2. |

## 23.3 What it is mixed into

- The Code Translation entity of §24, whose address field is computed by this behaviour.
- The platform's generic translation reader, which is extended so that every value it returns for a
  record that has an external identifier also carries the package name and the platform address.

---

# 24. Code Translation (`transifex.code.translation`)

Reference page: [`../../references/entities/transifex.code.translation.md`](../../references/entities/transifex.code.translation.md).

**Transport name** `transifex.code.translation`. **Storage name** `transifex_code_translation`.
**Kind** persistent, with audit fields switched off.

## 24.1 Purpose

A cache of the translatable terms that live in program text rather than in records, so that they can
be listed, searched, filtered by package and by language, and linked to the translation platform.

## 24.2 Field table

| Identifier | Full name | Type | Required | Default | Computed | Meaning |
|---|---|---|---|---|---|---|
| `source` | Code | long text | no | none | — | The source term. |
| `value` | Translation Value | long text | no | none | — | The translated term; empty when the term has not been translated. |
| `module` | Module | single line text | no | none | — | The package the term belongs to. Its help text reads "Module this term belongs to". |
| `lang` | Language | selection over the installed languages | no | none | — | The language. Values are not validated against the list, so a language that is later uninstalled does not break the row. |
| `transifex_url` | Transifex web address | single line text | no | none | computed, not stored | The deep link into the translation platform. Its help text reads "Propose a modification in the official version of the system". |

## 24.3 Loading and reloading

| Operation | Behaviour |
|---|---|
| Load | Takes an exclusive table lock without waiting. If the lock is unavailable, the operation returns false and does nothing — this is what guarantees that two workers never load the same pair twice. With the lock held, the packages default to every installed package and the languages default to every installed language except the base language; every pair already present in the table is skipped; the remaining pairs are read from the program text and created with elevated rights. |
| Open the list | Loads first, then opens the list view. |
| Reload | Empties the table and loads again. Run by a scheduled job every seven days. |

## 24.4 Uniqueness, ordering, display name, archival, company

- **Uniqueness**: none in the database; the skip-what-is-already-loaded rule keeps one set of rows
  per package and language.
- **Ordering**: platform default.
- **Display name**: platform default.
- **Archival, company**: none.
- **Access**: the settings-administration group may read; nobody may write, create or delete.

---

# 25. Onboarding (`onboarding.onboarding`)

Reference page: [`../../references/entities/onboarding.onboarding.md`](../../references/entities/onboarding.onboarding.md).

**Transport name** `onboarding.onboarding`. **Storage name** `onboarding_onboarding`. **Kind** persistent.

## 25.1 Purpose

One setup panel: a banner of step cards shown at the top of a screen until every step is done and
the user closes it.

## 25.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Stored | Meaning |
|---|---|---|---|---|---|---|---|---|
| `name` | Name of the onboarding | single line text | — | no | none | — | yes | The label. Translatable. |
| `route_name` | One word name | single line text | — | yes | none | — | yes | The single word that identifies the panel in its address, `/onboarding/<route name>`. |
| `step_ids` | Onboarding steps | many to many | `onboarding.onboarding.step` | no | none | — | yes | The cards of this panel, in their own sequence order. |
| `text_completed` | Message at completion | single line text | — | no | "Nice work! Your configuration is done." | — | yes | Shown once, when the last step completes. Its help text reads "Text shown on onboarding when completed". |
| `is_per_company` | Should be done per company? | boolean | — | no | none | computed, depends on the progress records, their companies, the steps and each step's own per-company flag | no | True as soon as any progress record carries a company or any step is per-company. Once true it stays true, even if the last per-company step is removed, so that existing progress never has to be merged. |
| `panel_close_action_name` | Closing action | single line text | — | no | none | — | yes | The name of the operation the client calls when the panel is closed. Its help text reads "Name of the onboarding model action to execute when closing the panel." |
| `current_progress_id` | Onboarding Progress | many to one | `onboarding.progress` | no | none | computed, not stored, depends on the current company and on the progress records | no | The progress record for the reader's company, or the company-less one. |
| `current_onboarding_state` | Completion State | selection | — | no | none | computed with the previous field | no | `not_done`, `just_done` or `done`; `not_done` when there is no progress record at all. |
| `is_onboarding_closed` | Was panel closed? | boolean | — | no | none | computed with the previous field | no | Whether the reader's company has dismissed the panel. |
| `progress_ids` | Onboarding Progress Records | one to many | `onboarding.progress` through `onboarding_id` | no | none | — | yes | Every progress record across every company. Read-only. Its help text reads "All Onboarding Progress Records (across companies)." |
| `sequence` | Sequence | integer | — | no | 10 | — | yes | Display order. |

## 25.3 Constraint

| Name | Condition | Message |
|---|---|---|
| `_route_name_uniq` | The one-word name is unique across panels | "Onboarding alias must be unique." |

## 25.4 Ordering, display name, archival, company

- **Ordering**: by ascending sequence, then by descending identifier.
- **Display name**: the name.
- **Archival**: none.
- **Company**: indirect. The panel itself is global; its completion is per company when
  `is_per_company` is true and global otherwise.

## 25.5 Operations

| Operation | Behaviour |
|---|---|
| Write | After the write, if the step list changed, every progress record recomputes which progress-step records it points at. |
| Close | Marks the reader's progress record as closed. |
| Close by external identifier | Resolves the panel from an external identifier and closes it; does nothing at all when the identifier resolves to nothing. |
| Refresh the progress records | For each panel that is per-company and whose progress records carry no company, deletes those records and creates fresh ones. Used when a step becomes per-company. |
| Toggle the visibility | Flips the closed flag of the reader's progress record. |
| Find or create the progress | Creates a progress record for every panel that has none for the reader's context, then returns them all. |
| Prepare the rendering values | Returns the closing operation name, the fixed record type `onboarding.onboarding`, the step list, the state mapping of [`state-machines.md`](state-machines.md) §5.3 and the completion message. |

---

# 26. Onboarding Step (`onboarding.onboarding.step`)

Reference page: [`../../references/entities/onboarding.onboarding.step.md`](../../references/entities/onboarding.onboarding.step.md).

**Transport name** `onboarding.onboarding.step`. **Storage name** `onboarding_onboarding_step`.
**Kind** persistent.

## 26.1 Purpose

One card. A step may belong to several panels at once, which is why the link is many-to-many.

## 26.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Stored | Meaning |
|---|---|---|---|---|---|---|---|---|
| `onboarding_ids` | Onboardings | many to many | `onboarding.onboarding` | no | none | — | yes | The panels this card appears on. |
| `title` | Title | single line text | — | no | none | — | yes | The card heading. Translatable. |
| `description` | Description | single line text | — | no | none | — | yes | The card subtitle. Translatable. |
| `button_text` | Button text | single line text | — | yes | "Let's do it" | — | yes | The label of the card's action button. Translatable. Its help text reads "Text on the panel's button to start this step". |
| `done_icon` | Font Awesome Icon when completed | single line text | — | no | `fa-star` | — | yes | The icon name used once the card is complete. When empty, the client falls back to `fa-check`. |
| `done_text` | Text to show when step is completed | single line text | — | no | "Step Completed!" | — | yes | Translatable. When empty, the client falls back to "All done!". |
| `step_image` | Step Image | binary | — | no | none | — | yes | The illustration. Its placeholder is the shipped default onboarding image. |
| `step_image_filename` | Step Image Filename | single line text | — | no | none | — | yes | The uploaded file's name. |
| `step_image_alt` | Alt Text for the Step Image | single line text | — | no | `Onboarding Step Image` | — | yes | The alternative text. Translatable. Its help text reads "Show when impossible to load the image". |
| `panel_step_open_action_name` | Opening action | single line text | — | no | none | — | yes | The operation the client calls when the card is clicked. Its help text reads "Name of the onboarding step model action to execute when opening the step, e.g. action_open_onboarding_1_step_1". |
| `current_progress_step_id` | Step Progress | many to one | `onboarding.progress.step` | no | none | computed, not stored, depends on the current company and the progress-step records | no | The progress-step record for the reader's company. |
| `current_step_state` | Completion State | selection | — | no | none | computed with the previous field | no | `not_done`, `just_done` or `done`; `not_done` when there is no progress-step record. |
| `progress_ids` | Onboarding Progress Step Records | one to many | `onboarding.progress.step` through `step_id` | no | none | — | yes | Every progress-step record across companies. Read-only. |
| `is_per_company` | Is per company | boolean | — | no | true | — | yes | Whether the completion is tracked per company. |
| `sequence` | Sequence | integer | — | no | 10 | — | yes | Order within a panel. |

## 26.3 Constraint

| Trigger | Condition | Message |
|---|---|---|
| The panel list changes | A step that belongs to at least one panel must have an opening operation | "An "Opening Action" is required for the following steps to be linked to an onboarding panel: %(step_titles)s", the placeholder being the list of offending step titles |

## 26.4 Ordering, display name, archival, company

- **Ordering**: by ascending sequence, then by ascending identifier.
- **Display name**: the title.
- **Archival**: none.
- **Company**: through the per-company flag; see §26.5.

## 26.5 Behaviour on write

1. Note which steps are about to have their per-company flag actually changed, and which panels the
   step already belonged to.
2. Apply the write.
3. For every step whose per-company flag changed, delete all of its progress-step records — the
   completion has to start again in the new granularity.
4. Ask every panel the step now belongs to to refresh its progress records.
5. For panels the step has just joined, recompute which progress-step records their progress
   records point at.

## 26.6 Operations

| Operation | Behaviour |
|---|---|
| Mark as just done | Creates the missing progress-step records for the reader's context, then moves those in state `not_done` to `just_done` and returns the steps that actually moved. |
| Validate a step by external identifier | Resolves the step; returns `NOT_FOUND` when it does not exist, `JUST_DONE` when the step moved, and `WAS_DONE` when it was already complete. |
| Create the progress steps | For every progress record of the step's panels that belongs to the reader's company or to no company, creates one progress-step record per step, linked to the matching progress records, carrying the reader's company when the step is per-company and no company otherwise. |

---

# 27. Onboarding Progress Tracker (`onboarding.progress`)

Reference page: [`../../references/entities/onboarding.progress.md`](../../references/entities/onboarding.progress.md).

**Transport name** `onboarding.progress`. **Storage name** `onboarding_progress`. **Kind** persistent.

## 27.1 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Stored | Meaning |
|---|---|---|---|---|---|---|---|---|
| `onboarding_state` | Onboarding progress | selection | — | no | none | computed, depends on the panel's step list and on each progress step's state | yes | `not_done` or `done`. It is `done` exactly when the number of progress steps in state `just_done` or `done` equals the number of steps on the panel. |
| `is_onboarding_closed` | Was panel closed? | boolean | — | no | false | — | yes | Whether the panel has been dismissed for this company. |
| `company_id` | Company | many to one | `res.company` | no | none | — | yes | The company this progress belongs to; empty for a global panel. Deleting the company deletes the progress. |
| `onboarding_id` | Related onboarding tracked | many to one | `onboarding.onboarding` | yes | none | — | yes | The panel. Indexed. Deleting the panel deletes the progress. |
| `progress_step_ids` | Progress Steps Trackers | many to many | `onboarding.progress.step` | no | none | — | yes | The per-step progress records this panel-level progress aggregates. |

## 27.2 Unique index

A unique index named `_onboarding_company_uniq` covers the panel together with the company, where a
missing company counts as zero. One progress record therefore exists per panel and company, and at
most one company-less progress record per panel.

## 27.3 Ordering, display name, archival

- **Ordering**: platform default.
- **Display name**: the panel.
- **Archival**: none.

## 27.4 Operations

| Operation | Behaviour |
|---|---|
| Recompute the progress steps | Repoints the progress-step list at the current progress-step record of every step of the panel. |
| Close | Sets the closed flag. |
| Toggle the visibility | Inverts the closed flag. |
| Read and consolidate the state | Specified in [`state-machines.md`](state-machines.md) §5.3. It is the only operation that turns `just_done` into `done`, and it is called only when the panel is rendered. |

---

# 28. Onboarding Progress Step Tracker (`onboarding.progress.step`)

Reference page: [`../../references/entities/onboarding.progress.step.md`](../../references/entities/onboarding.progress.step.md).

**Transport name** `onboarding.progress.step`. **Storage name** `onboarding_progress_step`.
**Kind** persistent.

## 28.1 Field table

| Identifier | Full name | Type | Target | Required | Default | Meaning |
|---|---|---|---|---|---|---|
| `progress_ids` | Related Onboarding Progress Tracker | many to many | `onboarding.progress` | no | none | The panel-level progress records this step progress feeds. |
| `step_state` | Onboarding Step Progress | selection | — | no | `not_done` | `not_done`, `just_done` or `done`. |
| `step_id` | Onboarding Step | many to one | `onboarding.onboarding.step` | yes | none | The step. Indexed. Deleting the step deletes the progress. |
| `company_id` | Company | many to one | `res.company` | no | none | The company, or empty for a step that is not per-company. Deleting the company deletes the progress. |

## 28.2 Unique index

A unique index named `_company_uniq` covers the step together with the company, where a missing
company counts as zero.

## 28.3 Operations

| Operation | Behaviour |
|---|---|
| Consolidate | Moves every record in `just_done` to `done` and returns those that moved. |
| Mark as just done | Moves every record in `not_done` to `just_done` and returns those that moved. A record already in `just_done` or `done` is untouched. |

---

# 29. Tour (`web_tour.tour`)

Reference page: [`../../references/entities/web_tour.tour.md`](../../references/entities/web_tour.tour.md).

**Transport name** `web_tour.tour`. **Storage name** `web_tour_tour`. **Kind** persistent.

## 29.1 Purpose

One guided walkthrough: where it starts, what it does, what it says at the end, and which users have
already seen it.

## 29.2 Field table

| Identifier | Full name | Type | Target | Required | Default | Computed | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | single line text | — | yes | none | — | The identifier a client uses to start the tour. Unique. |
| `step_ids` | Steps | one to many | `web_tour.tour.step` through `tour_id` | no | none | — | The ordered steps. Editable on the form only for a recorded tour. |
| `url` | Starting URL | single line text | — | no | `/odoo` | — | Where the browser goes before the first step. |
| `sharing_url` | Sharing web address | single line text | — | no | none | computed, depends on `name` | The address that starts this tour for a colleague. Formula in [`calculations.md`](calculations.md) §17.1. |
| `rainbow_man_message` | Closing message | rich text | — | no | "<b>Good job!</b> You went through all steps of this tour." | — | The celebration message shown at the end. Translatable. |
| `sequence` | Sequence | integer | — | no | 1000 | — | Which tour is offered first. |
| `custom` | Custom | boolean | — | no | false | — | True when the tour was recorded in the browser and therefore lives entirely in these records; false when the tour is shipped as program text and these records only describe it. |
| `user_consumed_ids` | Users who consumed the tour | many to many | `res.users` | no | none | — | Who has already been through it. |

## 29.3 Constraint

| Name | Condition | Message |
|---|---|---|
| `_uniq_name` | The name is unique | "A tour already exists with this name . Tour's name must be unique!" — reproduced including its spacing |

## 29.4 Ordering, display name, archival, company

- **Ordering**: by ascending sequence, then by name, then by identifier.
- **Display name**: the name.
- **Archival, company**: none.
- **Creation**: disabled on both the form and the list; tours arrive with their package or from the
  in-browser recorder.

## 29.5 Operations

| Operation | Behaviour |
|---|---|
| Consume | When the reader is an internal user, adds them to the consumed list of the tour with the given name, with elevated rights, and then returns the next tour to run. |
| Get the current tour | Returns nothing unless the reader is an internal user whose tour switch is on. Otherwise searches for the first non-recorded tour the reader has not consumed and returns its description; returns nothing when there is none. |
| Get a tour by name | Returns the description of the tour with that name. |
| Describe | Returns the name, the starting address, the recorded flag, the list of step descriptions and the closing message, and deliberately drops the identifier. |
| Export | Builds a client script that registers the tour under its name with its starting address and its steps, stores it as an attachment named after the tour with the script media type, attached to the tour itself, and returns a download address for it. |

---

# 30. Tour Step (`web_tour.tour.step`)

Reference page: [`../../references/entities/web_tour.tour.step.md`](../../references/entities/web_tour.tour.step.md).

**Transport name** `web_tour.tour.step`. **Storage name** `web_tour_tour_step`. **Kind** persistent.

## 30.1 Field table

| Identifier | Full name | Type | Target | Required | Default | Meaning |
|---|---|---|---|---|---|---|
| `trigger` | Trigger | single line text | — | yes | none | The selector of the element the step acts on. |
| `content` | Content | single line text | — | no | none | The bubble text. Omitted from the description when empty. |
| `tooltip_position` | Tooltip position | selection | — | no | `bottom` | `bottom` "Bottom", `top` "Top", `right` "Right", `left` "left" — the last label is reproduced with its lower-case spelling. |
| `tour_id` | Tour | many to one | `web_tour.tour` | yes | none | The owning tour. Indexed. Deleting the tour deletes the step. |
| `run` | Run | single line text | — | no | none | What to do at the element: a click, a typed value, or nothing. |
| `sequence` | Sequence | integer | — | no | none | Order within the tour. |

## 30.2 Ordering and description

- **Ordering**: by ascending sequence, then by ascending identifier.
- **Description**: each step is described by its trigger, its content, its run instruction and its
  tooltip position, the last being renamed to the client's spelling `tooltipPosition`; the
  identifier is dropped and an empty content is omitted entirely.

---

# 31. Sparse Fields Test (`sparse_fields.test`)

Reference page: [`../../references/entities/sparse_fields.test.md`](../../references/entities/sparse_fields.test.md).

**Transport name** `sparse_fields.test`. **Storage name** `sparse_fields_test`. **Kind** transient.

## 31.1 Purpose

The reference record that demonstrates and exercises sparse storage: a technique in which many
rarely filled fields share one column instead of each having their own, so that a record type can
carry far more fields than a table may hold columns.

## 31.2 Field table

| Identifier | Full name | Type | Stored in | Meaning |
|---|---|---|---|---|
| `data` | Data | serialised | its own column | The mapping that holds every sparse value of the record. |
| `boolean` | Boolean | boolean | `data` | A sparse boolean. |
| `integer` | Integer | integer | `data` | A sparse integer. |
| `float` | Float | decimal number | `data` | A sparse decimal number. |
| `char` | Char | single line text | `data` | A sparse single line of text. |
| `selection` | Selection | selection with the values `one` "One" and `two` "Two" | `data` | A sparse choice. |
| `partner` | Partner | many to one to `res.partner` | `data` | A sparse link to a Contact. |

## 31.3 How sparse storage behaves

| Aspect | Rule |
|---|---|
| Storage | A sparse field has no column; it is neither stored nor copied on duplication by default. |
| Reading | The value is read out of the mapping held by the named serialised field; a missing key reads as empty. |
| Reading a link | After reading, a sparse link is reduced to the records that still exist, so a dangling identifier reads as empty. |
| Writing | A non-empty value is written into the mapping only when it differs from what is already there; an empty value removes the key from the mapping entirely. |
| Serialised column | Stored as text. An empty value is stored as nothing; a mapping is stored as its structured-text rendering; reading back an absent value yields an empty mapping. |
| Prefetching | A serialised field is never prefetched with the rest of the record. |

## 31.4 What the technique adds to the field catalogue

| Addition | Effect |
|---|---|
| Field kind | The field catalogue gains the kind `serialized`; deleting the package cascades to fields of that kind. |
| Serialisation link | The field catalogue gains `serialization_field_id` "Serialization Field", a many-to-one to the field catalogue itself, restricted to fields of kind `serialized` on the same record type, deleted in cascade. Its help text reads "If set, this field will be stored in the sparse structure of the serialization field, instead of having its own database column. This cannot be changed after creation." |
| Reflection | After the catalogue is refreshed, every sparse field's serialisation link is set to the identifier of the serialised field it names. A sparse field naming a serialised field that does not exist fails with "Serialization field "%(serialization_field)s" not found for sparse field %(sparse_field)s!". |
| Instantiation | A field whose catalogue row carries a serialisation link is instantiated as sparse, stored inside the named serialised field. |
| Form | The serialisation link is shown on both the record-type form and the field form, read-only for fields belonging to the base package. |

Two refusals guard the link; they are recorded in [`business-rules.md`](business-rules.md) §9.

## 31.5 Access

The settings-administration group may read, write and create; nobody may delete.

---

# 32. Attachment: the cloud-storage kind (`ir.attachment`)

Reference page: [`../../references/entities/ir.attachment.md`](../../references/entities/ir.attachment.md).

The Attachment belongs to [`../platform-foundation/`](../platform-foundation/). This folder
specifies the extra storage kind and the content extraction it gains.

## 32.1 The extra storage kind

| Identifier | Full name | Type | Addition |
|---|---|---|---|
| `type` | Type | selection | Gains the value `cloud_storage` "Cloud Storage". Removing the cloud-storage package turns such attachments into plain address attachments. |

## 32.2 Constants

| Constant | Value |
|---|---|
| Upload address lifetime | 300 seconds |
| Download address lifetime | 300 seconds, overridable per call through the reading context key `cloud_storage_download_url_time_to_expiry` |

## 32.3 Operations added

| Operation | Behaviour |
|---|---|
| Build the blob name | The attachment identifier, a slash, a freshly generated universally unique identifier, a slash, and the attachment name. |
| Build the blob address | Provider-specific; see [`calculations.md`](calculations.md) §14.1 and §14.2. |
| Build the download description | Provider-specific. Returns the signed address and the number of seconds until it expires. |
| Build the upload description | Provider-specific. Returns the signed address, the request method, the expected success status and, when needed, the request headers. |
| Serve over the web | When the attachment is of the cloud kind **and** the cloud configuration is complete, the request is answered with a redirection to the signed download address, cached until ten seconds before that address expires, or not cached at all when that leaves no time. Otherwise the platform's own answer is used. |
| Turn into a cloud attachment | Runs after creation when the creation call asked for it. Refuses with "Cloud Storage is not enabled" when no provider is configured. Otherwise rewrites each attachment: the media type is preserved deliberately so that it is not guessed again, the bytes are dropped, the kind becomes `cloud_storage` and the address becomes the freshly built blob address. |
| Bring back to local storage | For a cloud attachment, downloads the blob through its signed address with a ten-second timeout; a non-successful answer fails with "Failed to download attachment (%(id)s) from cloud: %(code)s - %(reason)s"; on success the attachment becomes a plain stored attachment holding the downloaded bytes and no address. |
| List the unsupported record types | Every record type that carries the main-attachment behaviour, plus, when the document-management package is installed, every record type carrying its own document behaviour and the document record type itself. Attachments of those record types are never uploaded to the cloud, because their bytes are read by business code. |

## 32.4 Content extraction added

The attachment-indexation package replaces the platform's content-extraction step. For a given set
of bytes it tries five extractors in this fixed order and keeps the first non-empty result:
word-processing documents, presentation documents, spreadsheet documents, open-standard office
documents, and page-description documents. When none of them produces anything, the platform's own
extraction is used. Every extracted text has null characters removed.

A one-entry cache keyed on the attachment checksum holds the last extracted text, so that
duplicating an attachment does not extract its content again; duplication explicitly seeds that
cache from the original before copying.

The five extractors are specified in [`calculations.md`](calculations.md) §18.

---

# 33. Extension points contributed to foreign entities

## 33.1 Scheduled Action (`ir.cron`)

Gains one operation: a jump to the automation rule that owns its server action. It exists only
because the Scheduled Action reaches its server action by delegation and therefore does not inherit
the jump automatically.

## 33.2 Module (`ir.module.module`)

| Identifier | Full name | Type | Default | Meaning |
|---|---|---|---|---|
| `imported` | Imported Module | boolean | false | The package was installed from an uploaded archive rather than found on disk. |
| `module_type` | Module Type | selection | `official` | `official` "Official Apps" or `industries` "Industries". The second value makes the catalogue read from the remote directory instead of the database. |

Behaviour added:

| Addition | Effect |
|---|---|
| Imported names | The set of installed imported package names is cached for the life of the process. |
| Loading domain | Imported packages are excluded from the set the platform loads at start-up; they have no program text to load. |
| Translation loading | For a package with no manifest on disk, translations are read from attachments named `<package>_<language>.po` at the address `/<package>/i18n/<language>.po`; each language falls back through its base languages; a failure to read one is logged as a warning and skipped, and a language with no translation at all is logged at information level. |
| Version | An imported package's installed version is forced to its latest version, because there is no on-disk version to compare with. |
| Icon | An imported package's icon is read from the attachment whose address equals the package's icon address. |
| Upgrade | After the platform marks packages for upgrade, every imported package that was marked is put straight back to `installed`: an imported package cannot be upgraded because its sources are not on disk. |
| Uninstall | Imported packages are computed **before** the platform uninstall runs, and are deleted from the catalogue afterwards, because an imported package can never be reinstalled without its archive. |
| Catalogue reads | A search or a read whose filter asks for the `industries` kind is served from the remote directory instead of the database; see [`workflows.md`](workflows.md) §9. |
| Search panel | A category range request on a filter asking for the `industries` kind returns the remote category list instead of the local one. |
| Client translations | Translations of an imported package are served to the client from the stored translation attachments, filtered to the terms marked as client terms. |
| Term extraction | Terms are extracted from the stored script and template attachments of an imported package, with the display path `addons` followed by the attachment address. |
| Install from the directory | Downloads the archive from the remote directory and opens the import wizard on it; see [`workflows.md`](workflows.md) §9.3. |

## 33.3 View Definition (`ir.ui.view`)

Views belonging to an imported package are added to the set validated as custom views: the newest
view per inheritance root, active, for the given record type, whose defining package is imported.
The result is the platform's own answer combined with the check of those views.

## 33.4 Request Routing (`ir.http`)

| Addition | Contributed by | Effect |
|---|---|---|
| Cloud-storage session keys | cloud storage | When a provider is configured, the session description carries the minimum file size in bytes and the list of record types whose attachments must not go to the cloud. |
| Autocomplete session key | company autocomplete | For an administrator, the session description carries whether the current company still needs its one-time enrichment. |
| Tour session keys | guided tours | The session description carries the reader's tour switch and the description of the next tour to run. |
| Imported translations | module import | The client translation bundle is assembled from the platform for ordinary packages and from the stored attachments for imported ones. |

## 33.5 Configuration Settings (`res.config.settings`)

Every setting is listed in [`configuration.md`](configuration.md) §1. The behaviours added are:

| Addition | Effect |
|---|---|
| Cloud provider selection | An empty selection that each provider package extends with its own value. |
| Save | Before saving, when a provider was configured and is being changed, the outgoing provider is asked whether it may be switched off. After saving, the configuration is re-read; a chosen provider with an incomplete configuration fails with "Please configure the Cloud Storage before enabling it"; a complete configuration that differs from the one before the save is verified against the provider. |
| Read | The minimum file size in megabytes is derived from the stored byte count. |
| Provider verification | Each provider uploads and downloads a probe blob whose name is `0/` followed by the current instant and `.txt`, and refuses to be enabled when either fails. |

## 33.6 Contact (`res.partner`)

| Addition | Effect |
|---|---|
| Privacy lookup entry point | An operation that opens the privacy lookup wizard with the Contact's electronic mail address and name filled in. |
| Autocomplete widgets | On the form, the name, the tax registration number and the company registration number fields are given the autocomplete widget. |
| Address autocomplete widget | On the form, the street fields are given the address autocomplete widget, but only when the city list is enforced by the country catalogue. |
| Enrichment note | An operation that posts the enrichment result as an internal note on the Contact, rendered by the shipped enrichment template. |
| Directory lookups | Four operations: search by name, search by tax registration number, enrich by company registration number, enrich by regional tax number and enrich by internet domain. |
| Reference translation | Country, state, city, industry and language codes coming back from the directory are turned into links to the matching records; see [`calculations.md`](calculations.md) §10. |

## 33.7 Company (`res.company`)

| Identifier | Full name | Type | Default | Meaning |
|---|---|---|---|---|
| `iap_enrich_auto_done` | Enrich Done | boolean | false | Whether the one-time automatic enrichment already ran for this company. |

Behaviour: at creation, a company is enriched automatically unless tests are running, in which case
the flag is simply set. The automatic enrichment only proceeds when the reader is a system user, the
registry is ready and demonstration data is not being installed; it then enriches every company
whose flag is false and sets the flag on all of them, so that it can never loop.

## 33.8 Mail Server (`ir.mail_server`)

| Addition | Contributed by | Effect |
|---|---|---|
| Authentication kind | first provider | Gains `gmail` "Gmail OAuth Authentication"; removing the package resets affected servers to the default kind. |
| Authentication kind | second provider | Gains `outlook` "Outlook OAuth Authentication", likewise. |
| Explanatory text | both | A fixed paragraph replaces the platform's, explaining that only a user with a matching address may use the server unless a default sender parameter is set. |
| Encryption change | both | The platform's automatic reconfiguration on an encryption change is suppressed for a server of either delegated kind, so that the already correct port is not overwritten. |
| Kind change | both | Choosing the kind sets the host, the encryption to `starttls` and the port to 587; choosing anything else clears the three stored tokens. The first provider's host is `smtp.gmail.com`; the second provider's host is `smtp.outlook.com`. |
| Sender filter | both | Choosing the kind or changing the user name copies the user name into the sender filter. |
| Login | both | A server of the delegated kind greets the service and issues the extended authorisation command with the encoded authentication string; anything else falls through to the platform's password login. |
| Personal sending limit | second provider | A server of that kind may send the number of messages per minute given by the system parameter `mail.server.personal.limit.minutes_outlook`, or ten when that parameter is zero or unset, rather than the platform's own limit. |

## 33.9 Incoming Mail Server (`fetchmail.server`)

| Addition | Contributed by | Effect |
|---|---|---|
| Server kind | first provider | Gains `gmail` "Gmail OAuth Authentication". |
| Server kind | second provider | Gains `outlook` "Outlook OAuth Authentication". |
| Explanatory text | both | A fixed paragraph replacing the platform's. |
| Kind change | both | Choosing the kind sets the host, the secure flag and port 993; the first provider's host is `imap.gmail.com`, the second's is `imap.outlook.com`. Choosing anything else clears the three tokens and falls through to the platform's own reconfiguration. |
| Login | both | A server of the delegated kind authenticates with the extended mechanism and selects the inbox; anything else falls through. |
| Connection kind | both | A server of the delegated kind always uses the message-access protocol, never the post-office protocol. |

## 33.10 Base behaviour (`base`)

| Addition | Contributed by | Effect |
|---|---|---|
| Import templates | data import | Every record type gains an operation that returns the downloadable example files for importing into it: a list of label and address pairs, empty by default. |
| Field translations | translation platform | The platform's field-translation reader is extended: when the record has an external identifier whose package is part of the current run, every returned value gains the package name and the deep link into the translation platform. |
| Sparse parameter | sparse storage | Every record type accepts `sparse` as a valid field parameter. |

## 33.11 User (`res.users`)

| Identifier | Full name | Type | Contributed by | Default | Meaning |
|---|---|---|---|---|---|
| `tour_enabled` | Onboarding | boolean | guided tours | computed from the creation date, stored, writable | Whether guided tours run for this user. Computed as: the user is an administrator, no package carrying demonstration data is installed, and no test is running. |
| `outgoing_mail_server_type` | Outgoing Mail Server Type | selection | first provider, second provider | `default` | The personal mail server kind; gains `gmail` "Gmail" and `outlook` "Outlook". |
| `microsoft_calendar_rtoken` | Microsoft Refresh Token | single line text | second service family | none | The long-lived calendar token. Visible only to the settings-administration group; never copied. |
| `microsoft_calendar_token` | Microsoft User token | single line text | second service family | none | The short-lived calendar token. Same visibility. |
| `microsoft_calendar_token_validity` | Microsoft Token Validity | date and time | second service family | none | When that token expires. |

Behaviour added:

| Addition | Effect |
|---|---|
| Remote-file import permission | A hook that decides whether this user may import binary content by address rather than by value. It answers yes for the administration group and no otherwise, because fetching an arbitrary address ties up a worker. |
| Tour switch | An operation that writes the tour switch with elevated rights and returns its new value. |
| Personal mail server values | For each delegated kind, the host and the authentication kind to preset. |
| Personal mail server closing step | For each delegated kind, the consent address to open once the server is saved. |
| Calendar tokens | An operation that writes the three calendar token fields at once. |

## 33.12 Lead (`crm.lead`)

| Identifier | Full name | Type | Contributed by | Meaning |
|---|---|---|---|---|
| `reveal_id` | Reveal identifier | single line text | the lead bridge package | The correlation identifier of the lead-generation request that produced the lead. Indexed, skipping empty values. Carried over when two leads are merged. |

The lead itself belongs to
[`../customer-relationship-management/`](../customer-relationship-management/).

## 33.13 Module Uninstall Wizard (`base.module.uninstall`)

The list of packages the uninstall wizard displays is extended so that imported packages appear
in it.

## 33.14 Field Definition (`ir.model.fields`)

Covered in §31.4: the `serialized` kind and the serialisation link.
