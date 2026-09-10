# Channel / Partners (Members) (`slide.channel.partner`)

**Transport name:** `slide.channel.partner`  
**Storage name:** `slide_channel_partner`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`  
**Extended by packages:** `hr_skills_slides`, `website_slides_survey`

Description: Channel / Partners (Members)

## Identity and behavior

- Display name field: `partner_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `channel_id` | Course | many to one | `slide.channel` | required; indexed; on delete of the target: cascade |
| `member_status` | Attendee Status | selection |  | required; read only; default `joined` |
| `completion` | % Completed Contents | integer |  | default ; aggregated with avg |
| `completed_slides_count` | # Completed Contents | integer |  | default  |
| `partner_id` | Partner | many to one | `res.partner` | required; indexed; on delete of the target: cascade |
| `partner_email` | Partner Email | single line text |  | read only; related through path `partner_id.email` |
| `channel_user_id` | Responsible | many to one | `res.users` | related through path `channel_id.user_id` |
| `channel_type` | Channel Type | selection |  | related through path `channel_id.channel_type` |
| `channel_visibility` | Channel Visibility | selection |  | related through path `channel_id.visibility` |
| `channel_enroll` | Channel Enroll | selection |  | related through path `channel_id.enroll` |
| `channel_website_id` | Website | many to one | `website` | related through path `channel_id.website_id` |
| `next_slide_id` | Next Lesson | many to one | `slide.slide` | computed by rule `_compute_next_slide_id` (not stored) |
| `invitation_link` | Invitation Link | single line text |  | computed by rule `_compute_invitation_link` (not stored) |
| `last_invitation_date` | Last Invitation Date | date and time |  |  |
| `nbr_certification` | Nbr Certification | integer |  | related through path `channel_id.nbr_certification` |
| `survey_certification_success` | Certified | boolean |  |  |

## Selection values

### `member_status` (Attendee Status)

| Value | Label |
|---|---|
| `invited` | Invite Sent |
| `joined` | Joined |
| `ongoing` | Ongoing |
| `completed` | Finished |

## State fields

State machine fields of this entity: `member_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_channel_partner_uniq` | Constraint | `unique(channel_id, partner_id)` | A partner membership to a channel must be unique! | `website_slides` |
| `_check_completion` | Constraint | `check(completion >= 0 and completion <= 100)` | The completion of a channel is a percentage and should be between 0% and 100. | `website_slides` |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_invitation_link` | computation | self | `website_slides` | depends: `channel_id`, `partner_id` | This sets the url used as hyperlink in the channel invitation email in template mail_notification_channel_invite. The partner_id is given in the url, as well as a hash based on the partner and channel id. |
| `_compute_next_slide_id` | computation | self | `website_slides` |  |  |
| `_recompute_completion` | internal rule | self | `website_slides` |  | This method computes the completion and member_status of attendees that are neither 'invited' nor 'completed'. Indeed, once completed, membership should remain so. We do not do any update on the 'invited' records. One should first set member_status to 'joined' before recomputing those values when enrolling an invited or archived attendee. It takes into account the previous completion value to add or remove karma for completing the course to the attendee (see _post_completion_update_hook) |
| `unlink` | lifecycle override | self | `website_slides` |  | Override unlink method : Remove attendee from a channel, then also remove slide.slide.partner related to. |
| `_get_invitation_hash` | preparation rule | self | `website_slides` |  | Returns the invitation hash of the attendee, used to access courses as invited / joined. |
| `_post_completion_update_hook` | internal rule | self, completed | `hr_skills_slides`, `website_slides` |  | Post hook of _recompute_completion. Adds or removes karma given for completing the course.  :param completed:     True if course is completed.     False if we remove an existing course completion. |
| `_send_completed_mail` | internal rule | self | `hr_skills_slides`, `website_slides` |  | Send an email to the attendee when they have successfully completed a course. |
| `_gc_slide_channel_partner` | background operation | self | `website_slides` | autovacuum | The invitations of 'invited' attendees are only valid for 3 months. Remove outdated invitations with no completion. A missing last_invitation_date is also considered as expired. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Channel Partner: officer: create/write/unlink own only | `[(4, ref('group_website_slides_officer'))]` | `[('channel_id.user_id', '=', user.id)]` | 0 | 1 | 1 | 1 |
| Channel Partner: manager: crud all | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_channel_partner_view_search` | search |  | `partner_id`, `partner_email`, `channel_id` |  | `Archived`, `Invite Sent`, `Joined`, `Ongoing`, `Completed`, `Course`, `Status` | `website_slides` |
| `website_slides.slide_channel_partner_view_tree` | list |  | `channel_id`, `partner_id`, `partner_email`, `create_date`, `write_date`, `last_invitation_date`, `member_status`, `completion`, `next_slide_id`, `channel_user_id`, `channel_type`, `channel_visibility`, `channel_enroll`, `channel_website_id`, `active` | `action_archive`, `action_unarchive` |  | `website_slides` |
| `website_slides.slide_channel_partner_view_kanban` | kanban |  | `channel_id`, `partner_id`, `completion`, `channel_user_id` |  |  | `website_slides` |
| `website_slides.slide_channel_partner_view_graph` | graph |  |  |  |  | `website_slides` |
| `website_slides.slide_channel_partner_view_pivot` | pivot |  | `completion` |  |  | `website_slides` |
| `website_slides_survey.slide_channel_partner_view_tree` | xpath | `website_slides.slide_channel_partner_view_tree` | `nbr_certification`, `survey_certification_success` |  |  | `website_slides_survey` |
| `website_slides_survey.slide_channel_partner_view_search` | xpath | `website_slides.slide_channel_partner_view_search` |  |  | `Certified` | `website_slides_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_channel_partner_action` | Attendees | list,kanban |  |  |  | `website_slides` |
| `website_slides.slide_channel_partner_action_report` | Attendees | graph,pivot,list,kanban |  | `{'search_default_groupby_member_status': 1}` |  | `website_slides` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `website_slides.mail_template_channel_completed` | Elearning: Completed Course | Congratulations! You completed {{ object.channel_id.name }} |
| `website_slides.mail_template_slide_channel_enroll` | Elearning: Add Attendees to Course | You have been invited to join {{ object.channel_id.name }} |
| `website_slides.mail_template_slide_channel_invite` | Elearning: Promotional Course Invitation | You have been invited to check out {{ object.channel_id.name }} |

Machine-readable definition: `../../../schemas/data/entities/slide.channel.partner.json`; views: `../../../schemas/interfaces/views/slide.channel.partner.json`.
