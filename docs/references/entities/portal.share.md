# Portal Sharing (`portal.share`)

**Transport name:** `portal.share`  
**Storage name:** `portal_share`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `portal`  
**Extended by packages:** `project`

Description: Portal Sharing

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model` | Related Document Model | single line text |  | required |
| `res_id` | Related Document identifier | integer |  | required |
| `resource_ref` | Related Document | reference |  | computed by rule `_compute_resource_ref` (not stored); values provided by rule `_selection_target_model` |
| `partner_ids` | Recipients | many to many | `res.partner` | required |
| `note` | Note | multi line text |  | Help: Add extra content to display in the email |
| `share_link` | Link | single line text |  | computed by rule `_compute_share_link` (not stored) |
| `access_warning` | Access warning | multi line text |  | computed by rule `_compute_access_warning` (not stored) |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `portal` | model |  |
| `_selection_target_model` | internal rule | self | `portal` | model |  |
| `_compute_resource_ref` | computation | self | `portal` | depends: `res_model`, `res_id` |  |
| `_compute_share_link` | computation | self | `portal` | depends: `res_model`, `res_id` |  |
| `_compute_access_warning` | computation | self | `portal` | depends: `res_model`, `res_id` |  |
| `_send_public_link` | internal rule | self, partners | `portal` |  |  |
| `_send_signup_link` | internal rule | self, partners | `portal` |  |  |
| `action_send_mail` | user action | self | `portal`, `project` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_partner_manager` | yes | yes | yes | no | `portal` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `portal.portal_share_wizard` | form |  | `access_warning`, `res_model`, `res_id`, `share_link`, `partner_ids`, `note` | `Send`, `Cancel` |  | `portal` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `portal.portal_share_action` | Share Document | form |  | `{'dialog_size': 'medium'}` | new | `portal` |

Machine-readable definition: `../../../schemas/data/entities/portal.share.json`; views: `../../../schemas/interfaces/views/portal.share.json`.
