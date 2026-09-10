# Sequence (`ir.sequence`)

**Transport name:** `ir.sequence`  
**Storage name:** `ir_sequence`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `point_of_sale`

Description: Sequence

## Identity and behavior

- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `code` | Sequence Code | single line text |  |  |
| `implementation` | Implementation | selection |  | required; default `standard`; Help: While assigning a sequence number to a record, the 'no gap' sequence implementation ensures that each previous sequence number has been assigned already. While this sequence implementation will not skip any sequence number upon assignment, there can still be gaps in the sequence if records are deleted. The 'no gap' implementation is slower than the standard one. |
| `active` | Active | boolean |  | default `True` |
| `prefix` | Prefix | single line text |  | Help: Prefix value of the record for the sequence |
| `suffix` | Suffix | single line text |  | Help: Suffix value of the record for the sequence |
| `number_next` | Next Number | integer |  | required; default `1`; Help: Next number of this sequence |
| `number_next_actual` | Actual Next Number | integer |  | computed by rule `_get_number_next_actual` (not stored); writable through an inverse rule; Help: Next number that will be used. This number can be incremented frequently so the displayed value might already be obsolete |
| `number_increment` | Step | integer |  | required; default `1`; Help: The next number of the sequence will be incremented by this number |
| `padding` | Sequence Size | integer |  | required; default ; Help: Odoo will automatically adds some '0' on the left of the 'Next Number' to get the required padding size. |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda s: s.env.company) |
| `use_date_range` | Use subsequences per date_range | boolean |  |  |
| `date_range_ids` | Subsequences | one to many | `ir.sequence.date_range` | inverse field `sequence_id` |

## Selection values

### `implementation` (Implementation)

| Value | Label |
|---|---|
| `standard` | Standard |
| `no_gap` | No gap |

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_number_next_actual` | preparation rule | self | `base` |  | Return number from ir_sequence row when no_gap implementation, and number from postgres sequence when standard implementation. |
| `_set_number_next_actual` | internal rule | self | `base` |  |  |
| `_get_current_sequence` | preparation rule | self, sequence_date | `base` | model | Returns the object on which we can find the number_next to consider for the sequence. It could be an ir.sequence or an ir.sequence.date_range depending if use_date_range is checked or not. This function will also create the ir.sequence.date_range if none exists yet for today |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi | Create a sequence, in implementation == standard a fast gaps-allowed PostgreSQL sequence is used. |
| `unlink` | lifecycle override | self | `base` |  |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_next_do` | internal rule | self | `base` |  |  |
| `_get_prefix_suffix` | preparation rule | self, date, date_range | `base` |  |  |
| `get_next_char` | operation | self, number_next | `base` |  |  |
| `_create_date_range_seq` | internal rule | self, date | `base` |  |  |
| `_next` | internal rule | self, sequence_date | `base` |  | Returns the next number in the preferred sequence in all the ones given in self. |
| `next_by_id` | operation | self, sequence_date | `base` |  | Draw an interpolated string using the specified sequence. |
| `next_by_code` | operation | self, sequence_code, sequence_date | `base` | model | Draw an interpolated string using a sequence with the requested code. If several sequences with the correct code are available to the user (multi-company cases), the one from the user's current company will be used. |
| `_unlink_sequence` | internal rule | self | `point_of_sale` | ondelete |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_prefix_suffix` | UserError | Invalid prefix or suffix for sequence “%s” | `base` |
| `_unlink_sequence` | UserError | You cannot delete a sequence used in an active POS config: %s | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.sequence_view` | form |  | `name`, `implementation`, `code`, `active`, `company_id`, `prefix`, `suffix`, `use_date_range`, `padding`, `number_increment`, `number_next_actual`, `date_range_ids`, `date_from`, `date_to`, `number_next_actual` |  |  | `base` |
| `base.sequence_view_tree` | list |  | `code`, `name`, `prefix`, `padding`, `company_id`, `number_next_actual`, `number_increment`, `implementation` |  |  | `base` |
| `base.view_sequence_search` | search |  | `name`, `code`, `company_id` |  | `Archived` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_sequence_form` | Sequences |  |  | `{'active_test': False}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.sequence.json`; views: `../../../schemas/interfaces/views/ir.sequence.json`.
