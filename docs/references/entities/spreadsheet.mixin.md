# Spreadsheet mixin (`spreadsheet.mixin`)

**Transport name:** `spreadsheet.mixin`  
**Storage name:** `spreadsheet_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `spreadsheet`

Description: Spreadsheet mixin

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `spreadsheet_binary_data` | Spreadsheet file | binary |  | default computed dynamically (lambda self: self._empty_spreadsheet_data_base64()) |
| `spreadsheet_data` | Spreadsheet Data | multi line text |  | computed by rule `_compute_spreadsheet_data` (not stored); writable through an inverse rule |
| `spreadsheet_file_name` | Spreadsheet File Name | single line text |  | computed by rule `_compute_spreadsheet_file_name` (not stored) |
| `thumbnail` | Thumbnail | binary |  |  |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_spreadsheet_data` | validation | self | `spreadsheet` | constrains: `spreadsheet_binary_data` |  |
| `_compute_spreadsheet_data` | computation | self | `spreadsheet` | depends: `spreadsheet_binary_data` |  |
| `_inverse_spreadsheet_data` | inverse computation | self | `spreadsheet` |  |  |
| `_compute_spreadsheet_file_name` | computation | self | `spreadsheet` | depends: `display_name` |  |
| `_onchange_data_` | on change | self | `spreadsheet` | onchange: `spreadsheet_binary_data` |  |
| `get_display_names_for_spreadsheet` | operation | self, args | `spreadsheet` | readonly; model |  |
| `_empty_spreadsheet_data_base64` | internal rule | self | `spreadsheet` |  | Create an empty spreadsheet workbook. Encoded as base64 |
| `_empty_spreadsheet_data` | internal rule | self | `spreadsheet` |  | Create an empty spreadsheet workbook. The sheet name should be the same for all users to allow consistent references in formulas. It is translated for the user creating the spreadsheet. |
| `_zip_xslx_files` | internal rule | self, files | `spreadsheet` |  |  |
| `_get_file_content` | preparation rule | self, file_path | `spreadsheet` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_spreadsheet_data` | ValidationError | Uh-oh! Looks like the spreadsheet file contains invalid data.  %(errors)s | `spreadsheet` |
| `_check_spreadsheet_data` | ValidationError | Uh-oh! Looks like the spreadsheet file contains invalid data. | `spreadsheet` |

Machine-readable definition: `../../../schemas/data/entities/spreadsheet.mixin.json`.
