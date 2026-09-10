# Metadata for voice attachments (`discuss.voice.metadata`)

**Transport name:** `discuss.voice.metadata`  
**Storage name:** `discuss_voice_metadata`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Metadata for voice attachments

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `attachment_id` | Attachment | many to one | `ir.attachment` | indexed; not copied on duplication; on delete of the target: cascade |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

Machine-readable definition: `../../../schemas/data/entities/discuss.voice.metadata.json`.
