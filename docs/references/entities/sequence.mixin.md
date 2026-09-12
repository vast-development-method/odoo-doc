# Automatic sequence (`sequence.mixin`)

**Transport name:** `sequence.mixin`  
**Storage name:** `sequence_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account`  
**Extended by packages:** `l10n_hr_edi`

Description: Automatic sequence

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence_prefix` | Sequence Prefix | single line text |  | computed by rule `_compute_split_sequence` and stored |
| `sequence_number` | Sequence Number | integer |  | computed by rule `_compute_split_sequence` and stored |

## Operations (21)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `account` |  |  |
| `_get_sequence_cache` | preparation rule | self | `account` |  |  |
| `write` | lifecycle override | self, vals | `account` |  |  |
| `_get_sequence_date_range` | preparation rule | self, reset | `account` |  |  |
| `_must_check_constrains_date_sequence` | internal rule | self | `account` |  |  |
| `_year_match` | internal rule | self, format_value, year | `account` |  |  |
| `_truncate_year_to_length` | internal rule | self, year, length | `account` |  |  |
| `_sequence_matches_date` | internal rule | self | `account` |  |  |
| `_constrains_date_sequence` | validation | self | `account` | constrains: |  |
| `_compute_split_sequence` | computation | self | `account` | depends: |  |
| `_deduce_sequence_number_reset` | internal rule | self, name | `account`, `l10n_hr_edi` | model | Detect if the used sequence resets yearly, montly or never.  :param name: the sequence that is used as a reference to detect the resetting     periodicity. Typically, it is the last before the one you want to give a     sequence. |
| `_make_regex_non_capturing` | internal rule | self, regex | `account` |  | Replace the "named capturing group" found in the regex by "non-capturing group" instead.  Example: `^(?P<prefix1>.*?)(?P<seq>\d{0,9})(?P<suffix>\D*?)$` will become `^(?:.*?)(?:\d{0,9})(?:\D*?)$` - `(?P<name>...)` = Named capturing groups - `(?:...)` = Non-capturing group  :param regex: the regex to modify  :return: the modified regex |
| `_get_last_sequence_domain` | preparation rule | self, relaxed | `account` |  | Get the sql domain to retreive the previous sequence number.  This function should be overriden by models inheriting from this mixin.  :param relaxed: see _get_last_sequence.  :returns: tuple(where_string, where_params): with     where_string: the entire SQL WHERE clause as a string.     where_params: a dictionary containing the parameters to substitute         at the execution of the query. |
| `_get_starting_sequence` | preparation rule | self | `account` |  | Get a default sequence number.  This function should be overriden by models heriting from this mixin This number will be incremented so you probably want to start the sequence at 0.  :return: string to use as the default sequence to increment |
| `_get_last_sequence` | preparation rule | self, relaxed, with_prefix | `account` |  | Retrieve the previous sequence.  This is done by taking the number with the greatest alphabetical value within the domain of _get_last_sequence_domain. This means that the prefix has a huge importance. For instance, if you have INV/2019/0001 and INV/2019/0002, when you rename the last one to FACT/2019/0001, one might expect the next number to be FACT/2019/0002 but it will be INV/2019/0002 (again) because INV > FACT. Therefore, changing the prefix might not be convenient during a period, and would only work when the numbering makes a new start (domain returns by _get_last_sequence_domain is [], |
| `_get_sequence_format_param` | preparation rule | self, previous | `account`, `l10n_hr_edi` | model | Get the python format and format values for the sequence.  :param previous: the sequence we want to extract the format from  tuple(format, format_values) :returns: a 2-elements tuple with:      - format is the format string on which we should call .format()     - format_values is the dict of values to format the `format` string       `format.format(**format_values)` should be equal to `previous` |
| `_locked_increment` | internal rule | self, format_string, format_values | `account` |  | Increment the sequence for the given format, returning the new value.  This method will lock the sequence in the database through its unique constraint, in order to ensure cross-transactional uniqueness of sequence numbers. If the sequence is already locked by another transaction, it will wait until the other one finishes, then grab the next available number.  Once the sequence has been locked by the transaction, further increments will rely on a cache, to avoid the need for multiple savepoints (see implementation comments)  At entry, the sequence record must be governed by the unique constrai |
| `_set_next_sequence` | internal rule | self | `account` |  | Set the next sequence.  This method ensures that the field is set both in the ORM and in the database. This is necessary because we use a database query to get the previous sequence, and we need that query to always be executed on the latest data. |
| `_get_next_sequence_format` | preparation rule | self | `account` |  | Get the next sequence format and its values.  This method retrieves the last used sequence and determines the next sequence format based on it. If there is no previous sequence, it initializes a new sequence using the starting sequence format.  :returns: a 2-element tuple with:      - format_string (str): the string on which we should call .format()     - format_values (dict): the dict of values to format `format_string` |
| `_is_last_from_seq_chain` | internal rule | self | `account` |  | Tells whether or not this element is the last one of the sequence chain.  :return: True if it is the last element of the chain. |
| `_is_end_of_seq_chain` | internal rule | self | `account` |  | Tells whether or not these elements are the last ones of the sequence chain.  :return: True if self are the last elements of the chain. |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_date_sequence` | ValidationError | The %(date_field)s (%(date)s) you've entered isn't aligned with the existing sequence number (%(sequence)s). Clear the sequence number to proceed. To maintain date-based sequences, select entries and use the resequence option from the actions menu, available in developer mode. | `account` |
| `_deduce_sequence_number_reset` | ValidationError | The sequence regex should at least contain the seq grouping keys. For instance: ^(?P<prefix1>.*?)(?P<seq>\d*)(?P<suffix>\D*?)$ | `account` |
| `_get_last_sequence` | ValidationError | %s is not a stored field | `account` |
| `_get_sequence_format_param` | ValidationError | Journal is not set for this invoice. | `l10n_hr_edi` |
| `_get_sequence_format_param` | ValidationError | Business premises label is not set on the journal. | `l10n_hr_edi` |
| `_get_sequence_format_param` | ValidationError | Issuing device label is not set on the journal. | `l10n_hr_edi` |
| `_get_sequence_format_param` | ValidationError | Invalid sequence number format. | `l10n_hr_edi` |
| `_get_sequence_format_param` | ValidationError | Invalid year format. | `l10n_hr_edi` |

Machine-readable definition: `../../../schemas/data/entities/sequence.mixin.json`.
