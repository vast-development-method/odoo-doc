# text message Template Preview (`sms.template.preview`)

**Transport name:** `sms.template.preview`  
**Storage name:** `sms_template_preview`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sms`

Description: SMS Template Preview

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sms_template_id` | Text message Template | many to one | `sms.template` | required; on delete of the target: cascade |
| `lang` | Template Preview Language | selection |  |  |
| `model_id` | Model | many to one | `ir.model` | related through path `sms_template_id.model_id` |
| `body` | Body | single line text |  | computed by rule `_compute_sms_template_fields` (not stored) |
| `resource_ref` | Record reference | reference |  | values provided by rule `_selection_target_model` |
| `no_record` | No Record | boolean |  | computed by rule `_compute_no_record` (not stored) |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_selection_target_model` | internal rule | self | `sms` | model |  |
| `_selection_languages` | internal rule | self | `sms` | model |  |
| `default_get` | lifecycle override | self, fields | `sms` | model |  |
| `_compute_no_record` | computation | self | `sms` | depends: `model_id` |  |
| `_compute_sms_template_fields` | computation | self | `sms` | depends: `lang`, `resource_ref` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `sms` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms.sms_template_preview_form` | form |  | `sms_template_id`, `no_record`, `model_id`, `resource_ref`, `lang`, `body` | `Discard` |  | `sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sms.sms_template_preview_action` | Template Preview | form |  | `{'default_sms_template_id':active_id}` | new | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.template.preview.json`; views: `../../../schemas/interfaces/views/sms.template.preview.json`.
