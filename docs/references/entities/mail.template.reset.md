# Mail Template Reset (`mail.template.reset`)

**Transport name:** `mail.template.reset`  
**Storage name:** `mail_template_reset`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail`

Description: Mail Template Reset

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `template_ids` | Template | many to many | `mail.template` |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `reset_template` | operation | self | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mail.group_mail_template_editor` | yes | yes | yes | yes | `mail` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_template_reset_view_form` | form |  |  | `Reset Template`, `Discard` |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_template_reset_action` | Reset Mail Template | form |  | `{             'default_template_ids': active_ids         }` | new | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.template.reset.json`; views: `../../../schemas/interfaces/views/mail.template.reset.json`.
