# Field html History (`html.field.history.mixin`)

**Transport name:** `html.field.history.mixin`  
**Storage name:** `html_field_history_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `html_editor`

Description: Field html History

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `html_field_history` | History data | structured document |  | read only |
| `html_field_history_metadata` | History metadata | structured document |  | computed by rule `_compute_metadata` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_versioned_fields` | preparation rule | self | `html_editor` | model | This method should be overriden  :return: List[string]: A list of name of the fields to be versioned |
| `_compute_metadata` | computation | self | `html_editor` | depends: `html_field_history` |  |
| `create` | lifecycle override | self, vals_list | `html_editor` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `html_editor` |  |  |
| `html_field_history_get_content_at_revision` | operation | self, field_name, revision_id | `html_editor` |  | Get the requested field content restored at the revision_id.  :param str field_name: the name of the field :param int revision_id: id of the last revision to restore  :return: string: the restored content |
| `html_field_history_get_comparison_at_revision` | operation | self, field_name, revision_id | `html_editor` |  | For the requested field, Get a comparison between the current content of the field and the content restored at the requested revision_id.  :param str field_name: the name of the field :param int revision_id: id of the last revision to compare  :return: string: the comparison |
| `html_field_history_get_unified_diff_at_revision` | operation | self, field_name, revision_id | `html_editor` |  | For the requested field, Get a unified diff between the current content of the field and the content restored at the requested revision_id.  :param str field_name: the name of the field :param int revision_id: id of the last revision to compare  :return: string: the unified diff |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | ValidationError | 'Ensure all versioned fields ( %s ) in model %s are declared as sanitize=True' % (str(versioned_fields), rec._name) | `html_editor` |

Machine-readable definition: `../../../schemas/data/entities/html.field.history.mixin.json`.
