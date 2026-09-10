# Channel Invitation Wizard (`slide.channel.invite`)

**Transport name:** `slide.channel.invite`  
**Storage name:** `slide_channel_invite`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website_slides`

Description: Channel Invitation Wizard

## Identity and behavior

- Mixins (classical inheritance): `mail.composer.mixin`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `attachment_ids` | Attachments | many to many | `ir.attachment` |  |
| `send_email` | Send Email | boolean |  | computed by rule `_compute_send_email` and stored |
| `partner_ids` | Recipients | many to many | `res.partner` |  |
| `channel_id` | Course | many to one | `slide.channel` | required |
| `channel_invite_url` | Course Link | single line text |  | computed by rule `_compute_channel_invite_url` (not stored) |
| `channel_visibility` | Channel Visibility | selection |  | related through path `channel_id.visibility` |
| `channel_published` | Channel Published | boolean |  | related through path `channel_id.is_published` |
| `enroll_mode` | Enroll partners | boolean |  | read only; Help: Whether invited partners will be added as enrolled. Otherwise, they will be added as invited. |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_channel_invite_url` | computation | self | `website_slides` | depends: `channel_id` |  |
| `_compute_render_model` | computation | self | `website_slides` | depends: `channel_id` |  |
| `_compute_send_email` | computation | self | `website_slides` | depends: `channel_id`, `enroll_mode` |  |
| `action_invite` | user action | self | `website_slides` |  | Process the wizard content and proceed with sending the related email(s), rendering any template patterns on the fly if needed. This method is used both to add members as 'joined' (when adding attendees) and as 'invited' (on invitation), depending on the value of enroll_mode. Archived members can be invited or enrolled. They will become 'invited', or another status if enrolled depending on their progress. Invited members can be reinvited, or enrolled depending on enroll_mode. |
| `_prepare_mail_values` | preparation rule | self, slide_channel_partner | `website_slides` |  | Create mail specific for recipient |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_invite` | UserError | Unable to post message, please configure the sender's email address. | `website_slides` |
| `action_invite` | UserError | Please select at least one recipient. | `website_slides` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `website_slides` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_channel_invite_view_form` | form |  | `channel_id`, `channel_published`, `channel_visibility`, `enroll_mode`, `channel_invite_url`, `send_email`, `partner_ids`, `lang`, `render_model`, `subject`, `can_edit_body`, `body`, `attachment_ids`, `template_id` | `Send`, `Close`, `Cancel` |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.channel.invite.json`; views: `../../../schemas/interfaces/views/slide.channel.invite.json`.
