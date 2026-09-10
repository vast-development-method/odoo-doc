# Store link preview data (`mail.link.preview`)

**Transport name:** `mail.link.preview`  
**Storage name:** `mail_link_preview`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Store link preview data

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`
- Display name field: `source_url`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `source_url` | uniform resource locator | single line text |  | required |
| `og_type` | Type | single line text |  |  |
| `og_title` | Title | single line text |  |  |
| `og_site_name` | Site name | single line text |  |  |
| `og_image` | Image | single line text |  |  |
| `og_description` | Description | multi line text |  |  |
| `og_mimetype` | MIME type | single line text |  |  |
| `image_mimetype` | Image MIME type | single line text |  |  |
| `create_date` | Create Date | date and time |  | indexed |
| `message_link_preview_ids` | Message Link Preview | one to many | `mail.message.link.preview` | visible only to groups `base.group_erp_manager`; inverse field `link_preview_id` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_source_url` | UniqueIndex | `(source_url)` |  | `mail` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_create_from_message_and_notify` | internal rule | self, message, request_url | `mail` | model |  |
| `_is_link_preview_enabled` | internal rule | self | `mail` | model |  |
| `_is_domain_thottled` | internal rule | self, url | `mail` |  |  |
| `_search_or_create_from_url` | search rule | self, url | `mail` | model | Return the URL preview, first from the database if available otherwise make the request. |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_erp_manager` | yes | yes | yes | yes | `mail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_link_preview_view_form` | form |  | `source_url`, `og_type`, `og_title`, `og_image`, `og_image`, `og_mimetype`, `image_mimetype`, `create_date`, `og_description`, `message_link_preview_ids` |  |  | `mail` |
| `mail.mail_link_preview_view_tree` | list |  | `id`, `source_url`, `og_title`, `og_type`, `image_mimetype` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_link_preview_action` | Link Previews | list,form |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.link.preview.json`; views: `../../../schemas/interfaces/views/mail.link.preview.json`.
