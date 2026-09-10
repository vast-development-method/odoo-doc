# text message Template Reset (`sms.template.reset`)

**Transport name:** `sms.template.reset`  
**Storage name:** `sms_template_reset`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sms`

Description: SMS Template Reset

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `template_ids` | Template | many to many | `sms.template` |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `reset_template` | operation | self | `sms` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mail.group_mail_template_editor` | yes | yes | yes | yes | `sms` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms.sms_template_reset_view_form` | form |  |  | `Proceed`, `Cancel` |  | `sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sms.sms_template_reset_action` | Reset SMS Template | form |  | `{             'default_template_ids': active_ids         }` | new | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.template.reset.json`; views: `../../../schemas/interfaces/views/sms.template.reset.json`.
