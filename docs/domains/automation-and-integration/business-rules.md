# Business rules

Every refusal, constraint, invariant, permission check and locking rule of the domain, numbered with
a stable identifier. The prefix `AUT` stands for *automation and integration*; the number never
changes once assigned, and gaps in the sequence are deliberate so that later rules can be inserted
next to the rules they belong with.

Each rule states the record it applies to, the condition that makes it fail, the exact text the
system shows, and what happens afterwards. Message texts are **reproduced**, not authored: they are
part of observable behaviour and a rebuild must produce them character for character. Placeholders
are written as they appear in the stored text and described in words underneath.

Three kinds of refusal are distinguished:

| Kind | Meaning | When it is evaluated |
|---|---|---|
| Constraint | A validation attached to a record type. | On every create and on every write that touches one of the fields it watches. |
| Guard | A check inside an operation. | When that operation is invoked. |
| Permission | An access right, a record rule, a group test or a field visibility restriction. | On every read, write, create or delete, before anything else. |

A **warning** is not a refusal: it appears while a form is open, blocks nothing, and is listed here
only where a matching constraint exists.

## Index of rules

| Rule | Subject | Kind |
|---|---|---|
| [`AUT-001`](#aut-001) | Automation Rule name is required | Constraint |
| [`AUT-002`](#aut-002) | Message trigger needs a discussion thread | Constraint |
| [`AUT-003`](#aut-003) | Delay must not be negative | Constraint |
| [`AUT-004`](#aut-004) | Action record type must match the rule | Constraint |
| [`AUT-005`](#aut-005) | Actions carrying their own warning | Constraint |
| [`AUT-006`](#aut-006) | Live-update trigger allows only code actions | Constraint |
| [`AUT-007`](#aut-007) | Deletion trigger forbids message, follower and activity actions | Constraint |
| [`AUT-008`](#aut-008) | Incoming webhook found no record | Guard |
| [`AUT-009`](#aut-009) | Outgoing webhook needs an address | Guard |
| [`AUT-010`](#aut-010) | Watched record type must be concrete | Constraint |
| [`AUT-011`](#aut-011) | The scheduler record must exist | Guard |
| [`AUT-012`](#aut-012) | A rule's action may not be a child of a multi-action | Constraint |
| [`AUT-013`](#aut-013) | Group-restricted fields in an outgoing payload | Warning |
| [`AUT-014`](#aut-014) | Action record type differs from the rule's | Warning |
| [`AUT-015`](#aut-015) | Only settings administrators manage rules | Permission |
| [`AUT-016`](#aut-016) | Time-based processing runs only unattended | Guard |
| [`AUT-017`](#aut-017) | A rule runs at most once per record per operation | Invariant |
| [`AUT-018`](#aut-018) | Unknown webhook address | Guard |
| [`AUT-019`](#aut-019) | Failing webhook answer | Guard |
| [`AUT-020`](#aut-020) | Unsupported file format | Guard |
| [`AUT-021`](#aut-021) | Reader for the format is unavailable | Guard |
| [`AUT-022`](#aut-022) | Error cell in the older workbook format | Guard |
| [`AUT-023`](#aut-023) | Error cell in the current workbook format | Guard |
| [`AUT-024`](#aut-024) | Unsupported cell format | Guard |
| [`AUT-025`](#aut-025) | The text file cannot be decoded | Guard |
| [`AUT-026`](#aut-026) | The text delimiter must be one character | Guard |
| [`AUT-027`](#aut-027) | The file has no content | Guard |
| [`AUT-028`](#aut-028) | At least one column must be mapped | Guard |
| [`AUT-029`](#aut-029) | Heading row and data row widths differ | Guard |
| [`AUT-030`](#aut-030) | A decimal column holds a value that is not a number | Guard |
| [`AUT-031`](#aut-031) | Loading a file from a web address is not allowed | Permission |
| [`AUT-032`](#aut-032) | Image data is neither an address nor encoded bytes | Guard |
| [`AUT-033`](#aut-033) | A date column holds an unparseable value | Guard |
| [`AUT-034`](#aut-034) | A date value does not fit the chosen pattern | Guard |
| [`AUT-035`](#aut-035) | A fetched file is larger than the configured maximum | Guard |
| [`AUT-036`](#aut-036) | A file could not be fetched from its address | Guard |
| [`AUT-037`](#aut-037) | A date value was mapped onto a field that is not a date | Guard |
| [`AUT-038`](#aut-038) | An import session belongs to its creator | Permission |
| [`AUT-039`](#aut-039) | Import sessions cannot be deleted by hand | Permission |
| [`AUT-040`](#aut-040) | Only settings administrators import packages | Permission |
| [`AUT-041`](#aut-041) | No file was sent | Guard |
| [`AUT-042`](#aut-042) | The file is not an archive | Guard |
| [`AUT-043`](#aut-043) | An archive member is too large | Guard |
| [`AUT-044`](#aut-044) | The archive contains no manifest | Guard |
| [`AUT-045`](#aut-045) | A package failed to install | Guard |
| [`AUT-046`](#aut-046) | A dependency is unknown | Guard |
| [`AUT-047`](#aut-047) | A customisation package needs its design application | Guard |
| [`AUT-048`](#aut-048) | Asset declarations may not contain wildcards | Guard |
| [`AUT-049`](#aut-049) | Only administrators may upload over the unattended route | Permission |
| [`AUT-050`](#aut-050) | The remote catalogue answered with a failure | Guard |
| [`AUT-051`](#aut-051) | The remote catalogue could not be reached | Guard |
| [`AUT-052`](#aut-052) | Unsupported catalogue filter | Guard |
| [`AUT-053`](#aut-053) | Dependencies of a catalogue package are unavailable | Guard |
| [`AUT-054`](#aut-054) | A catalogue package could not be downloaded | Guard |
| [`AUT-055`](#aut-055) | The catalogue download connection failed | Guard |
| [`AUT-056`](#aut-056) | The unattended upload named an unknown database | Guard |
| [`AUT-057`](#aut-057) | Views of an imported package are validated as custom views | Invariant |
| [`AUT-058`](#aut-058) | Uninstalling an imported package erases it | Invariant |
| [`AUT-059`](#aut-059) | The package import screen cannot be deleted | Permission |
| [`AUT-060`](#aut-060) | No package selected for activation | Guard |
| [`AUT-061`](#aut-061) | The package is already active | Guard |
| [`AUT-062`](#aut-062) | Only inactive packages may be requested | Constraint |
| [`AUT-063`](#aut-063) | Only settings administrators review a request | Permission |
| [`AUT-070`](#aut-070) | No metered service with that technical name | Guard |
| [`AUT-071`](#aut-071) | Metered calls are refused while tests run | Guard |
| [`AUT-072`](#aut-072) | The metered service timed out | Guard |
| [`AUT-073`](#aut-073) | The metered service could not be reached | Guard |
| [`AUT-074`](#aut-074) | The balance is exhausted | Guard |
| [`AUT-075`](#aut-075) | The alert threshold must not be negative | Constraint |
| [`AUT-076`](#aut-076) | Every alert recipient needs an address | Constraint |
| [`AUT-077`](#aut-077) | The account token must not be empty | Guard |
| [`AUT-078`](#aut-078) | A service technical name is unique | Constraint |
| [`AUT-079`](#aut-079) | Accounts are visible only to their companies | Permission |
| [`AUT-080`](#aut-080) | The chosen geocoding provider is not implemented | Guard |
| [`AUT-081`](#aut-081) | The second geocoding provider needs a service key | Guard |
| [`AUT-082`](#aut-082) | The geocoding service could not be reached | Guard |
| [`AUT-083`](#aut-083) | The second geocoding provider refused a paid feature | Guard |
| [`AUT-084`](#aut-084) | Full address completion is for internal users | Permission |
| [`AUT-085`](#aut-085) | Reverse lookup is disabled while tests run | Guard |
| [`AUT-086`](#aut-086) | Geocoding providers are read-only to users | Permission |
| [`AUT-090`](#aut-090) | Archiving needs a record type that supports it | Constraint |
| [`AUT-091`](#aut-091) | The notification frequency must be positive | Constraint |
| [`AUT-092`](#aut-092) | The age field must be a stored date field | Constraint |
| [`AUT-093`](#aut-093) | Only settings administrators can be notified | Constraint |
| [`AUT-094`](#aut-094) | Recycling is for settings administrators | Permission |
| [`AUT-095`](#aut-095) | The looked-up address must normalise | Guard |
| [`AUT-096`](#aut-096) | A deleted line cannot be deleted again | Guard |
| [`AUT-097`](#aut-097) | Masking an address without an at-sign | Compatibility finding |
| [`AUT-098`](#aut-098) | Privacy handling is for settings administrators | Permission |
| [`AUT-099`](#aut-099) | Deleting a found record is confirmed | Guard |
| [`AUT-100`](#aut-100) | First provider's attachments are still in use | Guard |
| [`AUT-101`](#aut-101) | Second provider's attachments are still in use | Guard |
| [`AUT-102`](#aut-102) | The store must be configured before it is enabled | Guard |
| [`AUT-103`](#aut-103) | First provider refuses the upload probe | Guard |
| [`AUT-104`](#aut-104) | First provider refuses the download probe | Guard |
| [`AUT-105`](#aut-105) | Second provider refuses the upload probe | Guard |
| [`AUT-106`](#aut-106) | Second provider refuses the download probe or the cross-origin write | Guard |
| [`AUT-107`](#aut-107) | The store was switched off while the page was open | Guard |
| [`AUT-108`](#aut-108) | No store is enabled | Guard |
| [`AUT-109`](#aut-109) | A blob could not be brought back | Guard |
| [`AUT-110`](#aut-110) | First provider: no password on an outgoing server | Constraint |
| [`AUT-111`](#aut-111) | First provider: outgoing encryption must be the negotiated one | Constraint |
| [`AUT-112`](#aut-112) | First provider: the user name is required | Constraint |
| [`AUT-113`](#aut-113) | Second provider: no password on an outgoing server | Constraint |
| [`AUT-114`](#aut-114) | Second provider: outgoing encryption must be the negotiated one | Constraint |
| [`AUT-115`](#aut-115) | Second provider: the user name is required | Constraint |
| [`AUT-116`](#aut-116) | First provider: the incoming server must be secure | Constraint |
| [`AUT-117`](#aut-117) | Second provider: the incoming server must be secure | Constraint |
| [`AUT-118`](#aut-118) | First provider: only the administrator links a server | Permission |
| [`AUT-119`](#aut-119) | Second provider: only the administrator links a server | Permission |
| [`AUT-120`](#aut-120) | The mailbox address must normalise | Guard |
| [`AUT-121`](#aut-121) | Provider credentials are not configured | Guard |
| [`AUT-122`](#aut-122) | The relay could not be reached | Guard |
| [`AUT-123`](#aut-123) | The returned record type does not carry the behaviour | Guard |
| [`AUT-124`](#aut-124) | The returned record does not exist | Guard |
| [`AUT-125`](#aut-125) | The cross-site protection token does not match | Guard |
| [`AUT-126`](#aut-126) | The consented address differs from the server's | Guard |
| [`AUT-127`](#aut-127) | Second provider: no consent has been given yet | Guard |
| [`AUT-128`](#aut-128) | First provider: the token exchange failed | Guard |
| [`AUT-129`](#aut-129) | Second provider: the token exchange failed | Guard |
| [`AUT-130`](#aut-130) | A panel step needs an opening operation | Constraint |
| [`AUT-131`](#aut-131) | A panel's route segment is unique | Constraint |
| [`AUT-132`](#aut-132) | A tour name is unique | Constraint |
| [`AUT-133`](#aut-133) | The reflection documentation is group-restricted | Permission |
| [`AUT-134`](#aut-134) | The typed endpoint refuses an unknown record type | Guard |
| [`AUT-135`](#aut-135) | The typed endpoint refuses an unknown or private operation | Guard |
| [`AUT-136`](#aut-136) | The typed endpoint refuses identifiers on a type-level operation | Guard |
| [`AUT-137`](#aut-137) | The typed endpoint refuses mismatched arguments | Guard |
| [`AUT-138`](#aut-138) | Any other typed path is not found | Guard |
| [`AUT-139`](#aut-139) | The storing system of a field cannot be changed | Guard |
| [`AUT-140`](#aut-140) | A sparse field cannot be renamed | Guard |
| [`AUT-141`](#aut-141) | The translation cache is loaded once at a time | Guard |
| [`AUT-142`](#aut-142) | Some record types never store attachments outside | Invariant |
| [`AUT-143`](#aut-143) | Onboarding records are managed by settings administrators | Permission |

---

# 1. Automation rules

## AUT-001

**Rule name is required.**

- **Applies to** Automation Rule, field `name` (Automation Rule Name).
- **Kind** Constraint, enforced by the platform's required-field check.
- **Condition** The name is empty on create or on write.
- **Message** The platform's own required-field message, which names the field's label
  "Automation Rule Name".
- **Effect** The write is refused; nothing is stored.

## AUT-002

**A message trigger needs a record type with a discussion thread.**

- **Applies to** Automation Rule, fields `trigger` and `model_id` (Model).
- **Kind** Constraint.
- **Condition** The trigger is `on_message_received` (a message arrives) or `on_message_sent` (a
  message is sent) and the watched record type does not carry a discussion thread.
- **Message** "Mail event can not be configured on model %s. Only models with discussion feature can
  be used." The placeholder is the display name of the watched record type.
- **Effect** The rule is not saved.

## AUT-003

**A delay must not be negative.**

- **Applies to** Automation Rule, fields `trigger` and `trg_date_range` (Delay).
- **Kind** Constraint.
- **Condition** The trigger is one of the three time triggers — `on_time` (a date field is reached),
  `on_time_created` (a period after creation) or `on_time_updated` (a period after the last update)
  — and the delay is below zero.
- **Message** "Delay must be positive. Set 'Delay mode' to 'Before' to negate the delay."
- **Effect** The rule is not saved. While the form is open the sign is silently dropped instead, and
  for the date-field trigger the delay mode is flipped between `before` and `after`, so a user who
  types a negative number normally never meets this refusal.

## AUT-004

**Every action must target the rule's record type.**

- **Applies to** Automation Rule, fields `model_id` and `action_server_ids` (Actions).
- **Kind** Constraint.
- **Condition** At least one attached Server Action has a record type different from the rule's.
- **Message** "Target model of actions %(action_names)s are different from rule model." The
  placeholder is the comma-separated list of the offending action names.
- **Effect** The rule is not saved. Changing the rule's record type does not raise this refusal: it
  silently detaches every action whose record type no longer matches.

## AUT-005

**No attached action may carry its own warning.**

- **Applies to** Automation Rule, fields `trigger` and `action_server_ids`.
- **Kind** Constraint.
- **Condition** At least one attached Server Action reports a warning of its own — for instance an
  action whose own record type differs (see [`AUT-014`](#aut-014)) or an action whose configuration
  the platform considers incomplete.
- **Message** "Following child actions have warnings: %(children)s" The placeholder is the
  comma-separated list of the names of the warning actions.
- **Effect** The rule is not saved.

## AUT-006

**The live-update trigger allows only code actions.**

- **Applies to** Automation Rule, fields `trigger` and `action_server_ids`.
- **Kind** Constraint.
- **Condition** The trigger is `on_change` (a field changes while a form is open) and at least one
  attached action is not of the code kind.
- **Message** "\"On live update\" automation rules can only be used with \"Execute Python Code\"
  action type." The two quoted fragments inside the message are the stored labels of the trigger
  value and of the action kind and are reproduced as they stand.
- **Effect** The rule is not saved.
- **Reason** A live-update rule runs against an unsaved form. Any other action kind would write to
  the database from a record that does not exist yet.
- **Matching warning** While the form is open the interface shows the title "Warning" and the
  message "The \"%(trigger_value)s\" %(trigger_label)s can only be used with the \"%(state_value)s\"
  action type", whose placeholders are, in order, the trigger's own label "On UI change", the label
  of the trigger field, which is "Trigger", and the label of the code action kind, which is
  "Execute Code".

## AUT-007

**The deletion trigger forbids message, follower and activity actions.**

- **Applies to** Automation Rule, fields `trigger` and `action_server_ids`.
- **Kind** Constraint.
- **Condition** The trigger is `on_unlink` (a record is deleted) and at least one attached action is
  of the kind that posts a message, changes followers or schedules an activity.
- **Message** "Email, follower or activity action types cannot be used when deleting records, as
  there are no more records to apply these changes to!"
- **Effect** The rule is not saved.
- **Matching warning** While the form is open the interface shows the title "Warning" and the
  message "You cannot send an email, add followers or create an activity for a deleted record.  It
  simply does not work." — reproduced including the double space before "It".

## AUT-008

**An incoming webhook call must find a record.**

- **Applies to** Automation Rule, incoming webhook execution.
- **Kind** Guard.
- **Condition** After the record-finding expression has run, the resulting set is empty or its
  records no longer exist.
- **Message** "No record to run the automation on was found."
- **Effect** The execution stops. When call logging is switched on, a log line at level `ERROR` is
  written first, reading "Webhook #%s could not be triggered because no record to run it on was
  found." with the rule identifier substituted. The route answers with the status 500 and the body
  carrying the value `error` under the key `status` — see [`AUT-019`](#aut-019).

## AUT-009

**An outgoing webhook action must carry an address.**

- **Applies to** Server Action of the outgoing-webhook kind, field `webhook_url` (Webhook Web
  Address).
- **Kind** Guard.
- **Condition** The action runs on an existing record but no address is configured.
- **Message** "I'll be happy to send a webhook for you, but you really need to give me a URL to
  reach out to..."
- **Effect** The action fails and, with it, the operation that triggered the rule. When there is no
  record at all the action returns quietly and no failure is raised.

## AUT-010

**The watched record type must be concrete.**

- **Applies to** Automation Rule, field `model_id`.
- **Kind** Constraint, expressed as the field's selection filter.
- **Condition** The chosen record type is abstract, that is, it has no table of its own.
- **Message** The platform's own invalid-value message for a filtered relation field.
- **Effect** The value cannot be chosen. A record type with no fields at all also cannot carry a
  rule, because the trigger computation finds no field to watch.

## AUT-011

**The scheduler record must exist.**

- **Applies to** Automation Rule, operation *Scheduled action*.
- **Kind** Guard.
- **Condition** The shipped scheduled action of the automation package cannot be found.
- **Message** "The scheduled action for Automation Rules seems to have vanished."
- **Effect** The jump is refused. The same absence makes the interval recomputation of
  [`calculations.md`](calculations.md) §3.1 a no-operation rather than a failure.

## AUT-012

**A rule's action may not become a child of a multi-action.**

- **Applies to** Server Action, field `child_ids` (Child Actions).
- **Kind** Constraint, expressed as the child selection filter.
- **Condition** A candidate child action carries a back-link to an Automation Rule.
- **Message** The platform's own invalid-value message for a filtered relation field.
- **Effect** Actions owned by a rule never appear in the child list of a multi-action, because a
  rule's action has no parent and running it from two places would run the rule twice.

## AUT-013

**Group-restricted fields must not appear in an outgoing payload.**

- **Applies to** Server Action of the outgoing-webhook kind, field `webhook_field_ids` (Webhook
  Fields).
- **Kind** Warning; combined with [`AUT-005`](#aut-005) it becomes a refusal for an action attached
  to a rule.
- **Condition** At least one chosen field is visible only to a group.
- **Message** "Group-restricted fields cannot be included in webhook payloads, as it could allow any
  user to accidentally leak sensitive information. You will have to remove the following fields from
  the webhook payload:\n%(restricted_fields)s" The placeholder is the list of offending fields, one
  per line, each prefixed with a hyphen and a space.
- **Effect** The action reports a warning. Saving the action alone is still possible; saving the
  owning rule is not.

## AUT-014

**An action's record type should match its rule's.**

- **Applies to** Server Action, fields `model_id` and `base_automation_id` (Automation Rule).
- **Kind** Warning; combined with [`AUT-005`](#aut-005) it becomes a refusal.
- **Condition** The action carries a back-link to a rule and its record type is not the rule's.
- **Message** "Model of action %(action_name)s should match the one from automated rule
  %(rule_name)s." The placeholders are the action's name and the rule's name.
- **Effect** The action reports a warning.

## AUT-015

**Only settings administrators manage automation rules.**

- **Applies to** Automation Rule.
- **Kind** Permission.
- **Condition** The reader is not in the settings-administration group.
- **Effect** Reading, writing, creating and deleting are all refused with the platform's own access
  message. The rules themselves, once saved, run with elevated rights so that a rule configured by
  an administrator still fires for a user who cannot read it.

## AUT-016

**Time-based processing runs only unattended.**

- **Applies to** Automation Rule, the historic entry point of the time-based processor.
- **Kind** Guard.
- **Condition** The processor is called without the unattended flag.
- **Message** "can run time-based automations only in automatic mode" — a technical failure, not a
  user-facing message; it is reproduced because tests assert it.
- **Effect** Nothing is processed.

## AUT-017

**A rule runs at most once per record per operation.**

- **Applies to** Automation Rule processing.
- **Kind** Invariant.
- **Condition** The same rule would run a second time on the same record inside one operation, for
  instance because one of its own actions writes to the watched field.
- **Effect** The second run is skipped. The processing context carries a per-operation register
  keyed by rule; records already processed by that rule are removed from the candidate set before
  the actions run. A rule that changes the very field it watches therefore fires once, not
  endlessly.

## AUT-018

**An unknown webhook address is not found.**

- **Applies to** the incoming webhook route.
- **Kind** Guard.
- **Condition** No rule carries the identifier in the address.
- **Message** The answer is a structured body carrying the value `error` under the key `status`, sent
  with the status 404.
- **Effect** Nothing runs. The answer deliberately does not distinguish an unknown identifier from a
  disabled rule, so that an outsider cannot probe for valid addresses.

## AUT-019

**A failing webhook answers with a server failure.**

- **Applies to** the incoming webhook route.
- **Kind** Guard.
- **Condition** The execution raised anything at all — a failing record-finding expression, the
  empty result of [`AUT-008`](#aut-008), or a failure inside one of the actions.
- **Message** The answer is a structured body carrying the value `error` under the key `status`, sent
  with the status 500.
- **Effect** The transaction is rolled back. When call logging is switched on the failure text and
  its trace are written to the call log first, at level `ERROR`.

---

# 2. Data import

## AUT-020

**The file format must be one of the four supported ones.**

- **Applies to** Data Import Session, file reading.
- **Kind** Guard.
- **Condition** Neither the reported media type nor the file extension resolves to a reader.
- **Message** "Unsupported file format \"{}\", import only supports CSV, ODS, XLS and XLSX" The
  placeholder is the media type the browser reported. The four names in the message are the
  delimited text format, the open document spreadsheet format and the two office workbook formats.
- **Effect** The preview answer carries the failure text and, when the file looked like a delimited
  text file, the first two hundred bytes decoded with a single-byte encoding that never fails, so
  the user can see the beginning of the file.

## AUT-021

**The reader for a recognised format may be unavailable.**

- **Applies to** Data Import Session, file reading.
- **Kind** Guard.
- **Condition** The extension is recognised but the reading component for it is absent from the
  installation.
- **Message** "Unable to load \"{extension}\" file: requires Python module \"{modname}\"" The first
  placeholder is the file extension, the second the name of the missing reading component.
- **Effect** As [`AUT-020`](#aut-020).

## AUT-022

**An error cell in the older workbook format stops the read.**

- **Applies to** Data Import Session, reading the older office workbook format.
- **Kind** Guard.
- **Condition** A cell holds a spreadsheet error value.
- **Message** "Invalid cell value at row %(row)s, column %(col)s: %(cell_value)s" The placeholders
  are the one-based row number, the one-based column number and the error text of the cell; when the
  error code is not recognised the third placeholder becomes "unknown error code %s" with the raw
  code substituted.
- **Effect** As [`AUT-020`](#aut-020).

## AUT-023

**An error cell in the current workbook format stops the read.**

- **Applies to** Data Import Session, reading the current office workbook format.
- **Kind** Guard.
- **Condition** A cell is marked as holding an error.
- **Message** "Invalid cell value at row %(row)s, column %(col)s: %(cell_value)s" with the same
  placeholders as [`AUT-022`](#aut-022).
- **Effect** As [`AUT-020`](#aut-020).

## AUT-024

**A cell whose number format cannot be interpreted stops the read.**

- **Applies to** Data Import Session, reading the current office workbook format.
- **Kind** Guard.
- **Condition** A cell carries a date-like number format that the reader does not support.
- **Message** "Invalid cell format at row %(row)s, column %(col)s: %(cell_value)s, with format:
  %(cell_format)s, as (%(format_type)s) formats are not supported." The placeholders are the
  one-based row number, the one-based column number, the raw cell value, the cell's number format
  and the format family the reader derived from it.
- **Effect** As [`AUT-020`](#aut-020).

## AUT-025

**A delimited text file must decode.**

- **Applies to** Data Import Session, reading a delimited text file.
- **Kind** Guard.
- **Condition** The bytes cannot be decoded with the chosen encoding.
- **Messages** Two texts, chosen by how the encoding was arrived at:

  | Situation | Message |
  |---|---|
  | The encoding was detected | "There was an issue decoding the file using encoding “%s”.\nThis encoding was automatically detected." |
  | The encoding was chosen by the user | "There was an issue decoding the file using encoding “%s”.\nThis encoding was manually selected." |

  The placeholder is the encoding name; the quotation marks around it are the typographic ones and
  are reproduced as such; the line break inside the text is part of it.
- **Effect** As [`AUT-020`](#aut-020).

## AUT-026

**The text delimiter must be exactly one character.**

- **Applies to** Data Import Session, option `quoting`.
- **Kind** Guard.
- **Condition** The quoting option is empty or longer than one character.
- **Message** "Error while importing records: Text Delimiter should be a single character."
- **Effect** As [`AUT-020`](#aut-020).

## AUT-027

**The file must have content.**

- **Applies to** Data Import Session, preview.
- **Kind** Guard.
- **Condition** The reader returned a length of zero or less.
- **Message** "Import file has no content or is corrupt"
- **Effect** The preview answer carries the failure text and the byte preview described in
  [`AUT-020`](#aut-020).

## AUT-028

**At least one column must be mapped.**

- **Applies to** Data Import Session, conversion.
- **Kind** Guard.
- **Condition** Every entry of the column-to-field mapping is empty.
- **Message** "You must configure at least one field to import"
- **Effect** The run stops before the savepoint is taken; nothing is imported.

## AUT-029

**The heading row and the first data row must have the same width.**

- **Applies to** Data Import Session, conversion.
- **Kind** Guard.
- **Condition** The number of entries in the mapping differs from the number of cells in the first
  row of the file.
- **Message** "Error while importing records: all rows should be of the same size, but the title row
  has %(title_row_entries)d entries while the first row has %(first_row_entries)d. You may need to
  change the separator character." The placeholders are the two counts.
- **Effect** The run stops; nothing is imported.

## AUT-030

**A decimal column must hold numbers.**

- **Applies to** Data Import Session, decimal and monetary columns.
- **Kind** Guard.
- **Condition** After the currency symbol has been stripped and the grouping and decimal characters
  have been applied, the value still cannot be read as a number.
- **Message** "Column %(column)s contains incorrect values (value: %(value)s)" The placeholders are
  the field the column is mapped to and the offending value as it stood in the file.
- **Effect** The run returns immediately with this single message; nothing is imported.

## AUT-031

**Loading a binary value from a web address is a privilege.**

- **Applies to** Data Import Session, binary columns holding an address.
- **Kind** Permission.
- **Condition** The value looks like a web address and the reader is not allowed to fetch remote
  files. By default only the administration group is allowed.
- **Message** "You can not import file via URL, check with your administrator or support for the
  reason."
- **Effect** The run returns immediately; nothing is imported.
- **Reason** Fetching arbitrary addresses row by row would let one import occupy a worker
  indefinitely.

## AUT-032

**Image data must be an address or encoded bytes.**

- **Applies to** Data Import Session, image columns.
- **Kind** Guard.
- **Condition** The value is neither a web address nor a valid encoded byte string.
- **Message** "Found invalid image data, images should be imported as either URLs or base64-encoded
  data."
- **Effect** The run returns immediately; nothing is imported.

## AUT-033

**A date column must hold parseable dates.**

- **Applies to** Data Import Session, date and date-and-time columns.
- **Kind** Guard.
- **Condition** The value does not match the pattern in force for the column.
- **Message** "Column %(column)s contains incorrect values. Error in line %(line)d: %(error)s" The
  placeholders are the field, the one-based line number within the data rows and the parser's own
  description.
- **Effect** The run returns immediately; nothing is imported.

## AUT-034

**A date value that fits no pattern is reported with its position.**

- **Applies to** Data Import Session, date and date-and-time columns.
- **Kind** Guard.
- **Condition** The value matched no pattern at all, including the inferred ones.
- **Message** "Error Parsing Date [%(field)s:L%(line)d]: %(error)s" The placeholders are the field,
  the one-based line number and the parser's own description.
- **Effect** The run returns immediately; nothing is imported.

## AUT-035

**A fetched file must be within the configured maximum.**

- **Applies to** Data Import Session, binary columns holding an address.
- **Kind** Guard.
- **Condition** Either the announced length or the number of bytes actually read exceeds the
  configured maximum for an uploaded file.
- **Message** "File size exceeds configured maximum (%s bytes)" The placeholder is that maximum in
  bytes.
- **Effect** The run returns immediately; nothing is imported. The check is made twice: once on the
  announced length before reading, and once while reading in blocks of 32 768 bytes, so that a
  server that lies about the length cannot exhaust memory.

## AUT-036

**A file must be reachable at its address.**

- **Applies to** Data Import Session, binary columns holding an address.
- **Kind** Guard.
- **Condition** The fetch failed for any transport reason.
- **Message** "Could not retrieve URL: %(url)s [%(field_name)s: L%(line_number)d]: %(error)s" The
  placeholders are the address, the field, the one-based line number and the transport failure text.
- **Effect** The run returns immediately; nothing is imported.

## AUT-037

**A date value must be mapped onto a date field.**

- **Applies to** Data Import Session, conversion.
- **Kind** Guard.
- **Condition** A cell that the reader turned into a date or a date-and-time value is mapped onto a
  field that is neither.
- **Message** "Field '%(field)s' does not accept date/time values." The placeholder is the field.
  The message is wrapped by the platform's row-reporting envelope, which adds the row number and the
  row's own values.
- **Effect** The row is reported as failing; the run continues collecting further failures and
  imports nothing.

## AUT-038

**An import session belongs to the user who created it.**

- **Applies to** Data Import Session.
- **Kind** Permission, expressed as a record rule named "Import: access own records" and applying to
  every group.
- **Condition** The reader is not the creator.
- **Effect** The session is invisible. Two users importing at the same time therefore cannot see or
  disturb each other's files.

## AUT-039

**Import sessions are never deleted by hand.**

- **Applies to** Data Import Session.
- **Kind** Permission.
- **Condition** Any delete attempt.
- **Effect** Refused with the platform's own access message. Sessions disappear when the platform
  clears transient records; this entity keeps them for twelve hours first.

---

# 3. Package import and activation

## AUT-040

**Only settings administrators import packages.**

- **Applies to** Module Import Wizard.
- **Kind** Permission.
- **Condition** The reader is not in the settings-administration group.
- **Effect** Reading, writing and creating are refused. The menu entry itself is further restricted
  to the technical-features group, so an administrator who has not switched technical features on
  does not see it.

## AUT-041

**A file must be chosen.**

- **Applies to** Module Import Wizard, field `module_file` (Module Archive File).
- **Kind** Guard.
- **Condition** No file was sent.
- **Message** "No file sent."
- **Effect** Nothing is installed. The field is also declared required, so the form normally
  prevents this.

## AUT-042

**The file must be an archive.**

- **Applies to** Module Import Wizard.
- **Kind** Guard.
- **Condition** The bytes are not a valid archive.
- **Message** "Only zip files are supported."
- **Effect** Nothing is installed.

## AUT-043

**No archive member may exceed the maximum.**

- **Applies to** Module Import Wizard and the catalogue installer.
- **Kind** Guard.
- **Condition** A member of the archive declares a size above 104 857 600 bytes, that is one hundred
  mebibytes.
- **Message** "File '%s' exceed maximum allowed file size" The placeholder is the member's path
  inside the archive.
- **Effect** Nothing is installed and nothing is extracted.

## AUT-044

**Every package in the archive must have a manifest.**

- **Applies to** Module Import Wizard.
- **Kind** Guard.
- **Condition** A top-level directory of the archive contains no manifest file.
- **Message** "No manifest found in '%(modules)s'. Can't import the zip file." The placeholder is
  the comma-separated list of the directories without a manifest.
- **Effect** Nothing is installed.

## AUT-045

**A failing package installation reports its trace.**

- **Applies to** Module Import Wizard.
- **Kind** Guard.
- **Condition** Anything raised while one package of the archive was being installed.
- **Message** "Error while importing module '%(module)s'.\n\n %(error_message)s \n\n" The
  placeholders are the package name and the technical trace. The blank lines are part of the text.
- **Effect** The whole import is abandoned. Nothing is committed, so a partly installed archive
  leaves no trace in the database.

## AUT-046

**Dependencies must be known.**

- **Applies to** Module Import Wizard.
- **Kind** Guard.
- **Condition** A package in the archive depends on a package that is neither installed nor known to
  the installation nor contained in the archive.
- **Message** The fixed text "Unknown module dependencies:" followed by a line break and, for each
  unknown dependency, the three characters " - " and the dependency's name on its own line.
- **Effect** Nothing is installed. Dependencies that are known but not installed are installed first
  instead of being refused.

## AUT-047

**A customisation package needs its design application.**

- **Applies to** Module Import Wizard.
- **Kind** Guard.
- **Condition** The archive carries the marker of a package produced by the interface designer and
  that designer is not installed.
- **Message** "Studio customizations require the Odoo Studio app." — reproduced verbatim, including
  the product name it contains, because support procedures and tests key on it.
- **Effect** Nothing is installed.

## AUT-048

**Asset declarations may not contain wildcards.**

- **Applies to** Module Import Wizard, the manifest's asset declarations.
- **Kind** Guard.
- **Condition** A declared asset path contains a wildcard character.
- **Message** "The assets path in the manifest of imported module '%(module_name)s' cannot contain
  glob wildcards (e.g., *, **)." The placeholder is the package name.
- **Effect** Nothing is installed. A path that does not start with a separator silently gains one
  rather than being refused.

## AUT-049

**Only administrators may upload over the unattended route.**

- **Applies to** the unattended package-upload route.
- **Kind** Permission.
- **Condition** The credentials sent with the request do not belong to an administrator, or the
  second authentication factor is outstanding.
- **Message** "Only administrators can upload a module"
- **Effect** The answer carries the status 500 and the failure text as its body. The same check on
  the ordinary path uses the text "Only administrators can install data modules."

## AUT-050

**The remote catalogue may answer with a failure.**

- **Applies to** the remote catalogue listing.
- **Kind** Guard.
- **Condition** The catalogue answered with a failing status.
- **Message** "The list of industry applications cannot be fetched. Please try again later"
- **Effect** The listing is empty.

## AUT-051

**The remote catalogue may be unreachable.**

- **Applies to** the remote catalogue listing.
- **Kind** Guard.
- **Condition** The connection to the catalogue failed.
- **Message** "Connection to %s failed The list of industry modules cannot be fetched" The
  placeholder is the catalogue address `https://apps.odoo.com`, reproduced because it is part of the
  integration contract.
- **Effect** The listing is empty.

## AUT-052

**The catalogue understands only a small set of filters.**

- **Applies to** the catalogue browsing filter.
- **Kind** Guard.
- **Condition** The search filter contains a condition the catalogue reader cannot translate.
- **Message** "Unsupported domain condition " followed by the condition itself in its stored form.
- **Effect** The browse is refused.

## AUT-053

**A catalogue package whose dependencies are unavailable is refused.**

- **Applies to** the catalogue installer.
- **Kind** Guard.
- **Condition** The downloaded archive depends on packages that are neither installed nor part of
  the archive nor present in the installation.
- **Message** The fixed text "The installation of the data module would fail as the following
  dependencies can't be found in the addons-path:\n" followed by one line per missing package, each
  written as "- " and the name, and then the closing text, reproduced here on one line because it
  carries two addresses that are part of what the message asks the reader to do:
  "\nYou may need the Enterprise version to install the data module. Please visit https://www.odoo.com/pricing-plan for more information.\nIf you need Website themes, it can be downloaded from https://github.com/odoo/design-themes.\n"
- **Effect** Nothing is installed. When no dependency is missing, the same computation produces the
  informational text "Load demo data to test the industry's features with sample records. Do not
  load them if this is your production database." instead, which is shown on the confirmation screen.

## AUT-054

**A catalogue package may fail to download.**

- **Applies to** the catalogue installer.
- **Kind** Guard.
- **Condition** The download answered with a failing status.
- **Message** "The module %s cannot be downloaded" The placeholder is the package name.
- **Effect** Nothing is installed.

## AUT-055

**The catalogue download connection may fail.**

- **Applies to** the catalogue installer.
- **Kind** Guard.
- **Condition** The connection failed.
- **Message** "Connection to %(url)s failed, the module %(module)s cannot be downloaded." The
  placeholders are the catalogue address and the package name.
- **Effect** Nothing is installed.

## AUT-056

**The unattended upload must name a database.**

- **Applies to** the unattended package-upload route.
- **Kind** Guard.
- **Condition** The request does not resolve to a database.
- **Message** "Could not select database '%s'" The placeholder is the database name in the request.
- **Effect** The answer carries the status 500 and the failure text as its body.

## AUT-057

**Views coming from an imported package are validated as custom views.**

- **Applies to** View Definition.
- **Kind** Invariant.
- **Condition** A view belongs to a package marked as imported.
- **Effect** It is validated with the rules used for views a user wrote, not with the rules used for
  views shipped with the platform. A malformed imported view therefore reports its problem rather
  than preventing the installation from loading.

## AUT-058

**Uninstalling an imported package erases it.**

- **Applies to** Module.
- **Kind** Invariant.
- **Condition** The package being uninstalled carries the imported flag.
- **Effect** After the ordinary uninstall the package row itself is deleted rather than being left
  in the uninstalled state, because there is no source directory to reinstall it from. The uninstall
  screen also hides imported packages from the list of packages it says will be removed.

## AUT-059

**The package import screen cannot be deleted.**

- **Applies to** Module Import Wizard.
- **Kind** Permission.
- **Condition** Any delete attempt.
- **Effect** Refused. The record disappears when the platform clears transient records.

## AUT-060

**A package must be selected before activation is requested.**

- **Applies to** Module Activation Review.
- **Kind** Guard.
- **Condition** The review carries no package.
- **Message** "No module selected."
- **Effect** The dependency expansion is refused and the screen cannot be rendered.

## AUT-061

**An active package cannot be requested again.**

- **Applies to** Module Activation Review.
- **Kind** Guard.
- **Condition** The package is already installed.
- **Message** "The module is already installed."
- **Effect** The screen cannot be rendered.

## AUT-062

**Only inactive packages may be requested.**

- **Applies to** Module Activation Request and Module Activation Review, field `module_id` (Module).
- **Kind** Constraint, expressed as the field's selection filter.
- **Condition** The chosen package is in any state other than uninstalled.
- **Effect** The value cannot be chosen.

## AUT-063

**Only settings administrators review a request.**

- **Applies to** Module Activation Review.
- **Kind** Permission.
- **Condition** The reader is not in the settings-administration group.
- **Effect** The review screen cannot be opened. The request itself is available to every internal
  user, who may read, write and create it but never delete it; the same package grants every
  internal user read access to the package catalogue, its categories, its dependencies and its
  exclusions so that the request form can show them.

---

# 4. Metered outside services

## AUT-070

**A metered service must exist.**

- **Applies to** In-Application Purchase Account, the find-or-create operation.
- **Kind** Guard.
- **Condition** No In-Application Purchase Service carries the technical name the caller passed and
  no account exists for it either.
- **Message** "No service exists with the provided technical name"
- **Effect** No account is created and the calling feature cannot proceed.

## AUT-071

**Metered calls are refused while tests run.**

- **Applies to** every call to an outside metered service.
- **Kind** Guard.
- **Condition** A test is running.
- **Message** "Unavailable during tests."
- **Effect** The call is refused before any network activity. Individual services add their own
  variant: the company directory raises its own refusal carrying the text "Test mode", which the
  caller translates into the exhausted-balance marker.

## AUT-072

**A metered service may time out.**

- **Applies to** every call to an outside metered service.
- **Kind** Guard.
- **Condition** The call did not answer within its timeout, which is fifteen seconds by default,
  five seconds for the automatic company enrichment, two and a half seconds for address completion,
  five seconds for the delegated-authorisation relay and three hundred seconds for lead enrichment.
- **Message** "The request to the service timed out. Please contact the author of the app. The URL
  it tried to contact was %s" The placeholder is the address that was called.
- **Effect** The calling feature receives a refusal and decides for itself whether to report or to
  ignore it.

## AUT-073

**A metered service may be unreachable or answer badly.**

- **Applies to** every call to an outside metered service.
- **Kind** Guard.
- **Condition** Any transport failure, any failing status, or a structured answer carrying a failure
  that is not the exhausted-balance failure.
- **Message** "An error occurred while reaching %s. Please contact Odoo support if this error persists."
  The placeholder is the address that was called.
- **Effect** As [`AUT-072`](#aut-072).

## AUT-074

**An exhausted balance is a distinct failure.**

- **Applies to** every call to an outside metered service.
- **Kind** Guard.
- **Condition** The service answered with the insufficient-balance failure.
- **Message** "You don't have enough credits on your account to use this service."
- **Effect** A distinct failure is raised carrying, besides the message, the remaining balance, the
  service technical name and the purchase address, so that the caller can offer a top-up link. The
  company directory turns this failure into the marker `Insufficient Credit`.

## AUT-075

**The alert threshold must not be negative.**

- **Applies to** In-Application Purchase Account, fields `warning_threshold` (Email Alert Threshold)
  and `warning_user_ids` (Email Alert Recipients).
- **Kind** Constraint.
- **Condition** The threshold is below zero.
- **Message** "Please set a positive email alert threshold."
- **Effect** The account is not saved.

## AUT-076

**Every alert recipient needs an electronic mail address.**

- **Applies to** In-Application Purchase Account, same fields.
- **Kind** Constraint.
- **Condition** At least one chosen recipient has no address.
- **Message** "One of the email alert recipients doesn't have an email address set. Users: %s" The
  placeholder is the comma-separated list of the names of the recipients without an address, joined
  without a space after the comma.
- **Effect** The account is not saved.

## AUT-077

**The account token must not be empty.**

- **Applies to** In-Application Purchase Account, the token-hashing operation used to build the
  purchase address.
- **Kind** Guard.
- **Condition** The token is empty once the suffix after a plus sign has been removed.
- **Message** "The IAP token provided is invalid or empty."
- **Effect** No purchase address is produced.

## AUT-078

**A service technical name is unique.**

- **Applies to** In-Application Purchase Service, field `technical_name`.
- **Kind** Constraint, enforced by a unique index.
- **Condition** A second service claims a technical name that already exists.
- **Message** "Only one service can exist with a specific technical_name"
- **Effect** The service is not saved.

## AUT-079

**Accounts are visible only to their own companies.**

- **Applies to** In-Application Purchase Account.
- **Kind** Permission, expressed as a record rule named "User In Application Purchase Account"
  applying to every internal user.
- **Condition** The account's company list is neither empty nor intersecting the reader's allowed
  companies.
- **Effect** The account is invisible. Internal users may read accounts and create them but may
  neither change nor delete them; the settings-administration group has all four rights. The token
  itself is visible only to the settings-administration group, whatever the record rule says.

---

# 5. Geocoding and address completion

## AUT-080

**The chosen geocoding provider must be implemented.**

- **Applies to** the Geocoder, address resolution.
- **Kind** Guard.
- **Condition** The technical name stored on the chosen Geocoding Provider has no matching
  behaviour.
- **Message** "Provider %s is not implemented for geolocation service." The placeholder is the
  provider's technical name.
- **Effect** No coordinates are produced.

## AUT-081

**The second provider needs a service key.**

- **Applies to** the Geocoder, address resolution with the second provider.
- **Kind** Guard.
- **Condition** The system parameter `base_geolocalize.google_map_api_key` is empty.
- **Message** "API key for GeoCoding (Places) required.\nVisit
  https://developers.google.com/maps/documentation/geocoding/get-api-key for more information." The
  address is reproduced because it is the action the message asks the reader to take.
- **Effect** No coordinates are produced.

## AUT-082

**The geocoding service may be unreachable.**

- **Applies to** the Geocoder, both providers, forward and reverse resolution.
- **Kind** Guard.
- **Condition** The call raised anything at all.
- **Message** "Error with geolocation server: %s" The placeholder is the transport failure's own
  description.
- **Effect** No coordinates are produced. A failing status that still yields a readable answer is
  only logged as a warning and the answer is used.

## AUT-083

**The second provider may refuse a paid feature.**

- **Applies to** the Geocoder, the second provider.
- **Kind** Guard.
- **Condition** The provider answered with a status that is neither success nor the empty-result
  status.
- **Message** "Unable to geolocate, received the error:\n%s\n\nGoogle made this a paid feature.\nYou
  should first enable billing on your Google account.\nThen, go to Developer Console, and enable the
  APIs:\nGeocoding, Maps Static, Maps Javascript.\n" The placeholder is the provider's own error
  description. The line breaks are part of the text.
- **Effect** No coordinates are produced. The empty-result status yields no coordinates without any
  message, which is what makes the second attempt of the resolution procedure possible.

## AUT-084

**Full address completion is for internal users.**

- **Applies to** the address-completion detail route.
- **Kind** Permission.
- **Condition** The caller is not an internal user.
- **Message** "You don't have access to the full autocomplete feature."
- **Effect** No address is returned. The suggestion route, by contrast, answers anyone with an empty
  result list rather than a refusal, because it is reachable from public pages.

## AUT-085

**Reverse lookup is disabled while tests run.**

- **Applies to** the Geocoder, reverse resolution with the first provider.
- **Kind** Guard.
- **Condition** A test is running.
- **Message** "OpenStreetMap calls disabled in testing environment."
- **Effect** No address is produced.

## AUT-086

**Geocoding providers are read-only to users.**

- **Applies to** Geocoding Provider.
- **Kind** Permission.
- **Condition** Any write, create or delete by an internal user.
- **Effect** Refused. Every internal user may read the two shipped providers; only a package may add
  one.

---

# 6. Data recycling

## AUT-090

**Archiving needs a record type that supports it.**

- **Applies to** Recycling Model, field `recycle_action` (Recycle Action).
- **Kind** Constraint.
- **Condition** The action is `archive` and the chosen record type has no activation flag.
- **Message** "This model doesn't manage archived records. Only deletion is possible."
- **Effect** The rule is not saved.

## AUT-091

**The notification frequency must be positive.**

- **Applies to** Recycling Model, field `notify_frequency` (Notify).
- **Kind** Constraint, enforced by a database check.
- **Condition** The frequency is zero or below.
- **Message** "The notification frequency should be greater than 0"
- **Effect** The rule is not saved.

## AUT-092

**The age field must be a stored date field of the watched record type.**

- **Applies to** Recycling Model, field `time_field_id` (Time Field).
- **Kind** Constraint, expressed as the field's selection filter.
- **Condition** The chosen field does not belong to the watched record type, is not of a date or
  date-and-time kind, or is not stored.
- **Effect** The value cannot be chosen. Without a stored value the age comparison could not be
  pushed into the search.

## AUT-093

**Only settings administrators can be chosen as recipients.**

- **Applies to** Recycling Model, field `notify_user_ids` (Notify Users).
- **Kind** Constraint, expressed as the field's selection filter.
- **Condition** The chosen user is not in the settings-administration group.
- **Effect** The value cannot be chosen. The reasoning is that only such a user could act on the
  notification.

## AUT-094

**Recycling is for settings administrators.**

- **Applies to** Recycling Model and Recycling Record.
- **Kind** Permission.
- **Condition** The reader is not in the settings-administration group.
- **Effect** Both entities are invisible. The archive and delete performed when a candidate is
  validated are executed with elevated rights, so a candidate for a record type the administrator
  cannot otherwise touch is still disposed of.

---

# 7. Privacy handling

## AUT-095

**The looked-up address must normalise.**

- **Applies to** Privacy Lookup Wizard, field `email`.
- **Kind** Guard.
- **Condition** The address does not normalise to a canonical form.
- **Message** "Invalid email address “%s”" The placeholder is the address as typed; the quotation
  marks inside the message are the typographic ones and are reproduced as such.
- **Effect** No search is run.

## AUT-096

**A deleted line cannot be deleted again.**

- **Applies to** Privacy Lookup Wizard Line, operation *Delete*.
- **Kind** Guard.
- **Condition** The line is already marked as deleted.
- **Message** "The record is already unlinked."
- **Effect** Nothing happens. The mass deletion skips already deleted lines instead of raising.

## AUT-097

**Masking an address without an at-sign returns instead of raising.**

- **Applies to** Privacy Log, the address-masking helper.
- **Kind** **Compatibility finding**.
- **Observed** When the value handed to the address-masking helper contains no at-sign, the helper
  builds the failure "This email address is not valid (%s)" — the placeholder being the value — and
  **returns** it as the field value instead of raising it. The stored masked address is therefore the
  textual form of a failure object rather than a masked address.
- **Reachability** The wizard refuses an address that does not normalise before the log is ever
  written ([`AUT-095`](#aut-095)), so the path is reached only when a log is created directly.
- **Corrected behaviour** The helper should raise the failure so that the log is not written with a
  meaningless value; a rebuild should raise, and should keep the message text unchanged.

## AUT-098

**Privacy handling is for settings administrators.**

- **Applies to** Privacy Lookup Wizard, Privacy Lookup Wizard Line and Privacy Log.
- **Kind** Permission.
- **Condition** The reader is not in the settings-administration group.
- **Effect** All three are invisible, and the two action-menu entries that open the wizard from a
  Contact and from a User are hidden. Lines may be read, written and created but never deleted; the
  log may be read, written, created and deleted, but its list and form disable creation from the
  interface so that a log is only ever produced by a handling session.

## AUT-099

**Deleting a found record is confirmed.**

- **Applies to** Privacy Lookup Wizard Line, operation *Delete*.
- **Kind** Guard, enforced by the interface.
- **Condition** The button is pressed.
- **Message** "This operation is irreversible. Do you wish to proceed to the record deletion?"
- **Effect** The deletion happens only after the confirmation is accepted.

---

# 8. Attachments stored outside

## AUT-100

**The first provider's attachments must be migrated before it is switched off.**

- **Applies to** Configuration Settings, field `cloud_storage_provider` (Cloud Storage Provider for
  new attachments).
- **Kind** Guard.
- **Condition** The provider in force is the first one, the setting is being changed, and at least
  one attachment still points at that provider's blobs.
- **Message** "Some Azure attachments are in use, please migrate their cloud storages before disable
  this module"
- **Effect** The settings are not saved.

## AUT-101

**The second provider's attachments must be migrated before it is switched off.**

- **Applies to** the same field.
- **Kind** Guard.
- **Condition** As above, for the second provider.
- **Message** "Some Google attachments are in use, please migrate cloud storages before disable the
  provider"
- **Effect** The settings are not saved.

## AUT-102

**The store must be configured before it is enabled.**

- **Applies to** Configuration Settings.
- **Kind** Guard.
- **Condition** A provider is chosen but its configuration is incomplete, so the configuration
  computation returns nothing.
- **Message** "Please configure the Cloud Storage before enabling it"
- **Effect** The settings are not saved.

## AUT-103

**The first provider must accept the upload probe.**

- **Applies to** Configuration Settings, first provider verification.
- **Kind** Guard.
- **Condition** Uploading the probe blob through a freshly signed address answered with a failing
  status.
- **Message** "The connection string is not allowed to upload blobs to the container.\n%s" The
  placeholder is the provider's own answer body.
- **Effect** The settings are not saved.

## AUT-104

**The first provider must accept the download probe.**

- **Applies to** Configuration Settings, first provider verification.
- **Kind** Guard.
- **Condition** Downloading the probe blob through a freshly signed address answered with a failing
  status.
- **Message** "The connection string is not allowed to download blobs from the container.\n%s"
- **Effect** The settings are not saved.

## AUT-105

**The second provider must accept the upload probe.**

- **Applies to** Configuration Settings, second provider verification.
- **Kind** Guard.
- **Condition** Uploading the probe blob answered with a failing status.
- **Message** "The account info is not allowed to upload blobs to the bucket.\n%s"
- **Effect** The settings are not saved.

## AUT-106

**The second provider must accept the download probe and the cross-origin write.**

- **Applies to** Configuration Settings, second provider verification.
- **Kind** Guard.
- **Condition and message** Two failures share this rule because they belong to the same
  verification step:

  | Condition | Message |
  |---|---|
  | Downloading the probe blob answered with a failing status | "The account info is not allowed to download blobs from the bucket.\n%s" |
  | Writing the container's cross-origin rules answered with a failing status | "The account info is not allowed to set the bucket's CORS.\n%s" |

  The placeholder is the provider's own answer body in both cases.
- **Effect** The settings are not saved. The cross-origin rules written on success allow any origin,
  the read and write request methods, the media-type and disposition response headers, and a maximum
  age equal to the download address lifetime of three hundred seconds.

## AUT-107

**The store may have been switched off while the page was open.**

- **Applies to** the attachment upload route.
- **Kind** Guard.
- **Condition** The client asked for an outside upload but no provider is configured any more.
- **Message** "Cloud storage configuration has been changed. Please refresh the page."
- **Effect** The answer carries this text under the key `error` and no attachment is created.

## AUT-108

**No store is enabled.**

- **Applies to** Attachment, the conversion to an outside-stored attachment.
- **Kind** Guard.
- **Condition** The conversion was asked for but the system parameter `cloud_storage_provider` is
  empty.
- **Message** "Cloud Storage is not enabled"
- **Effect** The attachment stays as it was created, with its bytes in the ordinary store.

## AUT-109

**A blob must come back before the attachment is converted.**

- **Applies to** Attachment, bringing an outside-stored attachment back.
- **Kind** Guard.
- **Condition** The download through a freshly signed address answered with a status other than
  success. A failing status also raises the transport layer's own failure first.
- **Message** "Failed to download attachment (%(id)s) from cloud: %(code)s - %(reason)s" The
  placeholders are the attachment identifier, the answer status and the answer's reason text.
- **Effect** The attachment keeps its outside address and its kind.

## AUT-142

**Some record types never store attachments outside.**

- **Applies to** Attachment.
- **Kind** Invariant.
- **Condition** The attachment belongs to a record type whose own logic reads attachment bytes —
  every record type that carries the main-attachment behaviour of the messaging domain and, when the
  document management package is installed, every record type carrying its mixin and the document
  record type itself.
- **Effect** The client is told the list in the session description and never offers an outside
  upload for those record types.

---

# 9. Delegated mail authorisation

The two providers carry the same rules with different texts. The first provider is the one whose
outgoing host is `smtp.gmail.com`; the second is the one whose outgoing host is `smtp.outlook.com`.

## AUT-110

**First provider: an outgoing server carries no password.**

- **Applies to** Mail Server, fields `smtp_authentication`, `smtp_pass`, `smtp_encryption`,
  `from_filter` and `smtp_user`.
- **Kind** Constraint.
- **Condition** The authentication kind is `gmail` and a password is filled in.
- **Message** "Please leave the password field empty for Gmail mail server “%s”. The OAuth process
  does not require it" The placeholder is the server's name; the quotation marks are the typographic
  ones.
- **Effect** The server is not saved.

## AUT-111

**First provider: the outgoing encryption must be the negotiated one.**

- **Applies to** Mail Server, same fields.
- **Kind** Constraint.
- **Condition** The authentication kind is `gmail` and the encryption is not `starttls`.
- **Message** "Incorrect Connection Security for Gmail mail server “%s”. Please set it to \"TLS
  (STARTTLS)\"." The placeholder is the server's name.
- **Effect** The server is not saved. Choosing the authentication kind sets the encryption to
  `starttls` and the port to 587 automatically, so a user normally never meets this refusal.

## AUT-112

**First provider: the user name is required.**

- **Applies to** Mail Server, same fields.
- **Kind** Constraint.
- **Condition** The authentication kind is `gmail` and the user name is empty.
- **Message** "Please fill the \"Username\" field with your Gmail username (your email address).
  This should be the same account as the one used for the Gmail OAuthentication Token."
- **Effect** The server is not saved.

## AUT-113

**Second provider: an outgoing server carries no password.**

- **Applies to** Mail Server, fields `smtp_authentication`, `smtp_pass`, `smtp_encryption` and
  `smtp_user`.
- **Kind** Constraint.
- **Condition** The authentication kind is `outlook` and a password is filled in.
- **Message** "Please leave the password field empty for Outlook mail server “%s”. The OAuth process
  does not require it"
- **Effect** The server is not saved.

## AUT-114

**Second provider: the outgoing encryption must be the negotiated one.**

- **Applies to** Mail Server, same fields.
- **Kind** Constraint.
- **Condition** The authentication kind is `outlook` and the encryption is not `starttls`.
- **Message** "Incorrect Connection Security for Outlook mail server “%s”. Please set it to \"TLS
  (STARTTLS)\"."
- **Effect** The server is not saved.

## AUT-115

**Second provider: the user name is required.**

- **Applies to** Mail Server, same fields.
- **Kind** Constraint.
- **Condition** The authentication kind is `outlook` and the user name is empty.
- **Message** "Please fill the \"Username\" field with your Outlook/Office365 username (your email
  address). This should be the same account as the one used for the Outlook OAuthentication Token."
- **Effect** The server is not saved.

## AUT-116

**First provider: an incoming server must be marked secure.**

- **Applies to** Incoming Mail Server, fields `server_type` and `is_ssl`.
- **Kind** Constraint.
- **Condition** The server kind is `gmail` and the secure flag is not set.
- **Message** "SSL is required for server “%s”." The placeholder is the server's name.
- **Effect** The server is not saved. Choosing the kind sets the host `imap.gmail.com`, the secure
  flag and the port 993 automatically.

## AUT-117

**Second provider: an incoming server must be marked secure.**

- **Applies to** Incoming Mail Server, same fields.
- **Kind** Constraint.
- **Condition** The server kind is `outlook` and the secure flag is not set.
- **Message** "SSL is required for server “%s”."
- **Effect** The server is not saved. Choosing the kind sets the host `imap.outlook.com`, the secure
  flag and the port 993 automatically.

## AUT-118

**First provider: only the administrator links a server.**

- **Applies to** the first provider's connect operation.
- **Kind** Permission.
- **Condition** The reader is not the administrator.
- **Message** "Only the administrator can link a Gmail mail server."
- **Effect** No consent address is opened.

## AUT-119

**Second provider: only the administrator links a server.**

- **Applies to** the second provider's connect operation.
- **Kind** Permission.
- **Condition** The reader is not the administrator.
- **Message** "Only the administrator can link an Outlook mail server."
- **Effect** No consent address is opened.

## AUT-120

**The mailbox address must normalise.**

- **Applies to** both providers' connect operation.
- **Kind** Guard.
- **Condition** The address field of the record — the user name of an outgoing server, the user name
  of an incoming server — does not normalise.
- **Message** "Please enter a valid email address."
- **Effect** No consent address is opened.

## AUT-121

**Provider credentials must be configured, or the relay must be available.**

- **Applies to** both providers' connect operation.
- **Kind** Guard.
- **Condition** Neither the application identifier nor the secret is configured and the running
  edition is not the one that may use the relay; or the credentials are configured but the computed
  consent address is empty.
- **Messages** "Please configure your Gmail credentials." for the first provider and "Please
  configure your Outlook credentials." for the second.
- **Effect** No consent address is opened.

## AUT-122

**The relay must be reachable.**

- **Applies to** both providers' connect operation, relay path.
- **Kind** Guard.
- **Condition** The request to the relay failed or answered with a failing status, within a
  five-second timeout.
- **Message** "Oops, we could not authenticate you. Please try again later."
- **Effect** No consent address is opened. When the relay answers with a structured failure instead,
  its token is translated: `not_configured` becomes "Something went wrong. Try again later",
  `no_subscription` becomes "You don't have an active subscription", and any other token is shown
  unchanged.

## AUT-123

**The returned record type must carry the provider behaviour.**

- **Applies to** both providers' callback routes.
- **Kind** Guard.
- **Condition** The record type named in the returned state does not carry the provider's behaviour.
- **Message** None is shown; the answer is the forbidden status. A line naming the record type is
  logged at error level first.
- **Effect** Nothing is written.

## AUT-124

**The returned record must exist.**

- **Applies to** both providers' callback routes.
- **Kind** Guard.
- **Condition** The identifier in the returned state matches no record.
- **Message** None; the answer is the forbidden status, after an error line is logged.
- **Effect** Nothing is written.

## AUT-125

**The cross-site protection token must match.**

- **Applies to** both providers' callback routes.
- **Kind** Guard.
- **Condition** The token in the returned state is empty, or does not equal the token recomputed for
  that record. The comparison is made in constant time.
- **Message** None; the answer is the forbidden status, after an error line is logged.
- **Effect** Nothing is written. This is what stops a third party from making an administrator
  disconnect a mail server by following a crafted link.

## AUT-126

**The consented address must be the server's own.**

- **Applies to** the first provider's callback routes, and only for an outgoing server that has an
  owner or whose reader is not a settings administrator.
- **Kind** Guard.
- **Condition** The provider reports that the address is unverified, or the reported address does
  not normalise to the same value as the server's own address.
- **Message** "Oops, you're creating an authorization to send from %(email_login)s but your address
  is %(email_server)s. Make sure your addresses match!" The placeholders are the address the
  provider reported and the address on the server.
- **Effect** The error page is rendered with a link back to the record and nothing is written. When
  the provider's own verification call answers with a failing status the route answers with the
  forbidden status instead.

## AUT-127

**Second provider: consent must have been given before a send.**

- **Applies to** the second provider, building the authentication string.
- **Kind** Guard.
- **Condition** The access token is missing or stale and there is no refresh token to renew it with.
- **Message** "Please connect with your Outlook account before using it."
- **Effect** The send or fetch fails.

## AUT-128

**First provider: the token exchange may fail.**

- **Applies to** the first provider, exchanging an authorisation code or refreshing an access token
  directly against the provider.
- **Kind** Guard.
- **Condition** The exchange answered with a failing status.
- **Message** "An error occurred when fetching the access token."
- **Effect** During the callback the message is rendered on the error page with a link back to the
  record; during a send it fails the send.

## AUT-129

**Second provider: the token exchange may fail.**

- **Applies to** the second provider, same operations.
- **Kind** Guard.
- **Condition** The exchange answered with a failing status.
- **Message** "An error occurred when fetching the access token. %s" The placeholder is the
  provider's own description, or the fixed text "Unknown error." when the provider gave none.
- **Effect** As [`AUT-128`](#aut-128).

**The consent failure page.** When the provider itself sends the browser back with a failure rather
than a code, both callbacks render the error page with the text "An error occurred during the
authentication process." and a link to the application root.

---

# 10. Guidance, translation, documentation and remote calls

## AUT-130

**A step linked to a panel needs an opening operation.**

- **Applies to** Onboarding Step, field `onboarding_ids` (Onboardings).
- **Kind** Constraint.
- **Condition** The step is linked to at least one panel and its opening-operation name is empty.
- **Message** "An \"Opening Action\" is required for the following steps to be linked to an
  onboarding panel: %(step_titles)s" The placeholder is the list of the titles of the offending
  steps.
- **Effect** The link is refused.

## AUT-131

**A panel's route segment is unique.**

- **Applies to** Onboarding, field `route_name`.
- **Kind** Constraint, enforced by a unique index.
- **Condition** A second panel claims a route segment that already exists.
- **Message** "Onboarding alias must be unique."
- **Effect** The panel is not saved.

## AUT-132

**A tour name is unique.**

- **Applies to** Tour, field `name`.
- **Kind** Constraint, enforced by a unique index.
- **Condition** A second tour claims a name that already exists.
- **Message** "A tour already exists with this name . Tour's name must be unique!" — reproduced
  including the space before the full stop.
- **Effect** The tour is not saved.

## AUT-133

**The reflection documentation is group-restricted.**

- **Applies to** the documentation page and the two documentation listing routes reached with a
  session.
- **Kind** Permission.
- **Condition** The reader is not in the technical documentation group.
- **Message** "This page is only accessible to %s users." The placeholder is the group's own name,
  which is "Technical Documentation".
- **Effect** Nothing is served. The two routes reached with a bearer credential instead of a session
  do not repeat the check, because the credential itself carries the reader's groups.

## AUT-134

**The typed endpoint refuses an unknown record type.**

- **Applies to** the typed call route.
- **Kind** Guard.
- **Condition** The record type in the path does not exist.
- **Message** "the model %r does not exist" with the requested name substituted, answered with the
  not-found status.
- **Effect** Nothing runs.

## AUT-135

**The typed endpoint refuses an unknown or private operation.**

- **Applies to** the typed call route.
- **Kind** Guard.
- **Condition** The operation does not exist on the record type, or exists but is not public.
- **Message** The platform's own description of the missing public operation, answered with the
  not-found status.
- **Effect** Nothing runs.

## AUT-136

**The typed endpoint refuses identifiers on a type-level operation.**

- **Applies to** the typed call route.
- **Kind** Guard.
- **Condition** The operation is declared at the level of the record type rather than of a record,
  and the caller passed identifiers.
- **Message** "cannot call %s.%s with ids" with the record type and the operation substituted,
  answered with the unprocessable status.
- **Effect** Nothing runs.

## AUT-137

**The typed endpoint refuses mismatched arguments.**

- **Applies to** the typed call route.
- **Kind** Guard.
- **Condition** The named arguments do not fit the operation's signature.
- **Message** The platform's own description of the mismatch, answered with the unprocessable status.
- **Effect** Nothing runs.

## AUT-138

**Any other typed path is not found.**

- **Applies to** the typed call route family.
- **Kind** Guard.
- **Condition** The path is under the typed prefix but is not the call path.
- **Message** "Did you mean POST /json/2/<model>/<method>?" answered with the not-found status.
- **Effect** Nothing runs.

## AUT-139

**The storing system of a field cannot be changed.**

- **Applies to** Field Definition, field `serialization_field_id` (Serialization Field).
- **Kind** Guard.
- **Condition** A write would give an existing field a different serialisation field.
- **Message** "Changing the storing system for field \"%s\" is not allowed." The placeholder is the
  field's identifier.
- **Effect** The write is refused. Moving a value between a column of its own and a serialised
  structure would silently lose every stored value.

## AUT-140

**A sparse field cannot be renamed.**

- **Applies to** Field Definition, field `name`.
- **Kind** Guard.
- **Condition** A write would change the identifier of a field that is stored inside a serialisation
  field.
- **Message** "Renaming sparse field \"%s\" is not allowed" The placeholder is the field's current
  identifier.
- **Effect** The write is refused. The identifier is the key inside the serialised structure, so
  renaming it would orphan every stored value.

## AUT-141

**The translation cache is loaded once at a time.**

- **Applies to** Code Translation, the cache load.
- **Kind** Guard.
- **Condition** Another session already holds the exclusive lock on the cache table.
- **Message** None; the operation returns the value false.
- **Effect** Nothing is loaded and the caller carries on with whatever is already cached. The lock
  is what guarantees that the terms of one package and one language are created exactly once.

## AUT-143

**Onboarding records are managed by settings administrators.**

- **Applies to** Onboarding, Onboarding Step, Onboarding Progress Tracker and Onboarding Progress
  Step Tracker.
- **Kind** Permission.
- **Condition** The reader is not in the settings-administration group.
- **Effect** All four entities are invisible: the access declarations grant no right at all to
  anyone else, including the anonymous reader and the ordinary internal user, and the four rights
  are granted only to the settings-administration group. Progress is nevertheless recorded for
  ordinary users, because the operations that record it run with elevated rights.

---

# 11. Locking and ordering rules

| Identifier | Rule |
|---|---|
| Scheduler lock | Before the scheduler interval is recomputed the scheduled action row is locked for update, allowing references. When the lock cannot be taken the recomputation is abandoned silently, so that two workers saving rules at the same moment do not deadlock. |
| Separate connection for accounts | The find-or-create of a metered account runs on a separate database connection, and pending work is flushed before it, so that the exhausted-balance failure — which rolls the main transaction back — cannot undo the account creation. Deleting tokenless accounts uses the same technique. |
| Batch commits while recycling | Automatic recycling commits after each batch of five thousand candidates, manual recycling after each batch of fifty thousand, so that a long run that times out does not lose everything. Neither commits while a test is running. |
| Exclusive lock on the translation cache | See [`AUT-141`](#aut-141). |
| Savepoint around an import | Every import run, real or trial, is wrapped in a savepoint. A trial run always rolls back, clears the whole cache — because identifiers created during the run may have entered it — and discards pending registry changes so that other workers are not told about a rollback. |
| Send-and-forget delivery | An outgoing webhook is posted after the transaction commits, with a one-second timeout; a rollback cancels it and only writes a warning line. The call therefore never holds a user's transaction open. |
| Registry rewrite | Saving, deleting or duplicating a rule reinstalls the interception patches and marks the registry as changed so that every other worker reloads it. The rewrite is skipped while a file is being imported. |
| One log per privacy session | The Privacy Log is created the first time a session has anything to record and updated on every later change, so that a session produces exactly one log however many records it touches. |

---

# 12. Where these rules are exercised

- Step-by-step procedures that raise them: [`workflows.md`](workflows.md).
- The arithmetic they guard: [`calculations.md`](calculations.md).
- The states they gate: [`state-machines.md`](state-machines.md).
- Numbered scenarios that assert them: [`acceptance-criteria.md`](acceptance-criteria.md).
- The settings, groups and record rules they refer to: [`configuration.md`](configuration.md).
