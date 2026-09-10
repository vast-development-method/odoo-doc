# Email Aliases Mixin (`mail.alias.mixin`)

**Transport name:** `mail.alias.mixin`  
**Storage name:** `mail_alias_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Email Aliases Mixin

## Identity and behavior

- Mixins (classical inheritance): `mail.alias.mixin.optional`
- Delegation inheritance: embeds `mail.alias` through field `alias_id`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `alias_id` | Alias | many to one |  | required |
| `alias_name` | Alias Name | single line text |  |  |
| `alias_defaults` | Alias Defaults | multi line text |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_require_new_alias` | internal rule | self, record_vals | `mail` |  | alias_id field is always required, due to inherits |
| `_init_column` | internal rule | self, name | `mail` |  | Create aliases for existing rows. |
| `_init_column_alias_id` | internal rule | self | `mail` |  |  |

Machine-readable definition: `../../../schemas/data/entities/mail.alias.mixin.json`.
