# Base Import (`base_import.import`)

**Transport name:** `base_import.import`  
**Storage name:** `base_import_import`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base_import`

Description: Base Import

## Identity and behavior

- Transient maximum hours: 12.0

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model` | Model | single line text |  |  |
| `file` | File | binary |  | Help: File to check and/or import, raw binary (not base64) |
| `file_name` | File Name | single line text |  |  |
| `file_type` | File Type | single line text |  |  |

## Operations (31)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_fields_tree` | operation | self, model, depth | `base_import` | model | Recursively get fields for the provided model (through fields_get) and filter them according to importability  The output format is a list of :class:`Field`:  .. class:: Field      .. attribute:: id: str          A non-unique identifier for the field, used to compute         the span of the `required` attribute: if multiple         `required` fields have the same id, only one of them         is necessary.      .. attribute:: name: str          The field's logical (The system) name within the scope of         its parent.      .. attribute:: string: str          The field's human-readable name (`` |
| `_filter_fields_by_types` | internal rule | self, model_fields_tree, header_types | `base_import` |  | Remove from model_fields_tree param all the fields and subfields that do not match the types in `header_types`.  :param list[dict] model_fields_tree: Contains recursively all the importable fields of     the target model. Generated in :meth:`get_fields_tree`. :param list header_types: Contains the extracted fields types of the current header.     Generated in :meth:`_extract_header_types`. |
| `_read_file` | internal rule | self, options | `base_import` |  | Dispatch to specific method to read file content, according to its mimetype or file type  :param dict options: reading options (quoting, separator, ...) |
| `_read_xls` | internal rule | self, options | `base_import` |  |  |
| `_read_xls_book` | internal rule | self, book, sheet_name | `base_import` |  |  |
| `_read_xlsx` | internal rule | self, options | `base_import` |  |  |
| `_read_ods` | internal rule | self, options | `base_import` |  |  |
| `_read_csv` | internal rule | self, options | `base_import` |  | Returns file length and a CSV-parsed list of all non-empty lines in the file.  :raises csv.Error: if an error is detected during CSV parsing |
| `_extract_header_types` | internal rule | self, preview_values, options | `base_import` | model | Returns the potential field types, based on the preview values, using heuristics.  This methods is only used for suggested mapping at 2 levels:  1. for fuzzy mapping at file load -> Execute the fuzzy mapping only    on "most likely field types" 2. For "Suggested fields" section in the fields mapping dropdown list at UI side.  The following heuristic is used: If all preview values  - Start with `__export__`: return id + relational field types - Can be cast into integer: return id + relational field types, integer, float and monetary - Can be cast into Boolean: return boolean - Can be cast int |
| `_try_match_date_time` | internal rule | self, preview_values, options | `base_import` |  |  |
| `_extract_headers_types` | internal rule | self, headers, preview, options | `base_import` | model | For each column, this method will extract the potential data types based on the preview values  :param list headers: list of headers names. Used as part of key for                      returned headers_types to ease understanding of its usage :param list preview: list of the first file records (see "parse_preview" for more detail) e.g.::      [ ["lead_name1", "1", "partner_id1"], ["lead_name2", "2", "partner_id2"], ... ]  :param options: parsing options :returns: dict headers_types:      contains all the extracted header types for each header e.g.::          {             (header_index, header |
| `_get_mapping_suggestion` | preparation rule | self, header, fields_tree, header_types, mapping_fields | `base_import` |  | Attempts to match a given header to a field of the imported model.  We can distinguish 2 types of header format:  - simple header string that aim to directly match a field of the target model   e.g.: "lead_id" or "Opportunities" or "description". - composed '/' joined header string that aim to match a field of a   relation field of the target model (= subfield) e.g.:   'lead_id/description' aim to match the field `description` of the field lead_id.  When returning result, to ease further treatments, the result is returned as a list, where each element of the list is a field or a sub-field of |
| `_get_distance` | preparation rule | self, a, b | `base_import` |  | This method return an index that reflects the distance between the two given string a and b.  This index is a score between 0 and 1 where `0` indicates an exact match and `1` indicates completely different strings. |
| `_get_mapping_suggestions` | preparation rule | self, headers, header_types, fields_tree | `base_import` |  | Attempts to match the imported model's fields to the titles of the parsed CSV file, if the file is supposed to have headers.  Returns a dict mapping cell indices to key paths in the `fields` tree.  :param list headers: titles of the parsed file :param dict header_types:      extracted types for each column in the parsed file e.g.::          {             (header_index, header_name): ['int', 'float', 'char', 'many2one',...],              ...         }  :param list fields_tree:      list of the target model's fields e.g.::          [             {                 'name': 'fieldName',           |
| `_deduplicate_mapping_suggestions` | internal rule | self, mapping_suggestions | `base_import` |  | This method is meant to avoid multiple columns to be matched on the same field.  Taking `mapping_suggestions` as input, it will check if multiple columns are mapped to the same field and will only keep the mapping that has the smallest distance. The other columns that were matched to the same field are removed from the mapping suggestions.  Hierarchy mapping is considered as advanced and is skipped during this deduplication process. We consider that multiple mapping on hierarchy mapping will not occur often and due to the fact that this won't lead to any particular issues when a non 'char/te |
| `parse_preview` | operation | self, options, count | `base_import` |  | Generates a preview of the uploaded files, and performs fields-matching between the import's file data and the model's columns.  If the headers are not requested (not options.has_headers), returned `matches` and `headers` are both `False`.  :param int count: number of preview lines to generate :param options: format-specific options.                 CSV: {quoting, separator, headers} :type options: {str, str, str, bool} :returns: `{fields, matches, headers, preview} \| {error, preview}` :rtype: {dict(str: dict(...)), dict(int, list(str)), list(str), list(list(str))} \| {str, str} |
| `_convert_import_data` | internal rule | self, fields, options | `base_import` | model | Extracts the input BaseModel and fields list (with `False`-y placeholders for fields to *not* import) into a format Model.import_data can use: a fields list without holes and the precisely matching data matrix  :returns: (data, fields) :raises ValueError: in case the import data could not be converted |
| `_remove_currency_symbol` | internal rule | self, value | `base_import` | model |  |
| `_parse_float_from_data` | internal rule | self, data, index, name, options | `base_import` | model |  |
| `_infer_separators` | internal rule | self, value, options | `base_import` |  | Try to infer the shape of the separators: if there are two different "non-numberic" characters in the number, the former/duplicated one would be grouping ("thousands" separator) and the latter would be the decimal separator. The decimal separator should furthermore be unique. |
| `_parse_import_data` | internal rule | self, data, import_fields, options | `base_import` |  | Lauch first call to :meth:`_parse_import_data_recursive` with an empty prefix. :meth:`_parse_import_data_recursive` will be run recursively for each relational field. |
| `_parse_import_data_recursive` | internal rule | self, model, prefix, data, import_fields, options | `base_import` |  |  |
| `_parse_date_from_data` | internal rule | self, data, index, name, field_type, options | `base_import` |  |  |
| `_import_file_by_url` | internal rule | self, url, session, field, line_number | `base_import` |  | Imports a file by URL  :param str url: the original field value :param requests.Session session: :param str field: name of the field (for logging/debugging) :param int line_number: 0-indexed line number within the imported file (for logging/debugging) :return: the replacement value :rtype: bytes |
| `_stringify_date_like_objects` | internal rule | self, data, options, trim | `base_import` | model |  |
| `_build_import_error_msg` | internal rule | self, message, record, row_index, field | `base_import` |  |  |
| `_parse_datetime_data` | internal rule | self, import_fields, input_file_data | `base_import` |  |  |
| `execute_import` | operation | self, fields, columns, options, dryrun | `base_import` |  | Actual execution of the import  :param fields: import mapping: maps each column to a field,                `False` for the columns to ignore :type fields: list(str\|bool) :param columns: columns label :type columns: list(str\|bool) :param dict options: :param bool dryrun: performs all import operations (and                     validations) but rollbacks writes, allows                     getting as much errors as possible without                     the risk of clobbering the database. :returns: A list of errors. If the list is empty the import           executed fully and correctly. If the  |
| `_extract_binary_filenames` | internal rule | self, import_fields, data, model, prefix, binary_filenames | `base_import` |  |  |
| `_handle_multi_mapping` | internal rule | self, import_fields, input_file_data | `base_import` |  | This method handles multiple mapping on the same field.  It will return the list of the mapped fields and the concatenated data for each field:  - If two column are mapped on the same text or char field, they will end up   in only one column, concatenated via space (char) or new line (text). - The same logic is used for many2many fields. Multiple values can be   imported if they are separated by `,`.  Input/output Example:  input data     .. code-block:: python          [             ["Value part 1", "1", "res.partner_id1", "Value part 2"],             ["I am", "1", "res.partner_id1", "Batma |
| `_handle_fallback_values` | internal rule | self, import_field, input_file_data, fallback_values | `base_import` |  | If there are fallback values, this method will replace the input file data value if it does not match the possible values for the given field. This is only valid for boolean and selection fields.  .. note::      We can consider that we need to retrieve the selection values for     all the fields in fallback_values, as if they are present, it's because     there was already a conflict during first import run and user had to     select a fallback value for the field.  :param list import_field: ordered list of field that have been matched to import data :param list input_file_data: ordered list o |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_read_file` | UserError | Unsupported file format "{}", import only supports CSV, ODS, XLS and XLSX | `base_import` |
| `_read_file` | UserError | Unable to load "{extension}" file: requires Python module "{modname}" | `base_import` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `base_import` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Import: access own records | global (all users) | `[('create_uid', '=', user.id)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/base_import.import.json`.
