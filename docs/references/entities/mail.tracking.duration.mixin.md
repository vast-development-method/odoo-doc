# Mixin to compute the time a record has spent in each value a many2one field can take (`mail.tracking.duration.mixin`)

**Transport name:** `mail.tracking.duration.mixin`  
**Storage name:** `mail_tracking_duration_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Mixin to compute the time a record has spent in each value a many2one field can take

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `duration_tracking` | Status time | structured document |  | computed by rule `_compute_duration_tracking` (not stored); Help: JSON that maps ids from a many2one field to seconds spent |
| `rotting_days` | Days Rotting | integer |  | computed by rule `_compute_rotting` (not stored); Help: Day count since this resource was last updated |
| `is_rotting` | Rotting | boolean |  | computed by rule `_compute_rotting` (not stored); searchable through a search rule |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_duration_tracking` | computation | self | `mail` |  | Computes duration_tracking, a Json field stored as { <many2one_id (str)>: <duration_spent_in_seconds (int)> }      e.g. {"1": 1230, "2": 2220, "5": 14}  `_track_duration_field` must be present in the model that uses the mixin to specify on what field to compute time spent. Besides, tracking must be activated for that field.      e.g.     class MyModel(models.Model):         _name = 'my.model'         _track_duration_field = "tracked_field"          tracked_field = fields.Many2one('tracked.model', tracking=True) |
| `_get_duration_from_tracking` | preparation rule | self, trackings | `mail` |  | Calculates the duration spent in each value based on the provided list of trackings. It adds a "fake" tracking at the end of the trackings list to account for the time spent in the current value.  Args:     trackings (list): A list of dictionaries representing the trackings with:         - 'create_date': The date and time of the tracking.         - 'old_value_integer': The ID of the previous value.  Returns:     dict: A dictionary where the keys are the IDs of the values, and the values are the durations in seconds |
| `_is_rotting_feature_enabled` | internal rule | self | `mail` |  | To enable the rotting behavior, the following must be present:  * Stage-like model (linked by '_track_duration_field') must have a 'rotting_threshold_days' integer field     modeling the number of days before a record rots  * Model inheriting from duration mixin must have a 'date_last_stage_update' field tracking the last stage change   Also consider overriding _get_rotting_depends_fields() and _get_rotting_domain().  Certain views have access to widgets to display rotting status:     'rotting' for kanbans, 'rotting_statusbar_duration' for forms, 'badge_rotting' for lists.  :return: bool: whet |
| `_get_rotting_depends_fields` | preparation rule | self | `mail` |  | fields added to this method through override should likely also be returned by _get_rotting_domain() override  :return: the array of fields that can affect the ability of a resource to rot |
| `_get_rotting_domain` | preparation rule | self | `mail` |  | fields added to this method through override should likely also be returned by _get_rotting_depends_fields() override  :return: domain: conditions that must be met so that the field can be considered rotting |
| `_compute_rotting` | computation | self | `mail` | depends: | A resource is rotting if its stage has not been updated in a number of days depending on its stage's rotting_threshold_days value, assuming it matches _get_rotting_domain() conditions.  If the rotting_threshold_days field is not defined on the tracked module, or if the value of rotting_threshold_days is 0, then the resource will never rot. |
| `_search_is_rotting` | search rule | self, operator, value | `mail` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_search_is_rotting` | UserError | Model configuration does not support the rotting feature | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.tracking.duration.mixin.json`.
