# Additional resource for a particular slide (`slide.slide.resource`)

**Transport name:** `slide.slide.resource`  
**Storage name:** `slide_slide_resource`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`

Description: Additional resource for a particular slide

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `slide_id` | Slide | many to one | `slide.slide` | required; indexed; on delete of the target: cascade |
| `resource_type` | Resource Type | selection |  | required |
| `name` | Name | single line text |  | computed by rule `_compute_name` and stored |
| `data` | Resource | binary |  | computed by rule `_compute_reset_resources` and stored |
| `file_name` | File Name | single line text |  |  |
| `link` | Link | single line text |  | computed by rule `_compute_reset_resources` and stored |
| `download_url` | Download uniform resource locator | single line text |  | computed by rule `_compute_download_url` (not stored) |
| `sequence` | Sequence | integer |  |  |

## Selection values

### `resource_type` (Resource Type)

| Value | Label |
|---|---|
| `file` | File |
| `url` | Link |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_url` | Constraint | `CHECK (resource_type != 'url' OR link IS NOT NULL)` | A resource of type url must contain a link. | `website_slides` |
| `_check_file_type` | Constraint | `CHECK (resource_type != 'file' OR link IS NULL)` | A resource of type file cannot contain a link. | `website_slides` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_reset_resources` | computation | self | `website_slides` | depends: `resource_type` |  |
| `_compute_name` | computation | self | `website_slides` | depends: `file_name`, `resource_type`, `data`, `link` |  |
| `_compute_download_url` | computation | self | `website_slides` | depends: `name`, `file_name` |  |
| `_check_link_type` | validation | self | `website_slides` | constrains: `data` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_link_type` | ValidationError | Resource %(resource_name)s is a link and should not contain a data file | `website_slides` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website_slides` |
| `base.group_public` | no | no | no | no | `website_slides` |
| `base.group_portal` | no | yes | no | no | `website_slides` |
| `base.group_user` | no | yes | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Resource: read restricted to channel members and channel responsible | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('slide_id.channel_id.is_member', '=', True)]` | True | False | False | False |
| Resource: officer: read all | `[(4, ref('group_website_slides_officer'))]` | `[(1, '=', 1)]` | True | False | False | False |
| Resource: officer: crud own only | `[(4, ref('group_website_slides_officer'))]` | `[('slide_id.channel_id.user_id', '=', user.id)]` | True | True | True | True |
| Resource: manager: crud all | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |

Machine-readable definition: `../../../schemas/data/entities/slide.slide.resource.json`.
