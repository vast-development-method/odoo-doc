# Mail Main Attachment management (`mail.thread.main.attachment`)

**Transport name:** `mail.thread.main.attachment`  
**Storage name:** `mail_thread_main_attachment`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Mail Main Attachment management

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `message_main_attachment_id` | Main Attachment | many to one | `ir.attachment` | indexed (btree_not_null); not copied on duplication |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_message_post_after_hook` | messaging hook | self, message, msg_values | `mail` |  | Set main attachment field if necessary |
| `_message_set_main_attachment_id` | messaging hook | self, attachments, force, filter_xml | `mail` |  | Update 'main' attachment.  :param list attachments: new main attachment IDS; if several attachments   are given, we search for pdf or image first; :param boolean force: if set, replace an existing attachment; otherwise   update is skipped; :param filter_xml: filters out xml (and octet-stream) attachments, as in   most cases you don't want that kind of file to end up as main attachment   of records; |
| `_thread_to_store` | internal rule | self, store, fields, request_list | `mail` |  |  |

Machine-readable definition: `../../../schemas/data/entities/mail.thread.main.attachment.json`.
